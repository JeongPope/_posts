---
layout: post
category: devlog
tags: devops aws monitoring
title: "Cloudwatch를 활용한 AWS Resource 실시간 모니터링"
description: >
  Cloudwatch에 알람을 설정하고 Slack으로 알람 메세지를 발송합니다.
image:
  path: /assets/img/devlog/devops/aws/2021-03-04/main.png
---

<!--more-->

***

### 시스템의 구성
1. Cloudwatch : AWS Resource와 Running application을 모니터링 합니다.
2. AWS SNS : Publisher <> Subscriber 통신 채널입니다. Protocol을 이용하여 클라이언트에 메세지를 발송합니다.
3. Lambda : Serverless computing service. 서버를 관리하지 않아도 코드를 실행할 수 있습니다.
4. AWS KMS : 데이터 암호화에 사용되는 암호화 키이며 데이터 통신시에 사용합니다.
5. Slack : Slack app을 만들고, webhook을 통해 메세지를 수신합니다.
슬랙 앱은 [봇 만들기](/_posts/devlog/Golang/slackbot/2021-02-26-how-to-use-slack.md)를 통해서 확인해 보실 수 있습니다.
{:.note}

Cloudwatch에서 알람이 발생하면 `AWS SNS` Push service를 호출하여 `AWS Lambda` trigger를 실행, 마지막에 슬랙을 통해 알람을 보냅니다.

### AWS SNS(Simple Notification Service)
Amazon Simple Notification Service(Amazon SNS)는 애플리케이션 간(A2A) 및 애플리케이션과 사용자 간(A2P) 통신 모두를 위한 완전관리형 메시징 서비스입니다.<br>

A2A 게시/구독 기능에서는 분산 시스템, 마이크로서비스 및 이벤트 중심의 서버리스 애플리케이션 사이에서 처리량이 높은 푸시 기반의 다대다 메시징을 위한 주제를 제공합니다.<br>
Amazon SNS 주제를 사용하면 게시자 시스템에서 Amazon SQS 대기열, AWS Lambda 함수 및 HTTPS 엔드포인트를 비롯한 다수의 구독자 시스템과 병렬 처리를 위해 Amazon Kinesis Data Firehose로 메시지를 팬아웃할 수 있습니다. <br>
A2P 기능을 사용하면 SMS, 모바일 푸시 및 이메일을 통해 대규모로 사용자에게 메시지를 전송할 수 있습니다.

![Create SNS](/assets/img/devlog/devops/aws/2021-03-04/1.png){:.centered}{: style="width: 50%;"}
1. AWS SNS에서 주제를 생성하고 유형은 `표준`, 이름 까지 적어 생성해줍니다.

### Lambda : 서버리스 컴퓨팅 서비스
서버리스 컴퓨팅 서비스로 애플리케이션을 실행하기 위한 별도의 서버 셋업 없이 곧바로 코드를 실행할 수 있습니다.<br>
고정 비용없이 사용시간에 대해서만 비용이 발생합니다.

![Create Lambda - 1](/assets/img/devlog/devops/aws/2021-03-04/2.png){:.centered}{: style="width: 50%;"}
Blueprint - Cloudwatch-alarm-to-slack-python 을 사용합니다.
{:.figcaption}

![Create Lambda - 2](/assets/img/devlog/devops/aws/2021-03-04/3.png){:.centered}{: style="width: 50%;"}
함수이름은 아무렇게나 지어줍시다.
{:.figcaption}

![Create Lambda - 3](/assets/img/devlog/devops/aws/2021-03-04/4.png){:.centered}{: style="width: 50%;"}
기본 정책을 가지는 역할을 생성하고, SNS 주제를 위에 생성했던 SNS로 선택합니다.
{:.figcaption}

![Create Lambda - 4](/assets/img/devlog/devops/aws/2021-03-04/5.png){:.centered}{: style="width: 50%;"}
환경변수를 임의를 지정해줍니다.
{:.figcaption}

### AWS KMS(Key Management Service)
1. 데이터를 암호화하는 데 사용되는 암호화 키인 고객 마스터 키(CMKs)를 관리합니다.
2. Lambda에서 Slack으로 메시지를 전송할 때 webhook을 암호화하는데 사용합니다.


EC2에서 아래와 같이 실행시켜줍니다.
```shell
[admin@ip-10-0-0-51 ~]$ aws kms create-key --region ap-northeast-2
{
    "KeyMetadata": {
        "Origin": "AWS_KMS", 
        "KeyId": "2a77xxxx-xxxx-xxxx-xxxx-xxxx051bc618", 
        "Description": "", 
        "KeyManager": "CUSTOMER", 
        "EncryptionAlgorithms": [
            "SYMMETRIC_DEFAULT"
        ], 
        "Enabled": true, 
        "CustomerMasterKeySpec": "SYMMETRIC_DEFAULT", 
        "KeyUsage": "ENCRYPT_DECRYPT", 
        "KeyState": "Enabled", 
        "CreationDate": 1614868936.358, 
        "Arn": "arn:aws:kms:ap-northeast-2:111122223333:key/2a77xxxx-xxxx-xxxx-xxxx-xxxx051bc618", 
        "AWSAccountId": "111122223333"
    }
}
```
![KMS - 1](/assets/img/devlog/devops/aws/2021-03-04/6.png){:.centered}{: style="width: 50%;"}
AWS 웹화면에서도 확인할 수 있습니다.
{:.figcaption}

```shell
[admin@ip-10-0-0-51 ~]$ aws kms create-alias --alias-name alias/{keyName} --target-key-id {keyId} --region ap-northeast-2
```

![KMS - 2](/assets/img/devlog/devops/aws/2021-03-04/7.png){:.centered}{: style="width: 50%;"}
이름도 지어줬습니다.
{:.figcaption}


#### Key encrypt
Lambda에 적용할 함수 중, webhook을 암호화 하기 위해 `--encryption-context LambdaFunctionName={funcName}`을 사용합니다.
```
// Check AWS CLI Version
[admin@ip-10-0-0-51 ~]$ aws --version
aws-cli/1.18.107 Python/2.7.18 Linux/4.14.214-118.339.amzn1.x86_64 botocore/1.17.31

// Encrypt KMS Key
[admin@ip-10-0-0-51 ~]$ aws kms encrypt --key-id alias/mission-kms-key --plaintext "hooks.slack.com/services/XXXXXXXXXXX/XXXXXXXXXX/XXXXXXXXXXXXXXXXXXXXX" --region ap-northeast-2 --encryption-context LambdaFunctionName=mission-lambda
{
    "EncryptionAlgorithm": "SYMMETRIC_DEFAULT", 
    "KeyId": "arn:aws:kms:ap-northeast-2:79........97:key/2a777c24-9628-48eb-bda2-c5f7051bc618", 
    "CiphertextBlob": "{ecrypted kms key string}"
}
```

### Lambda 환경 변수 및 권한/정책 설정
위에서 암호화 후, `CiphertextBlob`을 `Lambda` - 환경변수에 `kmsEncryptedHookUrl`의 값을 사용해줍니다.<br>
그리고 암호화 구성은 고객 마스터키를 사용합니다. (당연히 마스터키는 KMS 키를 사용합니다.)

![Lambda role - 1](/assets/img/devlog/devops/aws/2021-03-04/8.png){:.centered}{: style="width: 50%;"}
![Lambda role - 2](/assets/img/devlog/devops/aws/2021-03-04/9.png){:.centered}{: style="width: 50%;"}
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1443036478000",
            "Effect": "Allow",
            "Action": [
                "kms:Decrypt"
                ],
            "Resource": [
                "{Your KMS Key ARN}"
                ]
        }]
}
```
이 정책을 람다와 연결해 주었습니다.

#### Lambda 메세지 보내기 (Test code)
```json
{
  "Records": [
    {
      "EventSource": "aws:sns",
      "EventVersion": "1.0",
      "EventSubscriptionArn": "arn:aws:sns:eu-west-1:000000000000:cloudwatch-alarms:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "Sns": {
        "Type": "Notification",
        "MessageId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "TopicArn": "arn:aws:sns:eu-west-1:000000000000:cloudwatch-alarms",
        "Subject": "ALARM: \"Example alarm name\" in EU - Ireland",
        "Message": "{\"AlarmName\":\"Example alarm name\",\"AlarmDescription\":\"Example alarm description.\",\"AWSAccountId\":\"000000000000\",\"NewStateValue\":\"ALARM\",\"NewStateReason\":\"Threshold Crossed: 1 datapoint (10.0) was greater than or equal to the threshold (1.0).\",\"StateChangeTime\":\"2017-01-12T16:30:42.236+0000\",\"Region\":\"EU - Ireland\",\"OldStateValue\":\"OK\",\"Trigger\":{\"MetricName\":\"DeliveryErrors\",\"Namespace\":\"ExampleNamespace\",\"Statistic\":\"SUM\",\"Unit\":null,\"Dimensions\":[],\"Period\":300,\"EvaluationPeriods\":1,\"ComparisonOperator\":\"GreaterThanOrEqualToThreshold\",\"Threshold\":1.0}}",
        "Timestamp": "2017-01-12T16:30:42.318Z",
        "SignatureVersion": "1",
        "Signature": "Cg==",
        "SigningCertUrl": "https://sns.eu-west-1.amazonaws.com/SimpleNotificationService-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.pem",
        "UnsubscribeUrl": "https://sns.eu-west-1.amazonaws.com/?Action=Unsubscribe&SubscriptionArn=arn:aws:sns:eu-west-1:000000000000:cloudwatch-alarms:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "MessageAttributes": {}
      }
    }
  ]
}
```

### Cloudwatch 알람 생성
추후 포스트에서 따로 다루어보겠습니다.

### Stress test
```shell
[admin@ip-10-0-0-51 ~]$ sudo yum install -y stress
Loaded plugins: priorities, update-motd, upgrade-helper
amzn-main                                                                                                                        | 2.1 kB  00:00:00     
amzn-updates                                                                                                                     | 3.8 kB  00:00:00     
Resolving Dependencies
--> Running transaction check
---> Package stress.x86_64 0:1.0.4-4.2.amzn1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

========================================================================================================================================================
 Package                          Arch                             Version                                    Repository                           Size
========================================================================================================================================================
Installing:
 stress                           x86_64                           1.0.4-4.2.amzn1                            amzn-main                            38 k

Transaction Summary
========================================================================================================================================================
Install  1 Package

Total download size: 38 k
Installed size: 89 k
Downloading packages:
stress-1.0.4-4.2.amzn1.x86_64.rpm                                                                                                |  38 kB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : stress-1.0.4-4.2.amzn1.x86_64                                                                                                        1/1 
  Verifying  : stress-1.0.4-4.2.amzn1.x86_64                                                                                                        1/1 

Installed:
  stress.x86_64 0:1.0.4-4.2.amzn1                                                                                                                       

Complete!
```

### TODO
1. Cloudwatch 알람 생성
2. Slack Msg 포맷
