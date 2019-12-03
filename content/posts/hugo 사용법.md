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


## <span class="section-num">2</span> 업그레이드 {#업그레이드}

```text
$ brew upgrade hugo
```


## <span class="section-num">3</span> 사이트 생성 {#사이트-생성}

```text
$ hugo new site hugo
```


## <span class="section-num">4</span> 테마 적용 {#테마-적용}

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


## <span class="section-num">5</span> 콘텐츠 추가 {#콘텐츠-추가}

```text
$ hugo new posts/first.org
```

content/posts/first.org 파일 생성됨  


## <span class="section-num">6</span> 테스트 {#테스트}

```text
$ hugo server -D
```

<http://localhost:1313/> 으로 연결  


## <span class="section-num">7</span> build static page {#build-static-page}

```text
$ hugo -D
```


## <span class="section-num">8</span> GitHub Actions for Hugo {#github-actions-for-hugo}

github action을 사용하면 hugo 소스만 관리하면 build는 자동화 할 수있다.  

1.  ssh key 생성  
    
    ```bash
    ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f gh-pages -N ""
    # You will get 2 files:
    #   gh-pages.pub (public key)
    #   gh-pages     (private key)
    ```

2.  github 저장소 설정 페이지 에서 아래 설정  
    -   Deploy keys 에 위에서 만든 public key 복사(allow write access  
        check)
    
    -   Secrets 에 위에서 만든 private key 복사 (이름을 ACTIONS\_DEPLOY\_KEY로)

3.  로컬 hugo 저장소의 .github/workflows/gh-pages.yml 작성  
    
    ```yaml
    name: github pages
    
    on:
      push:
        branches:
    ​    - src
    
    jobs:
      build-deploy:
        runs-on: ubuntu-18.04
        steps:
    ​    - uses: actions/checkout@master
          with:
    	submodules: true
    ​
        - name: Setup Hugo
          uses: peaceiris/actions-hugo@v2.2.0
          with:
       hugo-version: '0.59.1'
       extended: true
    ​
        - name: Build
          run: hugo --gc --minify --cleanDestinationDir
    ​
        - name: Deploy
          uses: peaceiris/actions-gh-pages@v2.4.0
          env:
       ACTIONS_DEPLOY_KEY: ${{ secrets.ACTIONS_DEPLOY_KEY }}
       PUBLISH_BRANCH: master
       PUBLISH_DIR: ./public
    ```
    
    저장소 타입이 project 일때와 User and Organization(url이  
    username.github.io) 일 때 설정이 다른데 위는 User and  
    Organization 일 경우다.  
    src 브랜치에 hugo src를 두고 github에 변경사항을 push 하면  
    github action에 의해 master 브랜치에 publish 되는 구조다.


### <span class="section-num">8.1</span> 참조 {#참조}

<https://github.com/peaceiris/actions-hugo>  
[gohugo.io](https://discourse.gohugo.io/t/deploy-hugo-project-to-github-pages-with-github-actions/20725)  
<https://blog.kye.dev/hugo-github-pages-actions/>
