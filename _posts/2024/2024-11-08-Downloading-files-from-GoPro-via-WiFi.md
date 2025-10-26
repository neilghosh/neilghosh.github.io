---
layout: post
title: "Downloading files from GoPro to Desktop using WiFi"
description: "In this post, we will download videos and photos from GoPro using its WiFi to desktop"
---

The most convenient way of downloading photos and videos from GoPro is to use the GoPro Quik in mobile and desktop. In Desktop it uses USB cable, however if for some reason it does not work via cable, once can download wirelessly.

1. Enable Wifi in GoPro. (Fastest way is to connect from mobile app and view media).
2. Connect your PC/Mac to the GoPro's WiFi e.g. "GoPro Hero 10 Black"
3. Navigate to http://10.5.5.9/videos/DCIM/100GOPRO/, it should give the directory listing.
4. You can download files one by one by right clicking and downloading 
5. You can use `wget` to download in a bulk as well. `wget -r -np -nH -nc -A MP4 "http://10.5.5.9/videos/DCIM/100GOPRO/?dd"`

One can play with `wget`'s options to download specific file pattern i.e. extension or even append `dd` to the query string to sort the listing in and download oldest file first.
    
