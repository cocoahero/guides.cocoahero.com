---
layout: post
title: "Android Google Maps SDK v2 - Custom Tile Providers"
summary: "The Google Maps SDK v2 for Android allows great flexibility. Providing custom tiles is just one of the many features available. In this inaugural guide, you will learn how you can use your own tiles with ease."
tags:
    - Android
    - Google Maps SDK
    - Custom
    - MBTiles
    - MapBox
    - Java
---
Back in February of 2013, Google released the Google Maps for Android SDK version 2. There were many improvements made over the initial version including first class fragment support, vector based tiles, and 3D building footprints. One of the most interesting new features of the API is the ability to add custom tile overlays. This allows developers to potentially use their own online (or offline) tile sources and render them easily on the map.

### Implementing TileProvider
The first step in getting your custom tiles rendered on to the map is to implement the [TileProvider][1] interface. This involves implementing a single method, `getTile(int x, int y, int zoom)`. The first two parameters describe the coordinate of the tile, while the third describes the requested zoom level. Note that in the "Google Maps" tile coordinate system, `'0,0'` is the top left corner of the map. Some provider's systems are inverted, with `'0,0'` being in the bottom left. Be sure to adjust the `'y'` coordinate if necessary to match your tile source.

What you must return at the end of this method is an instance of a [Tile][2]. A `Tile` is essentially a wrapper around a raw bitmap. To create a new instance, you must know the width and height in pixels (usually 256) of the tile, and have the raw `byte[]` containing the bitmap's image data. If you don't have the tile that is requested, simply return the `TileProvider#NO_TILE` static instance.

#### Online Tile Providers
At this point, if all you want to do is use an online tile provider such as [MapBox][3] you may be panicking a bit. Don't worry, you don't have to deal with multithreaded networking or even touch a single HTTP library! Provided within the SDK is a handy abstract class called [UrlTileProvider][4]. What this class does is implement the `TileProvider` interface and handles all the networking, threading, and caching. All you have to do is provide the URL! Below is an example of using `UrlTileProvider` to provide tiles from a [MapBox][3] map.

<script src="https://gist.github.com/cocoahero/f9e1553bb7f7fc121a35.js"></script>

### Adding a TileOverlay
Once you have your custom tile provider implemented, it's time to add it to the map view. In order to do so, you must first create an instance of [TileOverlayOptions][5] to configure your overlay. Below is some sample code on how this may be accomplished.

<script src="https://gist.github.com/cocoahero/5e3bd2f1abc5765711fb.js"></script>

### Ditching Google's Data
So you have your fancy new custom tile provider and you want to show it off. If you are using complete, opaque custom tiles, chances are you don't need Google's. Additionally, if your application is designed to work near offline, you may want to save the extra bandwidth. You can completely turn off the Google base layer by using the method `GoogleMap#setMapType(int)` and providing `GoogleMap#MAP_TYPE_NONE`. This will also consequently remove any Google data attributions from being displayed on top of your custom data. However, the "Google" watermark displayed in the bottom left will [unfortunately always be there][6]. So if you worry about your data possibly being incorrectly attributed to Google, this library may not be for you.

Overall, the Google Maps SDK v2 for Android is an extremely handy and flexible library that apps that use to display any sort of geographic information. Adding a map to an application can not only add a great user experience, but also a new way of visualizing data.


[1]: http://developer.android.com/reference/com/google/android/gms/maps/model/TileProvider.html "TileProvider"
[2]: http://developer.android.com/reference/com/google/android/gms/maps/model/Tile.html "Tile"
[3]: http://mapbox.com/ "MapBox"
[4]: http://developer.android.com/reference/com/google/android/gms/maps/model/UrlTileProvider.html "UrlTileProvider"
[5]: http://developer.android.com/reference/com/google/android/gms/maps/model/TileOverlayOptions.html "TileOverlayOptions"
[6]: https://code.google.com/p/gmaps-api-issues/issues/detail?id=4648
