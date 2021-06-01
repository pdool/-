[TOC]

![image-20201010234737816](..\..\img\20201010\1.png)

序言：继续接上篇的live template ，idea骚操作虽然好，但是使用范围有限，只能是一段代码，无法对一些重复的逻辑，重复的类进行处理，既然我们遇到了这个问题别人也会遇到，那有没有现成的技术方案呐？of course !今天就介绍下偷懒大杀器——Freemaker。【FreeMarker 是一款 *模板引擎*： 即一种基于模板和要改变的数据， 并用来生成输出文本(HTML网页，电子邮件，配置文件，源代码等)的通用工具。 它不是面向最终用户的，而是一个Java类库，是一款程序员可以嵌入他们所开发产品的组件】。官方的解释真的是高级，用一句话来说就是给程序员使用的，用来做内容生成的。

## 1、应用场景：

FreeMarker最初的设计，是被用来在MVC模式的Web开发框架中生成HTML页面的，它没有被绑定到 Servlet或HTML或任意Web相关的东西上。它也可以用于非Web应用环境中。在我们的游戏项目中有一些缓存类，和查询数据库的代码是重复的机械性代码，因此用来生成项目内的一些通用代码结构，提高了生产效率，也减少了出错的可能性，机智。

## 2、实战

不管是Jsp 还是freemaker 都是内容替换，用公式来表达就是：模板 + 数据模型 = 输出。

![image-20201010234737816](..\..\img\20201010\2.png)

你要做的就是理解，然后记住那些该死的标签，用完然后忘掉，重复，轮回，o(╯□╰)o。

### 1.环境搭建

创建maven 项目，或者直接下载 下面对应的包加入你的项目中，看你方便，建议使用maven，自动下载包，多happy。

```
    <dependency>
         <groupId>org.freemarker</groupId>
           <artifactId>freemarker</artifactId>
           <version>2.3.30</version>
    </dependency>
```

### 2.代码

模板文件：

```
package ${packagePath};

public class ${className} {

    public static void main(String[] args) {
        System.out.println("${helloWorld}");
    }
}
```

生成代码

```java
package org.pdool.d20201010;

import freemarker.template.Configuration;
import freemarker.template.Template;
import java.io.*;
import java.util.HashMap;
import java.util.Map;

/**
 * @author 香菜
 */
public class Aain {
    private static final String TEMPLATE_PATH = "src/main/java/org/pdool/d20201010";
    private static final String CLASS_PATH = "src/main/java/org/pdool/d20201010/gen/";
    private static final String PACKAGE_PATH = "org.pdool.d20201010.gen";

    public static void main(String[] args) throws Exception {
        //1、 创建freeMarker配置实例
        Configuration configuration = new Configuration();
        String genClassName = "HelloFreeMaker";
        // 2、 获取模版路径
        configuration.setDirectoryForTemplateLoading(new File(TEMPLATE_PATH));
        // 3、 准备数据，等会替换用内容，key为模板内变量
        Map<String, Object> dataMap = new HashMap<>();
        dataMap.put("packagePath", PACKAGE_PATH);
        dataMap.put("className", genClassName);
        dataMap.put("helloWorld", "hello freeMaker,from 香菜");
        // 4、 加载模版文件
        Template template = configuration.getTemplate("helloFreemaker.ftl");
        // 5、将生成的内容
        File docFile = new File(CLASS_PATH + genClassName + ".java");
        Writer out = new BufferedWriter(new OutputStreamWriter(new FileOutputStream(docFile)));
        // 6、输出文件
        template.process(dataMap, out);
        System.out.println(genClassName + ".java 文件创建成功 !");
    }
}

```

### 3.运行结果

![image-20201010231751947](..\..\img\20201010\3.png)

## 3、官方网站

上面介绍了基本的使用，在你使用的过程中可能需要一些其他的标签，可以查阅官方网站。

官方网站的链接：http://freemarker.foofun.cn/index.html，网站上有完整的介绍，今天主要还是介绍下怎么快速的入门，官方的网站上太全，等你遇到问题再去查也不着急。

## 4、总结：关注我公众号【香菜聊游戏】

不过是内容替换而已，相信会Java的同学基本一眼就能看明白，和Jsp 同理，跟着规则来，将生成的内容写入到文件，免去一些日常的代码操作。

疯狂提升开发效率，留点时间划划水，找朋友聊聊天，带其他的同学飞，展示下你的技术，何乐而不为。

使用步骤

   使用步骤：
第一步：创建一个Configuration对象，直接new一个对象。构造方法的参数就是freemarker对于的版本号。
第二步：设置模板文件所在的路径。
第三步：设置模板文件使用的字符集。一般就是utf-8.
第四步：加载一个模板，创建一个模板对象。
第五步：创建一个模板使用的数据集，可以是pojo也可以是map。一般是Map。
第六步：创建一个Writer对象，一般创建一FileWriter对象，指定生成的文件名。
第七步：调用模板对象的process方法输出文件。
第八步：关闭流。

写文章不容易，希望能获得大家的支持，点赞，转发 三连，谢谢支持。

下一期写下游戏内资源的管理，记得关注哦！