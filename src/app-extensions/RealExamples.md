# Real-life examples

The app-extensions library is not just hypothetical code - it's been used in several applications to great effect!
Here are just a few examples of actual extensions:

## MusicPlayer

The [MusicPlayer](https://github.com/scorbo2/musicplayer) application has had extension capabilities for several
versions now. Extensions can provide additional functionality not provided by the application itself. In particular,
extensions can provide new fullscreen visualization effects to show while music is playing. 

The [Scenery extension](https://github.com/scorbo2/ext-mp-scenery) for MusicPlayer is an interesting example.
It shows gently scrolling landscape scenery images, with programmable "tour guides" that can appear at configured
intervals to offer commentary either on the currently playing track, or on the currently shown scenery image.

## ImageViewer

The [ImageViewer](https://github.com/scorbo2/imageviewer) application was also designed with extension in mind.
Almost all built-in functionality within the app is extensible, allowing the creation of extensions that can do
almost any kind of manipulation on the currently shown image. Here are some example extensions:

- [Image Transform](https://github.com/scorbo2/ext-iv-image-transform) - rotate or flip an image
- [Image Resize](https://github.com/scorbo2/ext-iv-image-resize) - resize an image or a directory of images
- [Image Converter](https://github.com/scorbo2/ext-iv-image-converter) - convert an image or a directory of images between jpg and png formats
- [Full Screen](https://github.com/scorbo2/ext-iv-fullscreen) - presents a directory of images as a fullscreen slideshow

## It starts with extensible application design

ImageViewer is actually a good example of an application that was designed with extensibility in mind. 
Take a look in particular at the [ImageViewerExtension](https://github.com/scorbo2/imageviewer/blob/master/src/main/java/ca/corbett/imageviewer/extensions/ImageViewerExtension.java)
abstract class to see what kind of functionality is exposed to ImageViewer extensions. The
[ImageViewerExtensionManager](https://github.com/scorbo2/imageviewer/blob/master/src/main/java/ca/corbett/imageviewer/extensions/ImageViewerExtensionManager.java)
class handles interrogation of currently loaded and enabled extensions to see what features they offer.

It's difficult, but not impossible, to add extension capabilities to an application as an afterthought.
It's therefore recommended to think about possible extension points early in the application's development!
