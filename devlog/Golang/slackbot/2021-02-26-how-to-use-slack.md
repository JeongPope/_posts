---
layout: post
category: devlog
tags: golang slack
title: "Slack app with Golang - Text Msg"
subtitle: "Slack App - 1"
description: >
  Golang을 활용하여 클라우드 기반 팀 협업 도구인 'Slack'의 'Bot(app)'을 사용해본 경험을 기록합니다.
image:
  path: /assets/img/devlog/common/slack.jpg
related_posts:
  - _posts/devlog/Golang/package/2021-02-28-slack-json-msg.md
---

<!--more-->

* this unordered seed list will be replaced by the toc
{:toc}


## 사전 준비
- 메세지를 받고싶은 슬랙 채널

## 나만의 슬랙 봇 만들기
1. [Slack API](https://api.slack.com)에서 `Create a custom app`을 선택합니다.<br>
![Create slack custom app](/assets/img/devlog/golang/2021-02-26/1.png){: style="width: 80%;"}
<br>

2. `App Name`과 `Workspace`을 설정하고, 생성합니다.<br>
![App Name, Workspace](/assets/img/devlog/golang/2021-02-26/2.png){: style="width: 50%;"}
<br>

3. `Basic Information`에서 `Incoming Webhooks`를 클릭합니다. <br>
![Incoming Webhooks](/assets/img/devlog/golang/2021-02-26/3.png){: style="width: 50%;"}
<br>

4. 그리고 `Add New Webhook to Workspace`를 선택하여 `Webhook URL`을 생성합니다.<br>
![Webhook URL](/assets/img/devlog/golang/2021-02-26/4.png){: style="width: 50%;"}

![Integration](/assets/img/devlog/golang/2021-02-26/5.png){: style="width: 50%;"}
> 이후 채널에서 앱이 integration 된 것을 확인할 수 있습니다.<br>
> 이제부터 이 채널로 메세지를 보낼수 있게 되었습니다. 🎉

## 슬랙으로 `Text` 메세지 보내기
#### Model
```go
type slackTextBody struct {
	Text string `json:"text"`
}
```
> `string`으로 사용할 수 있었겠지만, 슬랙 메세지 형태 구분을 위해 `struct`를 사용했습니다.

#### Func : Send text message to slack
```go
func SendText(webhook string, msg string) {
    // 보내고 싶은 문자열을 마샬링 합니다.
	slackBody, _ := json.Marshal(slackTextBody{Text: msg})

    // 위에서 생성한 Webhook으로 POST Request 합니다.
	req, err := http.NewRequest(http.MethodPost, webhook, bytes.NewBuffer(slackBody))
	if err != nil {
		return err
	}

    // Request 헤더값을 추가해줍니다.
	req.Header.Add("Content-Type", "application/json")

	client := &http.Client{Timeout: 10 * time.Second}
	resp, err := client.Do(req)
	if err != nil {
		panic(err)
	}

	buf := new(bytes.Buffer)
	buf.ReadFrom(resp.Body)
	if buf.String() != "ok" {
		err = errors.New("Check slack status, response return : non-ok")

        panic(err)
	}
}
```

### Send : Hello, World!
```go
package main

type slackTextBody struct {
	// ...
}

func SendText(webhook string, msg string) {
    // ...
}

const(
    webhookURL = "https://hooks.slack.com/services/..."
)

func main() {
    SendText(webhookURL, "Hello, World!")
}
```

### Result
![Result](/assets/img/devlog/golang/2021-02-26/6.png){: style="width: 30%;"}

## JSON은? 파일 업로드는?
알림 메세지로 단순 텍스트만 보내지는 않고, 예쁘고 많은 정보를 담은 메세지를 보내야 할텐데 당연히 텍스트로 할리가 없다.<br><br><br>
그래서.....

## Next?
- Slack app with Golang - JSON