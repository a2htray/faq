# Laravel Form Class Not Found

在`Laravel`视图中编写模板代码，总是会有一些`CURD`的表单或列表，其中有就有可能使用到`Form`，`H`等帮助类，
这些原来是要安装依赖库的

[教程](https://laravelcollective.com/docs/5.4/html)

选择相应版本进行安装

```bash
composer require "laravelcollective/html":"^5.4.0"
```

在`app/config.php`中加入

```php
 'providers' => [
    // ...
    Collective\Html\HtmlServiceProvider::class,
    // ...
  ],
```

最后加入两个别名，同样在`app/config.php`中

```php
 'aliases' => [
    // ...
      'Form' => Collective\Html\FormFacade::class,
      'Html' => Collective\Html\HtmlFacade::class,
    // ...
  ],
```