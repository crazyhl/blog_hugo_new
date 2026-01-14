# Slim Framework中间件的分析


最近也在分析中间件的东西。翻看了一下 laravel 的实现。但是 laravel 的实现很复杂，其实也不是很复杂，就是它的结构么，跳来跳去的，所以最后还是看了slim，毕竟简单。容易分析。

通过源码我可能得知有一个 MiddlewareAwareTrait 这个 Trait，然后由 App 来 use 这个 trait。

这里面有3个方法，分别是 addMiddleware、seedMiddlewareStack 和 callMiddlewareStack。

addMiddleware 这个方法是注册中间件用的，进入后会检测当前是不是在运行中间件。如果在运行中，则不允许添加中间件。紧接着会检测当前的中间件的栈是不是空的，如果是空的，则创建一个栈。这是后就调用了 seedMiddlewareStack 这这个方法。这个方法很简单，就是初始化一个栈。并且吧使用这个 trait 的 App 放入到栈底，并且设置了这个 stack 的模式，是栈模式和便利模式。具体的可以看 splstack 的文档。好了继续回到 add 的方法，从栈的顶部取回回调函数然后跟当前的相关参数组装成一个新的闭包函数，放到栈顶。其实现在除站定意外的元素可以说都是没有什么用了，但是为什么还放到下面，需要在思考思考。然后返回this，这是为了方便链式调用。

这个添加的方法就完毕了，具体可以看源码
```php
 /**
     * Add middleware
     *
     * This method prepends new middleware to the application middleware stack.
     *
     * @param callable $callable Any callable that accepts three arguments:
     *                           1. A Request object
     *                           2. A Response object
     *                           3. A "next" middleware callable
     * @return static
     *
     * @throws RuntimeException         If middleware is added while the stack is dequeuing
     * @throws UnexpectedValueException If the middleware doesn't return a Psr\Http\Message\ResponseInterface
     */
    protected function addMiddleware(callable $callable)
    {
        if ($this->middlewareLock) {
            throw new RuntimeException('Middleware can’t be added once the stack is dequeuing');
        }

        if (is_null($this->stack)) {
            $this->seedMiddlewareStack();
        }
        $next = $this->stack->top();
        $this->stack[] = function (
            ServerRequestInterface $request,
            ResponseInterface $response
        ) use (
            $callable,
            $next
        ) {
            $result = call_user_func($callable, $request, $response, $next);
            if ($result instanceof ResponseInterface === false) {
                throw new UnexpectedValueException(
                    'Middleware must return instance of \Psr\Http\Message\ResponseInterface'
                );
            }

            return $result;
        };

        return $this;
    }

    /**
     * Seed middleware stack with first callable
     *
     * @param callable $kernel The last item to run as middleware
     *
     * @throws RuntimeException if the stack is seeded more than once
     */
    protected function seedMiddlewareStack(callable $kernel = null)
    {
        if (!is_null($this->stack)) {
            throw new RuntimeException('MiddlewareStack can only be seeded once.');
        }
        if ($kernel === null) {
            $kernel = $this;
        }
        $this->stack = new SplStack;
        $this->stack->setIteratorMode(SplDoublyLinkedList::IT_MODE_LIFO | SplDoublyLinkedList::IT_MODE_KEEP);
        $this->stack[] = $kernel;
    }

```

调用方法就没必要多了，就是一层层调用而已了。直接上源码

```php
/**
     * Call middleware stack
     *
     * @param  ServerRequestInterface $request A request object
     * @param  ResponseInterface      $response A response object
     *
     * @return ResponseInterface
     */
    public function callMiddlewareStack(ServerRequestInterface $request, ResponseInterface $response)
    {
        if (is_null($this->stack)) {
            $this->seedMiddlewareStack();
        }
        /** @var callable $start */
        $start = $this->stack->top();
        $this->middlewareLock = true;
        $response = $start($request, $response);
        $this->middlewareLock = false;
        return $response;
    }
```


---

> 作者:   
> URL: http://localhost:1313/posts/analysis-of-slim-framework-middleware/  

