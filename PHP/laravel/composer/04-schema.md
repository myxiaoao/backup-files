#Composer 结构

##一、Root Package（根目录包）

根目录包就是在你的项目的根目录由 composer.json 定义的包。主要就是由 composer.json 来定义你的项目的依赖。

某些字段只能在根目录包的中使用，比如 config 字段，只有根目录包能定义自己的配置。依赖包中的 config 字段是被忽略的。所以 config 字段是 root-only 的。

如果你克隆了其中一个依赖包并在上面工作，那么这个包就是根目录包。composer.json 还是一样的，但上下文不同。

（注：一个包是不是根目录包，取决于上下文。）

##二、composer.json 中的各个属性（字段）

###1、name

包的名字。由供应方（vendor）名和项目名组成，用 / 分隔。

在发布包的时候需要填。

###2、description

对包的一个简短描述，通常是一行的长度。

在发布包的时候需要填。

###3、version

包的版本。

格式必须是 `X.Y.Z`，选择性后缀：`-dev`、`-alphaN`、`-betaN`、`-RCN`。

###4、type

包的类型，默认为 library。

包类型用于定制安装逻辑。如果你的包的安装需要一些特殊的逻辑，你可以定义一个定制的类型。它可以是一个 symfony-bundle 的类型，或者 wordpress-plugin，或者 typo3-module。这些类型将被特定的项目所用，它们将提供安装器来安装这些类型的包。

####Composer 支持 3 种类型：

library：默认值。它将复制文件到 vendor 目录。

project：它表示这是个项目，而不是库。比如像 Symfony 标准版这种应用。

metapackage：一个含有依赖的空包，能触发安装，但不包含文件，不会向文件系统写任何东西。

composer-install：为其他的定制类型的包提供安装器的包。

###5、keywords

一个与包相关的关键词数组。用于包的搜索和过滤。

可选。

###6、homepage

项目的网站 URL。

可选。

###7、time

版本发布时间。必须是 YYYY-MM-DD 或 YYYY-MM-DD HH:MM:SS 格式。

可选。

###8、license

包的许可证。可以是字符串或字符串数组。

可选，但强烈建议加上。

###9、authors

包的作者。是个对象数组。

每个 author 对象有这些属性：

>name：作者名字  
>email：作者邮箱  
>homepage：作者网站 URL  
>role：作者在项目中的角色（如：developer 或 translator）

###10、support

各种关于该项目如何获取支持的信息。包含这些属性：

>email：获取支持的邮箱  
>issues：问题跟踪的 URL   
>forum：论坛的 URL  
>wiki：Wiki 的 URL  
>irc：IRC 的频道  
>source：查看或下载源码的 URL  

可选。

###11、Package links

依赖包的映射表，由包名映射版本约束。如：

```
{
    "require": {
        "monolog/monolog": "1.0.*"
    }
}
```

（1）require

列出包所依赖的包。除非这些依赖已经存在，否则这个包不会被安装。

（2）require-dev（root-only）

列出开发这个包（或跑测试等等）所依赖的包。在使用 install 命令时，只有带上 “–dev” 参数才能安装 dev 包。在使用 update 命令时，带上 “–no-dev” 则不更新。

（3）conflict

列出包会和哪些包发生冲突。它们将不被允许和你的包一起安装。如果约束了版本，则只会针对特定的版本。

（4）replace

列出哪些包要被这个包替代。

（5）provide

这个包所推荐的包列表。这个对公共接口最有用，一个包可以依赖一个虚拟的 logger 包，而实现 logger 接口的库可以放到 provide 字段中。

###12、suggest

建议一些能让这个包工作的更好或得到增强的包列表。这些信息只在包安装完成时给出，暗示用户可以添加更多包，虽然不是必须要安装的。

格式是，包名映射文字说明，如：

```
{
    "suggest": {
        "monolog/monolog": "Allows more advanced logging of the application flow"
    }
}
```

###13、autoload

提供给 PHP autoloader 的自动加载映射。

目前支持的有：PSR-0 自动加载规范，classmap 生成器，还有 files。

PSR-0 是比较推荐的，因为它的优秀的扩展性（在添加新的类的适合，不需要重新生成自动加载器）。

（1）PSR-0

在 psr-0 键名下，定义一个命名空间到路径的映射表，相对于包的根目录。注意，这也同样支持 PEAR-style 的没有命名空间的风格。

请注意命名空间的声明得以 \\ 结尾，确保自动加载器正确响应。

PSR-0 的引用可以在安装或更新时生成的文件中查看：
vendor/composer/autoload_namespaces.php

例子：

```
{
    "autoload": {
        "psr-0": {
            "Monolog\\": "src/",
            "Vendor\\Namespace\\": "src/",
            "Vendor_Namespace_": "src/"
        }
    }
}
```

如果你需要在多个目录里查找同一个前缀的命名空间，你可以用数组，如：

```
{
    "autoload": {
        "psr-0": { "Monolog\\": ["src/", "lib/"] }
    }
}
```

PSR-0 风格并不局限于加载命名空间的声明的东西，也可以用于类这个层级。当库中只有一个在全局命名空间中的类时，这种方式就能用上。比如你有个 PHP 源文件放在项目的根目录，你可以这样声明：

```
{
    "autoload": {
        "psr-0": { "UniqueGlobalClass": "" }
    }
}
```

如果你有个目录下全是用命名空间组织的，你可以用空前缀：

```
{
    "autoload": {
        "psr-0": { "": "src/" }
    }
}
```

（2）Classmap

classmap 的引用可以在安装或更新时生成的文件中查看：  
vendor/composer/autoload_classmap.php

类映射表是通过扫描指定的目录或文件下的所有的 .php 和 .inc 文件生成的。

你可以给任何不支持 PSR-0 的库用 classmap 生成器实现自动加载。配置上只要指定类所在的目录或文件即可：

```
{
    "autoload": {
        "classmap": ["src/", "lib/", "Something.php"]
    }
}
```

（3）files

如果你确定需要在任何请求中都加载某些文件，你可以使用 files 自动加载机制。对于那些包中有些 PHP 函数但不能自动加载时特别有用。例如：

```
{
    "autoload": {
        "files": ["src/MyLibrary/functions.php"]
    }
}
```

###14、include-path

（将被弃用，它的功能由 autoload 代替。其实就是设置 include_path，可选）

###15、target-dir

指定安装目标路径。

如果包的根目录是在命名空间下，自动加载就不正确了，所以才有 target-dir 来解决这个问题。

Symfony 就是个例子。它由很多组件包组成。Yaml 组件是在 `Symfony\Component\Yaml`  命名空间下的，它的根目录是 Yaml 目录。要让自动加载正常工作，我们要确保它不是安装在 `vendor/symfony/yaml` ，而是在 `vendor/symfony/yaml/Symfony/Component/Yaml` ，这样自动加载器才能从 `vendor/symfony/yaml` 加载它。

所以要定义 target-dir 如下：

```
{
    "autoload": {
        "psr-0": { "Symfony\\Component\\Yaml\\": "" }
    },
    "target-dir": "Symfony/Component/Yaml"
}
```

###16、minimum-stability（root-only）

定义根据稳定性如何过滤包。默认是 stable，如果你信赖一个 dev 包，你需要指明。

###17、prefer-stable（root-only）

如果开启，Composer 会在稳定包和不稳定包中选择前者。

###18、repositories（root-only）

定制包的仓库地址。

默认的，Composer 只使用 Packagist 仓库。通过指定仓库地址，你可以从任何地方获取包。

仓库不能递归。你只能将它们添加到主的 composer.json 中。所依赖包中 composer.json 文件中的仓库定义是被忽略的。

支持的仓库的类型有：

（1）composer

composer 仓库通过网络提供 packages.json 文件，它包含一个 composer.json 对象的列表，还有额外的 dist 或 source 信息。packages.json 文件通过 PHP 流加载。

（2）vcs

版本控制系统仓库，如：git、svn、hg。

（3）pear

通过它，你可以导入任何 pear 仓库到你的项目中。

（4）package

如果你依赖一个不支持 composer 的项目，你可以定义一个 package 类型的仓库，然后将 composer.json 对象直接写入。

完整的例子：


```
{
    "repositories": [
        {
            "type": "composer",
            "url": "http://packages.example.com"
        },
        {
            "type": "composer",
            "url": "https://packages.example.com",
            "options": {
                "ssl": {
                    "verify_peer": "true"
                }
            }
        },
        {
            "type": "vcs",
            "url": "https://github.com/Seldaek/monolog"
        },
        {
            "type": "pear",
            "url": "http://pear2.php.net"
        },
        {
            "type": "package",
            "package": {
                "name": "smarty/smarty",
                "version": "3.1.7",
                "dist": {
                    "url": "http://www.smarty.net/files/Smarty-3.1.7.zip",
                    "type": "zip"
                },
                "source": {
                    "url": "http://smarty-php.googlecode.com/svn/",
                    "type": "svn",
                    "reference": "tags/Smarty_3_1_7/distribution/"
                }
            }
        }
    ]
}
```

###19、config（root-only）

针对项目的一些配置。

* process-timeout：默认 300 秒，Composer 进程执行超时时间；
* use-include-path：默认 false，如果是 true，Composer 自动加载器也会到 PHP 的 include_path 中查找；
* preferred-install：默认 auto，设置 Composer 安装方式；
* github-protocols：默认 ["git", "https"]，设置与 github 通信协议；
* github-oauth：设置 oauth；
* vendor-dir：默认 vendor，你可以换成别的；
* bin-dir：默认 vendor/bin，如果项目有二进制文件，会链接到这；
* cache-dir：默认 $home/cache，存放 Composer 运行时产生的缓存；
* cache-files-dir：默认 $cache-dir/files，存放包的 zip 文件；
* cache-repo-dir：默认 $cache-dir/repo，存放仓库元数据；
* cache-vcs-dir：默认 $cache-dir/vcs，存放 vcs 克隆；
* cache-files-ttl：默认六个月，缓存的过期时间；
* cache-files-maxsize：默认 300M；
* notify-no-install：默认 true，从仓库安装包会有个通知，可以关掉；
* discard-changes：默认false，如何处理脏的更新；

###20、scripts（root-only）

Composer 允许你在安装进程中安装钩子脚本，钩子是基于事件的；

###21、extra

供 scripts 消费的额外数据；

###22、bin

指定哪些文件必须被当做二进制文件处理的；

###23、archive

设置创建包时的选项，exclude 属性可以设置排除哪些目录，例如：

```
{
    "archive": {
        "exclude": ["/foo/bar", "baz", "/*.test", "!/foo/bar/baz"]
    }
}
```

###参考
英文原文：http://getcomposer.org/doc/04-schema.md
 

