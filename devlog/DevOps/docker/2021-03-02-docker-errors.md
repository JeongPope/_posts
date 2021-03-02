---
layout: post
category: devlog
tags: devops docker
title: "Wiki : Docker errors"
description: >
  Docker를 사용하면서 겪었던 문제 모음
image:
  path: /assets/img/devlog/common/docker.png
---

<!--more-->

***

#### docker: Error response from daemon: pull access denied for test, repository does not exist or may require 'docker login': denied: requested access to the resource is denied.
1. 원인
도커에 로그인을 해달라고 에러메세지에 명시되어 있다.

2. 해결
```shell
docker login
```

***

#### OCI runtime exec failed: exec failed: container_linux.go:349: starting container process caused "exec: ...": stat /bash: no such file or directory": unknown
1. 원인
이 오류에는 다양한 원인이 있는것 같다. 내가 겪었던 문제들은,
* Multi-stage build 에서 두번째 스테이지에 `scratch` 이미지를 사용했다. 이 이미지를 사용한 컨테이너로 `exec -it {container ID} /bin/bash` 등등 으로 접근하려고 했을 때, 스크래치 이미지는 쉘이 없어 위와 같은 메세지를 확인할 수 있었다.
* `docker run` 이후 커맨드를 잘못 작성했을 때에도 위와 같은 메세지를 확인했다.

2. 해결
* 두번째 스테이지에서 `scratch`가 아닌 `alpine` 로 대체하자. (`scratch`는 디버그 하기도 어렵고, 쉘이 없다고 한다.)
* docker 명령어에 좀더 익숙해 지자..