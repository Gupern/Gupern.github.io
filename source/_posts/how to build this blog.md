---
title: how to build this blog
date: 2021-12-13 00:19:18
tags:
---


# installation

## install hexo 
`npm install -g hexo-cli`

## initial blog project 
`hexo init blog`

## install theme
`cd your-hexo-site`
`npm install hexo-theme-next`

## install photo plugin
`npm install hexo-asset-image --save`

## setting hexo ./_config.yml
```
theme: next
post_asset_folder: true
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git
  repo: git@github.com:Gupern/gupern.github.io.git
  branch: main
```

<!-- more --> 

## setting theme next ./themes/next/_config.yml
```
scheme: Gemini
# delete space between / and ||, if you did not do it, 'home' button will not be ok
menu:
  home: /|| home
  #about: /about/|| user
  #tags: /tags/|| tags
  #categories: /categories/|| th
  archives: /archives/|| archive
  #schedule: /schedule/|| calendar
  #sitemap: /sitemap.xml|| sitemap
  #commonweal: /404/|| heartbeat


# Automatically Excerpt. Not recommend.
# Please use <!-- more --> in the post to control excerpt accurately.
auto_excerpt:
  enable: auto
  length: 100
```


## change /node_modules/hexo-asset-image/index.js
```js
'use strict';
var cheerio = require('cheerio');

// http://stackoverflow.com/questions/14480345/how-to-get-the-nth-occurrence-in-a-string
function getPosition(str, m, i) {
  return str.split(m, i).join(m).length;
}

var version = String(hexo.version).split('.');
hexo.extend.filter.register('after_post_render', function(data){
  var config = hexo.config;
  if(config.post_asset_folder){
        var link = data.permalink;
    if(version.length > 0 && Number(version[0]) == 3)
       var beginPos = getPosition(link, '/', 1) + 1;
    else
       var beginPos = getPosition(link, '/', 3) + 1;
    // In hexo 3.1.1, the permalink of "about" page is like ".../about/index.html".
    var endPos = link.lastIndexOf('/') + 1;
    link = link.substring(beginPos, endPos);

    var toprocess = ['excerpt', 'more', 'content'];
    for(var i = 0; i < toprocess.length; i++){
      var key = toprocess[i];
 
      var $ = cheerio.load(data[key], {
        ignoreWhitespace: false,
        xmlMode: false,
        lowerCaseTags: false,
        decodeEntities: false
      });

      $('img').each(function(){
        if ($(this).attr('src')){
            // For windows style path, we replace '\' to '/'.
            var src = $(this).attr('src').replace('\\', '/');
            if(!/http[s]*.*|\/\/.*/.test(src) &&
               !/^\s*\//.test(src)) {
              // For "about" page, the first part of "src" can't be removed.
              // In addition, to support multi-level local directory.
              var linkArray = link.split('/').filter(function(elem){
                return elem != '';
              });
              var srcArray = src.split('/').filter(function(elem){
                return elem != '' && elem != '.';
              });
              if(srcArray.length > 1)
                srcArray.shift();
              src = srcArray.join('/');
              $(this).attr('src', config.root + link + src);
              console.info&&console.info("update link as:-->"+config.root + link + src);
            }
        }else{
            console.info&&console.info("no src attr, skipped...");
            console.info&&console.info($(this));
        }
      });
      data[key] = $.html();
    }
  }
});
```

## create new post
` hexo new post {your title}`

## add a photo

move your photo {photo name} to ./source/{your title}/ directory.

in ./source/_post/{your title}.md file, link it by the code follow:

`![{comment}]({photo name})`

![photo](1.jpg)

# tutorial

[《不用花一分线，松哥手把手教你上线个人博客》](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247487606&idx=1&sn=f836792babe2669def090163cd470898&scene=21#wechat_redirect)

[《HEXO插入图片（详细版）》](https://www.jianshu.com/p/f72aaad7b852)


# about this project

branch `dev` is hexo project
branch `main` is post source which publish by hexo

## usual command
- local test: `hexo s`
- clean: `hexo clean`
- build: `hexo g`
- deploy: `hexo d`
- build & deploy: `hexo g -d`
