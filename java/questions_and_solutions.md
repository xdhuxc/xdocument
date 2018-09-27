### Linux 下验证码无法显示问题
现象： 

web 应用在 windows 和 开发 Linux 服务器验证码显示正常，部署到新的生产 Linux 服务器上，验证码无法显示

异常日志：
```angularjs
Can't connect to X11 window server
```

问题原因：

Java 在图形处理时调用了本地的图形处理库。在利用 Java 做图形处理（比如，图片缩放、图片签名、生成报表），如果运行在 windows 上不会出问题。如果将程序移植到 Linux/Unix 上，有可能出现图形不能显示的错误，这是由于 Linux 的图形处理需要一个 X server 服务器。

问题解决方法：

1、检查 JDK 版本，不能使用 OpenJDK。

2、在 Tomcat 中，修改 catalina.sh，加入如下行：
```angularjs
JAVA_OPTS="-Djava.awt.headless=True"
```
如果名字为 java.awt.headless 的系统属性被设置为 True，那么 headless 工具包就会被使用。应用程序可以执行如下操作：
* 创建轻量级组件。
* 收集关于可用的字体、字体指标和字体设置的信息。
* 设置颜色来渲染准备图片。
* 创造和获取图像，为渲染准备图片。
* 使用 java.awt.PrintJob，java.awt.print.* 和 javax.print.* 类里的打印方法。

3、在 Jetty 中，修改 jetty.sh 中 JAVA_OPTIONS 参数，加入如下内容：
```angularjs
JAVA_OPTIONS+=("-Djava.awt.headless=True")
```

4、终极解决
一般是在程序开始激活 headless 模式，告诉程序，现在要工作在 Headless 模式下，就不要指望硬件帮忙了，得依靠系统的计算能力模拟出这些特性来
```angularjs
public void init(ServletConfig config) throws ServletException {
    super.init(config);
    height = Integer.parseInt(getServletConfig().getInitParameter("height"));
    width = Integer.parseInt(getServletConfig().getInitParameter("width"));
    System.setProperty("java.awt.headless", "True");
}
```


