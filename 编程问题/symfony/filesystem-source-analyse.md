# Symfony - filesystem component 源码分析及注释

```php
/*
 * This file is part of the Symfony package.
 *
 * (c) Fabien Potencier <fabien@symfony.com>
 *
 * For the full copyright and license information, please view the LICENSE
 * file that was distributed with this source code.
 */

namespace Symfony\Component\Filesystem;

use Symfony\Component\Filesystem\Exception\IOException;
use Symfony\Component\Filesystem\Exception\FileNotFoundException;

/**
 * @author Fabien Potencier <fabien@symfony.com>
 */
class Filesystem
{
    /**
     * 复制一个文件
     *
     * 如果目标文件较旧，总是会发生覆盖
     * 如果目标文件较新，则只有在 $overwriteNewerFiles = true 时发生覆盖
     *
     * @param string $originFile          源文件名
     * @param string $targetFile          目标文件名
     * @param bool   $overwriteNewerFiles 如果为真，即使目标文件较新也会发生覆盖
     *
     * @throws FileNotFoundException 源文件不存在
     * @throws IOException           复制失败
     */
    public function copy($originFile, $targetFile, $overwriteNewerFiles = false)
    {
        /**
         * 使用 stream_is_local 检查流或URL是否为本地
         * 如果 $originFile 中不包含"file://"，则表示本地
         */
        $originIsLocal = stream_is_local($originFile) || 0 === stripos($originFile, 'file://');

        /**
         * 当文件是本地文件时，检查该文件是否存在，如果不存在，报 FileNotFoundException
         */
        if ($originIsLocal && !is_file($originFile)) {
            throw new FileNotFoundException(sprintf('Failed to copy "%s" because file does not exist.', $originFile), 0, null, $originFile);
        }

        /**
         * 使用 $this->mkdir 创建目标文件所有目标
         */
        $this->mkdir(dirname($targetFile));

        /**
         * 先假设需要强制复制
         */
        $doCopy = true;

        /**
         * 当目标文件存在时，如果 $overwriteNewerFiles = false 时，并且源目标文件不是网络文件
         * 比较源文件和目标文件的修改时间，如果源文件较新，则进行复制
         * 
         * 当目标文件不存在时，不需要进行该判断
         * 
         * 当源文件是网络文件时，也不需要进行该判断
         */
        if (!$overwriteNewerFiles && null === parse_url($originFile, PHP_URL_HOST) && is_file($targetFile)) {
            $doCopy = filemtime($originFile) > filemtime($targetFile);
        }

        if ($doCopy) {
            /**
             * 源文件无法打开报 IOException，能打开则将文件句柄赋值给 $source
             */
            if (false === $source = @fopen($originFile, 'r')) {
                throw new IOException(sprintf('Failed to copy "%s" to "%s" because source file could not be opened for reading.', $originFile, $targetFile), 0, null, $originFile);
            }

            // Stream context created to allow files overwrite when using FTP stream wrapper - disabled by default
            /**
             * 创建一个目标句柄，并允许使用 ftp 的形式进行数据复制
             */
            if (false === $target = @fopen($targetFile, 'w', null, stream_context_create(array('ftp' => array('overwrite' => true))))) {
                throw new IOException(sprintf('Failed to copy "%s" to "%s" because target file could not be opened for writing.', $originFile, $targetFile), 0, null, $originFile);
            }

            /**
             * 使用流拷贝的方式进行复制，复制完成后，将复制的字节数赋值给 $bytesCopied
             */
            $bytesCopied = stream_copy_to_stream($source, $target);

            /**
             * 释放两个文件句柄
             */
            fclose($source);
            fclose($target);
            unset($source, $target);

            /**
             * 如果目标文件没有创建成功则报 IOException
             */
            if (!is_file($targetFile)) {
                throw new IOException(sprintf('Failed to copy "%s" to "%s".', $originFile, $targetFile), 0, null, $originFile);
            }

            /**
             * 在本地文件下，如果复制的字节数不等于源文件的字节数，则报 IOException
             * 
             * 使用 fileperms 得到源文件和目标文件的权限信息
             * 源文件的权限信息与 0111 进行按位与操作，得到结果后再与目标文件的权限信息进行按位或的操作
             * 目的在于要操作目标文件的执行权限
             */
            if ($originIsLocal) {
                @chmod($targetFile, fileperms($targetFile) | (fileperms($originFile) & 0111));

                if ($bytesCopied !== $bytesOrigin = filesize($originFile)) {
                    throw new IOException(sprintf('Failed to copy the whole content of "%s" to "%s" (%g of %g bytes copied).', $originFile, $targetFile, $bytesCopied, $bytesOrigin), 0, null, $originFile);
                }
            }
        }
    }

    /**
     * 以递归的形式创建目录
     *
     * @param string|iterable 可以是单个目标，也可以是可遍历的对象
     * @param int             目标权限
     *
     * @throws IOException 目标创建失败
     */
    public function mkdir($dirs, $mode = 0777)
    {
        /**
         * 使用 $this->toIterator 利用参数 $dirs，返回可遍历的对象
         * 
         * 如果目标已经存在，则跳过
         */
        foreach ($this->toIterator($dirs) as $dir) {
            if (is_dir($dir)) {
                continue;
            }

            /** 
             * 使用 mkdir 创建目标，使用@符号，屏蔽错误
             * 
             * 如果目录没有创建，将 error_get_last 方法取得错误信息，报 IOException
             * 
             * 如果有错误发生，则报的信息为错误的信息
             */
            if (true !== @mkdir($dir, $mode, true)) {
                $error = error_get_last();
                if (!is_dir($dir)) {
                    // The directory was not created by a concurrent process. Let's throw an exception with a developer friendly error message if we have one
                    if ($error) {
                        throw new IOException(sprintf('Failed to create "%s": %s.', $dir, $error['message']), 0, null, $dir);
                    }
                    throw new IOException(sprintf('Failed to create "%s"', $dir), 0, null, $dir);
                }
            }
        }
    }

    /**
     * 检查目录或者文件是否存在
     *
     * @param string|iterable 单个文件名，或者是一个可遍历的对象
     *
     * @return bool 存在为真，不存在为假
     */
    public function exists($files)
    {
        $maxPathLength = PHP_MAXPATHLEN - 2;

        foreach ($this->toIterator($files) as $file) {
            /**
             * 当文件或目标路径长度大于 PHP 限制长度时，报错
             */
            if (strlen($file) > $maxPathLength) {
                throw new IOException(sprintf('Could not check if file exist because path length exceeds %d characters.', $maxPathLength), 0, null, $file);
            }

            /**
             * 使用 file_exists 判断文件是否存在
             */
            if (!file_exists($file)) {
                return false;
            }
        }

        return true;
    }

    /**
     * 为文件设置访问时间和修改时间
     * 访问时间总是会被修改
     *
     * @param string|iterable 单个文件名(目录)，或一个可遍历的对象
     * @param int             $time  The touch time as a Unix timestamp
     * @param int             $atime 访问时间
     *
     * @throws IOException When touch fails
     */
    public function touch($files, $time = null, $atime = null)
    {
        foreach ($this->toIterator($files) as $file) {
            /**
             * 使用 touch 方法修改文件属性
             */
            $touch = $time ? @touch($file, $time, $atime) : @touch($file);
            if (true !== $touch) {
                throw new IOException(sprintf('Failed to touch "%s".', $file), 0, null, $file);
            }
        }
    }

    /**
     * 删除文件或者删除目录
     *
     * @param string|iterable 单个文件名(目录名)，或一个可遍历对象
     *
     * @throws IOException 删除失败
     */
    public function remove($files)
    {
        /**
         * 将 $files 转换为一个数组
         * 如果是一个可遍历对象，利用 iterator_to_array
         * 如果不是一个数组，利用 array 创建数组，该数组中只含单个元素
         */
        if ($files instanceof \Traversable) {
            $files = iterator_to_array($files, false);
        } elseif (!is_array($files)) {
            $files = array($files);
        }

        /**
         * 将数组进行反序
         */
        $files = array_reverse($files);

        /**
         * 遍历每一个文件
         */
        foreach ($files as $file) {

            if (is_link($file)) {
                // See https://bugs.php.net/52176
                /**
                 * 进行如果操作
                 * 1. 删除链接
                 * 2. 目录分隔符为 \\
                 * 3. 删除目录
                 * 
                 * 执行完上述操作做，文件如果还是存在，报错
                 */
                if (!@(unlink($file) || '\\' !== DIRECTORY_SEPARATOR || rmdir($file)) && file_exists($file)) {
                    $error = error_get_last();
                    throw new IOException(sprintf('Failed to remove symlink "%s": %s.', $file, $error['message']));
                }
            } elseif (is_dir($file)) {
                $this->remove(new \FilesystemIterator($file, \FilesystemIterator::CURRENT_AS_PATHNAME | \FilesystemIterator::SKIP_DOTS));
                
                /**
                 * 删除目录，如果还存在则报错
                 */
                if (!@rmdir($file) && file_exists($file)) {
                    $error = error_get_last();
                    throw new IOException(sprintf('Failed to remove directory "%s": %s.', $file, $error['message']));
                }
            } elseif (!@unlink($file) && file_exists($file)) {
                /**
                 * 直接删除链接，如果还存在则报错
                 */
                $error = error_get_last();
                throw new IOException(sprintf('Failed to remove file "%s": %s.', $file, $error['message']));
            }
        }
    }

    /**
     * 修改文件权限值
     *
     * @param string|iterable $files     单个文件，多个文件的数组(或目录)
     * @param int             $mode      新权限值
     * @param int             $umask     掩码
     * @param bool            $recursive 是否递归
     *
     * @throws IOException 修改失败报错
     */
    public function chmod($files, $mode, $umask = 0000, $recursive = false)
    {
        foreach ($this->toIterator($files) as $file) {
            /**
             * 对文件进行修改权限操作
             */
            if (true !== @chmod($file, $mode & ~$umask)) {
                throw new IOException(sprintf('Failed to chmod file "%s".', $file), 0, null, $file);
            }

            // 如果是目录类型且不是链接，$recursive = true 时，将进行递归操作，将该目录所有的上级目录一并修改权限
            if ($recursive && is_dir($file) && !is_link($file)) {
                $this->chmod(new \FilesystemIterator($file), $mode, $umask, true);
            }
        }
    }

    /**
     * 修改文件所有者
     *
     * @param string|iterable $files     单个文件，或可遍历对象
     * @param string          $user      新的用户
     * @param bool            $recursive 是否递归
     *
     * @throws IOException 失败报错
     */
    public function chown($files, $user, $recursive = false)
    {
        foreach ($this->toIterator($files) as $file) {

            /**
             * 需要递归，且为目录，不是链接，则递归修改所有者
             */
            if ($recursive && is_dir($file) && !is_link($file)) {
                $this->chown(new \FilesystemIterator($file), $user, true);
            }
            /**
             * 如果是一个链接，则需要使用 lchown 方法
             * 如果不是一个链接文件，则需要 chown 方法
             */
            if (is_link($file) && function_exists('lchown')) {
                if (true !== @lchown($file, $user)) {
                    throw new IOException(sprintf('Failed to chown file "%s".', $file), 0, null, $file);
                }
            } else {
                if (true !== @chown($file, $user)) {
                    throw new IOException(sprintf('Failed to chown file "%s".', $file), 0, null, $file);
                }
            }
        }
    }

    /**
     * 修改文件所有组
     *
     * @param string|iterable $files     单个文件，或可遍历对象
     * @param string          $group     新的组名
     * @param bool            $recursive 是否需要递归
     *
     * @throws IOException 失败报错
     */
    public function chgrp($files, $group, $recursive = false)
    {
        foreach ($this->toIterator($files) as $file) {
            /**
             * 需要递归，且为目录，不是链接，则递归修改所有组
             */
            if ($recursive && is_dir($file) && !is_link($file)) {
                $this->chgrp(new \FilesystemIterator($file), $group, true);
            }

            /**
             * 如果是一个链接，则需要使用 lchgrp 方法
             * 如果不是一个链接文件，则需要 chgrp 方法
             * posix_getgrnam 该方法返回一个组的信息，用于判断所给组是否存在
             */
            if (is_link($file) && function_exists('lchgrp')) {
                if (true !== @lchgrp($file, $group) || (defined('HHVM_VERSION') && !posix_getgrnam($group))) {
                    throw new IOException(sprintf('Failed to chgrp file "%s".', $file), 0, null, $file);
                }
            } else {
                if (true !== @chgrp($file, $group)) {
                    throw new IOException(sprintf('Failed to chgrp file "%s".', $file), 0, null, $file);
                }
            }
        }
    }

    /**
     * 为文件或目录重命名
     *
     * @param string $origin    源文件或源目录
     * @param string $target    新的文件名或新的目录名
     * @param bool   $overwrite 如果新的文件或目录已经存在是否进行覆盖
     *
     * @throws IOException 已有该文件名或已有该目录名
     * @throws IOException 源不能重命名
     */
    public function rename($origin, $target, $overwrite = false)
    {
        /**
         * 使用 $this-isReadable 还判断目录是否存在
         * 如果选择进行覆盖。即 $overwrite = true，则不管去理会目标是否存在
         */
        if (!$overwrite && $this->isReadable($target)) {
            throw new IOException(sprintf('Cannot rename because the target "%s" already exists.', $target), 0, null, $target);
        }

        /**
         * 使用 rename 进行修改
         * 使用 rename 可以进行 mv 命令的操作
         */
        if (true !== @rename($origin, $target)) {
            /**
             * 当使用 rename 失败时，则可以源是一个目录
             */
            if (is_dir($origin)) {
                // See https://bugs.php.net/bug.php?id=54097 & http://php.net/manual/en/function.rename.php#113943

                /**
                 * 如果是目录的话，使用 $this->mirror 做镜像操作，并删除源
                 */
                $this->mirror($origin, $target, null, array('override' => $overwrite, 'delete' => $overwrite));
                $this->remove($origin);

                return;
            }
            throw new IOException(sprintf('Cannot rename "%s" to "%s".', $origin, $target), 0, null, $target);
        }
    }

    /**
     * 判断一个 文件/目录 是否可读
     *
     * @param string 文件/目录 路径
     *
     * @return bool
     *
     * @throws IOException 长度太长
     */
    private function isReadable($filename)
    {
        $maxPathLength = PHP_MAXPATHLEN - 2;

        if (strlen($filename) > $maxPathLength) {
            throw new IOException(sprintf('Could not check if file is readable because path length exceeds %d characters.', $maxPathLength), 0, null, $filename);
        }

        /**
         * 调用 is_readable
         */
        return is_readable($filename);
    }

    /**
     * 创建链接文件或创建目录
     *
     * @param string $originDir     源目录
     * @param string $targetDir     目标
     * @param bool   $copyOnWindows 如果是 Windows 环境，是否需要复制文件
     *
     * @throws IOException 创建失败
     */
    public function symlink($originDir, $targetDir, $copyOnWindows = false)
    {
        /**
         * 表示目录的分隔符为 \\
         */
        if ('\\' === DIRECTORY_SEPARATOR) {
            /**
             * 使用 strtr 方法，将目录中的 / 换成 \\
             */
            $originDir = strtr($originDir, '/', '\\');
            $targetDir = strtr($targetDir, '/', '\\');

            if ($copyOnWindows) {
                $this->mirror($originDir, $targetDir);

                // 这里直接返回了！！！
                return;
            }
        }

        /**
         * 使用 $this->mkdir 创建目录
         */
        $this->mkdir(dirname($targetDir));

        $ok = false;
        if (is_link($targetDir)) {
            /**
             * 如果目标目录是一个链接，如果该链接并不是指向源目录，则将其删除
             */
            if (readlink($targetDir) != $originDir) {
                $this->remove($targetDir);
            } else {
                /**
                 * 当 $ok = true 时，已经不需要在执行后面的程序了
                 */
                $ok = true;
            }
        }

        /**
         * 使用 symlink 方法创建链接
         */
        if (!$ok && true !== @symlink($originDir, $targetDir)) {
            $report = error_get_last();
            if (is_array($report)) {
                if ('\\' === DIRECTORY_SEPARATOR && false !== strpos($report['message'], 'error code(1314)')) {
                    throw new IOException('Unable to create symlink due to error code 1314: \'A required privilege is not held by the client\'. Do you have the required Administrator-rights?', 0, null, $targetDir);
                }
            }
            throw new IOException(sprintf('Failed to create symbolic link from "%s" to "%s".', $originDir, $targetDir), 0, null, $targetDir);
        }
    }

    /**
     * 给定路径，转为相对路径
     *
     * @param string $endPath   目标的绝对路径
     * @param string $startPath Absolute path where traversal begins
     *
     * @return string Path of target relative to starting path
     */
    public function makePathRelative($endPath, $startPath)
    {
        if ('\\' === DIRECTORY_SEPARATOR) {
            /**
             * 将分隔符号进行替换
             */
            $endPath = str_replace('\\', '/', $endPath);
            $startPath = str_replace('\\', '/', $startPath);
        }

        $stripDriveLetter = function ($path) {
            /**
             * 需要对两个目录进行截取操作
             */
            if (strlen($path) > 2 && ':' === $path[1] && '/' === $path[2] && ctype_alpha($path[0])) {
                return substr($path, 2);
            }

            return $path;
        };

        $endPath = $stripDriveLetter($endPath);
        $startPath = $stripDriveLetter($startPath);

        // Split the paths into arrays
        $startPathArr = explode('/', trim($startPath, '/'));
        $endPathArr = explode('/', trim($endPath, '/'));

        $normalizePathArray = function ($pathSegments, $absolute) {
            $result = array();

            foreach ($pathSegments as $segment) {
                if ('..' === $segment && ($absolute || count($result))) {
                    array_pop($result);
                } elseif ('.' !== $segment) {
                    $result[] = $segment;
                }
            }

            return $result;
        };

        $startPathArr = $normalizePathArray($startPathArr, static::isAbsolutePath($startPath));
        $endPathArr = $normalizePathArray($endPathArr, static::isAbsolutePath($endPath));

        // Find for which directory the common path stops
        $index = 0;
        while (isset($startPathArr[$index]) && isset($endPathArr[$index]) && $startPathArr[$index] === $endPathArr[$index]) {
            ++$index;
        }

        // Determine how deep the start path is relative to the common path (ie, "web/bundles" = 2 levels)
        if (1 === count($startPathArr) && '' === $startPathArr[0]) {
            $depth = 0;
        } else {
            $depth = count($startPathArr) - $index;
        }

        // Repeated "../" for each level need to reach the common path
        $traverser = str_repeat('../', $depth);

        $endPathRemainder = implode('/', array_slice($endPathArr, $index));

        // Construct $endPath from traversing to the common path, then to the remaining $endPath
        $relativePath = $traverser.('' !== $endPathRemainder ? $endPathRemainder.'/' : '');

        return '' === $relativePath ? './' : $relativePath;
    }

    /**
     * 为一个目录制作一个镜像
     *
     * Copies files and directories from the origin directory into the target directory. By default:
     * 
     * 从源目录中复制文件或目录到目标目录
     *
     *  - existing files in the target directory will be overwritten, except if they are newer (see the `override` option)
     *  - files in the target directory that do not exist in the source directory will not be deleted (see the `delete` option)
     *
     * @param string       $originDir The origin directory 源目录
     * @param string       $targetDir The target directory 目标目录
     * @param \Traversable $iterator  Iterator that filters which files and directories to copy
     * @param array        $options   An array of boolean options
     *                                Valid options are:
     *                                - $options['override'] If true, target files newer than origin files are overwritten (see copy(), defaults to false)
     *                                - $options['copy_on_windows'] Whether to copy files instead of links on Windows (see symlink(), defaults to false)
     *                                - $options['delete'] Whether to delete files that are not in the source directory (defaults to false)
     *
     * $options 参数中包含三个配置项
     * @throws IOException When file type is unknown
     */
    public function mirror($originDir, $targetDir, \Traversable $iterator = null, $options = array())
    {
        /**
         * 使用 rtrim 方法，去除右边的 / 或 \
         */
        $targetDir = rtrim($targetDir, '/\\');
        $originDir = rtrim($originDir, '/\\');
        $originDirLen = strlen($originDir);

        // Iterate in destination folder to remove obsolete entries
        /**
         * 如果目标存在，则设置可以删除
         */
        if ($this->exists($targetDir) && isset($options['delete']) && $options['delete']) {
            $deleteIterator = $iterator;
            if (null === $deleteIterator) {
                /**
                 * 以 . 开头的文件，不去操作
                 */
                $flags = \FilesystemIterator::SKIP_DOTS;
                $deleteIterator = new \RecursiveIteratorIterator(new \RecursiveDirectoryIterator($targetDir, $flags), \RecursiveIteratorIterator::CHILD_FIRST);
            }
            $targetDirLen = strlen($targetDir);
            foreach ($deleteIterator as $file) {
                /**
                 * 将目标目录中的内容进行删除
                 */
                $origin = $originDir.substr($file->getPathname(), $targetDirLen);
                if (!$this->exists($origin)) {
                    $this->remove($file);
                }
            }
        }

        $copyOnWindows = false;
        if (isset($options['copy_on_windows'])) {
            $copyOnWindows = $options['copy_on_windows'];
        }

        if (null === $iterator) {
            $flags = $copyOnWindows ? \FilesystemIterator::SKIP_DOTS | \FilesystemIterator::FOLLOW_SYMLINKS : \FilesystemIterator::SKIP_DOTS;
            $iterator = new \RecursiveIteratorIterator(new \RecursiveDirectoryIterator($originDir, $flags), \RecursiveIteratorIterator::SELF_FIRST);
        }

        if ($this->exists($originDir)) {
            $this->mkdir($targetDir);
        }

        foreach ($iterator as $file) {
            /**
             * 遍历所有的内容信息
             */
            $target = $targetDir.substr($file->getPathname(), $originDirLen);

            if ($copyOnWindows) {
                /**
                 * 如果是在 Windows 中操作，则不需去进行链接文件的操作
                 */
                if (is_file($file)) {
                    $this->copy($file, $target, isset($options['override']) ? $options['override'] : false);
                } elseif (is_dir($file)) {
                    $this->mkdir($target);
                } else {
                    throw new IOException(sprintf('Unable to guess "%s" file type.', $file), 0, null, $file);
                }
            } else {
                if (is_link($file)) {
                    $this->symlink($file->getLinkTarget(), $target);
                } elseif (is_dir($file)) {
                    $this->mkdir($target);
                } elseif (is_file($file)) {
                    $this->copy($file, $target, isset($options['override']) ? $options['override'] : false);
                } else {
                    throw new IOException(sprintf('Unable to guess "%s" file type.', $file), 0, null, $file);
                }
            }
        }
    }

    /**
     * 判断是否是绝对路径
     *
     * @param string $file 文件路径
     *
     * @return bool
     */
    public function isAbsolutePath($file)
    {
        /**
         * 对路径的前两个字符进行检查，是否存在 / 或 \，如果包含则是绝对路径
         * 如果路径长度大于3，且第一个字符为纯字符 && 第二个字符为 : && 包含/ 或 \，三条满足表示绝对路径
         * 以URL的模式进行解析，解析成功表示为绝对路径
         */
        return strspn($file, '/\\', 0, 1)
            || (strlen($file) > 3 && ctype_alpha($file[0])
                && ':' === substr($file, 1, 1)
                && strspn($file, '/\\', 2, 1)
            )
            || null !== parse_url($file, PHP_URL_SCHEME)
        ;
    }

    /**
     * 创建一个临时文件
     *
     * @param string $dir    在哪个目录下
     * @param string $prefix 临时文件的前缀
     *
     * @return string 返回临时文件的文件名
     */
    public function tempnam($dir, $prefix)
    {
        /**
         * 对目录进行拆分
         */
        list($scheme, $hierarchy) = $this->getSchemeAndHierarchy($dir);

        // If no scheme or scheme is "file" or "gs" (Google Cloud) create temp file in local filesystem
        if (null === $scheme || 'file' === $scheme || 'gs' === $scheme) {
            /**
             * 创建临时文件
             */
            $tmpFile = @tempnam($hierarchy, $prefix);

            // If tempnam failed or no scheme return the filename otherwise prepend the scheme
            if (false !== $tmpFile) {
                if (null !== $scheme && 'gs' !== $scheme) {
                    return $scheme.'://'.$tmpFile;
                }

                return $tmpFile;
            }

            throw new IOException('A temporary file could not be created.');
        }

        // Loop until we create a valid temp file or have reached 10 attempts
        for ($i = 0; $i < 10; ++$i) {
            // Create a unique filename
            $tmpFile = $dir.'/'.$prefix.uniqid(mt_rand(), true);

            // Use fopen instead of file_exists as some streams do not support stat
            // Use mode 'x+' to atomically check existence and create to avoid a TOCTOU vulnerability
            $handle = @fopen($tmpFile, 'x+');

            // If unsuccessful restart the loop
            if (false === $handle) {
                continue;
            }

            // Close the file if it was successfully opened
            @fclose($handle);

            return $tmpFile;
        }

        throw new IOException('A temporary file could not be created.');
    }

    /**
     * 向文件写入内容
     *
     * @param string   $filename 目标文件
     * @param string   $content  写入的内容
     * @param null|int $mode     文件权限，如果为null，则不进行权限修改
     *
     * @throws IOException 如果失败
     */
    public function dumpFile($filename, $content, $mode = 0666)
    {

        $dir = dirname($filename);

        if (!is_dir($dir)) {
            /**
             * 目录不存在，需要创建目录
             */
            $this->mkdir($dir);
        }

        /**
         * 需要可写
         */
        if (!is_writable($dir)) {
            throw new IOException(sprintf('Unable to write to the "%s" directory.', $dir), 0, null, $dir);
        }

        /**
         * 使用 $this->tempnam 创建临时文件句柄
         */
        $tmpFile = $this->tempnam($dir, basename($filename));

        /**
         * 使用 file_put_contents 将内容写入到临时文件
         */
        if (false === @file_put_contents($tmpFile, $content)) {
            throw new IOException(sprintf('Failed to write file "%s".', $filename), 0, null, $filename);
        }

        if (null !== $mode) {
            if (func_num_args() > 2) {
                @trigger_error('Support for modifying file permissions is deprecated since Symfony 2.3.12 and will be removed in 3.0.', E_USER_DEPRECATED);
            }

            $this->chmod($tmpFile, $mode);
        } elseif (file_exists($filename)) {
            @chmod($tmpFile, fileperms($filename));
        }
        /**
         * 使用 $this->rename 进行 mv 操作
         */
        $this->rename($tmpFile, $filename, true);
    }

    /**
     * 转换为可遍历的类型，对单条信息同多条信息的统一处理
     * 
     * @param mixed $files
     *
     * @return \Traversable
     */
    private function toIterator($files)
    {
        /**
         * Traversable 可遍历的类型
         */
        if (!$files instanceof \Traversable) {
            $files = new \ArrayObject(is_array($files) ? $files : array($files));
        }

        return $files;
    }

    /**
     * 对路径进行拆分，file:///tmp -> array(file, tmp)
     *
     * @param string $filename 需要解析的文件名
     *
     * @return array 返回层次结构的数组
     */
    private function getSchemeAndHierarchy($filename)
    {
        /**
         * 第三个参数用于，只产生两个元素
         */
        $components = explode('://', $filename, 2);

        return 2 === count($components) ? array($components[0], $components[1]) : array(null, $components[0]);
    }
}
```