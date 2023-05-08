# Image Loading

<image src="assets/design.png" width="500"/>

## <a href="technical.md">Technical Documentation</a>

## Gathering Requirements
- Are we designing an application or only library?
- Are we designing a library that loads images or handles a collection of photos?
- Where do we need to load images from?
- Do we need to add local storage based on disk or only memory?
- Do we need to add expired time storage disk?
- Do we need to cancel download image when app page remove?
- Do we need to add dynamic image width and height?
- It would be good to provide thumbnails, customizable placeholders, loading indicators, and transition animation for UI-targets - but we should probably leave it out of scope.

## Memory cache
It would be 1/8 from available memory based on google docs <a href="https://developer.android.com/topic/performance/graphics/cache-bitmap#memory-cache">memory cache</a>.
Because there's a lot of factor for consider what size for cache like:
- How memory intensive is the rest of your activity and/or application?
- How many images will be on-screen at once? How many need to be available ready to come on-screen?
- What is the screen size and density of the device?
- What dimensions and configuration are the bitmaps and therefore how much memory will each take up?

## Disk cache
While the memory cache is checked in the UI thread, the disk cache is checked in the background thread. Disk operations should never take place on the UI thread. When image processing is complete, the final bitmap is added to both the memory and disk cache for future use.

Instead set maximum memory in disk cache, I think it better if we just add expire for image file like 1 week or 1 month.

## Slow Internet
If use has slow internet we can show low quality image first like if image not show in short amount of time, we show low quality image first like show placeholder.
```java
loadHiResImage() // 400 x 400
// if user has slow internet
loadLowResImage() // 50 x 50
// after finished high res image
showHiResImage()
```

## <a href="imageloading/technical.md">Technical Documentation</a>

## TODO
- Http Client Caching