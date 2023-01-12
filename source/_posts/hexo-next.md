---
title: hexo-next
date: 2023-01-12 15:56:50
categories:
- 笔记
tags:
- hexo
- next
---

### 背景
之前博客许久未打理，本地的node环境和hexo版本也比较陈旧，删除了github旧仓库重新跑了一遍，做下记录以便日后参考

### 参考文档
> [hexo文档](https://hexo.io/zh-cn/docs/)

> [next主题文档](https://theme-next.js.org/docs/getting-started/)

### 流程
1. 安装hexo及项目初始化
    
    ```bash
    $: npm install -g hexo-cli # 安装hexo
    
    $: hexo init my_project # 初始化项目
    ```
    
2. 项目配置
    
    [配置](https://hexo.io/zh-cn/docs/configuration)
    
3. 安装next主题
    
    [Getting Started](https://theme-next.js.org/docs/getting-started/)
    
    此处使用git方式下载的包在生成静态文件时index.html中包含模板标签，访问有问题，暂时没找到原因，所以使用了npm方式安装：
    
    ```bash
    $: cd my_project && npm install hexo-theme-next # 安装next主题
    $: cp -rp node_modules/hexo-theme-next themes/next # 此处将npm包拷贝到项目的themes目录，方便主题配置文件的版本管理
    $: npm uninstall hexo-theme-next
    ```
<!-- more -->

4. 启用next主题
    
    ```yaml
    # 项目的_config.yml
    # Extensions
    ## Plugins: https://hexo.io/plugins/
    ## Themes: https://hexo.io/themes/
    theme: next
    ```
    
5. 主题自定义配置
    
    [开始使用](https://theme-next.iissnan.com/getting-started.html)
    
6. 第三方插件
    1. 字数统计
        
        [字数统计](https://hexo-next.readthedocs.io/zh_CN/latest/next/advanced/%E5%AD%97%E6%95%B0%E7%BB%9F%E8%AE%A1/)
        
    2. 本地搜索
        
        [第三方服务集成](https://theme-next.iissnan.com/third-party-services.html#local-search)
        
    3. Disqus评论插件
        
        [第三方服务集成](https://theme-next.iissnan.com/third-party-services.html#disqus)
        
    4. Waline评论及阅读数统计插件
        
        [快速上手](https://waline.js.org/guide/get-started/)
        
        1. 根据上面文档指引，创建并且成功部署Vercel，*HTML 引入 (客户端)部分*不用做
        2. hexo安装waline扩展并且在主题中进行配置
            
            `$:npm install @waline/hexo-next`
            
            ```yaml
            waline:
              # New! Whether enable this plugin
              enable: true
            
              # Waline server address url, you should set this to your own link
              serverURL: https://your-domain.vercel.app # 替换为你自己的域名
            
              # Waline library CDN url, you can set this to your preferred CDN
              libUrl: https://unpkg.com/@waline/client@v2/dist/waline.js
            
              # Waline CSS styles CDN url, you can set this to your preferred CDN
              cssUrl: https://unpkg.com/@waline/client@v2/dist/waline.css
            
              # Custom locales
              # locale:
              #   placeholder: Welcome to comment # Comment box placeholder
            
              # If false, comment count will only be displayed in post page, not in home page
              commentCount: true # 开启评论数展示
            
              # Pageviews count, Note: You should not enable both `waline.pageview` and `leancloud_visitors`.
              pageview: true # 开启阅读数展示
            
              # Custom emoji
              emoji:
              #   - https://unpkg.com/@waline/emojis@1.0.1/weibo
              #   - https://unpkg.com/@waline/emojis@1.0.1/alus
                - https://unpkg.com/@waline/emojis@1.0.1/bilibili
                - https://unpkg.com/@waline/emojis@1.0.1/qq
              #   - https://unpkg.com/@waline/emojis@1.0.1/tieba
              #   - https://unpkg.com/@waline/emojis@1.0.1/tw-emoji
            
              # Comment infomation, valid meta are nick, mail and link
              # meta:
              #   - nick
              #   - mail
              #   - link
            
              # Set required meta field, e.g.: [nick] | [nick, mail]
              # requiredMeta:
              #   - nick
            
              # Language, available values: en-US, zh-CN, zh-TW, pt-BR, ru-RU, jp-JP
              lang: zh-CN
            
              # Word limit, no limit when setting to 0
              # wordLimit: 0
            
              # Whether enable login, can choose from 'enable', 'disable' and 'force'
              # login: enable
            
              # comment per page
              pageSize: 10
            ```
            
        3. 开启了**pageview**的话需要在leancloud应用的结构化数据中创建`Counter`这个Class，本质上阅读数存储在`Counter` 表中，评论数据存储在`Comment` 表中
    5. ~~leancloud阅读次数~~
        
        (Waline可以做)此路不通，不管是国内还国外的云函数部署都需要绑定独立的域名才能使用，暂时没有多余的域名
        
    
7. 增加标签页面和分类页面
    
    [主题配置](https://theme-next.iissnan.com/theme-settings.html#tags-page)
    
    [主题配置](https://theme-next.iissnan.com/theme-settings.html#categories-page)
    
    这样可以在主题的menu配置中加上这两个页面，不然`public/tags`和`public/categories`文件夹不会生成`index.html`

8. github-page配置
    [Github Page](https://docs.github.com/zh/pages/getting-started-with-github-pages/creating-a-github-pages-site)

9. 项目部署配置
    [Hexo一键部署](https://hexo.io/zh-cn/docs/github-pages#%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2)