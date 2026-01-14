---
title: 自己实现一个简易的 validator
categories: [PHP]
slug: implement-a-simple-validator-yourself
date: 2019-05-03 16:10:01
toc: false
---

最近在做一些自己的小东西，因为没用 laravel，所以对感觉很多东西都没有 laravel 那么顺手，很多东西都得自己搞定才行，不过也正是因为这样，很多东西弄起来，更符合自己的需求了。

以前写表单请求的时候没感觉一个验证器会让我写代码的时候轻松一些，但是因为最近写的表单更多，我越来越发现一个验证器会让我少些多少代码，所以，我自己实现了一个简易的验证器。

```php
<?php


namespace App\Validator;

use Slim\Http\Request;

/**
 * 验证器的根类
 * Class AbstractValidator
 * @package App\Validator
 */
abstract class AbstractValidator
{
    /**
     * @var Request
     */
    protected $request;
    /**
     * @var array
     */
    protected $extraParams;

    public function __construct()
    {
    }

    /**
     * 保存条件的数组
     * @var array
     * rules 的 key 是验证的变量名称
     * rules 的 value 是个数组
     * value 可包含 type type是声明变量的类型，可包含 string | integer | float | double | array | regex 其他的需要用到的时候在添加
     *       不同的类型会依赖后续不同的参数
     * value 包含 require 这个类型是 true | false, 如果是 true 就是必填项，如果请求参数不包含这个就会返回 false，默认 false
     * value 可包含 message 出错的提示信息，如果出错没有 message 的时候就考大家在 validation 里面自己实现默认的消息了,
     *       如果包含 type 键，这个 message 就是 type 出错的 message，如果没有 type 这个就可以是 require 为 true 的错误消息
     * value 可包含 requireMessage 这个是 require 为 true 时候的出错消息，如果这个没有，就返回默认的信息 'xxx必须填写'
     */
    abstract public function rules();

    public function validation($request, $extraParams)
    {
        $this->request = $request;
        $this->extraParams = $extraParams;

        return $this->doValidation();
    }

    /**
     * 执行验证流程
     */
    public function doValidation()
    {
        $rules = $this->rules();

        // 获取请求方法
        $requestMethod = $this->request->getMethod();
        // 获取参数，如果是get就直接获取get参数，如果是其他请求方法
        // 则优先获取post参数，如果不存在在获取get参数，如果依然不存在
        // 就获取 extraParams
        foreach ($rules as $paramName => $rule) {
            $rule = (array)$rule;
            $falseReturn = [false, array_key_exists('message', $rule) ? $rule['message'] : null];

            $value = $this->request->getParsedBodyParam($paramName);

            if (empty($value) && strtoupper($requestMethod) !== 'GET') {
                $value = $this->request->getQueryParam($paramName);
            }


            if (array_key_exists($paramName, $this->extraParams)) {
                $value = $this->extraParams[$paramName];
            }

            if ($rule['require'] == true && empty($value)) {
                return [
                    false,
                    array_key_exists('requireMessage', $rule)
                        ? $rule['requireMessage']
                        : (array_key_exists('message', $rule) ? $rule['message'] : $paramName . '必须填写')
                ];
            }

            // 如果什么都不存在就返回
            if (empty($value)) {
                continue;
            }

            if (!array_key_exists('requireMessage', $rule)) {
                $rule['type'] = 'string';
            }

            switch ($rule['type']) {
                case 'string':
                    if ($rule['length']) {
                        // 字符串情况下是长度必须相等
                        $length = $rule['length'];
                        if ($length !== mb_strlen($value)) {
                            return $falseReturn;
                        }
                    } else if ($rule['min'] || $rule['max']) {
                        // 字符串情况下是最大最小长度
                        if (array_key_exists('min', $rule) && mb_strlen($value) < $rule['min']) {
                            return $falseReturn;
                        }

                        if (array_key_exists('max', $rule) && mb_strlen($value) > $rule['max']) {
                            return $falseReturn;
                        }
                    }

                    break;
                case 'integer':
                    if (!is_numeric($value) || ($value + 0) !== intval($value)) {
                        return $falseReturn;
                    }

                    $value = intval($value);

                    if ($rule['length']) {
                        // 数字的情况下比较数字是否相同
                        if ($value !== $rule['length']) {
                            return $falseReturn;
                        }
                    } else if ($rule['min'] || $rule['max']) {
                        // 这种情况下是区间
                        if (array_key_exists('min', $rule) && $value < $rule['min']) {
                            return $falseReturn;
                        }

                        if (array_key_exists('max', $rule) && $value > $rule['max']) {
                            return $falseReturn;
                        }
                    } else if ($rule['list'] && is_array($rule['list']) && !in_array($value, $rule['list'])) {
                        // list 代表一个数组，数字不在数组里面就报错
                        return $falseReturn;
                    }
                    break;
                case 'float':
                case 'double':
                    if (!is_numeric($value) || ($value + 0) !== floatval($value)) {
                        return $falseReturn;
                    }
                    $value = floatval($value);

                    if ($rule['length']) {
                        // 数字的情况下比较数字是否相同
                        if ($value !== $rule['length']) {
                            return $falseReturn;
                        }
                    } else if ($rule['min'] || $rule['max']) {
                        // 这种情况下是区间
                        if (array_key_exists('min', $rule) && $value < $rule['min']) {
                            return $falseReturn;
                        }

                        if (array_key_exists('max', $rule) && $value > $rule['max']) {
                            return $falseReturn;
                        }
                    } else if ($rule['list'] && is_array($rule['list']) && !in_array($value, $rule['list'])) {
                        // list 代表一个数组，数字不在数组里面就报错
                        return $falseReturn;
                    }
                    break;
                case 'regex':
                    // 正则
                    if (!preg_match('~' . $rule['pattern'] . '~iu', $value)) {
                        return $falseReturn;
                    }
                    break;
                case 'array':
                    // 数组
                    if (!is_array($value)) {
                        return $falseReturn;
                    }

                    if ($rule['length']) {
                        // 数组长度
                        if ($value !== $rule['length']) {
                            return $falseReturn;
                        }
                    } else if ($rule['min'] || $rule['max']) {
                        // 数组长度
                        if (array_key_exists('min', $rule) && $value < $rule['min']) {
                            return $falseReturn;
                        }

                        if (array_key_exists('max', $rule) && $value > $rule['max']) {
                            return $falseReturn;
                        }
                    }
                    break;
            }
        }

        return [true, ''];
    }
}

```

这次写代码用的是 slim 框架，所以一部分的类依赖我就使用了 slim 的相关类。弄好了这个之后写表单使用起来就简易多了，写一个类继承这个 `AbstractValidator` 类，只需实现一个方法 `rules`, 这个方法返回一个数组，一个需要验证的表单数组就ok了。

在验证的地方，调用 `validation` 方法就可以了，在返回失败的时候实现自己的返回失败逻辑就OK了。实例如下

```php
<?php


namespace App\Validator;


class TestValidator extends AbstractValidator
{

    /**
     * 保存条件的数组
     * @var array
     * rules 的 key 是验证的变量名称
     * rules 的 value 是个数组
     * value 可包含 type type是声明变量的类型，可包含 string | integer | float | double | array | regex 其他的需要用到的时候在添加
     *       不同的类型会依赖后续不同的参数
     * value 包含 require 这个类型是 true | false, 如果是 true 就是必填项，如果请求参数不包含这个就会返回 false，默认 false
     * value 可包含 message 出错的提示信息，如果出错没有 message 的时候就考大家在 validation 里面自己实现默认的消息了,
     *       如果包含 type 键，这个 message 就是 type 出错的 message，如果没有 type 这个就可以是 require 为 true 的错误消息
     * value 可包含 requireMessage 这个是 require 为 true 时候的出错消息，如果这个没有，就返回默认的信息 'xxx必须填写'
     */
    public function rules()
    {
        // TODO: Implement rules() method.
        return [
            'field1' => [
                'require' => true,
            ],
        ];
    }
}

```

上面这个类就是我们自己的 `validator`， 下面就是我们的验证逻辑部分， 在 slim 中 只要实现一个 中间件就可以了，如下
```php
<?php
/**
 * Created by PhpStorm.
 * User: haoliang
 * Date: 2019-03-13
 * Time: 15:20
 */

namespace App\Middleware;


use App\Controller\User;
use App\Utils\JWT;
use App\Validator\AbstractValidator;
use Jose\Component\Core\Converter\StandardConverter;
use Slim\Container;
use Slim\Http\Request;
use Slim\Http\Response;
use App\Model\User as UserModel;

/**
 * 检测菜单权限的中间件
 * Class CheckUrl
 * @package App\Middleware
 */
class ValidateMiddleware
{
    /**
     * @var AbstractValidator $validator ;
     */
    private $validator;
    /**
     * @var Container $container ;
     */
    private $container;

    public function __construct(Container $container, AbstractValidator $validator)
    {
        $this->container = $container;
        $this->validator = $validator;
    }


    /**
     * @param Request $request
     * @param Response $response
     * @param $next
     * @return Response
     */
    public function __invoke(Request $request, Response $response, $next)
    {
        $routeParams = $request->getAttribute('routeInfo')[2];
        $this->container->get('logger')->info(json_encode($routeParams));

        list($result, $message) = $this->validator->validation($request, $routeParams);
        $this->container->get('logger')->info($result . '   '. $message);

        if ($result) {
            $response = $next($request, $response);
        } else {
            $response = $response->withJson([
                'status' => -19999,
                'message' => $message,
                'data' => '',
            ]);
        }

        return $response;
    }
}

```

在 slim 中，就是在 `__invoke` 方法中实现验证逻辑就ok了。


这次实现的比较简单，我自己都觉得还有很大的优化余地，慢慢优化ing。