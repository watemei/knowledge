**引言**

使用goole Kaptcha 生成的验证码样式如下图，

![](file:///C:\Users\WANGXB~1\AppData\Local\Temp\ksohtml\wpsDD37.tmp.jpg)

色彩比较朴素、当然也可以通过配置 init-param来实现背景，总的来说还是不太好看，所以本文预期结果如下图：

![](file:///C:\Users\WANGXB~1\AppData\Local\Temp\ksohtml\wpsDD38.tmp.jpg)

**Google Kaptcha简介**

kaptcha 是一个非常实用的验证码生成工具。有了它，你可以生成各种样式的验证码，因为它是可配置的。kaptcha工作的原理是调用 com.google.code.kaptcha.servlet.KaptchaServlet，生成一个图片。同时将生成的验证码字符串放到 HttpSession中。

使用kaptcha可以方便的配置：

验证码的字体

验证码字体的大小

验证码字体的字体颜色

验证码内容的范围\(数字，字母，中文汉字！\)

验证码图片的大小，边框，边框粗细，边框颜色

验证码的干扰线\(可以自己继承com.google.code.kaptcha.NoiseProducer写一个自定义的干扰线\)

验证码的样式\(鱼眼样式、3D、普通模糊……当然也可以继承com.google.code.kaptcha.GimpyEngine自定义样式\)

可以看出功能还是蛮多的，配置也提出来。

**google Kaptcha的使用及配置**

**maven配置引用**

```
<!-- https://mvnrepository.com/artifact/com.google.code.kaptcha/kaptcha -->
<dependency>
    <groupId>com.google.code.kaptcha</groupId>
    <artifactId>kaptcha</artifactId>
    <version>2.3</version>
</dependency>
```

**web.xml配置KaptchaServlet**

&lt;!--Kaptcha 验证码  --&gt;&lt;!--

&lt;servlet&gt;

&lt;servlet-name&gt;kaptcha&lt;/servlet-name&gt;

&lt;servlet-class&gt;com.google.code.kaptcha.servlet.KaptchaServlet&lt;/servlet-class&gt;

&lt;init-param&gt;

&lt;param-name&gt;kaptcha.border&lt;/param-name&gt;

&lt;param-value&gt;no&lt;/param-value&gt;

&lt;/init-param&gt;

&lt;init-param&gt;

&lt;param-name&gt;kaptcha.border.color&lt;/param-name&gt;

&lt;param-value&gt;105,179,90&lt;/param-value&gt;

&lt;/init-param&gt;

&lt;init-param&gt;

&lt;param-name&gt;kaptcha.textproducer.font.color&lt;/param-name&gt;

&lt;param-value&gt;red&lt;/param-value&gt;

&lt;/init-param&gt;

&lt;init-param&gt;

&lt;param-name&gt;kaptcha.image.width&lt;/param-name&gt;

&lt;param-value&gt;250&lt;/param-value&gt;

&lt;/init-param&gt;

&lt;init-param&gt;

&lt;param-name&gt;kaptcha.image.height&lt;/param-name&gt;

&lt;param-value&gt;90&lt;/param-value&gt;

&lt;/init-param&gt;

&lt;init-param&gt;

&lt;param-name&gt;kaptcha.textproducer.font.size&lt;/param-name&gt;

&lt;param-value&gt;70&lt;/param-value&gt;

&lt;/init-param&gt;

&lt;init-param&gt;

&lt;param-name&gt;kaptcha.session.key&lt;/param-name&gt;

&lt;param-value&gt;code&lt;/param-value&gt;

&lt;/init-param&gt;

&lt;init-param&gt;

&lt;param-name&gt;kaptcha.textproducer.char.length&lt;/param-name&gt;

&lt;param-value&gt;4&lt;/param-value&gt;

&lt;/init-param&gt;

&lt;init-param&gt;

&lt;param-name&gt;kaptcha.textproducer.font.names&lt;/param-name&gt;

&lt;param-value&gt;宋体,楷体,微软雅黑&lt;/param-value&gt;

&lt;/init-param&gt;

&lt;/servlet&gt;

&lt;servlet-mapping&gt;

&lt;servlet-name&gt;kaptcha&lt;/servlet-name&gt;

&lt;url-pattern&gt;/ClinicCountManager/kaptcha.jpg&lt;/url-pattern&gt;

&lt;/servlet-mapping&gt;

**前端页面设置**

&lt;img src="captcha.jpg" class="login\_code\_icon"

```
                            onclick="this.src='captcha.jpg?'+Math.random\(\)" /&gt;
```

**效果展示**

![](file:///C:\Users\WANGXB~1\AppData\Local\Temp\ksohtml\wpsDD39.tmp.jpg)

开始尝试通过配置实现，但是效果一字体只能单一颜色，背景样式鱼眼样式、3D、普通模糊,不太好看，只能看源码找方案;

**源码分析**

进入com.google.code.kaptcha.servlet.KaptchaServlet类中是通过BufferedImage bi = this.kaptchaProducer.createImage\(capText\);来绘制图片的。

![](file:///C:\Users\WANGXB~1\AppData\Local\Temp\ksohtml\wpsDD3A.tmp.jpg)

默认使用DefaultKaptcha

![](file:///C:\Users\WANGXB~1\AppData\Local\Temp\ksohtml\wpsDD3B.tmp.jpg)

继续查看createImage\(String text\)方法，要实现彩色效果的方法映入眼帘。

![](file:///C:\Users\WANGXB~1\AppData\Local\Temp\ksohtml\wpsDD4C.tmp.jpg)

**重写DefaultKaptcha为DefaultCASKaptcha**

重写需要干两件事情：

1、验证码背景图片为彩色；

2、验证码字体的位置及彩色设置；

**验证码背景图片为彩色**

![](file:///C:\Users\WANGXB~1\AppData\Local\Temp\ksohtml\wpsDD4D.tmp.jpg)

通过生成随机干扰线实现。

**验证码字体的位置及彩色设置**

![](file:///C:\Users\WANGXB~1\AppData\Local\Temp\ksohtml\wpsDD4E.tmp.jpg)

需要根据字体大小实现动态位置确定，此处使用源码中的思路：

**1、定位初始位置；**

```
    int startPosX = width / \(2 + text.length\(\)\);

    int startPosY =\(height - fontSize\) / 5 + fontSize;
```

**2、通过GlyphVector获取字体外边框的宽度；**

```
    GlyphVector gv =  chosenFonts\[i\].createGlyphVector\(frc, rand\);

    double charWidth = gv.getVisualBounds\(\).getWidth\(\);
```

**3、通过位移增加绘制下一个验证码字。**

```
     startPosX = startPosX + \(int\) charWidth + 2;
```

![](file:///C:\Users\WANGXB~1\AppData\Local\Temp\ksohtml\wpsDD5F.tmp.jpg)

**修改web.xml配置**

&lt;servlet&gt;

&lt;servlet-name&gt;Kaptcha&lt;/servlet-name&gt;

&lt;servlet-class&gt;com.google.code.kaptcha.servlet.KaptchaServlet&lt;/servlet-class&gt;

&lt;init-param&gt;

&lt;param-name&gt;kaptcha.border&lt;/param-name&gt;

&lt;param-value&gt;no&lt;/param-value&gt;

&lt;/init-param&gt;

&lt;init-param&gt;

&lt;param-name&gt;kaptcha.producer.impl&lt;/param-name&gt;

&lt;param-value&gt;com.iigeo.cas.DefaultCASKaptcha&lt;/param-value&gt;

&lt;/init-param&gt;

...

&lt;/servlet&gt;

**最终效果图**

![](file:///C:\Users\WANGXB~1\AppData\Local\Temp\ksohtml\wpsDD60.tmp.jpg)

t/http�����n

