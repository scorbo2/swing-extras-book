# ImageUtil

The `ImageUtil` utility class provides convenient wrappers around some functionality in Java's image IO library.

## Loading and saving images

We get the following utility methods (javadoc omitted for brevity):

```java
public static ImageIcon loadImageIcon(final File file) {...}
public static ImageIcon loadImageIcon(final URL url) {...}
public static BufferedImage loadImage(final URL url) throws IOException {...}
public static BufferedImage loadImage(final File file) throws IOException {...}
public static void saveImage(final BufferedImage image, final File file) throws IOException {...}
public static void saveImage(final BufferedImage image, final File file, float compressionQuality) {...}
public static void saveImage(final BufferedImage image, final File file, ImageWriteParam writeParam) {...}
public static void saveImage(final BufferedImage image, final File file, ImageWriter writer, ImageWriteParam writeParam) {...}
```

We see that we have numerous options for easily loading and saving images in the following formats:
- jpeg, with configurable compression options
- png for lossless image storage, includes support for transparency
- gif, including support for animated gifs

## Generating thumbnail images

`ImageUtil` also contains facilities for easily generating "thumbnail" images (these are scaled-down versions
of larger images, suitable for use as an image preview):

```java
public static BufferedImage generateThumbnail(final File image,
                                              final int width,
                                              final int height) throws IOException {...}
public static BufferedImage generateThumbnail(final BufferedImage sourceImage,
                                              final int width,
                                              final int height) {...}
public static BufferedImage generateThumbnailWithTransparency(final File image,
                                                              final int width,
                                                              final int height) throws IOException {...}
public static BufferedImage generateThumbnailWithTransparency(final BufferedImage sourceImage,
                                                              final int width,
                                                              final int height) {...}
```

We see that it can work either with images already in memory (in the form of a `BufferedImage`), or with
images saved on disk (sparing you from having to do the I/O).
