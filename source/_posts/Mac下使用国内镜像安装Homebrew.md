---
title: Mac下使用国内镜像安装Homebrew
date: 2019-01-12 15:28:15
tags: brew安装
categories: 操作系统
---

<!--more-->

## 通用安装方式

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

## 使用国内镜像安装步骤

第一步，

```
cd ~
curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install >> brew_install
```

第二步，修改`brew_install`文件

```
#!/System/Library/Frameworks/Ruby.framework/Versions/Current/usr/bin/ruby
# This script installs to /usr/local only. To install elsewhere you can just
# untar https://github.com/Homebrew/brew/tarball/master anywhere you like or
# change the value of HOMEBREW_PREFIX.
HOMEBREW_PREFIX = "/usr/local".freeze
HOMEBREW_REPOSITORY = "/usr/local/Homebrew".freeze
HOMEBREW_CACHE = "#{ENV["HOME"]}/Library/Caches/Homebrew".freeze
HOMEBREW_OLD_CACHE = "/Library/Caches/Homebrew".freeze

# 下面这行是原始文件，将此行注释掉，换成它下面这行
#BREW_REPO = "https://github.com/Homebrew/brew".freeze	
BREW_REPO = "git://mirrors.ustc.edu.cn/brew.git".freeze

# 下面这行是原始文件，将此行注释掉，换成它下面这行
#CORE_TAP_REPO = "https://github.com/Homebrew/homebrew-core".freeze
CORE_TAP_REPO = "git://mirrors.ustc.edu.cn/homebrew-core.git".freeze
```

第三步，终端输入

```
/usr/bin/ruby ~/brew_install 
```

第四步，替换homebrew默认源

```
cd "$(brew --repo)"
git remote set-url origin git://mirrors.ustc.edu.cn/brew.git
```

第五步，替换homebrew-core源

```
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin git://mirrors.ustc.edu.cn/homebrew-core.git
```

第六步，设置 bintray镜像

```
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile
```