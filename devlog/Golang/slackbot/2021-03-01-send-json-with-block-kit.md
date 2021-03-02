---
layout: post
category: devlog
tags: golang slack
title: "Slack app with Golang - JSON"
subtitle: "Slack App - 2"
description: >
  'Slack block kit'을 사용하여 'Bot(app)' 메세지를 커스텀한 경험을 기록합니다.
image:
  path: /assets/img/devlog/common/slack.jpg
related_posts:
  - _posts/devlog/Golang/package/2021-02-28-slack-json-msg.md
  - _posts/devlog/Golang/slackbot/2021-02-26-how-to-use-slack.md
---

<!--more-->

* this unordered seed list will be replaced by the toc
{:toc}

## Slack Block Kit Builder
![BlockKitBuilder](/assets/img/devlog/golang/2021-03-01/1.png){:.centered}{: style="width: 80%;"}

Slack Message Preview
{:.figcaption}

[Slack Block Kit Builder](https://app.slack.com/block-kit-builder)는 서로 다른 방식으로 정보를 표시하는 블록을 쌓을 수 있습니다.<br>
이 블록은 `JSON`으로 편집할 수 있고, 미리보기를 통해 어떻게 표시될지 확인할 수 있기때문에 굉장히 편리합니다.

## Block
왼쪽 사이드바에서 내가 원하는 블록을 찾아 쉽게 추가할 수 있습니다.<br>
이 포스팅에서는 예제로, 아래 이미지처럼 메세지를 보내보겠습니다.

![Sample](/assets/img/devlog/golang/2021-03-01/2.png){:.centered}{: style="width: 80%;"}

Sample JSON Message
{:.figcaption}

```json
{
	"blocks": [
		{
			"type": "section",
			"text": {
				"type": "mrkdwn",
				"text": "Hello, Assistant to the Regional Manager Dwight! *Michael Scott* wants to know where you'd like to take the Paper Company investors to dinner tonight.\n\n *Please select a restaurant:*"
			}
		},
		{
			"type": "divider"
		},
		{
			"type": "section",
			"text": {
				"type": "mrkdwn",
				"text": "*Farmhouse Thai Cuisine*\n:star::star::star::star: 1528 reviews\n They do have some vegan options, like the roti and curry, plus they have a ton of salad stuff and noodles can be ordered without meat!! They have something for everyone here"
			},
			"accessory": {
				"type": "image",
				"image_url": "https://s3-media3.fl.yelpcdn.com/bphoto/c7ed05m9lC2EmA3Aruue7A/o.jpg",
				"alt_text": "alt text for image"
			}
		},
		{
			"type": "divider"
		}
	]
}
```

오른쪽 `Payload`를 통해, 위와 같은 JSON을 webhook으로 보내고,<br>
`Blocks`에 배열의 형태로 `Block`이 들어있으면 되겠네요.

## Model
```go
// Blocks : block
type Blocks struct {
	Data []Block `json:"blocks"`
}

// Block : 슬랙 메세지 json을 사용하기 위한 구조체
type Block struct {
	Type      string         `json:"type"`
	Elements  BlockContext   `json:"elements,omitempty"`
	Text      BlockText      `json:"text,omitempty"`
	Accessory BlockAccessory `json:"accessory,omitempty"`
}

// BlockText : Block 내 Text 구조체
type BlockText struct {
	Type string `json:"type"`
	Text string `json:"text"`
}

// BlockAccessory : Block 내 Accessory 구조체
type BlockAccessory struct {
	Type    string `json:"type"`
	ImgURL  string `json:"image_url"`
	AltText string `json:"alt_text"`
}
```

저는 위와 같이 구조체를 만들었고, 블록의 특성들에 따라 `omitempty`를 활용하거나 변경해야할 수 있습니다. <br>예시를 위해 위와 같은 구조체를 작성하였기 때문에 필요에 따라 충분히 수정해야합니다.
{:.note}

## Build Blocks
1. 첫번째 `section`
```go
head := Block{
		Type: "section",
		Text: BlockText{
			Type: "mrkdwn",
			Text: "Hello, Assistant to the Regional Manager Dwight! *Michael Scott* wants to know where you'd like to take the Paper Company investors to dinner tonight.\n\n *Please select a restaurant:*",
		},
	}
```
첫번째 블록은 `Type`, `Text` 만 가지고 있습니다.<br>
저는 `Text`를 BlockText라는 구조체를 사용하였습니다.

2. 두번째, 네번째 `divider`
```go
divider := Block{
		Type: "divider",
	}
```
두번째 블록은 `Type`만 갖고있습니다.

3. 세번째 `section`
```go
body := Block{
		Type: "section",
		Text: BlockText{
			Type: "mrkdwn",
			Text: "*Farmhouse Thai Cuisine*\n:star::star::star::star: 1528 reviews\n They do have some vegan options, like the roti and curry, plus they have a ton of salad stuff and noodles can be ordered without meat!! They have something for everyone here",
		},
		Accessory: BlockAccessory{
			Type:    "image",
			ImgURL:  "https://s3-media3.fl.yelpcdn.com/bphoto/c7ed05m9lC2EmA3Aruue7A/o.jpg",
			AltText: "alt text for image",
		},
	}
```
첫번째 블록과 다르게 `Accessory`블록을 가지고 있어서 추가해 주었습니다.

4. Block 들을 하나의 슬라이스에 담아줍니다.
```go
tempBlocks := []Block{}
	tempBlocks = append(tempBlocks, head, divider, body, divider)
```

5. 마지막으로 `Blocks` 구조체에 담도록 합니다.
```go
msg := Blocks{
		Data: tempBlocks,
	}
```

## Send JSON to Slack
```go
slackBody, _ := json.MarshalIndent(msg, "", " ")

req, err := http.NewRequest(http.MethodPost, webhook, bytes.NewBuffer(slackBody))
if nil != err {
	panic(err)
}
req.Header.Add("Content-Type", "application/json")

client := &http.Client{Timeout: 10 * time.Second}
resp, err := client.Do(req)

if nil != err {
	panic(err)
}
buf := new(bytes.Buffer)
buf.ReadFrom(resp.Body)
if buf.String() != "ok" {
	panic(errors.New("Check slack status, responsereturn : non-ok"))
}
```
먼저 JSON 형태로 담아둔 `msg`를 마샬링해야 합니다.<br>
이때 주의해야할 점이 한가지 있었습니다.

### omitempty
```go
slackBody, _ := json.MarshalIndent(msg, "", " ")
fmt.Println(string(slackBody))
```

```shell
{
 "blocks": [
  {
   "type": "section",
   "text": {
    "type": "mrkdwn",
    "text": "Hello, Assistant to the Regional Manager Dwight! *Michael Scott* wants to know where you'd like to take the Paper Company investors to dinner tonight.\n\n *Please select a restaurant:*"
   },
   "accessory": {
    "type": "",
    "image_url": "",
    "alt_text": ""
   }
  },
  {
   "type": "divider",
   "text": {
    "type": "",
    "text": ""
   },
   "accessory": {
    "type": "",
    "image_url": "",
    "alt_text": ""
   }
  },
  {
   "type": "section",
   "text": {
    "type": "mrkdwn",
    "text": "*Farmhouse Thai Cuisine*\n:star::star::star::star: 1528 reviews\n They do have some vegan options, like the roti and curry, plus they have a ton of salad stuff and noodles can be ordered without meat!! They have something for everyone here"
   },
   "accessory": {
    "type": "image",
    "image_url": "https://s3-media3.fl.yelpcdn.com/bphoto/c7ed05m9lC2EmA3Aruue7A/o.jpg",
    "alt_text": "alt text for image"
   }
  },
  {
   "type": "divider",
   "text": {
    "type": "",
    "text": ""
   },
   "accessory": {
    "type": "",
    "image_url": "",
    "alt_text": ""
   }
  }
 ]
}
```
중간중간 사용하지 않는 `text`, `type`, `accessory`와 같은 구조체들이 값이 없음에도 `omitempty` 처리를 하지 않았습니다.

기본 패키지인 `encoding/json`에서는 struct가 omitempty 일때를 처리해주지 못합니다. <br>

### github.com/clarketm/json
struct가 `omitempty`일 경우가 있을 때, 위 패키지를 사용하면 마샬링할 때 구조체도 같이 처리할 수 있습니다.

```shell
{
 "blocks": [
  {
   "type": "section",
   "text": {
    "type": "mrkdwn",
    "text": "Hello, Assistant to the Regional Manager Dwight! *Michael Scott* wants to know where you'd like to take the Paper Company investors to dinner tonight.\n\n *Please select a restaurant:*"
   }
  },
  {
   "type": "divider"
  },
  {
   "type": "section",
   "text": {
    "type": "mrkdwn",
    "text": "*Farmhouse Thai Cuisine*\n:star::star::star::star: 1528 reviews\n They do have some vegan options, like the roti and curry, plus they have a ton of salad stuff and noodles can be ordered without meat!! They have something for everyone here"
   },
   "accessory": {
    "type": "image",
    "image_url": "https://s3-media3.fl.yelpcdn.com/bphoto/c7ed05m9lC2EmA3Aruue7A/o.jpg",
    "alt_text": "alt text for image"
   }
  },
  {
   "type": "divider"
  }
 ]
}
```

슬랙으로 JSON 메세지를 보내는 것은 Text를 보낼때 사용한 함수를 동일하게 사용했습니다.
```go
// 위에서 생성한 Webhook으로 POST Request 합니다.
req, err := http.NewRequest(http.MethodPost,webhook, bytes.NewBuffer(slackBody))
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
	err = errors.New("Check slack status, responsereturn : non-ok")
    panic(err)
}
```

## Result
![Result](/assets/img/devlog/golang/2021-03-01/3.png){:.centered}{: style="width: 70%"}

## Next
- Slack app with Golang - File Upload