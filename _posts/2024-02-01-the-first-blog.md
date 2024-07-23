---
title: 第一篇博客
author: Enokiy
date: 2024-02-01 
categories: [其他]
tags: [git-page,jekyll]
pin: true
---

倒腾许久，我的博客终于开张了~~

整个架子使用github-page、Jekyll完成，用了Jekyll现有的模板[jekyll-theme-chirpy](https://github.com/cotes2020/jekyll-theme-chirpy)，不太习惯原来的页面比例，修改了一下页面比例。

记录一下搭建的大概步骤：

1. 
    使用jekyll-theme-chirpy模板需要仔细阅读[使用说明](https://chirpy.cotes.page/posts/getting-started/)，使用说明里面的用Chirpy Starter生成仓库或者直接fork[jekyll-theme-chirpy](https://github.com/cotes2020/jekyll-theme-chirpy)两种方法都可以，推荐使用[ Chirpy Starter](https://github.com/cotes2020/chirpy-starter)生成。

2. 
    使用[ Chirpy Starter](https://github.com/cotes2020/chirpy-starter)生成名为[username].github.io软件仓之后，在对应仓的settings-->Code and automation-->Pages中进行配置部署:

    ![](/assets/images/the-first-blog/20240206143728.png)

3. 
    到这里访问[username].github.io应该会有初步的博客的架子了，然后clone仓库到本地开始修改_config.yml，添加自己的文章，但是添加了markdown文章之后再push到github上，发现博客起不起来了，通过github仓的actions可以看到具体的报错信息：
  ![](/assets/images/the-first-blog/20240206144708.png)

    排错不可能每次都改一下推上去再看github的actions到底报了什么错，效率太低了，所以需要在本地搭建环境。基于[Jekyll的指导](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll)安装Jekyll, 我是在Ubuntu下:

    ```console
    # Install Ruby and other prerequisites:
    sudo apt-get install ruby-full build-essential zlib1g-dev

    #  set up a gem installation directory for your user account
    echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
    echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
    echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
    source ~/.bashrc

    #  install Jekyll and Bundler
    gem install jekyll bundler
    ```

    在本地仓目录执行 `bundle install`安装仓库所需依赖gems之后执行`jekyll s -H 0.0.0.0 -P 8888  -I -t`启用本地服务就可以预览了。

## 遇到的坑点

1. clone下来的仓中的Gemfile文件不要修改，最开始的时候直接fork了jekyll-theme-chirpy，有个依赖版本冲突了，就把Gemfile里面的版本改了，改完push到github发现博客起不来，原因是github支持的jekyll版本有限，参考[依赖版本](https://pages.github.com/versions/)。

2. 通过[ Chirpy Starter](https://github.com/cotes2020/chirpy-starter)生成的仓库，缺少assets/下面的静态文件

> （又是一个没有仔细阅读使用说明的锅），通过[chirpy静态文件](https://github.com/cotes2020/chirpy-static-assets#readme)解决，这里一开始通过git submodule init的方式更新了assets/lib之后把代码推到github上发现部署不起来，报错assets/lib/jquerys等文件找不到，后面又git submodule  deinit --all -f 之后再推代码又部署成功了，没有特别理解原理。）


3. 通过[ Chirpy Starter](https://github.com/cotes2020/chirpy-starter)生成的仓库，模板不支持中文

> 我的解决方法是把jekyll-theme-chirpy中的_data、_includes、_plugins、_sass、_tabs都拷贝了过来，然后在_config.yml中把lang修改为zh-CN。

4. 报错：internal script reference /assets/js/dist/app.min.js does not exist：

> 解决办法: 把本地生成的_site/app.js拷贝到/assets/js/dist/app.min.js中，然后去掉.gitignore中的`assets/js/dist`前面的注释。

5. 报错：` At _site/index.html:1:'link' tag is missing a reference `

> 原因是因为我为了修改markdown的渲染css,引入了assets/css/sytax.css，然后在head.html中添加了`  <link rel="stylesheet" href="assets/css/syntax.css">` ,写成了相对地址，找不到这个css文件。后面发现修改过的markdown渲染好像在github上不生效，所以又把这个css删了，然后问题也就解决了。

6. 开始提到不习惯原来模板的页面比例，就调整了一下，方法是把本地生成的/_site/assets/css/jekyll-theme-chirpy.css拷贝到/assets/css/jekyll-theme-chirpy.css，然后修改了其中的

```css
#main-wrapper {
    position: relative;
    padding-left: 0;
    padding-right: 0;
    margin-left: 15%;  /*  新增 */
    margin-top: 2%; /*  新增 */
}

#main-wrapper>.container {
    min-height: 100vh;
    margin-left: 1%;  /*  新增 */
    min-width:100% /*  新增 */
}
```

7. 原样式对于markdown中的图片是居中显示的，当图片比较小的时候居中显示看着有点难受，决定把图片修改为靠左显示，方法如下：修改_includes/refactor-content.html文件，在原来\<img\>标签外的\<a\>标签外面加了一层\<p style="display:inline-block;text-align: left;"\>，**注意后面的`\</p`没有`\>`**,修改完的代码如下:

![html](/assets/images/the-first-blog/html.png)

8. 修改完全部之后在本地使用以下命令可以先检查一下有没有报错，没有报错就可以推代码了。

```console
bundle exec htmlproofer _site \
    \-\-disable-external=true \
    \-\-ignore-urls "/^http:\/\/127.0.0.1/,/^http:\/\/0.0.0.0/,/^http:\/\/localhost/"
```

9. md文件里面的图片暂时先放到了assets静态文件目录下了，对于github仓库1G的限制，这样放可能不太合理，后续找一个合适的图床平台再来修改。


就酱紫吧~ 开门大吉~~
