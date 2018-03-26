# PHP Dotenv 源码注释

`Dotenv`包就两个重要的类，一个`Dotenv类`，对外提供接口，一个`Loader类`，用于解析与设置环境变量实现。

```php
// Dotenv.php
<?php

namespace Dotenv;

/**
 * Dotenv 类对外提供统一的接口，隐藏 Loader 类的具体实现
 */
class Dotenv
{
    protected $filePath; // 保存配置文件的路径 
    protected $loader;

     // 创建一个 Dotenv 实例，
     // $path . DIRECTORY_SEPARATOR . $file 为配置文件路径，最终赋值给 $filePath 属性
    public function __construct($path, $file = '.env')
    {
        $this->filePath = $this->getFilePath($path, $file); // 处理得到配置文件路径
        $this->loader = new Loader($this->filePath, true); // 具体的配置文件解析操作其实是由 Loader 类所提供
    }

    // load 和 overload 方法，是为了隐藏 Loader 类的接口
    // 向外界提供统一的操作方法，区别在于：在重新实例化 Loader 类时的第二个参数
    public function load()
    {
        return $this->loadData();
    }

    public function overload()
    {
        return $this->loadData(true);
    }

     // 用于得到配置文件路径
    protected function getFilePath($path, $file)
    {
        if (!is_string($file)) {
            $file = '.env';
        }
        // 使用 rtrim 方法将字符串右边的目录分隔符号去除，没有则不做改变
        // 使用 DIRECTORY_SEPARATOR ，统一处理 Unix 和 Windows 环境
        $filePath = rtrim($path, DIRECTORY_SEPARATOR).DIRECTORY_SEPARATOR.$file;

        return $filePath;
    }

    protected function loadData($overload = false)
    {
        $this->loader = new Loader($this->filePath, !$overload);

        return $this->loader->load();
    }

    public function required($variable)
    {
        return new Validator((array) $variable, $this->loader);
    }
}
```

```php
// Loader.php
<?php

namespace Dotenv;

use Dotenv\Exception\InvalidFileException;
use Dotenv\Exception\InvalidPathException;

class Loader
{
    protected $filePath;

    protected $immutable;

    public function __construct($filePath, $immutable = false)
    {
        $this->filePath = $filePath;
        $this->immutable = $immutable;
    }

    public function load()
    {
        $this->ensureFileIsReadable(); // 确定配置文件是否有效，无效则抛异常

        $filePath = $this->filePath;
        $lines = $this->readLinesFromFile($filePath); // 读入配置文件中每一行信息
        foreach ($lines as $line) { // 遍历进行解析
            // 判断是否为注释
            if (!$this->isComment($line) && $this->looksLikeSetter($line)) {
                // 在调用时没有传入第二个参数
                // $name = $line, $value = null
                // 原因在于，setEnvironmentVariable 对该库对外提供的方法，用户可以在程序中自行设置环境变量
                $this->setEnvironmentVariable($line); // 如果有效，则将其设置为环境变量，使用 getenv 方法就可以读取
            }
        }

        return $lines;
    }

    protected function ensureFileIsReadable()
    {
        // 这里只判断了两个条件，是否可读、是否是文件
        if (!is_readable($this->filePath) || !is_file($this->filePath)) {
            throw new InvalidPathException(sprintf('Unable to read the environment file at %s.', $this->filePath));
        }
    }

    protected function normaliseEnvironmentVariable($name, $value)
    {
        list($name, $value) = $this->splitCompoundStringIntoParts($name, $value);
        list($name, $value) = $this->sanitiseVariableName($name, $value);
        list($name, $value) = $this->sanitiseVariableValue($name, $value);

        // 如果 value 值包含 $ 符号，将进行变量解析
        $value = $this->resolveNestedVariables($value); // 这里主要是考虑到，如果在配置中取环境变量的情况

        return array($name, $value);
    }

    public function processFilters($name, $value)
    {
        // 通过 $this->splitCompoundStringIntoParts 返回的信息才真正符合 name = value 的配对
        list($name, $value) = $this->splitCompoundStringIntoParts($name, $value);
        // 处理 name，去除 export \ " 这三个字符
        list($name, $value) = $this->sanitiseVariableName($name, $value);
        // 处理 value，看 sanitiseVariableValue 的注释
        list($name, $value) = $this->sanitiseVariableValue($name, $value);

        return array($name, $value);
    }

    protected function readLinesFromFile($filePath)
    {
        // auto_detect_line_endings：ini 配置信息，是否读取行尾换行符号，默认为0,
        // 读取 -> 修改 -> 操作 -> 复位
        $autodetect = ini_get('auto_detect_line_endings');
        ini_set('auto_detect_line_endings', '1');
        // 将新行与空行忽略，不进行读取
        $lines = file($filePath, FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES);
        ini_set('auto_detect_line_endings', $autodetect);

        return $lines;
    }

    // 判断该行是否为注释
    protected function isComment($line)
    { // 将左边空白符去除后，判断第一个字符是否为 #
        return strpos(ltrim($line), '#') === 0;
    }

    protected function looksLikeSetter($line)
    {   // 这里判断很简单，只是判断该行是否包含 =
        return strpos($line, '=') !== false;
    }

    protected function splitCompoundStringIntoParts($name, $value)
    {
        if (strpos($name, '=') !== false) { // 这里很关键，主要区别在于是否是通过写配置文件方式进行配置的
            // 如果是配置文件中进行配置，则将 name 拆成 新的 name 和 value
            // 拆的方法不错，使用 array_map
            list($name, $value) = array_map('trim', explode('=', $name, 2));
        }

        return array($name, $value);
    }

    
    protected function sanitiseVariableValue($name, $value)
    {
        $value = trim($value); // 两边抹去空白
        if (!$value) { // 如果什么都没写，表示变量值就是null
            return array($name, $value);
        }

        if ($this->beginsWithAQuote($value)) { // 如果 value 包含一个单引号
            $quote = $value[0];
            $regexPattern = sprintf(
                '/^
                %1$s          # match a quote at the start of the value
                (             # capturing sub-pattern used
                 (?:          # we do not need to capture this
                  [^%1$s\\\\] # any character other than a quote or backslash
                  |\\\\\\\\   # or two backslashes together
                  |\\\\%1$s   # or an escaped quote e.g \"
                 )*           # as many characters that match the previous rules
                )             # end of the capturing sub-pattern
                %1$s          # and the closing quote
                .*$           # and discard any string after the closing quote
                /mx',
                $quote
            );
            $value = preg_replace($regexPattern, '$1', $value);
            $value = str_replace("\\$quote", $quote, $value);
            $value = str_replace('\\\\', '\\', $value); // 需要使用转义的方式写配置文件
        } else {
            $parts = explode(' #', $value, 2); // 担心配置行后也会有注释，使用 explode 进行分割
            $value = trim($parts[0]);

            if (preg_match('/\s+/', $value) > 0) { // 不能包含空白符
                throw new InvalidFileException('Dotenv values containing spaces must be surrounded by quotes.');
            }
        }

        return array($name, trim($value));
    }

    protected function resolveNestedVariables($value)
    {
        if (strpos($value, '$') !== false) {
            $loader = $this;
            // preg_replace_callback，先进行匹配，匹配通过，返回处理后的字符串
            $value = preg_replace_callback(
                '/\${([a-zA-Z0-9_]+)}/',
                function ($matchedPatterns) use ($loader) {
                    // 环境信息中读取变量值
                    $nestedVariable = $loader->getEnvironmentVariable($matchedPatterns[1]);
                    if ($nestedVariable === null) {
                        return $matchedPatterns[0];
                    } else {
                        return $nestedVariable;
                    }
                },
                $value
            );
        }

        return $value;
    }

    protected function sanitiseVariableName($name, $value)
    {   // 去除一些不必要的字符
        $name = trim(str_replace(array('export ', '\'', '"'), '', $name));

        return array($name, $value);
    }

    protected function beginsWithAQuote($value)
    {
        return strpbrk($value[0], '"\'') !== false;
    }

    // 取变量值
    public function getEnvironmentVariable($name)
    {
        switch (true) { // 使用 switch(true)，操作可以，说明这里主要是为了和 case 后面的值进行比较
            case array_key_exists($name, $_ENV):
                return $_ENV[$name];
            case array_key_exists($name, $_SERVER):
                return $_SERVER[$name];
            default:
                $value = getenv($name);
                return $value === false ? null : $value;
        }
    }

    public function setEnvironmentVariable($name, $value = null)
    {
        list($name, $value) = $this->normaliseEnvironmentVariable($name, $value);
        // 对于已经的环境变量，通过判断是否可改，再进行操作
        if ($this->immutable && $this->getEnvironmentVariable($name) !== null) {
            return; // 第二个参数有作用了，应该是为了避免修改系统已有的环境变量值
        }
    
        if (function_exists('apache_getenv') && function_exists('apache_setenv') && apache_getenv($name)) {
            apache_setenv($name, $value); // 在 apache 的环境下
        }

        if (function_exists('putenv')) {
            putenv("$name=$value");
        }

        $_ENV[$name] = $value; // 在 $_ENV 和 $_SERVER 两个环境常量中也保存了该信息
        $_SERVER[$name] = $value;
    }

    public function clearEnvironmentVariable($name)
    {
        if ($this->immutable) {
            return;
        }

        if (function_exists('putenv')) {
            putenv($name);
        }

        unset($_ENV[$name], $_SERVER[$name]);
    }
}
```