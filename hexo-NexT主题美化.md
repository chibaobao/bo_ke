---
title: hexo-NexT主题美化
date: 2016-10-12 00:40:34
tags: hexo
categories: 博客搭建
---
# hexo主题NexT美化

## 配置安装
[NexT首页](http://theme-next.iissnan.com/)
- 下载
	1. 切换到hexo的目录
	2. 执行
		`` git clone https://github.com/iissnan/hexo-theme-next themes/next``

- 配置hexo配置文件_config.yml
	theme: nex	
## 主题美化
- 样式选择
	```
	#scheme: Muse
   #scheme: Mist
   scheme: Pisces
   ```

- 设置标签，归档，分类
	修改配置文件并执行相应命令生成
	都是分3步完成

		1. hexo new page tags
		2. 修改source/tags/index.md中加一行内容 type: "tags"
		3. 配置文件中 menu:下的tags: /tags去掉注释

		1. hexo new page archives
		2. 修改source/archives/index.md中加一行内容 type: "archives"
		3. 配置文件中 menu:下的archives: /archives去掉注释

		1. hexo new page categories
		2. 修改source/categories/index.md中加一行内容 type: "categories"
		3. 配置文件中 menu:下的categories: /categories去掉注释

	```
	到此配置结束
	完成以上操作后每次new一个file后
	---
	title: 题目
	date: 2016-10-09 18:44:46
	tags: 标签
	categories: 分类
	---
	网页中就会自动出现相应的tags(标签)，categories(分类)内容
	```

- 设置头像
	avatar:  /images/tux.png（source下的images目录）

- 设置网页图标
	favicon:  /images/tux.png（source下的images目录）

- 侧边栏社交链接
	```
	# Social links
	social:
	GitHub: https://github.com/your-user-name
 	```

- 设置首页预览效果,而不是全文显示
	```
	auto_excerpt:
	enable: true
	length: 150
	```
	
- 设置多说评论
    - [注册多说](http://duoshuo.com/) 
    - 在首页选择我要安装
    - 配置多说
    - 在主题（next）目录查找comments.***文件，改为一下内容
 
    ```
      <section id="comments">
    <!-- 多说评论框 start -->
    <div class="ds-thread" data-thread-key="<%= post.layout %>-<%= post.slug %>" data-title="<%= post.title %>" data-url="<%= page.permalink %>"></div>
    <!-- 多说评论框 end -->
    <!-- 多说公共JS代码 start (一个网页只需插入一次) -->
    <script type="text/javascript">
    var duoshuoQuery = {short_name:'<%= config.duoshuo_shortname %>'};
      (function() {
        var ds = document.createElement('script');
        ds.type = 'text/javascript';ds.async = true;
        ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
        ds.charset = 'UTF-8';
        (document.getElementsByTagName('head')[0] 
         || document.getElementsByTagName('body')[0]).appendChild(ds);
      })();
      </script>
    <!-- 多说公共JS代码 end -->
  </section>


    ```