---
title: Chrome grid justify/align-content bug
date: 2018-04-18 23:20
categories: code
tags:
- chrome
- bug
- grid
---

最近写[项目](https://github.com/MondoGao/by-cobweb-project-progress-post)的时候发现 Chrome 66 和 FF 59 在 Grid 上表现不一致，写了个[测试 Demo](https://codepen.io/MondoGao/pen/KoYBja)

Chrome 在 `flex-grow` 的增长条件下，devtools 显示高度正常，但 `justify-content: stretch` 不生效

