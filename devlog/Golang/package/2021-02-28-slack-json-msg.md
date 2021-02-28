---
layout: post
category: devlog
tags: golang slack
title: "Golang package : encoding/json"
subtitle: "Golang package : encoding/json"
description: >
  Golang에서는 JSON을 어떻게 다룰까?
image:
  path: /assets/img/devlog/common/json.png
related_posts:
  - _posts/category/devlog/Golang/slackbot/2021-02-26-how-to-use-slack.md
---
JSON : **J**ava**S**cript **O**bject **N**otation <br>

Software는 `Byte`단위로 데이터를 인식합니다.<br>
메모리 바이트는 해석하기에 따라 달라지는데 이러한 변환을 '인코딩' 또는 '마샬링'이라고 합니다.

Golang에서는 encoding이 이를 담당하는 기본 패키지이며, 실제로는 인터페이스 타입만 정의 되어 있고 데이터 형태에 따라 서브 패키지로 기능을 제공합니다.

<!--more-->

* this unordered seed list will be replaced by the toc
{:toc}


## Package
* encoinding/json
* github.com/clarketm/json

## Marshal
논리적 구조를 로우 바이트로 변경하는 것을 **마샬링(Marshaling)** 또는 **인코딩(Encoding)**이라고 합니다.<br>
Golang value를 바이트 슬라이스로 변경하는 것입니다.

이 역할을 json.**Marshal** 함수가 담당합니다.

```go
func Marshal(v interface{}) ([]byte, error)
```

Golang에서 모든 타입을 받을 수 있는 `interface{}`를 인자로 받아 바이트 슬라이스로 변환합니다.

```go
type member struct {
	Email string
	Level int
}

func main() {
	// Marshaling : bool
	fmt.Println("Marshaling : Boolean")
	result, _ := json.Marshal(false)

	fmt.Println(result)
	fmt.Println(string(result))

	// Marshaling : struct
	fmt.Println("Marshaling : Struct")
	userA := member{
		Email: "member@member.com",
		Level: 1,
	}
	result, _ = json.Marshal(userA)
	fmt.Println(result)
	fmt.Println(string(result))
}
```

```shell
Marshaling : Boolean
[102 97 108 115 101]
false
Marshaling : Struct
[123 34 69 109 97 105 108 34 58 34 109 101 109 98 101 114 64 109 101 109 98 101 114 46 99 111 109 34 44 34 76 101 118 101 108 34 58 49 125]
{"Email":"member@member.com","Level":1}
```

JSON 의 Key 값은 보통 소문자로 시작하고, 구조체로 타입을 정의하는 경우 태그를 이용해 프로퍼티를 변경할 수 있습니다.
```go
type member struct {
  // JSON 태그를 참고하여 Email -> email, Level -> level 로 변환
	Email string `json:"email"`
	Level int    `json:"level"`
}
```

프로퍼티의 이름 변경 뿐 아니라, 제어를 할 수도 있습니다.
```go
type kyc struct {
  Name string `json:"-"`              // 출력하지 않음
  Phone string `json:"phone,string"`  // 문자열로 출력함
  Bank string `json:"bank,omitempty"` // 값이 비어있다면 출력하지 않음
}
```

가독성을 높이려면 json.**MarshalIndent** 를 쓰도록 합니다.
```go
fmt.Println("Marshaling : Struct")
userA := member{
	Email: "member@member.com",
	Level: 1,
}
result, _ = json.Marshal(userA)
fmt.Println(result)
fmt.Println(string(result))

result, _ = json.MarshalIndent(userA, "", " ")
fmt.Println(result)
fmt.Println(string(result))
```

```shell
Marshaling : Struct
[123 34 101 109 97 105 108 34 58 34 109 101 109 98 101 114 64 109 101 109 98 101 114 46 99 111 109 34 44 34 108 101 118 101 108 34 58 49 125]
{"email":"member@member.com","level":1}
[123 10 32 34 101 109 97 105 108 34 58 32 34 109 101 109 98 101 114 64 109 101 109 98 101 114 46 99 111 109 34 44 10 32 34 108 101 118 101 108 34 58 32 49 10 125]
{
 "email": "member@member.com",
 "level": 1
}
```

## Unmarshal
마샬링과 반대로 바이트 슬라이스나 문자열을 논리적 자료구조로 변경하는 것을 말합니다.<br>
이 역할을 json.**Unmarshal** 함수가 담당합니다.

```go
func Unmarshal(data []byte, v interface{}) error
```

마샬링과 마찬가지로 먼저 Boolean을 언마샬링 해보면, `false` 를 확인할 수 있습니다.
```go
var result bool
json.Unmarshal([]byte("false"), &result)
fmt.Printf("%t\n", result) // false
```

첫번째 인자로 바이트 슬라이스를 넘겨주고, 두번째 인자로 결과를 담아줄 포인터 변수를 넘겨줍니다.<br>
이 두번째 변수의 시그니처가 `interface{}`이기 때문에 타입에 따라 언마샬링을 합니다.<br>
> 당연히 Boolean 뿐 아니라 여러 타입도 사용할 수 있습니다.

## Next
- encoding, decoding : 인코딩과 디코딩을 알아보고, 마샬링/언마샬링과의 차이점

### 예정
- File Upload : slack API도 있겠지만 외부 패키지를 사용해보자
- Mysql : MySQL에 연결하여 DB 데이터를 사용해보자

## Reference
- [김정환님 깃허브 블로그](https://jeonghwan-kim.github.io/dev/2019/01/18/go-encoding-json.html)