---
title: slim framework 定时任务
categories: [PHP]
tags: [slim, symfony, 定时任务]
slug: slim-framework
date: 2019-02-12 07:38:00
updated: 2019-02-12 07:38:00
toc: false
---

最近想要弄点东西，不想用 laravel 这种框架了，因为想要更细致的了解一些底层框架相关的东西，所以采用了 slim 框架，首先这个框架比较轻便，看源码也比较快捷，当然现在更细以后有很多东西也都不熟悉了，正好可以再次看看。

不过 slim 这个框架因为比较轻便，所以周边也都比较少，我有个需求是定时任务，这个时候 slim 就不支持了，所以得需要自己搞定。

做这个的时候想到了以前看 symfony 的时候有个 console 模块，正好可以用到，google 了一下发现也有人是这么实现的，所以我也就参考着弄了下来。

<!--more-->

先从 composer 引入 console
```php
composer require symfony/console
```
接下来我们创建了一个 application.php 的文件，这个文件主要是初始化 symfony/console 的任务，代码类似下面的
```php
require __DIR__ . '/../vendor/autoload.php';
require __DIR__ . '/bootstrap.php'; // 这个是初始化 slim

use Symfony\Component\Console\Application;

$application = new Application();
// TODO 这块等待注册各种 Command
$application->run();
```

上面的代码可以看到有个 bootstrap.php 的引入，这块使我们初始化 slim 用的，代码类似下面的
```php
if (PHP_SAPI == 'cli') {
    // Instantiate the app
    $settings = require __DIR__ . '/../src/settings.php';

    //I try adding here path_info but this is wrong, I'm sure
//    $settings['environment'] = $env;

    $app = new \Slim\App($settings);

    $container = $app->getContainer();

    // Set up dependencies
    require __DIR__ . '/../src/dependencies.php';
}
```
上面的初始化我们写在了 PHP_SAPI 是 cli 的情况下，因为在其他情况下我们不想让这个脚本执行，并且我们也可以看到这里面我们仅仅是配置了 container 相关的东西，因为我们不需要让 slim 的 app 运行起来，只需要配置好 container 就可以了。

到这，我们所有的初始化工作就全部做完了，下面我可以搞一个 command 的实例出来，看看前面配置的东西是否 ok ，可以运行起来，具体可以参看 console 的文档 [https://symfony.com/doc/current/components/console.html](https://symfony.com/doc/current/components/console.html) 以及 [https://symfony.com/doc/current/console.html](https://symfony.com/doc/current/console.html) 这两个里面有详尽的介绍，代码如下

新建命令文件 TestCommand.php
```php
<?php
/**
 * Created by PhpStorm.
 * User: haoliang
 * Date: 2019-02-12
 * Time: 15:12
 */

namespace App\Command;


use Psr\Container\ContainerInterface;
use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

class TestCommand extends Command
{
    // the name of the command (the part after "bin/console")
    protected static $defaultName = 'test:slim-command';
    private $container;

    public function __construct(ContainerInterface $container, $name = null)
    {
        parent::__construct($name);
        $this->container = $container;
    }

    protected function configure()
    {
        $this->setDescription('测试 slim 和 symfony console 结合情况');
    }

    protected function execute(InputInterface $input, OutputInterface $output)
    {
        // ...
        $output->writeln([
            'symfony console 的输出跑起来了',
        ]);

        $logger = $this->container->get('logger');
        $logger->info("slim 的日志也跑起来了");
    }
}
```

上面就是我们的命令我文件，需要注意的是，我们重写了 `__contruct` 方法，这里面我们把 slim 的 container 注入进来了，方便我们后续使用。

这个文件写完了之后我们可以在 application.php 里面注册命令 
```php
$application->add(new TestCommand($app->getContainer()));
```

然后我们就可以在控制台执行命令了，
```php
php console/application.php test:slim-command
```
![图片alt](https://www.cimple.ink/images/2019/02/12/10806644011804ad9a6e148fc4d81150.png)


再看，我们用 slim container 里面设置的 log 是不是生效了，
![图片alt](https://www.cimple.ink/images/2019/02/12/cf90c886050387daf3adbf878a8af453.png)

可以看到 slim 里面设置的 log 也都 ok 了，至此，所有的东西都可以搞定了，接下来我们就可以在 crontab 里面使用这些命令了。