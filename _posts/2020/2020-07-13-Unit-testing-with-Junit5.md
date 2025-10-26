---
layout: post
title: "Unit Testing With JUnit5"
description: ""
---
# Unit Testing With JUnit5
This is not an article about why you should do unit testing, rather this is about the how part. If you are writting in Java, probably its very synonymous to "writing JUnits", this is because [junit](https://junit.org/junit5/) is one of the most popular libraries out there.

Let's get started with a simple example

For Junit4 you would need the following dependency if you are doing maven.
```
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

App.java
```
public class App {
    int add(int x, int y) {
        return x+y;
    }
}
```
AppTest.java
```
public class AppTest {
    private App app;

    @Before
    public void setup() {
         app = new App();
    }

    @Test
    public void testAdd() {
        Assert.assertEquals(5, app.add(2, 3));
    }
}

```
**Exceptions**

Its good to test the negative scenarios also i.e. if you are returning error in some condition , validate that as well.

For example if we want to avoid integer overflow or underflow 

App.java

```
    int addSafely(int x, int y) {
        return Math.addExact(x, y);
    }
```
AppTest.java
```
    @Test(expected = ArithmeticException.class)
    public void testAddSafely() {
        app.addSafely(Integer.MAX_VALUE, 1);
    }
```

in junit 5

```
    @Test
    public void testAddSafely() {
        final Exception exception = assertThrows(ArithmeticException.class, () -> app.addSafely(Integer.MAX_VALUE, 1));
        assertEquals("integer overflow", exception.getMessage());
    }
```
In above example you are just testing one case. However when there is a complex logic in your main method, you may want to have test for several combinations (including exceptions cases)

```
    @ParameterizedTest
    @CsvSource(value = { "2, 3, 5", "1, 1, 2" }, delimiter = ',')
    public void testAdd(final int x, final int y, final int sum) {
        assertEquals(sum, app.add(x, y));
    }
```
The above test runs all the combinations mentioned in the @CsvSource annotation. However  @[ParameterizedTest](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests) is a Junit5 feature. This needs the following additional dependency. 

```
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-params</artifactId>
      <version>5.4.2</version>
      <scope>test</scope>
    </dependency>
```
Along with the following for the basic Junit5 tests
```
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-engine</artifactId>
      <version>5.5.2</version>
      <scope>test</scope>
    </dependency>
```
For junit4 there is only [@RunWith(Parameterized.class)](https://github.com/junit-team/junit4/wiki/Parameterized-tests) is available so the whole class is repeatably run instead of just a specific method.

**Mocking**

Not all methods are so simple like above which has just some input and output without any call other class/methods. If those other method and class are already having tests (in fact they should have their own test) we should not worry about them and assume they work properly while writing test for the caller method. This is why we need to "mock" them.

Let's say we use a logger which is a core jdk feature and we know it would work, so we can mock it and just make sure it was called properly from our code because that's where we can goof up. 

`App.java`
 ```
     int add(int x, int y) {
        logger.info("Adding numbers");
        return x + y;
    }
 ```
 
 `AppTest.java`
 ```
 @ExtendWith(MockitoExtension.class)
 public class AppTest {

    private App app;
    
    @Mock
    private Logger logger;

    @BeforeEach
    public void setup() {
        doNothing().when(logger).info(any(String.class));
        app = new App(logger);
    }
    
    public void testAdd() {
        assertEquals(5, app.add(2, 3));
        verify(logger, times(1)).info("Adding numbers");
    }
 ```

If you don't have exact match to be done,  there are several other matches available 
e.g.
If you have the following log statement `logger.info("Result is "+x);`
You need
```
org.mockito.ArgumentMatchersstartsWith("Result is");
```

**Asserting objects**
Assert scenarios are not always  simple literal match, it could be a complex object in which you want to match some attributes. `ReflectionMatcher` comes handy in this case.

```
Assert.assertTrue(new ReflectionEquals(expected, excludeFields).matches(actual));
```


