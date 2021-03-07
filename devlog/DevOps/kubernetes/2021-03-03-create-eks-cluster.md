---
layout: post
category: devlog
tags: devops kubernetes
title: "Create EKS Cluster and deploying nginx"
description: >
  EKS 탐방기 시리즈 - 1편
image:
  path: /assets/img/devlog/devops/kubernetes/2021-03-03/eks/main.jpg
---

<!--more-->

* this unordered seed list will be replaced by the toc
{:toc}

***

### 쿠버네티스
![Kubernetes](/assets/img/devlog/devops/kubernetes/2021-03-03/eks/1.png){:.centered}{: style="width: 50%;"}
Kubernetes logo
{:.figcaption}

쿠버네티스는 **컨테이너를 쉽고 빠르게 배포, 확장하고 관리를 자동화해주는 오픈소스 플랫폼** 입니다.<br>
`Docker`가 컨테이너 기반 가상화 플랫폼이라면, 이 컨테이너를 관리하는 것이 쿠버네티스 입니다.<br>

좀 더 자세한 정보를 다음 페이지에서 쉽게 설명해 주시고 계십니다.
- [오픈소스 컨설팅](https://tech.osci.kr/2020/06/06/97465347/)
- [Subicura](https://subicura.com/2019/05/19/kubernetes-basic-1.html)

### kubectl
Kubectl은 쿠버네티스 클러스터를 제어하기 위한 커맨드 라인 도구입니다.<br>
구성을 위해, kubectl 은 config 파일을 $HOME/.kube 에서 찾고, KUBECONFIG 환경 변수를 설정하거나 `--kubeconfig` 플래그를 설정하여 다른 kubeconfig 파일을 지정할 수 있습니다.<br>
- [kubeconfig 파일을 사용하여 클러스터 접근 구성하기](https://kubernetes.io/ko/docs/concepts/configuration/organize-cluster-access-kubeconfig/
{:.note})

#### 설치 <br>
```shell
// 1.8 버전 Kubectl 을 다운로드 했습니다.
$ curl -LO https://storage.googleapis.com/kubernetes- release/release/v1.18.0/bin/linux/amd64/kubectl

// 실행 권한 추가
$ chmod +x ./kubectl

// PATH가 설정된 디렉터리로 옮깁니다.
$ sudo mv ./kubectl /usr/local/bin/kubectl

$ kubectl version --client
```

### eksctl
`Amazon EKS` 클러스터 작업을 위한 명령줄 도구입니다. 

#### 설치 <br>
```shell
// eksctl 설치파일 다운로드
$ curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp 

// PATH가 설정된 디렉터리로 옮깁니다.
$ sudo mv /tmp/eksctl /usr/local/bin

$ eksctl version
```

### Create EKS Cluster
#### Workernode Key
```shell
// Set region
$ aws configure set region ap-northeast-2
$ cd .ssh

// Key 생성
$ ssh-keygen 
```

생성된 공개키를 사용중인 EC2에 리전으로 업로드 합니다.
```shell
$ aws ec2 import-key-pair --key-name "<key-name>" --public-key-material file://~/.ssh/id_rsa.pub
```

#### EKS Cluster와 worker node 생성
```shell
$ eksctl create cluster --name <cluster-name> --version 1.17 --region ap-northeast-2 --nodegroup-name <worker-node-name> --node-type <instanceType> --nodes 1 --nodes-min 1 --nodes-max 1 --ssh-access --ssh-public-key <keyName> --managed 
2021-03-03 13:51:01 [ℹ]  eksctl version 0.38.0
2021-03-03 13:51:01 [ℹ]  using region ap-northeast-2
2021-03-03 13:51:01 [ℹ]  setting availability zones to [ap-northeast-2a ap-northeast-2b ap-northeast-2d]
2021-03-03 13:51:01 [ℹ]  subnets for ap-northeast-2a - public:192.168.0.0/19 private:192.168.96.0/19
2021-03-03 13:51:01 [ℹ]  subnets for ap-northeast-2b - public:192.168.32.0/19 private:192.168.128.0/19
2021-03-03 13:51:01 [ℹ]  subnets for ap-northeast-2d - public:192.168.64.0/19 private:192.168.160.0/19
2021-03-03 13:51:01 [ℹ]  using EC2 key pair %!q(*string=<nil>)
2021-03-03 13:51:01 [ℹ]  using Kubernetes version 1.17
2021-03-03 13:51:01 [ℹ]  creating EKS cluster "cluster" in "ap-northeast-2" region with managed nodes
2021-03-03 13:51:01 [ℹ]  will create 2 separate CloudFormation stacks for cluster itself and the initial managed nodegroup
2021-03-03 13:51:01 [ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=ap-northeast-2 --cluster=cluster'
2021-03-03 13:51:01 [ℹ]  CloudWatch logging will not be enabled for cluster "cluster" in "ap-northeast-2"
2021-03-03 13:51:01 [ℹ]  you can enable it with 'eksctl utils update-cluster-logging --enable-types={SPECIFY-YOUR-LOG-TYPES-HERE (e.g. all)} --region=ap-northeast-2 --cluster=cluster'
2021-03-03 13:51:01 [ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "cluster" in "ap-northeast-2"
2021-03-03 13:51:01 [ℹ]  2 sequential tasks: { create cluster control plane "cluster", 2 sequential sub-tasks: { wait for control plane to become ready, create managed nodegroup "wn" } }
2021-03-03 13:51:01 [ℹ]  building cluster stack "eksctl-cluster-cluster"
2021-03-03 13:51:02 [ℹ]  deploying stack "eksctl-cluster-cluster"
2021-03-03 13:51:02 [ℹ]  waiting for CloudFormation stack "eksctl-cluster-cluster"
2021-03-03 14:05:30 [ℹ]  building managed nodegroup stack "eksctl-cluster-nodegroup-wn"
2021-03-03 14:05:30 [ℹ]  deploying stack "eksctl-cluster-nodegroup-wn"
2021-03-03 14:05:30 [ℹ]  waiting for CloudFormation stack "eksctl-cluster-nodegroup-wn"
2021-03-03 14:09:05 [ℹ]  waiting for the control plane availability...
2021-03-03 14:09:05 [✔]  saved kubeconfig as "/home/admin/.kube/config"
2021-03-03 14:09:05 [ℹ]  no tasks
2021-03-03 14:09:05 [✔]  all EKS cluster resources for "cluster" have been created
2021-03-03 14:09:05 [ℹ]  nodegroup "wn" has 1 node(s)
2021-03-03 14:09:05 [ℹ]  node "ip-192-168-17-251.ap-northeast-2.compute.internal" is ready
2021-03-03 14:09:05 [ℹ]  waiting for at least 1 node(s) to become ready in "wn"
2021-03-03 14:09:05 [ℹ]  nodegroup "wn" has 1 node(s)
2021-03-03 14:09:05 [ℹ]  node "ip-192-168-17-251.ap-northeast-2.compute.internal" is ready
2021-03-03 14:09:06 [ℹ]  kubectl command should work with "/home/admin/.kube/config", try 'kubectl get nodes'
2021-03-03 14:09:06 [✔]  EKS cluster "cluster" in "ap-northeast-2" region is ready
```

```shell
// 생성된 노드 확인
[admin@ip-10-0-0-51 ~]$ kubectl get node
NAME                                                STATUS   ROLES    AGE     VERSION
ip-192-168-17-251.ap-northeast-2.compute.internal   Ready    <none>   2m20s   v1.17.12-eks-7684af
```

### Pod
#### Deployment 배포
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx   # Pod에 label:my-nginx 가 생성되면, selector에 정의된대로 해당 pod만 관리한다.
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx  # Pod에 label:my-nginx를 적용함
    spec:
      containers:
      - name: my-nginx
        image: nginx    # 도커 허브의 nginx (여기에서는 latest를 사용할 것으로 예상)이미지를 실행하는 nginx 컨테이너를 Pod에서 실행함
        ports:
        - containerPort: 80
```

```shell
[admin@ip-10-0-0-51 ~]$ kubectl apply -f nginx-deployment.yaml
deployment.apps/my-nginx created
[admin@ip-10-0-0-51 ~]$ kubectl get pod
NAME                        READY   STATUS              RESTARTS   AGE
my-nginx-75897978cd-6qps2   0/1     ContainerCreating   0          8s
my-nginx-75897978cd-9g9st   0/1     ContainerCreating   0          8s
```

### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx
  type: LoadBalancer  # 생성 타입은 로드밸런서
```

```shell
[admin@ip-10-0-0-51 ~]$ kubectl apply -f nginx-service.yaml
service/my-nginx created
[admin@ip-10-0-0-51 ~]$ kubectl get svc
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP                                                                   PORT(S)        AGE
kubernetes   ClusterIP      10.100.0.1      <none>                                                                        443/TCP        14m
my-nginx     LoadBalancer   10.100.138.30   ab...........................1af-809455680.ap-northeast-2.elb.amazonaws.com   80:32404/TCP   8s
```

위 Loadbalancer `EXTERNAL-IP`의 80번 포트로 접근하면 nginx를 확인할 수 있다.

### Reference
- [오픈소스 컨설팅](https://tech.osci.kr/2020/06/06/97465347/)
- [Subicura](https://subicura.com/2019/05/19/kubernetes-basic-1.html)