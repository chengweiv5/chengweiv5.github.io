---
layout: post
title: Mac OS X EI Captain 安装 Jekyll
tags:
  - mac
  - ruby
---

今天重新开始写博客，然后发现系统升级到 EI Captain 后 jekyll
没有了，看来是被升级覆盖了，然后开始安装这套博客环境，一堆坑。。。

## 安装 jekyll 报错
通过 `sudo gem install jekyll` 的时候报错：

```
 chengwei@Chengweis-MacBook-Air  ~  sudo gem install jekyll
 Password:
 Building native extensions.  This could take a while...
 ERROR:  Error installing jekyll:
    ERROR: Failed to build gem native extension.

        /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/bin/ruby extconf.rb
        mkmf.rb can't find header files for ruby at /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ruby/include/ruby.h


        Gem files will remain installed in /Library/Ruby/Gems/2.0.0/gems/redcarpet-3.3.3 for inspection.
        Results logged to /Library/Ruby/Gems/2.0.0/gems/redcarpet-3.3.3/ext/redcarpet/gem_make.out
```

看起来像是找不到头文件，网上搜了一下发现有说是 Xcode 不是最新，或者没有安装，
包括 Xcode command line tools 没有安装之类的。

我又重新打开 App store 检查了一遍，表示 Xcode
已经是最新版本了；然后又看到说是找不到 make，但是我用 `which make`，却输出了
`/usr/bin/make`。

而且但从错误输出看，却是找不到头文件，而我 ls 了一下，确实没有下面这个文件。

```
/System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ruby/include/ruby.h
```

最后实在没办法，运行了 `make` 指令试试。

结果提示要首先同意使用条款才能使用，如下：

```
 chengwei@Chengweis-MacBook-Air  ~  make


 Agreeing to the Xcode/iOS license requires admin privileges, please re-run as root via sudo.
 ```

 然后，我就运行了 `sudo make`，然后按照提示，翻到条款最底部，然后输入了
 `agree`。

 完成后，make 就可以使用了。

 最后，见证奇迹的时刻到了，重新执行 `sudo gem install jekyll`，成功了。

 ```
 sudo gem install jekyll
 Building native extensions.  This could take a while...
 Successfully installed redcarpet-3.3.3
 Fetching: jekyll-paginate-1.1.0.gem (100%)
 Successfully installed jekyll-paginate-1.1.0
 Fetching: jekyll-gist-1.3.5.gem (100%)
 Successfully installed jekyll-gist-1.3.5
 ...
 Parsing documentation for yajl-ruby-1.2.1
 Installing ri documentation for yajl-ruby-1.2.1
 Done installing documentation for redcarpet, jekyll-paginate, jekyll-gist, coffee-script-source, execjs, coffee-script, jekyll-coffeescript, sass, jekyll-sass-converter, listen, jekyll-watch, classifier-reborn, jekyll, yajl-ruby after 27 seconds
 14 gems installed
 ```

## bundle install 报错

 jekyll 安装成功后，还远远没有完成，接下来需要使用 `bundle install` 安装依赖的
 gem。

 这里需要修改一下本项目的 Gemfile，将源修改为淘宝的 gem 源 `source
 'https://ruby.taobao.org'`。

安装到 nokogiri 的时候，又会出错，表示 nokogiri 不能安装，错误可能如下：

```
checking for xmlParseDoc() in -lxml2... no
checking for xmlParseDoc() in -llibxml2... no
-----
libxml2 is missing.  Please locate mkmf.log to investigate how it is failing.
-----
*** extconf.rb failed ***
Could not create Makefile due to some reason, probably lack of necessary
libraries and/or headers.  Check the mkmf.log file for more details.  You may
need configuration options.
```

但是，用 `brew install libxml2` 后还是不行。解决办法如下：

```
$ bundle config build.nokogiri "--use-system-libraries --with-xml2-include=/usr/local/opt/libxml2/include/libxml2"
```

然后，再执行 `bundle install`，安装完成。

## jekyll serve 报错

最后，`jekyll serve` 会错误，如下：

```
...
      Generating...
  Dependency Error:  Yikes! It looks like you don't have redcarpet or one of its dependencies installed. In order to use Jekyll as currently configured, you'l
l need to install this gem. The full error message from Ruby is: 'cannot load such file -- redcarpet' If you run into trouble, you can find helpful resources
at http://jekyllrb.com/help/!
  Conversion error: Jekyll::Converters::Markdown encountered an error while converting '_posts/2008-04-14-java-send-email.md/#excerpt':
                    redcarpet
             ERROR: YOUR SITE COULD NOT BE BUILT:
                    ------------------------------------
                    redcarpet
```

需要通过 `bundle exec` 来运行 jekyll 命令。

```
$ bundle exec jekyll serve
```

成功了，对 ruby 一窍不通，搜索了好久才最后解决。
