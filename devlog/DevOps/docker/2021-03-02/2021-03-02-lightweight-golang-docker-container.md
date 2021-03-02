---
layout: post
category: devlog
tags: devops docker golang
title: "Lightwegith golang container use multi-stage build"
subtitle: "Lightweight golang docker container"
description: >
  멀티스테이지 빌드를 사용하여 Golang 컨테이너를 경량화 하는 방법을 기록하였습니다.
image:
  path: /assets/img/devlog/devops/2021-03-02/main.png
---

slack app을 만들고 나서, 도커 이미지로 빌드를 했더니 700MB 이상의 크기나 되는 이미지가 만들어졌습니다.<br>
1,000 line도 되지 않는 slack app 소스코드를 위해 1 GB나 되는 용량을 할당해주는 것은 도저히 참을수가 없었습니다.

<!--more-->

* this unordered seed list will be replaced by the toc
{:toc}

## Multi-stage build
컨테이너 이미지를 만들면서 빌드 등에는 필요하지만 최종 컨테이너 이미지에는 필요 없는 환경을 제거할 수 있도록 단계를 나누어서 기반 이미지를 만드는 방법<br>

### Background
Docker가 등장한 이후, Image를 작게 만드려는 노력들이 있었는데 그 이유는 Image 의 사이즈가 작을수록 빌드, 배포시간이 그만큼 단축되기 때문입니다.<br>
각 `Instruction`들은 Dockerfile 하나의 Layer로 추가가 되기 때문에 여러 최적화가 필요했고, 이를 위해 나온 방법이 **Multi stage build** 패턴입니다.

## Dockerfile
```shell
### Builder
FROM golang:1.13-alpine as builder
RUN apk update && apk add git

WORKDIR /tmp/slackbot
COPY . .

RUN go mod tidy
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -a -ldflags '-s -w' -o main .

### Make executable image
FROM scratch

COPY --from=builder /tmp/slackbot /

CMD [ "/main" ]
```
최종 Dockerfile
{:figcaption}

일반적인 Dockerfile과는 다르게 `as builder`, `--from`을 볼 수 있다.

## First stage : Builder
```shell
### Builder
FROM golang:1.13-alpine as builder
RUN apk update && apk add git

WORKDIR /tmp/slackbot
COPY . .

RUN go mod tidy
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -a -ldflags '-s -w' -o main .
```

- `go mod tidy`
`go mod`파일과 소스코드를 비교하여 import되지 않은 의존성은 제거하고, import 되었지만 의존성 리스트에 추가되지 않은 모듈은 추가합니다.<br>

- `CGO_ENABLED=0`, `-a`
C Binary 를 사용하지 않으므로 표준 라이브러리를 모두 다시 빌드합니다.<br>

- `ldflags '-s -w'`
디버그 정보를 삭제하여 바이너리의 크기를 줄입니다.

## Second stage
```shell
### Make executable image
FROM scratch

COPY --from=builder /tmp/slackbot /

CMD [ "/main" ]
```

- `FROM scratch`
말 그대로 텅 비어있는 이미지입니다.

예시에서는 scratch 이미지를 사용했지만, /bin/bash 등 쉘을 사용하기 위해서는 alpine 를 사용하는게 좋을듯...
{:.note}

- `COPY --from=builder /tmp/slackbot /`
`builder`에서 `/tmp/slackbot`폴더에서 `/`로 복사합니다.

## Result
```shell
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
slackbot            0.1                 52139609f4e0        20 hours ago        6.81MB
```

용량에는 `builder` 스테이지의 파일들은 최종 빌드된 이미지에 포함되지 않았습니다.

## 주의해야 할 점
- Go는 의존성 관리에는 `git`이 꼭 필요합니다. `alpine`에는 `git`이 포함되어 있지 않으므로 꼭 포함시켜줍니다.
- `scratch` 이미지에는 암호화된 통신을 위한 CA 인증서가 들어있지 않습니다. 때문에 HTTPS를 비롯한 SSL 통신을 하는 경우 오류가 발생하게 됩니다.
- 간혹 유저 관리를 위해 `/etc/passwd` 바이너리 등이 필요한 때도 있습니다.
- 또한 `scratch` 는 디버그가 어렵습니다. alpine을 씁시다.

## Next
- Slack app with Golang - File Upload