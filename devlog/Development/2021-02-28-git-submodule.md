---
layout: post
category: devlog
tags: development git
title: "Git Submodule를 활용한 Jekyll Blog comment 기능 활용하기"
subtitle: "Git tool - Submodule"
image:
  path: /assets/img/devlog/development/2021-02-28/3.png
---

저는 Github Blog 의 코드는 `Private repository`에 저장되어 있습니다.<br>
이로 인해 포스트들에 Comment 기능을 사용할 수 없어 `Public`으로 분리해야 했습니다.

이때 `git submodule`을 사용하였습니다.

<!--more-->

## [Git submodule](https://git-scm.com/book/ko/v2/Git-도구-서브모듈)
프로젝트를 수행하다 보면 다른 프로젝트를 함께 사용해야 하는 경우가 종종 있다. 함께 사용할 다른 프로젝트는 외부에서 개발한 라이브러리라던가 내부 여러 프로젝트에서 공통으로 사용할 라이브러리일 수 있다. 이런 상황에서 자주 생기는 이슈는 두 프로젝트를 서로 별개로 다루면서도 그 중 하나를 다른 하나 안에서 사용할 수 있어야 한다는 것입니다.

Git 저장소 안에 다른 Git 저장소를 디렉토리로 분리해 넣는 것이 서브모듈이다. 다른 독립된 Git 저장소를 Clone 해서 내 Git 저장소 안에 포함할 수 있으며 각 저장소의 커밋은 독립적으로 관리합니다.

## My Blog
* Jekyll
* Theme : Hydejack PRO
* Comments : Utterance

코멘트 기능을 사용하기 위해 `Utterance`를 선택했습니다. <br>
기능을 사용하려면, `Jekyll`에서 포스트 글을 담는 `_posts`폴더가 **public repository**에 있어야 했습니다..

## Public Repository에 _posts 분리
![Public repository](/assets/img/devlog/development/2021-02-28/4.png){:.centered}{: style="width: 50%;"}

실제로 분리한 Public repository
{:.figcaption}

## Private Repository에 Submodule 추가
```shell
// posts를 가지고 있는 public repository 추가
git submodule add https://github.com/JeongPope/_postgit
```

![Git module](/assets/img/devlog/development/2021-02-28/5.png){:.centered}{: style="width: 50%;"}
.gitmodules 파일이 추가되었습니다.
{:.figcaption}

이제 하위 프로젝트를 포함하여 커밋을 생성하면, `_posts`의 디렉토리 모드가 `160000`인것을 볼 수 있습니다.<br>
Git 에게 있어서는 일반적 파일이나 디렉토리가 아니라는 뜻입니다.
```shell
git commit -am 'Update : Added _posts module'
[master abcdef] added _posts module
 2 files changed, 4 insertions(+)
 create mode 100644 .gitmodules
 create mode 160000 DbConnector
```

끝으로 `push`합니다.
```shell
git push origin master
```

이제 `Private repository`에서는 기존에 보던것과 다른 폴더가 하나 생겨났습니다.
