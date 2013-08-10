![Composer](http://getcomposer.org/img/logo-composer-transparent.png)


#Composer (作曲家)，PHP 的依赖管理器

##介绍

Composer是PHP中的一个依赖管理工具. 它可以让你声明自己项目所依赖的库，然后它将会在项目中为你安装这些库。

##依赖管理
Composer不是包管理器。是的，它实际上和"包"或者库打交道，但是它是以项目为单位进行管理，把它们安装到你项目中的一个目录（例如vendor）。默认情况下它不会以全局的方式安装任何东西。因此，它是一个依赖管理器。

这个想法并不新鲜，Composer的灵感是来自于node的npm和ruby的bundler。但是目前PHP还没有一个这样的工具。

###Composer解决的问题是：

a) 你有一个依赖N多库的项目。    
b) 这些库中一些又依赖于其他的库。    
c) 你声明你所依赖的库。   
d) Composer找出哪些包的哪个版本将会被安装，然后安装它们（也就是把它们下载到你的项目中）。

##声明依赖关系
假设你正在创建一个项目，然后你需要一个日志操作的库。你决定使用monolog。为了把它加入到你的项目中，你需要做的就是创建一个名为composer.json的文件，其描述这个项目的依赖关系。
```
{
    "require": {
        "monolog/monolog": "1.2.*"
    }
}
```
我们简单的描述说我们的项目依赖某个monolog/monolog包，版本只要是以1.2开头的就行。

##系统要求
Composer需要PHP 5.3.2+才能运行。一些灵敏的PHP设置和编译选项也是必须的，不过安装程序（installer）会警告你任何不兼容的地方。

如果想要从源码而不是简单的从zip压缩包中安装软件包的话，你将需要git，svn或者hg，这依赖于软件包是通过什么进行版本控制的。

Composer是兼容多平台的，并且我们力争使其在Windows，Linux和OSX上的运行无差异。

##安装 - *nix平台
下载Composer可执行程序 

###局部安装

为了获取Composer，我们需要做两件事。第一个是安装Composer（前面说过了，这意味下载它到你的项目中）：

```
$ curl -sS https://getcomposer.org/installer | php
```

这只会检查一些PHP设置，然后下载composer.phar到你的工作目录中。这个文件是Composer二进制文件。它是一个PHAR (PHP archive)，PHP的归档格式，也可以像其他命令一样在命令行上运行。

你可以使用`--install-dir`选项，并且提供一个目标目录（可以是绝对或者相对路径）从而把Composer安装到一个指定的目录：

```
$ curl -sS https://getcomposer.org/installer | php -- --install-dir=bin
```

###全局安装

你可以把这个文件放到任何你想放的地方。如果你把它放到你的PATH中，你就可以全局访问它了。在类unix系统中你甚至可以使它可执行，并且调用的时候不需要php。

你可以执行这些命令从而能够在你的系统上简单的访问composer：

```
$ curl -sS https://getcomposer.org/installer | php
$ sudo mv composer.phar /usr/local/bin/composer
```

然后，只需要执行composer命令来运行Composer，而不是php composer.phar。

##安装 - Windows平台

###使用安装程序

这是在你的机器上安装Composer最简单的方法。

下载并运行Composer-Setup.exe，它将会安装最新的Composer版本并且设置好PATH，然后你就可以在命令中的任何目录下调用composer了。

###手动安装

切换到一个存在于PATH环境变量中的目录，然后执行安装代码片段来下载composer.phar：

```
C:\Users\username>cd C:\bin
C:\bin>php -r "eval('?>'.file_get_contents('https://getcomposer.org/installer'));"
```

创建一个新的以.bat结尾的composer文件:

```
C:\bin>echo @php "%~dp0composer.phar" %*>composer.bat
```

关闭你当前的终端。打开一个新的终端测试一下：

```
C:\Users\username>composer -V   
Composer version 27d8904

C:\Users\username>
```

##使用Composer

我们接下来要使用Composer来安装项目的依赖。如果你在当前目录下没有一个叫作composer.json的文件，请跳到基本使用章节。

为了解决并下载依赖，运行install命令：

```
$ php composer.phar install
```

如果你是全局安装，并且目录下没有phar文件，那么运行这个：

```
$ composer install
```

如果是上面的例子，这个操作将会下载monolog到`vendor/monolog/monolog`目录。

###自动加载 

除了下载库之外，Composer也会创建一个自动加载文件，这个文件能够自动加载Composer下载的库中所有的类。如果想使用它，只需要在你代码启动的地方加上如下代码：

```
require 'vendor/autoload.php';
```

哇哦！现在开始使用monolog吧! 如果想进一步学习Composer，继续阅读「基本使用」章节。 如果想要找需要的package，到Packagist。

###参考
英文原文：http://getcomposer.org/doc/00-intro.md