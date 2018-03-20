# Symfony2 Request Boot 源码学习

学习 Symfony2 一个请求的执行流程。

<!-- more -->

这里使用的是`web/app_dev.php`文件。

```php
/**
 * 因为是开发环境，故在入口脚本中加入了对请求源地址的校验
 */
if (isset($_SERVER['HTTP_CLIENT_IP'])
    || isset($_SERVER['HTTP_X_FORWARDED_FOR'])
    || !(in_array(@$_SERVER['REMOTE_ADDR'], array('127.0.0.1', '::1'), true) || PHP_SAPI === 'cli-server')
) {
    header('HTTP/1.0 403 Forbidden');
    exit('You are not allowed to access this file. Check '.basename(__FILE__).' for more information.');
}

require __DIR__.'/../app/autoload.php';
Debug::enable();

$kernel = new AppKernel('dev', true);
$kernel->loadClassCache();

/**
 * 利用预定义变量，如 $_SERVER、$_POST、$_GET、$_COOKIE、$_FILES
 * 使用 Request 的静态方法创建一个 Request 对象
 */
$request = Request::createFromGlobals();


$response = $kernel->handle($request);
$response->send();
$kernel->terminate($request, $response);
```

下面看下 Request 类中的 `public static function createFromGlobals()`方法：

```php
public static function createFromGlobals()
    {
        // With the php's bug #66606, the php's built-in web server
        // stores the Content-Type and Content-Length header values in
        // HTTP_CONTENT_TYPE and HTTP_CONTENT_LENGTH fields.
        $server = $_SERVER;

        /**
         * 将预定义变量 $_SERVER 中的已有两个值换成新的键值(去除 HTTP 前缀) 
         */
        if ('cli-server' === PHP_SAPI) {
            if (array_key_exists('HTTP_CONTENT_LENGTH', $_SERVER)) {
                $server['CONTENT_LENGTH'] = $_SERVER['HTTP_CONTENT_LENGTH'];
            }
            if (array_key_exists('HTTP_CONTENT_TYPE', $_SERVER)) {
                $server['CONTENT_TYPE'] = $_SERVER['HTTP_CONTENT_TYPE'];
            }
        }

        // 利用私有的静态方法 createRequestFromFactory 生成 Request 对象
        // 参数则将预定义的变量传入
        // 看下面对 createRequestFromFactory 方法的注释
        $request = self::createRequestFromFactory($_GET, $_POST, array(), $_COOKIE, $_FILES, $server);

        if (0 === strpos($request->headers->get('CONTENT_TYPE'), 'application/x-www-form-urlencoded')
            && in_array(strtoupper($request->server->get('REQUEST_METHOD', 'GET')), array('PUT', 'DELETE', 'PATCH'))
        ) {
            parse_str($request->getContent(), $data);
            $request->request = new ParameterBag($data);
        }

        return $request;
    }
```

```php
private static function createRequestFromFactory(array $query = array(), array $request = array(), array $attributes = array(), array $cookies = array(), array $files = array(), array $server = array(), $content = null)
    {
        /**
         * 如果你自定义了如何生成 Request 对象的方法，则通过定义的方法返回
         * 启动阶段没有定义该方法，故直接使用 new Request 一个对象
         * 在 Request 的构造方法中，又直接调用 $this->initialize 方法
         */
        if (self::$requestFactory) {
            $request = call_user_func(self::$requestFactory, $query, $request, $attributes, $cookies, $files, $server, $content);

            if (!$request instanceof self) {
                throw new \LogicException('The Request factory must return an instance of Symfony\Component\HttpFoundation\Request.');
            }

            return $request;
        }

        return new static($query, $request, $attributes, $cookies, $files, $server, $content);
    }
```

直接来看`$this->initialize`方法

```php
public function initialize(array $query = array(), array $request = array(), array $attributes = array(), array $cookies = array(), array $files = array(), array $server = array(), $content = null)
{
    /**
     * 在 initialize 方法中，将以数组类型存储的信息，分别转换成以对象的形式
     * $this->request 代表了 $_POST
     * $this->query 代表了 $_GET
     * $this->attributes 启动阶段为 null
     * $this->cookies 代表了 $_COOKIE
     * $this->files 代表了 $_FILES
     * $this->server 代表了 $_SERVER
     * $this->headers 的值则是通过对 $_SERVER 的解析而来
     *
     * 在继承关系上，FileBag、ServerBag、HeaderBag 都是继承了 ParameterBag
     */
    $this->request = new ParameterBag($request);
    $this->query = new ParameterBag($query);
    $this->attributes = new ParameterBag($attributes);
    $this->cookies = new ParameterBag($cookies);
    $this->files = new FileBag($files);
    $this->server = new ServerBag($server);
    $this->headers = new HeaderBag($this->server->getHeaders());

    $this->content = $content;
    $this->languages = null;
    $this->charsets = null;
    $this->encodings = null;
    $this->acceptableContentTypes = null;
    $this->pathInfo = null;
    $this->requestUri = null;
    $this->baseUrl = null;
    $this->basePath = null;
    $this->method = null;
    $this->format = null;
}
```

生成好 Request 对象后，在`createFromGlobals`方法中下面一段代码

```php
/**
 * 为了区别如果不是普通的提交数据，则会以另一种方式解析所提交的数据
 */
if (0 === strpos($request->headers->get('CONTENT_TYPE'), 'application/x-www-form-urlencoded')
    && in_array(strtoupper($request->server->get('REQUEST_METHOD', 'GET')), array('PUT', 'DELETE', 'PATCH'))
) {
    parse_str($request->getContent(), $data);
    $request->request = new ParameterBag($data);
}
```

经过特殊处理生成后的 Request 对象，被传递给`new AppKernel()->handle($request)`进行处理

```php
public function handle(Request $request, $type = HttpKernelInterface::MASTER_REQUEST, $catch = true)
{
    if (false === $this->booted) {
        /**
         * 通过 $this->boot 方法，会去初始化组件，初始化服务容器，为每个组件设置服务容器
         */
        $this->boot();
    }

    /**
     * 最终使用`HttpKernel->handle`方法来处理当前请求
     */
    return $this->getHttpKernel()->handle($request, $type, $catch);
}
```

以请求成功为例，在`handle`方法中又去调用了`handleRaw`，其中的参数`$type`为`HttpKernelInterface::MASTER_REQUEST = 1`

```php
private function handleRaw(Request $request, $type = self::MASTER_REQUEST)
{
    $this->requestStack->push($request);

    // 声明一个 GetResponseEvent 事件
    $event = new GetResponseEvent($this, $request, $type);
    // 让所有 Listener 触发该事件
    $this->dispatcher->dispatch(KernelEvents::REQUEST, $event);

    if ($event->hasResponse()) {
        return $this->filterResponse($event->getResponse(), $request, $type);
    }

    // load controller
    if (false === $controller = $this->resolver->getController($request)) {
        throw new NotFoundHttpException(sprintf('Unable to find the controller for path "%s". The route is wrongly configured.', $request->getPathInfo()));
    }

    // 对于没有响应结果的，则表示需要执行控制器的代码
    $event = new FilterControllerEvent($this, $controller, $request, $type);
    $this->dispatcher->dispatch(KernelEvents::CONTROLLER, $event);
    $controller = $event->getController();

    // 利用反射类，验证请求所给参数与控制器方法中参数是否符合，并返回方法所需参数
    $arguments = $this->resolver->getArguments($request, $controller);

    // 调用一个控制器方法
    $response = \call_user_func_array($controller, $arguments);

    // 当返回的视图类型的返回时
    if (!$response instanceof Response) {
        $event = new GetResponseForControllerResultEvent($this, $request, $type, $response);
        $this->dispatcher->dispatch(KernelEvents::VIEW, $event);

        if ($event->hasResponse()) {
            $response = $event->getResponse();
        }

        if (!$response instanceof Response) {
            $msg = sprintf('The controller must return a response (%s given).', $this->varToString($response));

            // the user may have forgotten to return something
            if (null === $response) {
                $msg .= ' Did you forget to add a return statement somewhere in your controller?';
            }
            throw new \LogicException($msg);
        }
    }

    return $this->filterResponse($response, $request, $type);
}
```

Request 对象处理完成后，返回一个 Response 对象，调用该对象的 send 方法实现将结果返回给浏览器。最后，结束后触发各类相关终止事件。

## 参考

* Event Listener Tutorial [http://www.chrisbrand.co.za/2013/06/22/design-pattern-event-dispatcher/](http://www.chrisbrand.co.za/2013/06/22/design-pattern-event-dispatcher/)