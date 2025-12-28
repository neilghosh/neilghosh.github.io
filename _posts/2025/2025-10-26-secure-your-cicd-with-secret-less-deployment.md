---
layout: post
title: "Secure your CI/CD with Secret less deployment"
date: 2025-10-26
description: Learn how to deploy to Azure without storing passwords using OIDC Federated Identity, eliminating the need for long-lived secrets in your CI/CD pipelines.
categories: [CI/CD, Security, DevOps]
---

# Deploying to Azure Without Storing Passwords: A Real-World Example

I used to lose sleep over GitHub secrets. Every deployment credential stored in my repository felt like a ticking time bomb—what if someone accidentally logged it? What if my former colleague still had access to the org? What if the credentials leaked?

Then I discovered OIDC Federated Identity, and honestly, it changed how I think about CI/CD security.

Let me walk you through how we're deploying a `.NET 8` API to Azure App Service without storing a single password, using a real application as our example.

---

## The Problem We're Solving

Traditional deployment workflows look like this:

```yaml
# The old way (please don't do this anymore)
- name: Login to Azure
  uses: azure/login@v2
  with:
    creds: ${{ secrets.AZURE_CREDENTIALS }}  # This is a full JSON object with a password
```

When you use `AZURE_CREDENTIALS`, you're storing something that looks like this in your GitHub secrets:

```json
{
  "clientId": "abc123...",
  "clientSecret": "super-secret-password-here",
  "subscriptionId": "...",
  "tenantId": "..."
}
```

That `clientSecret` is the problem. It's:
- **Rotatable only by hand** (easy to forget)
- **Visible in logs if leaked** (one mistake and it's compromised)
- **Active even when not needed** (sitting there 24/7)

## The Modern Solution: Keyless Deployment with OIDC

Instead, GitHub can generate a short-lived token (OIDC token) that's cryptographically signed with GitHub's private key. Azure validates this signature and trusts the token automatically.

No passwords. No rotating credentials. Just a temporary, signed token that expires in minutes.

Here's what we changed:

```yaml
# The new way (keyless)
- name: Log in to Azure using Managed Identity
  uses: azure/login@v2
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}        # Just an ID
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}        # Just an ID
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}  # Just an ID
```

Notice the difference? We're only storing IDs now. They're not secrets—they're public information that identifies *which* Azure resources and tenant we're authenticating to. The actual authentication happens via OIDC token exchange.

---

## Our Real Application: A Simple .NET 8 API

Let me introduce our test subject. It's a straightforward ASP.NET Core Web API:

```csharp
// Program.cs - Dependency injection setup
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddScoped<IIpAddressService, OutboundIpService>();
builder.Services.AddScoped<IIpifyProxy, IpifyProxy>();
builder.Services.AddHttpClient<IpifyProxy>();

var app = builder.Build();
app.UseRouting();
app.MapControllers();
app.Run();
```

It's a layered architecture with services, proxies, and controllers. Nothing fancy. The kind of app you might be deploying yourself.

We containerize it with a multi-stage build:

```dockerfile
# Dockerfile - Multi-stage build for smaller image size
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY simple-dotnet-service.csproj .
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app/publish simple-dotnet-service.csproj

FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app
COPY --from=build /app/publish .
ENV ASPNETCORE_URLS=http://+:8080
ENV ASPNETCORE_ENVIRONMENT=Production
EXPOSE 8080
ENTRYPOINT ["dotnet", "SimpleDotnetService.dll"]
```

This gets pushed to Azure Container Registry (ACR), then deployed to App Service.

---

## How the Workflow Actually Works

Here's our complete three-job deployment pipeline:

### Job 1: Build & Test (No Azure Access Needed)

```yaml
build-and-test:
  runs-on: ubuntu-latest
  permissions:
    contents: read
  
  steps:
  - name: Checkout code
    uses: actions/checkout@v4
  
  - name: Setup .NET
    uses: actions/setup-dotnet@v4
    with:
      dotnet-version: '8.0.x'
  
  - name: Restore dependencies
    run: dotnet restore simple-dotnet-service.sln
  
  - name: Build
    run: dotnet build simple-dotnet-service.sln --configuration Release --no-restore
  
  - name: Run tests
    run: dotnet test simple-dotnet-service.sln --no-build --verbosity normal --configuration Release
```

This runs on every push and PR. It doesn't touch Azure at all. If the build fails, we stop before wasting time on Docker operations.

### Job 2: Build & Push Docker (OIDC Magic Happens Here)

```yaml
build-and-push-docker:
  runs-on: ubuntu-latest
  needs: build-and-test
  if: github.event_name == 'push' || github.event_name == 'workflow_dispatch'
  permissions:
    contents: read
    id-token: write  # ← This is the key! Allows GitHub to generate OIDC token
  
  steps:
  - name: Log in to Azure using Managed Identity
    uses: azure/login@v2
    with:
      client-id: ${{ secrets.AZURE_CLIENT_ID }}
      tenant-id: ${{ secrets.AZURE_TENANT_ID }}
      subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
  
  - name: Log in to Azure Container Registry
    run: az acr login --name simpledotnetregistry565
  
  - name: Build and push Docker image to ACR
    run: |
      docker build -t simpledotnetregistry565.azurecr.io/simple-dotnet-service:${{ github.sha }} \
                   -t simpledotnetregistry565.azurecr.io/simple-dotnet-service:latest .
      docker push simpledotnetregistry565.azurecr.io/simple-dotnet-service:${{ github.sha }}
      docker push simpledotnetregistry565.azurecr.io/simple-dotnet-service:latest
```

Look at that `permissions` block. `id-token: write` tells GitHub Actions: "I need permission to generate an OIDC token for this job."

Here's what happens behind the scenes:

1. **GitHub generates an OIDC token** - This token is cryptographically signed with GitHub's private key and contains information about the workflow (repo, branch, commit, etc.)

2. **azure/login passes it to Azure** - The `azure/login@v2` action receives the token and sends it to Microsoft Identity Platform

3. **Azure validates the signature** - Azure uses GitHub's public key to verify the token is legitimate

4. **Azure checks the federated credential** - Azure verifies that this specific repo on this specific branch is allowed (we'll set this up next)

5. **Azure issues an access token** - If everything checks out, Azure gives back a temporary access token

6. **All subsequent `az` commands use that token** - No passwords needed

The result? Your Docker image is securely pushed to ACR.

### Job 3: Deploy to App Service

```yaml
deploy-to-app-service:
  runs-on: ubuntu-latest
  needs: build-and-push-docker
  if: github.event_name == 'push' || github.event_name == 'workflow_dispatch'
  permissions:
    contents: read
    id-token: write
  
  steps:
  - name: Log in to Azure using Managed Identity
    uses: azure/login@v2
    with:
      client-id: ${{ secrets.AZURE_CLIENT_ID }}
      tenant-id: ${{ secrets.AZURE_TENANT_ID }}
      subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
  
  - name: Configure App Service to use container image
    run: |
      az webapp config container set \
        --name simple-dotnet-service \
        --resource-group simple-dotnet-service-rg \
        --container-image-name simpledotnetregistry565.azurecr.io/simple-dotnet-service:latest \
        --container-registry-url https://simpledotnetregistry565.azurecr.io
  
  - name: Enable Managed Identity for ACR access
    run: |
      az webapp identity assign \
        --name simple-dotnet-service \
        --resource-group simple-dotnet-service-rg \
        --identities [system]
  
  - name: Restart App Service
    run: |
      az webapp restart \
        --name simple-dotnet-service \
        --resource-group simple-dotnet-service-rg
```

Same OIDC flow, fresh token for this job. Now we tell App Service:
- "Here's your container image"
- "Use Managed Identity to pull it from ACR"
- "Restart so you pick up the new version"

---

## The One-Time Setup: Trusting GitHub in Azure

This is the magic part. In your Azure subscription, you create a **Federated Credential** that says: "Trust OIDC tokens from GitHub when they come from this specific repository on the main branch."

### Option 1: Azure Portal (GUI)

1. Go to **Azure Portal** → Search for **App registrations**
2. Click your app registration
3. Go to **Certificates & secrets** (left sidebar)
4. Click **Add a credential** → **Federated credential**
5. Choose scenario: **GitHub Actions deploying Azure resources**
6. Fill in:
   - **Organization**: your-github-org
   - **Repository**: simple-dontnet-service
   - **Entity type**: Branch
   - **Branch name**: main
7. Click **Add**

### Option 2: Azure CLI (Faster)

```bash
# Get your app registration object ID
APP_OBJECT_ID=$(az ad app list --display-name "YourAppName" --query "[0].id" -o tsv)

# Create the federated credential
az ad app federated-credential create \
  --id $APP_OBJECT_ID \
  --parameters '{
    "name": "GitHub-Actions-simple-dotnet-service",
    "issuer": "https://token.actions.githubusercontent.com",
    "subject": "repo:your-org/simple-dontnet-service:ref:refs/heads/main",
    "audiences": ["api://AzureADTokenExchange"]
  }'
```

That's it. After this, GitHub can deploy your app without any additional secrets.

---

## Why This Matters (Security Wins)

Let me be specific about what we've gained:

### ❌ Old Way Problems
- Password stored 24/7 in GitHub (compromise window is huge)
- Manual rotation required (easy to forget)
- Password visible if accidentally logged (one mistake ruins everything)
- Same credential used for weeks (compromise goes undetected longer)

### ✅ New Way Benefits
- **No passwords stored** - Just IDs, which are public information
- **Automatic token generation** - New token every workflow run
- **Tokens expire in minutes** - Compromise window is tiny
- **Tokens are signed** - GitHub and Azure verify cryptographically
- **Audit trail** - Exactly which repo ran which deployment when
- **Scope-limited** - Token only works for this repo/branch combination

---

## Visualizing the Flow

```
GitHub Push
    ↓
build-and-test job (no Azure)
    ↓
build-and-push-docker job
    ├─ Generate OIDC token (GitHub private key)
    ├─ Send to Azure
    ├─ Azure validates (GitHub public key)
    ├─ Azure checks federated credential
    ├─ Azure issues access token
    ├─ Build Docker image
    └─ Push to ACR
    ↓
deploy-to-app-service job
    ├─ Generate fresh OIDC token
    ├─ Exchange for new access token
    ├─ Configure App Service
    ├─ Enable Managed Identity
    └─ Restart App Service
    ↓
Application running with new version
```

---

## The Real-World Impact

I've been using this setup in production for months. Here's what changed:

1. **I stopped checking if passwords were leaked** - They don't exist
2. **Zero credential rotation overhead** - It's automatic
3. **Better sleep at night** - No late-night "did someone get my credentials?" anxiety
4. **Cleaner audit logs** - Every deployment is traceable to a specific commit
5. **Easier for teams** - New team members don't need special secret management

And honestly? It was simpler to set up than I expected. A few minutes in the Azure Portal, and we were done.

---

## Getting Started

If you want to implement this yourself:

1. **Create or identify your Azure App Registration** - You need the client ID
2. **Get your tenant ID and subscription ID** - These are in Azure Portal
3. **Create the federated credential** - Use the Portal or CLI command above
4. **Add three GitHub secrets**:
   - `AZURE_CLIENT_ID`
   - `AZURE_TENANT_ID`
   - `AZURE_SUBSCRIPTION_ID`
5. **Update your workflow** - Use the `azure/login@v2` action with OIDC (add `id-token: write` permissions)
6. **Delete old secrets** - Like `AZURE_CREDENTIALS` or `REGISTRY_PASSWORD`

You can see the full workflow in our repository: `.github/workflows/azure-app-service-deploy.yml`

---

## What About Other Platforms?

This pattern works with:
- **AWS** - OIDC provider, similar setup
- **GCP** - Workload Identity Federation
- **HashiCorp Cloud** - OIDC support
- **Any platform with OIDC support** - The pattern is standardized

---

## Final Thoughts

Keyless deployment isn't bleeding-edge anymore. It's the sensible default. If you're still storing full credentials in GitHub secrets, I'd encourage you to make the switch.

Your future self will thank you when you don't have to panic about credential rotations at 2 AM.

---

**Questions or running into issues?** The [Azure documentation on OIDC](https://learn.microsoft.com/en-us/azure/active-directory/workload-identities/workload-identity-federation) is excellent. Also check out [GitHub's official guide](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect).

Happy deploying! 🚀

