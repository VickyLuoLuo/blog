# READ ME

> Ported Theme of [Hux Blog](https://github.com/Huxpro/huxpro.github.io), Thank [Huxpro](https://github.com/Huxpro) for designing such a flawless theme.
>
> This LiveMyLife theme created by [Vincent](https://wvincen.gitee.io/) modified from the original Porter [YenYuHsuan](https://github.com/YenYuHsuan/hexo-theme-beantech) , refer to the Themes of [dusign](https://github.com/dusign/hexo-theme-snail)、[Utone](https://github.com/shixiaohu2206/hexo-theme-huhu), Thanks [dusign](https://github.com/dusign/hexo-theme-snail)、[Utone](https://github.com/shixiaohu2206/hexo-theme-huhu).
>   

## Quick Start

### Install Node.js and Git
```shell
#For Mac
brew install node
brew install git
```
> Windows: Download & install Node.js. -> [Node.js](https://nodejs.org/zh-cn/download/)
>
> Windows: Download & install Git. -> [Git](https://git-scm.com/download/win)

### Install Hexo
```shell
$ npm install -g hexo-cli
```
> What is [Hexo](https://hexo.io/docs/)?
>
> Hexo is a fast, simple and powerful blog framework. You write posts in Markdown (or other markup languages) and Hexo generates static files with a beautiful theme in seconds.

### Init
```shell
git clone https://gitee.com/vickyluo/blog.git # clone this product
npm install
```

### Start the Server
```shell
hexo generate # or hexo g
hexo server   # or hexo s
```
Starts a local server. By default, this is at `http://localhost:4000/`.
> More Commands -> [Hexo Commands](https://hexo.io/docs/commands)

## Configuration
Modify `_config.yml` file with your own info, Especially the section:

### Site
Replace the following information with your own.
```yml
# Site
title: Live My Life
subtitle: 淡而无味也是一种味道
author: Mr.Vincent
language: zh-CN
timezone:
```

## Hexo Basics

### CDN Settings

JsDelivr is A free CDN for Open Source fast、reliable and automated. If Github Pages deploy，you can config CDN settings. The images of the Hexo-theme-livemylife has added JsDelivr CDN Setting. How to use Jsdelivr? -> Docs：[免费 CDN 提速 Github 静态资源访问](https://v-vincen.life/2020/07/15/Github-%E5%8A%A0%E9%80%9F%E4%BC%98%E5%8C%96/#%E5%85%8D%E8%B4%B9-CDN-%E6%8F%90%E9%80%9F-Github-%E9%9D%99%E6%80%81%E8%B5%84%E6%BA%90%E8%AE%BF%E9%97%AE)
```yml
# CDN Setting
# Docs: https://www.jsdelivr.com/?docs=gh
# If Github Pages deploy，you can ues jsdelivr settings
#
jsdelivr:
  jsdelivr_url: https://cdn.jsdelivr.net/gh/
  github_username: V-Vincen
```

### Site Settings
Put customized pictures in img directory.
```yml
# Site settings
SEOTitle: dream big
email: 
description: ""
keyword: ""
header-img: img/header_img/newhome_bg.jpg
archives-img: img/header_img/archive_bg.jpg
```

### Favicon Settings
```yml
favicon: img/avatar/favicon.jpg
```

### Signature Settings
Copy your signature image to `<root>/img/signature` and modify the `_config.yml`.
```yml
signature: true   # show signature
signature-img: img/signature/<your-signature>
```
> How to create signature -> [Free Online Signature](https://fontmeme.com/signature-fonts/)

### Wave Settings
```yml
# Wave settings
wave: true
```

### SNS Settings
If you don’t want to display it, you can delete it directly.
```yml
# SNS settings
# RSS: true
# github_username: V-Vincen
# twitter_username: V_Vincen_
# instagram_username: V_Vincen_
# facebook_username:  yourAccount
# linkedin_username:  yourAccount
# zhihu_username: yourAccount
# weibo_username: WVincen
```

### Sidebar Settings
Copy your avatar image to `<root>/img/avatar` and modify the `_config.yml`:
```yml
sidebar: true   # whether or not using Sidebar.
sidebar-about-description: ""
sidebar-avatar: img/avatar/vincnet.jpg    # use absolute URL, seeing it's used in both `/` and `/about/`
widgets:
- visitor   # busuanzi: https://busuanzi.ibruce.info/
- featured-tags
- short-about
- recent-posts
- friends-blog
- archive
- category

# widget behavior
## Archive
archive_type: 'monthly'
show_count: true

## Featured Tags
featured-tags: true   # whether or not using Feature-Tags
featured-condition-size: 0    # A tag will be featured if the size of it is more than this

## Friends
friends: [
    {
        title: "",
        href: ""
    },{
        title: "",
        href: ""
    }
]
```

### Comment Settings
Hexo-Theme-LiveMyLife temporarily supports three Comments. I use gitalk comment system.

#### Gitalk
Gitalk is a modern comment component based on GitHub Issue and Preact. See [Gitalk](https://github.com/gitalk/gitalk) for detailed configuration method.
```yml
# Gitalk Settings
# Doc:https://github.com/gitalk/gitalk/blob/master/readme-cn.md
gitalk:
  owner:                          # 'GitHub repo owner'
  admin:                          # 'GitHub repo'
  repo:                           # ['GitHub repo owner and collaborators, only these guys can initialize github issues']
  clientID:                       # 'GitHub Application Client ID'
  clientSecret:                   # 'GitHub Application Client Secret'
  perPage: 10                     # Pagination size, with maximum 100.
  pagerDirection: last            # Comment sorting direction, available values are last and first.
  createIssueManually: false      # By default, Gitalk will create a corresponding github issue for your every single page automatically when the logined user is belong to the admin users. You can create it manually by setting this option to true
  language: en                    # Localization language key, en, zh-CN and zh-TW are currently available.
  maxCommentHeight: 250           # An optional number to limit comments' max height, over which comments will be folded.Default 250.
```

#### Gitment
Gitment is a comment system based on GitHub Issues, which can be used in the frontend without any server-side implementation. See [Gitment](https://github.com/imsun/gitment) for detailed configuration method.
```yml
## Gitment Settings
## Doc: https://github.com/imsun/gitment
gitment:
  owner:                          # Your GitHub ID. Required.
  repo:                           # The repository to store your comments. Make sure you're repo's owner. Required.
  client_id:                      # GitHub client ID. Required.
  client_secret:                  # GitHub client secret. Required.
  desc:                           # An optional description for your page, used in issue's body. Default ''.
  perPage: 10                     # An optional number to which comments will be paginated. Default 20.
  maxCommentHeight: 250           # An optional number to limit comments' max height, over which comments will be folded. Default 250.
```

#### Disqus
If you want use [Disqus](https://disqus.com/), you must have a circumvention (proxy, clime over the firewall) technology.
```yml
# Disqus settings
disqus_username: your-disqus-ID
```


### Analytics Settings
How to config Analytics? -> Docs:[Analytics and Sitemap Settings](https://v-vincen.life/2020/04/17/Hexo-Theme-LiveMyLife/#Analytics-Settings)
```yml
# Analytics settings
# Google Analytics
ga_track_id: UA-xxxxxx-xx   # Format: UA-xxxxxx-xx

# Baidu Analytics
ba_track_id: ba_track_id
```

### Sitemap Settings
How to config Sitemap? -> Docs:[Analytics and Sitemap Settings](https://v-vincen.life/2020/04/17/Hexo-Theme-LiveMyLife/#Analytics-Settings)
```yml
# Google sitemap
sitemap:
  path: sitemap.xml

# Baidu sitemap
baidusitemap:
  path: baidusitemap.xml
  baidu_push: true
```


### Go to top icon Setup
My icon is using point, you can change to your own icon at `sourcre/css/images`.

### Post tag
You can decide to show post tags or not.
```yml
home_posts_tag: true
```

### Markdown render
My markdown render engine plugin is [hexo-renderer-markdown-it](https://github.com/celsomiranda/hexo-renderer-markdown-it).
```yml
# Markdown-it config
## Docs: https://github.com/celsomiranda/hexo-renderer-markdown-it/wiki
markdown:
  render:
    html: true
    xhtmlOut: false
    breaks: true
    linkify: true
    typographer: true
    quotes: '“”‘’'
```

### Anchorjs Settings
And if you want to change the header anchor '❡', you can go to `layout/_partial/anchorjs.ejs` to change it. How to use anchorjs, see [AnchorJS](https://www.bryanbraun.com/anchorjs/#examples) for detailed examples.
```yml
# Anchorjs Settings
anchorjs: true    # if you want to customize anchor. check out line:26 of `layout/_partial/anchorjs.ejs`
```

```javascript
async("//cdn.bootcss.com/anchor-js/1.1.1/anchor.min.js",function(){
        anchors.options = {
          visible: 'hover',
          placement: 'left',
          icon: '❡'
          // icon: 'ℬ'
        };
        anchors.add().remove('.intro-header h1').remove('.subheading').remove('.sidebar-container h5');
    })
```

### Article Top
```yml
# article top
top: true
```
Hexo-theme-livemylife has added the article top function, just add `top: number` configuration to your markdown notes, articles are sorted by this number.

### WordCount Settings
A Word Count Plugin for Hexo. See [WordCount](https://github.com/willin/hexo-wordcount) for detailed configuration method.
```yml
# Dependencies: https://github.com/willin/hexo-wordcount
# Docs: https://www.npmjs.com/package/hexo-wordcount
wordcount: true
```

### Top scroll progress
```yml
# top scroll progress
scroll: true
```

### Tip
```yml
tip:
  enable: true
  copyright: Say what you think...
```

### Social Share Post
```yml
#Docs: https://github.com/overtrue/share.js
share: true
```

### Viewer Config
Viewer is a simple jQuery image viewing plugin. Let us first look at a [demo](https://fengyuanchen.github.io/viewer/). See [Viewer](https://github.com/fengyuanchen/viewer) for detailed configuration. If you want to modify the [options](https://github.com/fengyuanchen/viewerjs#options) of Viewer, you can go to `sourcre/js/viewer/pic-viewer.js` to change it.
```yml
# Viewer config
viewer: true
```

### Theme Color Config
Hexo-Theme-LiveMyLife temporarily supports two themes color.
```yml
themecolor: true
```

### Search Settings
```yml
# Dependencies: https://github.com/V-Vincen/hexo-generator-zip-search
search:
  enable: true
  path: search.json
  zipPath: search.flv
  versionPath: searchVersion.json
  field: post
  # if auto, trigger search by changing input
  # if manual, trigger search by pressing enter key or search button
  trigger: auto
  # show top n results per article, show all results by setting to -1
  top_n_per_article: 1
```

### Gitter
Gitter is a chat and network platform that helps manage, develop and connect communities through messages, content and discovery.See [Gitter](https://gitter.im/) for detailed configuration method.
```yml
## Docs:https://gitter.im/?utm_source=left-menu-logo
##
gitter:
  room: your-community/your-room
```

### Deployment
Replace to your own repo!
```yml
deploy:
  type: git
  repo: https://github.com/<yourAccount>/<repo> # or https://gitee.com/<yourAccount>/<repo>
  branch: <your-branch>
```

## Hexo Basics

Some hexo command:

```bash
hexo new post "<post name>"   # you can change post to another layout if you want
hexo clean && hexo generate   # generate the static file
hexo server   # run hexo in local environment
hexo deploy   # hexo will push the static files automatically into the specific branch(gh-pages) of your repo!
```

## Have fun ^\_^

Please [Star](https://github.com/V-Vincen/hexo-theme-livemylife) this Project if you like it! [Follow](https://github.com/V-Vincen) would also be appreciated! Peace!

