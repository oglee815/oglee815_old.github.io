---
layout: post
date: 2022-04-13 00:00
created_date: 2022-04-13 00:00
title: "[work] vim에서 default로 mouse=r 설정"
author: oglee
description: vim에서 default로 mouse=r 설정
comments: true
math: true
category:
- Work
tags:
- Vim
- Docker
---

/usr/share/vim/vim82/defaults.vim 에서 mouse=r 설정

<!--more-->

- Docker를 새로 만들고 Vim을 다시 설치하자, 마우스 더블클릭 시에 내용이 복사되는 기능이 사리짐.
- 구글링 해보니 \:set mouse=r 로 변경하면 되지만 매번 할 수는 없었음.
- default로 설정하기 위해 이것저것 해본 결과, /usr/share/vim/vim82/defaults.vim 에서 아래 라인을 바꾸니까 됐음.
  - ![image](https://user-images.githubusercontent.com/18374514/168773675-ebd4814e-30f5-46bf-8ab4-66f4ac87459b.png)
- 근데 왜 xterm에서 실행할 때만 적용되는 옵션이 있는거지? 
