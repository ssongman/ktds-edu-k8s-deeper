# < K9s >

> K9s를 이용한 Kubernetes 클러스터 관리





# 1. K9s 소개

## 1) K9s란 무엇인가?

K9s는 Kubernetes 클러스터를 터미널에서 쉽게 관리할 수 있도록 도와주는 오픈 소스 CLI 도구이다. 

직관적인 사용자 인터페이스와 강력한 기능을 제공하여, 복잡한 명령어 없이도 Kubernetes 자원을 효율적으로 관리할 수 있다.



## 2) K9s의 주요 기능 및 장점

* **직관적인 터미널 UI**: 터미널 기반의 UI를 제공하여 Kubernetes 자원을 쉽게 탐색하고 관리할 수 있다.
* **실시간 모니터링**: Kubernetes 클러스터의 자원을 실시간으로 모니터링할 수 있다.
* **편리한 검색 및 필터링**: 자원 검색 및 필터링 기능을 통해 원하는 정보를 빠르게 찾을 수 있다.
* **다양한 자원 관리**: Pod, 서비스, 디플로이먼트 등 다양한 Kubernetes 자원을 관리할 수 있다.







# 2. K9S Setup

## 1) K9S 설치

```sh
# root 권한으로

$ sudo -s

$ mkdir -p ~/temp/k9s
  cd  ~/temp/k9s

# relese 확인
# https://github.com/derailed/k9s/releases

$ wget https://github.com/derailed/k9s/releases/download/v0.32.4/k9s_Linux_amd64.tar.gz
  tar -xzvf k9s_Linux_amd64.tar.gz

$ ll
-rw-r--r-- 1  501 staff    10174 Mar 22  2021 LICENSE
-rw-r--r-- 1  501 staff    40905 Mar 20 19:01 README.md
-rwxr-xr-x 1  501 staff 93470872 Mar 20 19:17 k9s*
-rw-r--r-- 1 root root  29966459 Mar 20 19:21 k9s_Linux_amd64.tar.gz



$ cp ./k9s /usr/local/bin/

$ ll /usr/local/bin/
-rwxr-xr-x  1 root root 60559360 May 15 13:05 k9s*


# 일반 사용자로 전환
$ exit 

```



## 2) K9S 실행

```sh

# 일반 사용자로 전환

# 실행
$ k9s

```

![image-20240607213706737](./K9s.assets/image-20240607213706737.png)



### **기본 화면 이해**

- 터미널 UI가 나타나며, 기본적으로 현재 네임스페이스의 Pod 목록이 표시된다.
- 상단에는 현재 컨텍스트와 네임스페이스 정보가 표시된다.



### **네임스페이스 전환**

- `:ns` 명령어를 입력하고 원하는 네임스페이스를 선택한다.





# 3. K9s 주요 기능 실습



## 1) Pod 관리

**Pod 목록 보기**:

- 네임스페이스를 선택후 `Enter`키를 누르면 해당 NS의 모든 Pod가 표시된다.

![image-20240607220647982](./K9s.assets/image-20240607220647982.png)



**Pod 상세 정보 확인**:

- 특정 Pod를 선택하고 `Enter` 키를 눌러 상세 정보를 확인한다.

**Pod Describe 확인**:

- 특정 Pod를 선택하고 `D` 키를 눌러 상세 정보를 확인한다.

**Pod 내 Shell 실행**:

- 특정 Pod를 선택하고 `S` 키를 눌러 Shell을 실행한다.
- Pod 내에서 다양한 명령을 수행해 볼 수 있다.

**Pod 삭제**:

- 삭제할 Pod를 선택하고 `Ctrl+d` 키를 누른다.

**Pod 로그 확인**:

- 로그를 보고 싶은 Pod를 선택하고 `l` 키를 누른다.
- 로그를 실시간으로 스트리밍하여 볼 수 있다.

![image-20240607220529540](./K9s.assets/image-20240607220529540.png)





## 2) Node

**Node 목록 보기**:

- `:node` 명령어를 입력하면 모든 Node가 표시된다.
- Node 당 CPU / Memory 정보를 확인할 수 있다. 총량과 현재 사용중인 지표를 확인한다.
- 실제 운영하면서 자주 확인하는 지표이다.

![image-20240607215138458](./K9s.assets/image-20240607215138458.png)

**Node 목록 보기**:

- Node 선택후 `Enter`키를 누르면 해당 Node의 모든 Pod가 표시된다.



## 3) deploy

**deployment 목록 보기**:

- `:deploy` 명령어를 입력하면 모든 deployment가 표시된다.
- deployment 선택후 `Enter`키를 누르면 해당 deployment 의 모든 Pod가 표시된다.



## 4) svc

**Service 목록 보기**:

- `:svc`명령어를 입력하면 모든 Service 가 표시된다.
- Service 선택후 `Enter`키를 누르면 해당 Service 의 모든 Pod가 표시된다.





## 6) xray

**xray 목록 보기**:

* `:xray [resource]` 명령어를 입력하면 resource(service,deploy, pod 등) 의 모든 하위 구조가 표시된다.

- `:xray service` 명령어를 입력하면 동Namespace 모든 service들 및 하위 구조가 표시된다.
- 방향키로 이동후 해당 리소스를 클릭하면 상세보기로 변경된다.

![image-20240607215851601](./K9s.assets/image-20240607215851601.png)

- `:xray pods` 명령어를 입력하면 동Namespace 모든 pod들 및 하위 구조가 표시된다.

![image-20240607220321327](./K9s.assets/image-20240607220321327.png)





# 4. 주요 단축키

아래 단축키는 실제 운영환경에서 자주 사용되는 단축키이다.

- `?`: 도움말

- `d`: Describe

- `l`: 로그 보기

- `Ctrl+w`: 와이드 보기

  - Node 창에서 Node IP 등을 확인할때 사용한다.

- `Shift+f`: Port-Forward

- `space`: Mark

- `Ctrl+\`: Mark Clear

- `:q`: Quit

- `y`: YAML 보기

  

  

