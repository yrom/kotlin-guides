---
layout: page
title: Contribute
site_nav_category: contribute
site_nav_category_order: 400
is_site_nav_category: true
---

欢迎并感谢您参与进来为本站供献内容！

供献内容时，如是小的改动可以直接发起 pull request；如是大型改动，我们建议先通过[issue tracker](https://github.com/android/kotlin-guides/issues) 发起一个讨论。

Pull request 需指向 [GitHub 仓库](https://github.com/android/kotlin-guides)上的 `master` 分支。 
每过几周，所有的改动都会统一发布到`gh-pages`分支，[“变更日志”](changelog.html)页面也会随之更新。

**我们期待着您的参与！**


## 开发

你在做改动时，可以在你自己的电脑上运行这个网站

### 配置 Ruby 和 Bundler

首先请确认你安装了 Ruby 和 [Bundler](http://bundler.io/)。

    gem install bundler

### 一次性配置

    bundle install --path vendor/bundle

_注意: 如果你是 Mac OS 系统，安装 nokogiri 时发生错误，你可以先手动执行命令 `brew unlink xz`，再重试安装，然后再执行 `brew link xz`。_

### 运行网站

    bundle exec jekyll serve

用浏览器打开 [http://127.0.0.1:4000/kotlin-guides/](http://127.0.0.1:4000/kotlin-guides/)。
