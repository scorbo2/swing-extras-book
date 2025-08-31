# LogoGenerator

It's worth mentioning an older class that's still around in `swing-extras`, and that is `LogoGenerator`.
This is the class that ultimately led to the creation of [ImageTextUtil](ImageTextUtil.md).

`LogoGenerator` has been used for years as a way to very quickly and easily generate very small,
text-based images, which can be suitable for use for application logo images or banner images
to be shown in an application. It can also be used to generate small square images suitable for
an application icon.

![LogoGenerator](logogenerator1.png "LogoGenerator")

## Generator logo images

We start by populating a `LogoProperty` object describing the image to be generated:

```java
public final class LogoProperty extends AbstractProperty {
    // ...

    private ColorType bgColorType;
    private Color bgColor;
    private Gradient bgGradient;

    private ColorType borderColorType;
    private Color borderColor;
    private Gradient borderGradient;

    private ColorType textColorType;
    private Color textColor;
    private Gradient textGradient;

    private int borderWidth;
    private boolean hasBorder;
    private boolean autoSize;
    private Font font;
    private int logoWidth;
    private int logoHeight;
    private int yTweak;
    
    // ... getters and setters omitted...
}
```

Once we have configured the properties we want, we can talk to the `LogoGenerator` class:

```java
public final class LogoGenerator {
    // ...
    
    public static BufferedImage generateImage(String text, LogoProperty preset) {...}
    public static void generateAndSaveImage(String text, LogoProperty preset, File outputFile) throws IOException {...}}
}
```

We see that the API here is quite simple. Just specify the text to render and your `LogoProperty` object,
and decide whether you want to receive a `BufferedImage` generated in memory, or whether you want to save
the resulting image directly to disk. Easy!
