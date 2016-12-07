---
layout: page
title: About
description: -------
keywords: kjson
comments: true
menu: 关于
permalink: /about/
---

我是kjson, 坚信代码和数据可以改变世界。

仰慕「优雅编码的艺术」。

## 坚信

* 熟能生巧
* 努力改变人生

## 联系

* GitHub：[@kimjayson](https://github.com/kimjayson)
* 博客：[{{ site.title }}]({{ site.url }})
* 微博: [@kjson1992](http://weibo.com/kjson1992)
* 知乎: [@金子](http://www.zhihu.com/people/金子)
* 豆瓣: [@kjson](http://www.douban.com/people/kjson)

## Skill Keywords

#### Software Engineer Keywords
<div class="btn-inline">
    {% for keyword in site.skill_software_keywords %}
    <button class="btn btn-outline" type="button">{{ keyword }}</button>
    {% endfor %}
</div>

#### Mobile Developer Keywords
<div class="btn-inline">
    {% for keyword in site.skill_mobile_app_keywords %}
    <button class="btn btn-outline" type="button">{{ keyword }}</button>
    {% endfor %}
</div>

#### Windows Developer Keywords
<div class="btn-inline">
    {% for keyword in site.skill_windows_keywords %}
    <button class="btn btn-outline" type="button">{{ keyword }}</button>
    {% endfor %}
</div>
