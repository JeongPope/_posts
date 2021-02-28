---
layout: post
category: devlog
tags: development git mac
title: "xcrun: error: invalid active developer path"
subtitle: "Mac Update 이후 개발 도구 에러"
image:
  path: /assets/img/devlog/development/2021-02-28/1.png
---

Big sur 업데이트 후, 반복적으로 발생하는 에러가 있었습니다. <br>
기록을 남겨두면서 기억해두고자 합니다.

## Summary
```shell
xcode-select --install
```

<!--more-->

## Steps
1. 여러 개발 도구들이 아래와 같은 xcrun 에러가 발생합니다.

    ```shell
    xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
    ```

2. `xcode-select` 커맨드를 통해 `xcode-cli`를 설치해 줍니다.

    ```shell
    xcode-selector --install
    ```

3. 설치가 완료된 것을 확인합니다.<br>
![Install](/assets/img/devlog/development/2021-02-28/2.png){: style="width: 50%;"}



