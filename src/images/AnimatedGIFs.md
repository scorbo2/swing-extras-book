# Animated GIFs

The `ImagePanel` class offers native support for animated GIF images! This is actually supplied
by the built-in Java class `ImageIcon`.

```java
if (imageFile.getName().toLowerCase().endsWith(".gif")) {
    ImageIcon icon = new ImageIcon(imageFile.getAbsolutePath());
    imagePanel.setImageIcon(icon);
}
else {
    BufferedImage image = ImageUtil.loadImage(imageFile);
    imagePanel.setImage(image);
}
```

The above example code will invoke the correct `ImagePanel` method depending on whether
the image is a GIF image or not. Generally speaking, `ImagePanel` prefers dealing with
`BufferedImage` instances, because we can store the entire raster of pixels in memory
to make it easier to handle things like scaling or stretching. But, animated GIF images
must be represented by an `ImageIcon` instance, so slightly different handling is required.

All of the same display options still work! You can stretch, scale, and zoom animated
GIF images even as the animation is playing. This can either be done programmatically
or via user input (via mouse clicks or mouse wheel, if those options are enabled in the image panel).
