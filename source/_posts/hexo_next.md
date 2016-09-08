---
title: Hexo 功能完善
category: Hexo
date: 2013-12-25
---
### 给Hexo添加“上一篇”、“下一篇”导航
我使用的是Hexo的默认主题light，该主题的文章页面没有提供“上一篇”和“下一篇”导航功能，而我又非常喜欢默认主题的清爽简洁，只能自己折腾了。
在Hexo官方文档中变量说明部分我们可以知道，在 Article (post, page, …) 中page.prev和page.next分别代表上一篇和下一篇文章，这两个变量都是一个page对象，所以通过page.prev.path和page.prev.title就可以获取到上一篇文章的相对路径和标题，然后通过修改/themes/light/layout/文件夹中的文件来实现我们的功能。
首先，在/themes/light/layout/_partial/post/文件夹中新建prev_next.ejs文件。
<!-- more -->
prev_next.ejs
```
<% if (page.prev || page.next){ %>
<div class="prev_next clearfix">
  <% if (page.prev){ %>
    <a href="<%- config.root %><%- page.prev.path %>" class="alignleft prev" title="<%= page.prev.title %>"><%= page.prev.title %></a>
  <% } %>
  <% if (page.next){ %>
    <a href="<%- config.root %><%- page.next.path %>" class="alignright next" title="<%= page.next.title %>"><%= page.next.title %></a>
  <% } %>
</div>
<% } %>
```
然后，修改/themes/light/layout/_partial/article.ejs文章模板，找到需要加入“上一篇”和“下一篇”导航功能的位置，加上<%- partial('post/prev_next') %>，比如这样：
```
      <% } else { %>
        <%- partial('post/category') %>
        <%- partial('post/tag') %>
        <%- partial('post/share') %>
        <%- partial('post/prev_next') %>
      <% } %>
      <div class="clearfix"></div>
    </footer>
  </div>
</article>

<%- partial('comment') %>
```
最后，在/themes/light/source/css/_partial/article.styl文件末尾添加上相应的CSS。
```
.prev_next
  margin 1em 0
  clear both
  overflow hidden
  a
    display block
    float left
    width 50%
    background #dbdbdb
    text-align center
    padding 0.4em 0
    color #1ba1e2
    transition background .45s color .45s
    &:hover
      color #fafafa
      background #717171
    &.prev::before
      content "上一篇："
      padding-right 0.5em
    &.next::before
      content "下一篇："
      padding-right 0.5em
```
### 给博客和文章添加keywords
---
默认情况下博客和文章是没有关键字的，可以安装如下方法修改。
首先，在博客配置文件/Hexo/_config.yml中添加keywords:字段，关键字以英文,分割，如下:
```
# Site
title: typeof this  #站点名
subtitle:    # 副标题
description: # 站点描述，搜索引擎
author: typeof
email: xxx@admin.com
language: zh-CN
keywords: Web,前端,JavaScript,html5,css3 # 博客关键字
```
然后，修改模板文件，我用的是light模板，修改/themes/light/layout/_partial/head.ejs
```
#删除下面这行
<% if (page.keywords){ %><meta name="keywords" content="<%= page.keywords %>"><% } %>
#增加以下内容
<% if (page.keywords){ %>
<meta name="keywords" content="<%= page.keywords %>,<%= config.keywords %>">
<% } else if (config.keywords){ %>
<meta name="keywords" content="<%= config.keywords %>">
<%} %>
```
简单说明一下：如果页面有关键字，则用页面的关键字加上配置文件里面的关键字，如果没有关键字，则用配置文件的关键字。
要给文章添加关键字，只需要在文章里面加入keywords:即可。也可以直接修改创建文章的模板/scaffolds/post.md，在最下面添加keywords:，如下：
```
title: {{ title }}
date: {{ date }}
updated: {{ date }}
tags:
categories:
keywords:
```
### 添加自定义widget
---
添加widget的方法很简单，首先在/themes/light/layout/_widget/文件夹中创建widget文件your_widget.ejs，然后在主题配置文件中加载你的widget，下面通过创建一个友情链的widget来看看具体操作。
友情链接包含连接名称和连接地址两个属性，看到有的教程中把友情连接直接静态地写在widget中，修改起来不方便，所以要借助主题配置文件_config.yml和Hexo提供的访问配置文件的对象theme。
首先，在/themes/flight/_config.yml文件中添加如下节。
### 友情链接
```
blogrolls: #友情链接
  - Hexo: http://hexo.io
  - Hexo Document: http://hexo.io/docs
  - github: https://github.com/
  - jQuery: http://jquery.com/
```
然后，在/themes/light/layout/_widget/文件夹中创建友情连接widget文件blogroll.ejs，内容可以这样：
```
<% if (theme.blogrolls && theme.blogrolls.length > 0) { %>
<div class="widget tag">
  <h3 class="title"><%= __('blogroll') %></h3>
  <ul class="entry">
  <% theme.blogrolls.forEach(function(item){ %>
    <%
      var description, linkURL
      for (var tmp in item) {
        description = tmp;
        linkURL = item[tmp];
      }
    %>
    <li><a href="<%- linkURL %>" target="_blank"><%= description %></a></li>
  <% }); %>
  </ul>
</div>
<% } %>
```
最后，修改主题配置文件/themes/flight/_config.yml，在widgets下增加blogroll。如下：
```
widgets:
  - category
  - tag
  - recent_posts
  - blogroll
```
友情连接widget就创建好了，hexo server，在本地http://localhost:4000/查看效果吧！同样，我们可以使用此方法添加个人说明，微博秀等widget。
添加最新评论widget
首先需要声明的是本屌用的是多说评论系统，所以最新评论widget也是利用多说提供的API来实现，上一节已经分享了怎么创建自定义的widget，现在我们按照上面的方法来一步一步实现该widget。
在/themes/light/layout/_widget/文件夹下创建最新评论小挂件recent_comments.ejs，内容如下：
```
<% if (theme.duoshuo){ %>
<div class="widget recent_comments">
  <h3 class="title"><%= __('recent_comments') %></h3>
  <ul class="entry ds-recent-comments" data-num-items="5" data-show-avatars="0" data-show-title="1" data-show-time="1"></ul>
  <script type="text/javascript">
  if(typeof duoshuoQuery === 'undefined'){
    var duoshuoQuery = {short_name:"<%- theme.duoshuo %>"};
    (function() {
      var ds = document.createElement('script');
      ds.type = 'text/javascript';ds.async = true;
      ds.src = 'http://static.duoshuo.com/embed.js';
      ds.charset = 'UTF-8';
      (document.getElementsByTagName('head')[0]
      || document.getElementsByTagName('body')[0]).appendChild(ds);
    })();
  }
  </script>
</div>
<% } %>
```
简单说明，在每个页面中，如果使用多个多说控件，只需要添加一次多说js，所以这里有这样的判断if(typeof duoshuoQuery==='undefined')，在需要用到多说的位置都加上这个判断，避免多次加载js文件；另外多说评论相关的参数：data-num-items显示的评论条数，data-show-avatars是否显示用户头像，data-show-title是否显示文章标题，data-show-time是否显示评论时间，具体参数说明可以参考多说官方说明文档，按照官方文档还可以添加多说最近访客小部件和热评文章小部件。
