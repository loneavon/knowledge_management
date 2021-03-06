title: Emacs Org-mode 入门学习
date: 2017-01-03
tags:
- Emacs
- Org-mode
categories: work
---
### 为什么要学习 Emacs Org-mode

1. 知识管理 & 时间管理是我进入到下一阶段的有效理论，但缺少有效的工具
2. 知识管理重点在于记录和分享，现有的常使用的工具有 Evernote 、Typora 以及各种 Wiki & JIRA，这些工具可以记录但是缺少有效的分享机制，且记录零散，尤其厌烦各种格式
3. 时间管理重点在于记录和反馈，目前的方法只记录完成了什么，手动更新状态，无法记录耗时
4. 记录格式为 Markdown、Text 等，各种格式之间不兼容。虽然 Markdown 可以作为中间语言，但工具残次不齐，选择恐惧症
5. 好奇心作祟，Vim vs Emacs

基于以上，在茫茫互联网中发现 Emacs Org-mode，有丰富的插件来完成知识的记录和管理，有强大的 GTD 支持。Org-mode 作为中间语言存在，可以支持转化为多种格式，目前最有用的为转化为 HTML & Markdown，这样可以将自己整理的内容结合 git 分享到网上，构建一个从终端记录 -> 半自动整理 -> 发布 -> 分享的机制

### Org-mode Blog Example

[前百度同事, 页面极简, 技术干货,翻译较多](http://dirtysalt.info/)

### 学习 Emacs Org-mode 切入点

Emacs Org-mode 已有初步了解，需要慢慢融入到工作 & 生活中，先考虑工作相关，初步有以下需求：

1. 学习 & 调研文档整理，转化为 HTML 或者 Markdown
2. 每日工作计划 & GTD （GTD 正在学习中），先期记录事件，熟悉工具

#### 需求细分

1. 文档整理 & GTD 需要统一模板，考虑 Org-mode 生成自定义模板，部分信息如时间 & 作者等自动填充。其他信息手动补充，如标题、KeyWords 、主题框架等
2. 文档整理中少不了图片，引用已有图片和截图功能
3. 导出 HTML & Markdown

### 调研

##### 插入模板

分为两部分：a. 插入一些基本信息（用于分类和索引）b. 插入文档的主题框架

基本信息在 Org-mode 中用于 export，所以 “export setting” 可以检索出来大部分有用的信息。Org-mode 提供了几个导出的模板，见下：

 ![Org-mode Export template](/Users/loneavon/Downloads/DB4D3C15-45BF-4E07-BEAD-EE05814A2BB1.png)

通过 C-c C-e  + ‘#’ 可以调出选择 template 对话，但是默认的 template 对应的元素都不是特别匹配。需要元素见下：

```powe
#+TITLE: Emacs Org Mode
#+DATE: 2014-02-02
#+KEYWORDS: Emacs, Makeup, 知识管理, 时间管理
#+DESCRIPTION: 知识管理，时间管理
#+AUTHOR: levin
#+EMAIL: loneavon1@gmail.com
```

找了很长时间发现 yasnippet 这个插件，这个插件常用来做编程语言的自动填充，同时你可以自定义填充框架，这种方法有一个很麻烦的地方，就是需要扩充模板元素时之前添加过的 org 文件已经无法再更新模板元素。

[Yasnippet 简介](http://www.cnblogs.com/qazwsxedc121/p/3285552.html)

[Yasnippet 官方文档](https://www.emacswiki.org/emacs/Yasnippet)

但是目前官方默认支持的都是各种编程语言的 mode，不支持 org-mode，要想在 org-mode 中添加填充的片段需要手动支持 org-mode。目前不太清楚实现 org-mode 的逻辑，不过我猜应该是在 snippets 目前下直接创建 org-mode 的文件夹，加入对应的 snippet 文件即可

[Using Yasnippet to Reduce Blogging Friction](http://irreal.org/blog/?p=2211)

发现已经有人这么做了，还给出了实现方法，跟我猜的一样：

``` 
I loaded yasnippet with ELPA and I'm pretty sure it came from MELPA. My configuration is merely

(require 'yasnippet)
(yas-global-mode 1)

I had to restart Emacs to get it to see the "built-in" snippets. There are no predefined Org Mode snippets but you can define your own (as I did) by adding the directories ~/.emacs.d/snippets/org-mode and adding your custom Org Mode snippets to the org-mode directory
```

添加时要插入当前时间(Date)，发现可以在  snippet 文件执行 Lisp 代码，加上下面的代码即可：

```Lisp
`(format-time-string "<%Y-%m-%d %H:%M>" (current-time))`
```

另外，考虑也加入 modify time，但是这个时间以什么为准还没有确定，可以用文件 Modify 时间来填充，这部分获取比较复杂，大概分为下面两个步骤：

1. 获取当前文件的文件属性

   ```lisp
   file-attributes (buffer-file-name)
   ```

2. 从获取到的文件属性中截取 Modify Time，不过 Mac 的文件属性跟 Linux 上不太一样，格式见下：

   ```po
   16777220 123547654 -rw-r--r-- 1 loneavon staff 0 220 "Sep  8 17:33:36 2016" "Sep  8 17:31:28 2016" "Sep  8 17:31:28 2016" "Sep  8 17:31:28 2016" 4096 8 0 blog-template.snippet
   ```

获取当前文件名：

```Lisp
(buffer-file-name)
```

目前还没有办法正常获取，[Lisp 操作对象之三 文件 - 水木社区 Emacs 版](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwj4ruCDwP_OAhXDnpQKHbXDDucQFggcMAA&url=http%3A%2F%2Fsmacs.github.io%2Felisp%2F14-file.html&usg=AFQjCNFyn-IbxH1HqrX8bbNZUUYzwUxBlg&sig2=Dui7i8aysA6oy52Bv9iajQ) 这篇文章中介绍了文件操作的内容，但是对分割字符串还不清楚用什么方法，后面再跟进看看

所以折腾完了后，自动填充 Blog 导出元素的基本步骤如下：

1. Emacs 安装 Yasnippet

   ```po
   # 启动 Emacs 后，使用 Emacs 自带的 package-install 安装
   M-x package-install [return] yasnippet [return]
   ```

2. 创建 snippet 的 org-mode 目录

   ```po
   # 使用的 Emacs 24.5 版本，默认使用 elpa 来管理 package，本地路径见下
   mkdir /Users/loneavon/.emacs.d/elpa/yasnippet-0.10.0/snippets/org-mode
   ```

3. 创建 blog-template 的 snippet 文件

   ```po
   cd /Users/loneavon/.emacs.d/elpa/yasnippet-0.10.0/snippets/org-mode && vim blog-template.snippet
   ```

   blog-template.snippet 文件内容：

   ```Li
   # -*- mode: snippet -*-
   # name: blog header template
   # key: bt
   # --
   #+TITLE: ${title}
   #+DATE: `(format-time-string "<%Y-%m-%d %H:%M>" (current-time))`
   #+KEYWORDS: ${keywords}
   #+AUTHOR: levin
   #+EMAIL: loneavon1@gmail.com
   ```

   注：在 Org-mode 下 [TAB] 进行填充时，依赖的 Key 为 snippet 文件中的 key 字段的值，与 snippet 文件名无关

4. 修改 ~/.emacs，增加 Yasnippet 配置

   ```lisp
   ;; yasnippet config
   (add-to-list 'load-path "/Users/loneavon/.emacs.d/elpa/yasnippet-0.10.0")
   (require 'yasnippet)
   (yas-global-mode 1)
   ```

##### 导出到 HTML

[Org-mode Publish 官方文档](http://orgmode.org/worg/org-tutorials/org-publish-html-tutorial.html) 这个文档中详细说明了如果发布一个包含多个 org 文件的 Project，重点关注 Publishing the *org* project

强制发布 HTML：

```Lisp
C-u M-x org-publish-project [return] knowledge
```

注：C-u 为 Force 前缀

导出时自动添加 CSS 文件：

1. (测试生效)在插入导出元素时，默认添加。在 snippet 文件中添加:

   ```lisp
   #+HTML_HEAD_EXTRA: <link rel="stylesheet" type="text/css" href="css/levin.css"/>
   ```

2. (测试不生效)在 ~/.emacs 中对应导出配置添加 style，见下：

   ```lisp
   :style "<link rel=\"stylesheet\" href=\"style.css\" type=\"text/css\" />"
   ```

3. (测试生效)在 ~/.emacs 全局配置文件中设置 html-head，见下：

   ```lisp
   (setq org-html-head "<link rel=\"stylesheet\" type=\"text/css\" href=\"css/levin.css\"/>")
   ```

4. (测试不生效)通过在 org 文件中设置 #+SETUPFILE

   ```Lisp
   #+SETUPFILE: ~/.emacs.d/org-templates/level-N.org
   #+TITLE: My Title
   ```

如果 org 文件中存在下划线时， 在导出的时候会自动将其转义，在配置文件中设置：

```lisp
;; 禁用默认下划线转义, 使用 {} 作为转义标识符, 参考: http://blog.waterlin.org/articles/emacs-org-mode-subscripter-setting.html
;; 此为全局设置，所有 org 文件均生效
(setq org-export-with-sub-superscripts '{})
```

需要对现有的各个 org 生成一个 sitemap，相当于一个索引，sitemap 在第一次产生后当后面 org 文件变化默认不会更新 sitemap，强制更新 sitemap 见下：还没找到解决办法，先删除后生成

###### 生成 HTML 时增加 HOME & UP 链接

在 ~/.emacs.d/init.el 文件 publish 导出 html 的配置增加一下配置即可

```lisp
:html-link-home "index.html"
:html-link-up "sitemap.html"
```

生成 HTML 时可添加 OPTION，可用的 OPTION 请参见 [Publishing Options](http://orgmode.org/manual/Publishing-options.html)

###### 导出代码段语法高亮

1. 在配置文件中设置 org-src-fontify-natively 为 non-nil，使 org buffer 中代码段高亮

   ```lisp
   ;; org 文件中代码段语法高亮
   (setq org-src-fontify-natively t)
   ```

2. 安装 htmlize 使在导出 html 时保持高亮，emacs package-install 无法安装，手动下载文件到 ~/.emacs.d/lisp 下，在 init-org.el 中直接引用即可

###### 增加评论框

目前暂时使用国内的多说，设置方法见下：

1. 在多说申请一个域名，登陆后跳转到 http://duoshuo.com/create-site/，short_name 为“多说域名”去掉.duoshuo.com 的部分（即你所填的字符串）

2. 在 org publish function 中添加以下代码

   ```lisp
   ;; 添加评论框
   (setq org-html-postamble t)
   (setq org-html-postamble-format
         '((	"en"
                "<!-- Duoshuo Comment BEGIN -->
       		<div class='ds-thread'></div>
       		<script type='text/javascript'>
       			var duoshuoQuery = {short_name:'第一步申请的 short_name'};
       			(function() {
                   	var ds = document.createElement('script');
                   	ds.type = 'text/javascript';ds.async = true;
                   	ds.src = 'http://static.duoshuo.com/embed.js';
                   	ds.charset = 'UTF-8';
                   	(document.getElementsByTagName('head')[0]
                     	|| document.getElementsByTagName('body')[0]).appendChild(ds);
                   })();
       		</script>
       		<!-- Duoshuo Comment END -->"
           )))
   ```

多说的界面还不是特别简洁，后面再研究，可以考虑用国外的 Disqus

##### 插入 LaTex 数学公式

Org-mode 默认内置了 LaTex，所以可以直接使用相应的语句即可插入数学公式
```orgmode
     \begin{equation}
     x=\sqrt{b}
     \end{equation}
     
     If $a^2=b$ and \( b=2 \), then the solution must be
     either $$ a=+\sqrt{2} $$ or \[ a=-\sqrt{2} \].
```
插入公式有两种方式：
1. 在行首或者行间空格后，使用 \begin 方式声明，见上
2. 使用 LaTex 常见的数据分隔符, 使用如第二段代码所示，使用单$对、圆括号对在行内插入，使用双$对、中括号在行间插入。请注意，当需要在行内插入使用单个 $ 符号对时，请保证在两个 $ 符号之间紧邻 $ 符号的位置没有空格、圆空号后者引号，且前后需要留有空格。具体请参见[LaTex fragments](http://orgmode.org/manual/LaTeX-fragments.html)
### 其他内容

#### Emacs 报错 & 调试

打印 Emacs Trace 信息：[how-to-show-backtrace-for-emacs](http://stackoverflow.com/questions/14067524/how-to-show-backtrace-for-emacs)

##### Emacs 安装 yasnippet 加载后报错

Emacs 默认的 package 安装路径为：/Users/loneavon/.emacs.d/elpa/，Yasnippet 已安装但在启动加载时找不到对应的路径，报错信息见下：

```lisp
Debugger entered--Lisp error: (file-error "Cannot open load file" "no such file or directory" "yasnippet")
```

解决方法：在 ~/.emacs 中添加以下代码：

```lisp
;; 指定加载路径
(add-to-list 'load-path "/Users/loneavon/.emacs.d/elpa/yasnippet-0.10.0")
```

