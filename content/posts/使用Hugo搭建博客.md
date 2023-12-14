---
title: '使用 Hugo 搭建博客'
date: '2023-12-04 21:02:11'
---

1. 下载 Hugo

   https://github.com/gohugoio/hugo

   ```bash
   tar xvf hugo_extended_0.120.4_linux-amd64.tar.gz
   sudo mv hugo /usr/local/bin
   ```

2. 创建站点

   ```bash
   sudo chmod o+w /usr/local/nginx/html
   cd /usr/local/nginx/html
   hugo new site blog --format yaml
   ```

3. 下载主题

   ```bash
   cd /usr/local/nginx/html/blog/themes
   git clone https://gitee.com/lzhuii/hugo-PaperMod.git PaperMod
   ```

4. 修改配置文件

   ```bash
   vim /usr/local/nginx/html/blog/hugo.yaml
   ```

   ```yaml
   baseURL: "https://lzhui.top/"
   title: 临渊羡鱼
   paginate: 5
   theme: PaperMod
   languageCode: zh-cn
   defaultContentLanguage: zh
   timeZone: Asia/Shanghai
   
   enableRobotsTXT: true
   buildDrafts: false
   buildFuture: false
   buildExpired: false
   cleanDestinationDir: true
   
   googleAnalytics: UA-123-45
   
   minify:
     disableXML: true
     minifyOutput: true
     
   outputs:
     home:
       - HTML
       - RSS
       - JSON # is necessary
       
   params:
     env: production # to enable google analytics, opengraph, twitter-cards and schema.
     title: 临渊羡鱼
     description: "一个正在路上的程序猿"
     keywords: [Blog, Portfolio, PaperMod]
     author: 临渊羡鱼
     images: ["<link or path of image for opengraph, twitter-cards>"]
     DateFormat: "2006年01月02日 15:04:05"
     defaultTheme: auto # dark, light
     disableThemeToggle: false
   
     ShowReadingTime: true
     ShowShareButtons: true
     ShowPostNavLinks: true
     ShowBreadCrumbs: true
     ShowCodeCopyButtons: false
     ShowWordCount: true
     ShowRssButtonInSectionTermList: true
     UseHugoToc: true
     disableSpecial1stPost: false
     disableScrollToTop: false
     comments: false
     hidemeta: false
     hideSummary: false
     showtoc: false
     tocopen: false
   
     assets:
       # disableHLJS: true # to disable highlight.js
       # disableFingerprinting: true
       favicon: "<link / abs url>"
       favicon16x16: "<link / abs url>"
       favicon32x32: "<link / abs url>"
       apple_touch_icon: "<link / abs url>"
       safari_pinned_tab: "<link / abs url>"
   
     label:
       text: "临渊羡鱼"
       icon: /apple-touch-icon.png
       iconHeight: 35
   
     # profile-mode
     profileMode:
       enabled: true # needs to be explicitly set
       title: 临渊羡鱼
       subtitle: "一个正在路上的程序猿"
       imageUrl: "img/avatar.jpg"
       imageWidth: 120
       imageHeight: 120
       imageTitle: 
       buttons:
         - name: 文章
           url: posts
         - name: 标签
           url: tags
         - name: 时间轴
           url: archives
   
     # home-info mode
     homeInfoParams:
       Title: '临渊羡鱼'
       Content: '一个正在路上的程序猿'
   
     socialIcons:
       - name: github
         url: "https://github.com/lzhuii"
   
     cover:
       hidden: true # hide everywhere but not in structured data
       hiddenInList: true # hide on list pages and home
       hiddenInSingle: true # hide on single page
   
     # for search
     # https://fusejs.io/api/options.html
     fuseOpts:
       isCaseSensitive: false
       shouldSort: true
       location: 0
       distance: 1000
       threshold: 0.4
       minMatchCharLength: 0
       keys: ["title", "permalink", "summary", "content"]
   
   menu:
     main:
       - identifier: tags
         name: 标签
         url: /tags/
         weight: 1
       - identifier: archives
         name: 时间轴
         url: /archives/
         weight: 2
       - identifier: search
         name: 搜索
         url: /search/
         weight: 3
         
   # Read: https://github.com/adityatelange/hugo-PaperMod/wiki/FAQs#using-hugos-syntax-highlighter-chroma
   pygmentsUseClasses: true
   markup:
     highlight:
       noClasses: false
       # anchorLineNos: true
       # codeFences: true
       guessSyntax: true
       lineNos: true
       style: monokai
   ```
   
5. 编写文章

   /usr/local/nginx/html/blog/content/posts 目录下上传 markdown 文档

6. 生成网站

   ```bash
   cd /usr/local/nginx/html/blog
   hugo
   ```


7. 安装foottools，配置字体

   ```bash
   sudo mkdir -p /usr/share/fonts
   # sftp上传字体文件 LXGWWenKai-Regular.ttf
   
   pip install fontTools
   # 提取字体
   find /usr/local/nginx/html/blog/public -type f -name "*[html,css,js,json,txt]" | xargs cat >> tmp.txt
   fonttools subset "/usr/share/fonts/LXGWWenKai-Regular.ttf" --text-file="tmp.txt" --output-file="/usr/local/nginx/html/blog/public/fonts/LXGWWenKai-Regular.ttf"
   ```

   