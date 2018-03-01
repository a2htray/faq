# Mac下安装haskell开发环境

## 前期准备

> 使用`brew install haskell-stack`提示只能在`Ruby2.3`下工作

解决:

```bash
brew install ruby@2.3
```

之后提示

```bash
Error: Xcode alone is not sufficient on Sierra.
Install the Command Line Tools:
  xcode-select --install
```

运行`xcode-select --install`,点击`Next`,完成后安装`brew install ruby@2.3`

## 安装记录

```bash
brew install haskell-stack
```

运行

```bash
stack setup
```

可能是网络的原因,没有办法下载文件,最终选择下载平台版本。

[下载地址](https://www.haskell.org/platform/mac.html)

