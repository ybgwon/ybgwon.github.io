+++
title = "hugo 사용법"
author = ["Yong-Beom Gwon"]
date = 2019-11-14T00:00:00+09:00
tags = ["hugo", "blog"]
draft = false
+++

## <span class="section-num">1</span> 설치 {#설치}

```text
$ brew install hugo
```


## <span class="section-num">2</span> 사이트 생성 {#사이트-생성}

```text
$ hugo new site hugo
```


## <span class="section-num">3</span> 테마 적용 {#테마-적용}

1.  테마 파일을 선택한 뒤 submodule 로 받아온다.  
    
    ```text
    $ cd hugo
    $ git init
    $ git submodule add https://github.com/digitalcraftsman/hugo-cactus-theme.git themes/cactus
    ```
2.  config.toml 파일에서 받아온 theme을 설정한다.  
    
    ```text
    $ cat config.toml
    baseURL = "http://ybgwon.github.io/"
    languageCode = "ko-kr"
    title = "YB Gwon's ..."
    theme = "cactus"
    ```


## <span class="section-num">4</span> 콘텐츠 추가 {#콘텐츠-추가}

```text
$ hugo new posts/first.org
```

content/posts/first.org 파일 생성됨  


## <span class="section-num">5</span> 테스트 {#테스트}

```text
$ hugo server -D
```

<http://localhost:1313/> 으로 연결  


## <span class="section-num">6</span> build static page {#build-static-page}

```text
$ hugo -D
```
