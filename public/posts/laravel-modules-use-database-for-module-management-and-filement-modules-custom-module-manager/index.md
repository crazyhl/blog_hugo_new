# Laravel Modules 使用数据库进行模块管理以及 Filement Modules 自定义模块管理器


## Laravel Modules 使用数据库进行模块管理
`Laravel Modules` 默认是使用文件系统进行进行模块管理的，如果本地开发是没有问题的，但是如果放到线上，由于模块信息默认是在项目根目录的，那么就要把根目录的这个文件进行权限配置。背离了我们只放开 public/storage 目录权限的目标（这里描述的的可能不准确），所以下面只有两个选择

1. 把模块信息文件放到 public 或者 storage 下。
2. 采用数据库进行模块管理

在这里我选择使用数据库进行模块管理的操作，创建 `model` 和迁移文件 `php artisan make:model ModuleActivator -m`
编辑迁移文件

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('module_activators', function (Blueprint $table) {
            $table->id();
            $table->string('module_name')->unique();
            $table->boolean('module_status');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('module_activators');
    }
};
```

增加两个字段 `module_name` 和 `module_status` 分别表示模块名称和模块状态
执行迁移 `php artisan migrate`。
接下来修改配置文件`modules.php` 如果没有文件，记得 publish 一下就可以了

```php
'activators' => [
    'file' => [
        'class' => FileActivator::class,
        'statuses-file' => base_path('modules_statuses.json'),
    ],
    'database' => [
        'class' => DatabaseActivator::class,
        'model' => ModuleActivator::class,
    ]
],

'activator' => 'database',
```
`activators` 增加 `database` 配置，`activator` 修改为 `database`。
`database` 配置中的 `model` 就是刚才创建的 `ModuleActivator` 模型。
`database` 配置中的 `class` 是 `DatabaseActivator` 数据库管理器，我们自己创建，代码如下
```php
<?php

namespace App\Activators;

use Nwidart\Modules\Contracts\ActivatorInterface;
use Nwidart\Modules\Module;
use Illuminate\Container\Container;
use Illuminate\Database\Eloquent\Model;

class DatabaseActivator implements ActivatorInterface
{

    /**
     * 模块管理的模型
     * @var class-string<Model> $modelClass
     */
    private string $modelClass;

    public function __construct(Container $app)
    {
        $this->modelClass = $app['config']['modules.activators.database.model'];
    }

    /**
     * Enables a module
     */
    public function enable(Module $module): void
    {
        $this->setActiveByName($module->getName(), true);
    }

    /**
     * Disables a module
     */
    public function disable(Module $module): void
    {
        $this->setActiveByName($module->getName(), false);
    }

    /**
     * Determine whether the given status same with a module status.
     */
    public function hasStatus(Module|string $module, bool $status): bool
    {
        $name = $module instanceof Module ? $module->getName() : $module;
        $moduleRecord = $this->modelClass::where('module_name', $name)->first();
        if ($moduleRecord) {
            return boolval($moduleRecord['module_status']) === $status;
        } else {
            return $status === false;
        }
    }

    /**
     * Set active state for a module.
     */
    public function setActive(Module $module, bool $active): void
    {
        $this->setActiveByName($module->getName(), $active);
    }

    /**
     * Sets a module status by its name
     */
    public function setActiveByName(string $name, bool $active): void
    {
        $this->modelClass::upsert(['module_name' => $name, 'module_status' => $active], ['module_name'], ['module_status']);
    }

    /**
     * Deletes a module activation status
     */
    public function delete(Module $module): void
    {
        $moduleRecord = $this->modelClass::where('module_name', $module->getName())->first();
        if ($moduleRecord) {
            $moduleRecord->delete();
        }
    }

    /**
     * Deletes any module activation statuses created by this class.
     */
    public function reset(): void
    {
        $this->modelClass::truncate();
    }
}
```
`database` 配置中的 `model`, 就是为了可以更换为你自己的模块状态管理模型。
这个类实现了 `ActivatorInterface` 接口。

这样模块的激活状态就会存储在数据库中了。可以使用 `laravel modules` 的 `artisan` 命令来测试模块的激活状态。 

## filament-module 自动注册模块，只激活启用模块中的 Plugin

[filament-modules](https://github.com/savannabits/filament-modules) 是 `filament` 中的一个插件，可以搭配 `laravel modules` 来进行开发，每一个模块都是一个 `filament` 插件。方便我们模块化的组织系统。

但是默认的配置`filament-modules.auto-register-plugins` 存在一个问题，如果打开这个配置，那么所有的模块中的 `Plugin` 都会被注册到 `filament` 中，即使这个模块没有被激活。
这显然不是我们想要的，我们只希望激活的模块中的 `Plugin` 被注册到 `filament` 中。
但是如果把这个配置设置为 `false` 又得自己手动注册管理，会更麻烦。
所以我们需要一个自定义的模块管理器，来管理模块的激活状态，并且根据模块的激活状态来注册 `Plugin`。

```php
<?php

namespace App\Plugin;

use Coolsam\Modules\Facades\FilamentModules;
use Coolsam\Modules\ModulesPlugin as BaseModulesPlugin;
use Nwidart\Modules\Facades\Module;

class ModulesPlugin extends BaseModulesPlugin
{
    protected function getModulePlugins(): array
    {
        if (! config('filament-modules.auto-register-plugins', false)) {
            return [];
        }
        $enabledModules = Module::allEnabled();
        $pluginPaths = [];
        foreach ($enabledModules as $module) {
            $path = $module->getPath() . DIRECTORY_SEPARATOR . 'app' . DIRECTORY_SEPARATOR . 'Filament' . DIRECTORY_SEPARATOR . '*Plugin.php';
            $modulePluginPaths = glob($path);
            if(is_array($modulePluginPaths)) {
                $pluginPaths = array_merge($pluginPaths, $modulePluginPaths);
            }
        }

        return collect($pluginPaths)
            ->map(fn ($path) => FilamentModules::convertPathToNamespace($path))
            ->filter()->toArray();

        // return collect($enabledModules)->map(function ($module) {
        //     $path = $module->getPath() . DIRECTORY_SEPARATOR . 'app' . DIRECTORY_SEPARATOR . 'Filament' . DIRECTORY_SEPARATOR . $module->getName() . 'Plugin.php';
        //     if(file_exists($path)) {
        //         return FilamentModules::convertPathToNamespace($path);
        //     }
        //     return null;
        // })->filter()->toArray();
    }
}
```

注意我注释的部分，一般我个人都会使用我注释部分的代码。因为我的规划是每个模块都只有一个插件，且插件名跟模块名称统一（这样略显死板）。这样注释部分的速度会更快。未注释的部分，单纯是为了防止模块有多个插件的情况。喜欢用哪个就自己开放关闭就可以了。
这个类继承了 `use Coolsam\Modules\ModulesPlugin` ，并重写了 `getPlugins` 方法。这个方法用于获取所有激活模块的插件类。

这样修改`PanelProvider` 中的 plugin 方法 `->plugin(ModulesPlugin::make())`, 参数设定为上面那个自定义类就可以了，这样就可以自动注册已激活模块的插件了。

---

> 作者:   
> URL: http://localhost:1313/posts/laravel-modules-use-database-for-module-management-and-filement-modules-custom-module-manager/  

