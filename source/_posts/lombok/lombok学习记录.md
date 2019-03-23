layout: title
title: lombok学习记录
date: 2019-03-21 22:45:01
tags:
- lombok
categories:
- lombok
---

### lombok介绍
lombok是一个java的jar包，第三方字节码修改库，简化重复性简单的代码，比如java类的getter、setter、tostring()、equals()、构造函数、hashcode等方法在编译期的自动生成，以及builder模式创建实例。

### 原理
使用JSR 269: Pluggable Annotation Processing API来实现https://jcp.org/en/jsr/detail?id=269，编译期修改抽象语法树，如增加get方法节点等，然后生成字节码

### 利弊
利:
- 简化重复的代码，使类看起来简洁
- builder模式好用
  
弊:
- 破坏源代码完整性，可阅读性
- 依赖额外lombok jar包，增加出问题的风险
- ide等必须安装lombok插件，不然报错，而且自动生成的方法没法从调用处跳转
### 遇到的坑
#### lombok显示声明构造函数的一个坑-字段值顺序乱掉
有下例：

```
import lombok.Builder;
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;
@Setter
@Getter
@ToString
@Builder
public class Button {
    private String content;

    private String color;

    private String url;

    public Button(String content, String url, String color) {
        this.content = content;
        this.color = color;
        this.url = url;
    }

    public static final Button DEFAULT_BUTTON = Button.builder()
            .content("xxcontent")
            .color("#FE6D4D")
            .url("http://www.xx.com")
            .build();

    public static void main(String[] args) {
        Button button = Button.DEFAULT_BUTTON;
        System.out.println(button);
    }
}
```

##### 问题
上例子main函数预期输出为:

```
    Button(content=xxcontent, color=#FE6D4D, url=http://www.xx.com)

```
而实际为：
```
Button(content=xxcontent, color=http://www.xx.com,url=#FE6D4D)
```

url的值与color的值顺序乱了，为啥？

##### 分析编译后的class文件
```
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//
public class Button {
    private String content;
    private String color;
    private String url;
    public static final Button DEFAULT_BUTTON = builder().content("xxcontent").color("#FE6D4D").url("http://www.xx.com").build();

    ////这里..，url为第二个形参，与字段声明顺序不一致
    public Button(String content, String url, String color) {
        this.content = content;
        this.color = color;
        this.url = url;
    }

    public static void main(String[] args) {
        Button button = DEFAULT_BUTTON;
        System.out.println(button);
    }

    public static Button.ButtonBuilder builder() {
        return new Button.ButtonBuilder();
    }
    //getter/setter

    public String toString() {
        return "Button(content=" + this.getContent() + ", color=" + this.getColor() + ", url=" + this.getUrl() + ")";
    }

    public static class ButtonBuilder {
        private String content;
        private String color;
        private String url;

        ButtonBuilder() {
        }

        public Button.ButtonBuilder content(String content) {
            this.content = content;
            return this;
        }

        public Button.ButtonBuilder color(String color) {
            this.color = color;
            return this;
        }

        public Button.ButtonBuilder url(String url) {
            this.url = url;
            return this;
        }

        public Button build() {
            return new Button(this.content, this.color, this.url);////这里..
        }
        //tostring()
    }
}

```

##### 结论
不仔细看代码或者debug，其实不容易看出来区别，问题就在于ButtonBuilder的build()方法内部使用了Button类的全参构造函数，而且其字段顺序和Button类各个字段顺序一致；而观察Button类内部的全参构造函数发现，url是第二个形参，跟字段顺序不一致，所以导致值错乱。

建议：构造函数最好使用ide自动生成，形参顺序保持和字段顺序一致，避免出现类似问题。


说明：如果不显示声明带参数的构造函数，lombok会自动生成带参数的构造函数，是没问题的，顺序是一致的。

