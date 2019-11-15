---
title: 怎样书写你的Git Commit Message
date: 2019-11-11 17:54:38
tags:
  - git
categories:
  - [技术, git]
---

> 最近在组内搞 Git 的规范，之前 CTO 在 engineering doc 里面有提到一篇如何写 commit message 的文章，把它翻译出来，供大家学习参考。原文详见于 https://chris.beams.io/posts/git-commit/

![](/img/git-commit.png)

# 介绍：为什么好的 commit messages 很重要

如果你随便打开一个 Git 仓库的 commit log，你会发现他们都会有程度不一的混乱。看下我前段时间提交到[Spring 项目的 💩💩💩](https://github.com/spring-projects/spring-framework/commits/e5f4b49?author=cbeams)。

```BASH
$ git log --oneline -5 --author cbeams --before "Fri Mar 26 2009"

e5f4b49 Re-adding ConfigurationPostProcessorTests after its brief removal in r814. @Ignore-ing the testCglibClassesAreLoadedJustInTimeForEnhancement() method as it turns out this was one of the culprits in the recent build breakage. The classloader hacking causes subtle downstream effects, breaking unrelated tests. The test method is still useful, but should only be run on a manual basis to ensure CGLIB is not prematurely classloaded, and should not be run as part of the automated build.
2db0f12 fixed two build-breaking issues: + reverted ClassMetadataReadingVisitor to revision 794 + eliminated ConfigurationPostProcessorTests until further investigation determines why it causes downstream tests to fail (such as the seemingly unrelated ClassPathXmlApplicationContextTests)
147709f Tweaks to package-info.java files
22b25e0 Consolidated Util and MutableAnnotationUtils classes into existing AsmUtils
7f96f57 polishing
```

🤮 到自己都看不下去了。下面是我[最近](https://github.com/spring-projects/spring-framework/commits/5ba3db?author=philwebb)提交的，一样的仓库，对比一下：

```BASH
$ git log --oneline -5 --author pwebb --before "Sat Aug 30 2014"

5ba3db6 Fix failing CompositePropertySourceTests
84564a0 Rework @PropertySource early parsing logic
e142fd1 Add tests for ImportSelector meta-data
887815f Update docbook dependency and generate epub
ac8326d Polish mockito usage
```

哪个你更有读下去的欲望？

前者长短不一，格式不一；后者语言简洁、风格统一。
前者是每个开发者习惯性会出现的；后者则需要一个好的习惯和规范，不是随意做到的。

绝大多数 repo 的 log 都跟前者类似，不过也有例外。[Linux kernel](https://github.com/torvalds/linux/commits/master)项目和[Git](https://github.com/git/git/commits/master)项目本身就是一个非常好的例子，还有[Spring Boot](https://github.com/spring-projects/spring-boot/commits/master)和任何[Tim Pope](https://github.com/tpope/vim-pathogen/commits/master)维护的项目。

这些优秀项目的贡献者深知一个道理：一个精心编写的 commit message 是一个非常好的沟通方式，其他开发者可以很容易理解代码变更的上下文语义环境，还有未来的自己。你一定经历过正准备对着一次历史 commit 骂娘的时候，发现那个坑爹的是你自己 🤔。

Git diff 可以告诉你哪里发生了变更，但只有 commit message 会让你知道为什么会这么做。Peter Hutterer [总结](http://who-t.blogspot.com/2009/12/on-commit-messages.html)的很好：

> Re-establishing the context of a piece of code is wasteful. We can’t avoid it completely, so our efforts should go to reducing it [as much] as possible. Commit messages can do exactly that and as a result, a commit message shows whether a developer is a good collaborator.

> 重新建立起一段代码更改的上下文是非常头痛的。我们很难去完全避免它，但是可以尽可能的减少这类情况的发生，好的 commit 习惯可以帮助我们做到。并且，commit message 可以反映出来一个开发者是不是一个好的团队合作者

![WTF](/img/wtfm.png)

如果你从来没怎么想过关于 commit 好坏的问题，一定是你从来没用过`git log` 或者别的比如第三方 GUI 工具查看 log。

这是一个恶性循环：因为我们的 commit log 本身就是一堆 💩💩💩，也不介意再多我这坨 💩。因为大家都是这个想法，所以 log 变得越来越 💩。

但是呢，一个精心维护的 log 是一个美观又很有用的东西。`git blame`, `revert`, `rebase`, `log`, `shortlog` 这些命令都值得被使用。去 review 别人的 commit 和 PR 也很值得去做，特别是优秀的开发者，看着看着保不齐你哪天自己也会了。大家都这么做的话，**理解项目前几个月甚至前几年发生了什么**就不是没可能，而且是可以有效做到的。

**一个项目长期的成功与否取决于它的可维护性**。所以我们值得花一些时间去做这些规范化，养成良好的习惯。一开始习惯的养成，一直到最后看到成果，这一路下来肯定是比较痛苦的。

本帖我主要阐述了关于如何保持健康的 commit history 的最基本的概念：开发者个人如何写好 commit message。还有些别的方面比如`commit squashing`, 这里就先不细说。

每个编程语言都有自己的格式化、样式、命名规范等。当然了，具体到细节，这些规范会有一些差异。不过大家都知道，大家按照一种规范来，肯定比每个人有一套规范要好。

Git commit 规范也一样，一个项目组里，commit 规范必须要统一。要想把 commit history 搞得漂亮可读性强，首先要声明一个规范，至少要包括以下三个方面：

### 样式

包括标记语法（Markdown），宽度限制（比如 80 个字母换行），语法，首字母大写，标点符号等。把这些点都明确出来，并尽可能的简化，避免过于复杂。最后一个漂亮风格一致的 log 就此诞生 🥳。这样的 log 大家不仅乐于去看，而且确实会把阅读 log 当成一种习惯。

### 正文

Commit message 里面什么信息应该出现，什么不应该出现。

### 元数据

How should issue tracking IDs, pull request numbers, etc. be referenced? （这句话我没看懂，元数据这些怎么被人为管理，理解的同学可以跟我说下，不胜感激。）

不过好在有一些业内通用的现成规则供我们借鉴使用，下面列举了七大规则，让你成为 commit 专家：

# 七个优化你的 commit message 的规则

1. Separate subject from body with a blank line
1. Limit the subject line to 50 characters
1. Capitalize the subject line
1. Do not end the subject line with a period
1. Use the imperative mood in the subject line
1. Wrap the body at 72 characters
1. Use the body to explain what and why vs. how

---

1. 标题与正文用空行隔开
1. 标题不超过 50 个字母
1. 标题行行首字母大写
1. 不要以标点符号结束标题行
1. 标题行要用势在必行的语气
1. 正文部分 72 个字符换行
1. 使用正文部分来解释

举个 🌰

```MARKDOWN
Summarize changes in around 50 characters or less

More detailed explanatory text, if necessary. Wrap it to about 72
characters or so. In some contexts, the first line is treated as the
subject of the commit and the rest of the text as the body. The
blank line separating the summary from the body is critical (unless
you omit the body entirely); various tools like `log`, `shortlog`
and `rebase` can get confused if you run the two together.

Explain the problem that this commit is solving. Focus on why you
are making this change as opposed to how (the code explains that).
Are there side effects or other unintuitive consequences of this
change? Here's the place to explain them.

Further paragraphs come after blank lines.

 - Bullet points are okay, too

 - Typically a hyphen or asterisk is used for the bullet, preceded
   by a single space, with blank lines in between, but conventions
   vary here

If you use an issue tracker, put references to them at the bottom,
like this:

Resolves: #123
See also: #456, #789
```

## 1. 标题与正文用空行隔开

引用`git commit` [手册](https://www.kernel.org/pub/software/scm/git/docs/git-commit.html#_discussion):

> 虽然不是硬性要求，但是这是个好主意：50 个字母总结，空出一行，然后详细描述。第一行被看作是标题，这个在 Git 里有应用场景,比如 Git-format-patch 把 commit 转换成邮件，第一行就会被当做邮件标题，空行下面的描述部分就是邮件主体。

> 译者注：这种方式有个显而易见的好处是，当你的 merge request 只有一个 commit 的时候，commit message 中的标题和正文会被自动解析为 merge request 的标题和描述
> 例子：
> ![我的commit](/img/git-commit-rule-1-1.png)
> ![我的merge request](/img/git-commit-rule-1.png)

当然了，不可能每次 commit 都要求你写这么多，有的简单的写一行即可，比如：

```BASH
Fix typo in introduction to user guide
```

一目了然，无需解释。如果有人想知道是修改了哪个拼错的词，自己用`git show`, `git diff`, `git log -p`看一下就行了

这种一行的 message，直接用下面这条命令去 commit 就行：

```BASH
$ git commit -m "Fix typo in introduction to user guide"
```

不过对于比较复杂的更改，需要别人理解上下文的，还是要写清楚描述，像这样：

```MARKDOWN
Derezz the master control program

MCP turned out to be evil and had become intent on world domination.
This commit throws Tron's disc into MCP (causing its deresolution)
and turns it back into a chess game.
```

不过这种多行的没法使用上面的命令。

下面是用`git log`查看完整的 log：

```BASH
$ git log
commit 42e769bdf4894310333942ffc5a15151222a87be
Author: Kevin Flynn <kevin@flynnsarcade.com>
Date:   Fri Jan 01 00:00:00 1982 -0200

Derezz the master control program

MCP turned out to be evil and had become intent on world domination.
This commit throws Tron's disc into MCP (causing its deresolution)
and turns it back into a chess game.
```

然后你可以用`git log --oneline`去查看标题：

```BASH
$ git log --oneline
42e769 Derezz the master control program
```

或者`git shortlog`， 按作者分组，只显示主题：

```BASH
$ git shortlog
Kevin Flynn (1):
      Derezz the master control program

Alan Bradley (1):
      Introduce security program "Tron"

Ed Dillinger (3):
      Rename chess program to "MCP"
      Modify chess program
      Upgrade chess program

Walter Gibbs (1):
      Introduce protoype chess program
```

还有些别的规范，但是如果 subject 和 body 之间不用空行隔开，就都是扯淡了

## 2. 标题不超过 50 个字符

50 字符不是硬性限制，但确实是一个首要规则。保持标题的精简可以确保容易阅读，一眼看出作者想要解释什么。

> 小贴士：如果你很难去总结这次 commit，肯定是你一次 commit 了太多东西，难以总结。移步[这里](https://www.freshconsulting.com/atomic-commits/)学习一下吧~

Github 在你 commit 标题超过 50 个字符的时候会发出警告。如果超过 72 个字符，就会省略后面的。
![](/img/git-commit-50-limit.png)

所以 72 个字符是硬性限制，但我们尽量做到少于 50 字符。

## 3. 标题行行首字母大写

很简单无需解释

```
Accelerate to 88 miles per hour
```

```
accelerate to 88 miles per hour
```

哪个好哪个坏，一眼就能看出来。无意间单压 ✖️2，skr

## 4. 不要以标点符号结束标题行

标点符号完全没有必要出现在标题行。此外，因为 50 个字符的限制，空格很珍贵，谨慎使用。

好的：

```
Open the pod bay doors
```

不好的

```
Open the pod bay doors.
```

## 5. 标题行要用势在必行的语气

**势在必行的语气** 指得是命令式的语气，比如：

- 去打扫屋子
- 把门关上
- 把垃圾倒了

还有这七条准则也是这种语气。事实上，git 本身也是这种语气：

```MARKDOWN
Merge branch 'myfeature'
```

```MARKDOWN
Revert "Add the thing with the stuff"

This reverts commit cc87791524aedd593cff5a74532befe7ab69ce9d.
```

```MARKDOWN
Merge pull request #123 from someuser/somebranch
```

干净利落，没有废话

一个技巧就是，在你的 commit 标题前面加上 **"If applied, this commit will"**, 也就是你的 commit 将要解决一件什么事，比如：

- If applied, this commit will **refactor subsystem X for readability**
- If applied, this commit will **update getting started documentation**
- If applied, this commit will **remove deprecated methods**
- If applied, this commit will **release version 1.0.0**
- If applied, this commit will **merge pull request #123 from user/branch**

> 小贴士：在标题里可以简单粗暴些，在下面解释体里语气可以缓和一些。

## 6. 正文部分 72 个字符换行

Git 不会自动帮你换行，建议每 72 个字符就自己手动换行一下。可以借助一些编辑器，这里就不解释了。

## 7. 使用正文部分来解释

这儿有个不错的[例子](https://github.com/bitcoin/bitcoin/commit/eb0b56b19017ab5c16c745e6da39c53126924ed6)

```MARKDOWN
commit eb0b56b19017ab5c16c745e6da39c53126924ed6
Author: Pieter Wuille <pieter.wuille@gmail.com>
Date:   Fri Aug 1 22:57:55 2014 +0200

   Simplify serialize.h's exception handling

   Remove the 'state' and 'exceptmask' from serialize.h's stream
   implementations, as well as related methods.

   As exceptmask always included 'failbit', and setstate was always
   called with bits = failbit, all it did was immediately raise an
   exception. Get rid of those variables, and replace the setstate
   with direct exception throwing (which also removes some dead
   code).

   As a result, good() is never reached after a failure (there are
   only 2 calls, one of which is in tests), and can just be replaced
   by !eof().

   fail(), clear(n) and exceptions() are just never called. Delete
   them.
```

可以看看作者的更改，就会明白作者为别人节省了多少时间。如果他现在不在这里解释上下文语义，他的思路就永远失去了。

多数情况下，你无需细说你是怎样更改 code 的，除非它非常复杂。只需要非常清晰的阐述你为什么这样做，之前的逻辑是怎样的，现在的是怎样的，你为什么用现在的方式。

# 一些建议

## 爱上命令行，摒弃 IDE

Git 子命令非常多，命令行本身非常强大。当然 IDE 里集成的 Git 也不错。我每天使用 IDE （IntelliJ IDEA）, 还有 Eclipse 等，但是我从来没见过哪个 IDE 集成的 Git 可以如 Git 命令行一样便捷强大.

记得当你使用命令行的时候,不管是 Bash 还是 Zsh 还是 Powershell, 都有 Tab 键自动补全的功能，记得去使用他们。

> 译者注：Git 命令行非常强大，强大的同时命令又过于繁杂，学习成本比较高。Windows 和 Mac 下都有很多强大的第三方 Git GUI 软件，比如 Sourcetree 等，可以为我们节省不少学习时间。但是如果在服务器上操作 Git，或者 Linus 发行版又没有好的第三方软件，那就只能依赖命令行了。总之学会 Git 命令行绝对是一件非常酷炫的事儿（只会 fetch, pull, checkout, branch 等基本命令的路过 🥵 ）。

## 阅读 Pro Git

[这本书](https://git-scm.com/book/en/v2)可以在线免费阅读，也有中文版，好好利用它吧~
