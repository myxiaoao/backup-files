#Composer 的库

##适合阅读对象：
1、了解什么是 Composer（不了解的看这里：[《Composer (作曲家)，PHP 的依赖管理器》](https://github.com/huanghua581/laravel/blob/master/composer/00-intro.md)）  
2、想了解如何让你的包能通过 Composer 安装  
3、想了解 Composer 大的体系  

##一、所有项目都是一个包

如果你的目录里有一个 `composer.json`，那这个目录就是个包。当你在项目中添加了一项 `require`，你就创建了一个依赖其他包的包。你的项目和外部库的唯一区别就是：你的项目是一个没有名字的包。

如果想让你的包能通过 Composer 安装，你需要给它指定一个名字。通过给 composer.json 添加 name 字段即可：

```
{
    "name": "acme/hello-world",
    "require": {
        "monolog/monolog": "1.0.*"
    }
}
```

例子中，项目的名称为 `acme/hello-world`，其中 `acme` 是供应方（vendor）的名字。供应方名是必须强制提供的。

（注：如果你不知道供应方名该怎么取，你的 GitHub 用户名通常是不错的选择。因为包名是大小写敏感的，所以一般用小写字符并在词之间用横杠分隔）

##二、平台的包

Composer 也有平台的包，这些包是并不是由 Comoposer 安装，它们是安装在操作系统上的东西，Composer 将它们作为虚拟的包，以方便管理。它们包括：PHP 本身，PHP 的扩展，还有一些系统中的库。

###1、php 包
代表用户的 PHP 版本，这样就允许你对 PHP 版本定义依赖，比如，`>=5.4.0`。如果必须要 64 位版本的 PHP，你可以依赖 `php-64bit` 这个包。

###2、ext-<name> 包
你可以依赖 PHP 的扩展（包括核心扩展）。至于包的版本，这里有很多不一致，建议就用 `*`。比如你的项目用到 GD 库，就可以依赖 `ext-gd` 这个包。

###3、lib-<name> 包
你可以依赖一些 PHP 用到的系统库。比如：`curl`、`iconv`、`libxml`、`openssl` 等。

你可以用命令 `composer show –platform` 查看你本地的有效的平台包。比如我的结果是：

```
MacBook-Pro:~ composer$ composer show --platform
 
ext-bcmath     0         The bcmath PHP extension
ext-bz2        0         The bz2 PHP extension
ext-calendar   0         The calendar PHP extension
ext-ctype      0         The ctype PHP extension
ext-curl       0         The curl PHP extension
...
lib-curl       7.24.0    The curl PHP library
lib-iconv      1.11      The iconv PHP library
lib-libxml     2.7.8     The libxml PHP library
lib-openssl    0.9.8.18  The openssl PHP library
lib-pcre       8.02      The pcre PHP library
lib-xsl        1.1.26    The xsl PHP library
php            5.3.15    The PHP interpreter
php-64bit      5.3.15    The PHP interpreter (64bit)
```

##三、指定版本

你需要通过某种方式指定包的版本。不过当你在 Packagist 上发布包时，它会通过 VCS（version control system， 如：git，svn，hg）推断出版本信息，所以这种情况下就不需要指定版本。

如果你手工创建包，则必须指定它的版本，可以用 version 字段：

```
{
    "version": "1.0.0"
}
```

###1、Tags（标签）

对应每个 tag，都会有一个版本的包创建。tag 名称必须匹配 `X.Y.Z` 或 `vX.Y.Z`，还有一些可选的后缀名，如：`RC`，`beta`，`alpha`，`patch` 等。

这里有些 tag 名称的例子：

```
1.0.0
v1.0.0
1.10.5-RC1
v4.4.4beta2
v2.0.0-alpha
v2.0.4-p1
```

###2、Branches（分支）

对应每个分支，都会有一个开发版本的包创建。如果分支的名字看起来像个版本的命名，则版本名将会是 `{branchname}-dev`。比如，一个叫 `2.0` 的分支，会得到一个 `2.0.x-dev` 的版本（`.x` 是因为技术原因而添加的，目的是让它看起来像个分支，`2.0.x` 这种分支也是有效的，也会被转换为 `2.0.x-dev`）。

如果分支名看起来不像版本号，它的版本名会是 `dev-{branchname}`。比如，`master` 分支的版本名为 `dev-master`。

（注：如果你安装一个开发版本，Composer 将会从来源来安装它）

###3、Aliases（别名）

可以将分支名化名为版本名。比如，你可以将 `dev-master` 化名为 `1.0.x-dev`，以后你可以在其他包里依赖 `1.0.x-dev`。

##四、锁文件

如果你愿意，可以为你的代码库提交 `composer.lock` 文件。这可以保证你的团队始终在测试同一份依赖的包，不会因为依赖的包的更新带来干扰。但是，这个锁文件不会对其他依赖它的项目造成任何影响，它只影响当前这个项目。

##五、发布到 VCS（版本控制系统）

如果你的 VCS 中有一个项目含有 `composer.json` 文件，那你的库已经可以被 Composer 安装了。下面这个例子，假设我们在 GitHub 上发布了 `acme/hello-world`，位置在 `github.com/username/hello-world`。

我们在本地创建一个项目，我们叫它 `acme/blog`。这个项目将依赖 `acme/hello-world`，而后者又依赖了 `monolog/monolog`。我们可以通过下面一两步完成这些处理。

先创建个 `blog` 目录，里面创建个 `composer.json` 文件：

```
{
    "name": "acme/blog",
    "require": {
        "acme/hello-world": "dev-master"
    }
}
```

`name` 字段在这里不是必须的，因为我们没想要将它发布为一个库。

然后我们要告诉这个博客应用怎么去找 `hello-word` 这个依赖。我们可以添加一个包的仓库定义语句到 `composer.json`：

```
{
    "name": "acme/blog",
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/username/hello-world"
        }
    ],
    "require": {
        "acme/hello-world": "dev-master"
    }
}
```

OK 了。现在你可以运行 `Composer` 的 `install` 命令来安装依赖。

（再次提醒：任何 git/svn/hg 仓库，如果含有 `composer.json`，就能添加到你的项目中，只要你指定包的仓库地址，并声明依赖它）。

##六、发布到 Packagist（packagist.org 这个网站）

好了，现在你能够发布你的包了。但是每次指定 VCS 仓库会显得很笨重。而且你也不想强迫你的用户来做这事情。

有件事你可能注意到了，就是上面例子中，我们没有指定 'monolog/monolog' 的仓库。那它是如何工作的？答案就是：Packagist。

Packagist 是 Composer 的首要的包的仓库，而且这个仓库默认就是可用的，不需要额外定义。任何发布到 Packagist 的包都能被 Composer 自动使用。monolog 就是在 Packagist 上，所以我们依赖它，可以不需要添加一个额外的仓库地址。

如果你要和世界分享你的 ”hello-world“，你可以将它发布到 Packagist。很简单的。

###参考
英文原文：http://getcomposer.org/doc/02-libraries.md
