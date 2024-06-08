# ktds-edu-k8s-deeper
> kubernetes 기반 업무에서 활용할 수 있는 활용 Tip 및  Skill Up



본 교육은 Kubernetes 에서의 트래픽 흐름을 좀더 깊이 있게 알아보며 실 업무 환경에서 활용 할 수 있는 POD 관리, Container 관리, 모니터링 등에 대해서 실습 기반으로 학습한다.

문의: 송양종( yj.song@kt.com / ssongmantop@gmail.com )





# K8s 구축 및 Traffic Flow ([바로가기](./textbook/K8sTrafficFlow.md))  

* 파일명 : K8sTrafficFlow.md
* k3s구축
* Cloud에서 Kubernetes Traffic Flow
  * LB --> Node --> Service --> Ingress Controller



# K9s 활용한 Cluster 관리 ([바로가기](./textbook/K9s.md))  

* K9s의 주요 기능 및 장점
* K9S 설치 및 기본화면 이해
* K9s 주요 기능 실습
  * deploy, svc, pod, xray 등



# K8s Monitoring ([바로가기](./textbook/K8sMonitoring.md))

* 파일명 : K8sMonitoring.md
* kube-prometheus-stack 설치
*  Prometheus 확인
  * Target, Graph, PromQL 예제 확인
* Grafana UI / Dashboard 확인
  * CoreDNS, Namespace, Pods, Workdloads, Nodes 등



# POD Health Check 관리 ([바로가기](./textbook/HealthCheck.md))

* 파일명 : HealthCheck.md
* Health Check의 중요성
* Health Check의 종류
* LivenessProbe, ReadinessProbe, StartupProbe 실습



# POD 리소스 관리 ([바로가기](./textbook/Resource.md))

* 파일명 : Resource.md



# Headless Service 활용한 POD 접근 ([바로가기](./textbook/HeadlessService.md))

* 파일명 : HeadlessService.md



# K8s 멀티클러스터링 ([바로가기](./textbook/MultiCluster.md))

* 멀티 클러스터 개요 : 클러스터 확정 및 멀티 클러스터링 필요성
* 사례로 보는 멀티 클러스터 가 필요성 
* Clium 을 활용한 멀티 클러스터링 구조 및 실습



# Init Container ([바로가기](./textbook/InitContainer.md))

* 파일명 : InitContainer.md






















# 1. 시작전에 ( [가이드 문서 보기](./beforebegin/beforebegin.md) )  



## 1) 실습 환경 준비(개인PC)

- mobaXterm 설치
- gitbash 설치
- typora 설치



## 2) 교육문서 Download

- 교육자료 download
- Typora 로 readme.md 파일오픈



## 3) 실습 환경 준비(Cloud)

- 개인 VM 서버 주소 확인 및  KtdsEduCluster Namespace 확인
- SSH (Mobaxterm) 실행
- 실습자료 download






# 2. Class1: kubernetes 맛보기 ( [가이드 문서 보기](./kubernetes/kubernetes.md) )  



## 1) Kubernetes 란 무엇인가?

- k8s 개요

- k8s 이전에는?

- 왜 k8s 가 필요한가?



## 2) [개인VM] Docker 실습

- sample app 실행
- Scale Out
- 부하분산 고민
- clean up



## 3) [개인VM] k3s 설치

- k3s 란?
- vm에 k3s 설치



## 4) [개인Cluster] Kubernetes 실습

- 개인 Namespace 확인
- sample app deploy
  - Deployment
  - Service 
  - Scale Out
  - Ingress



## 5) [EduCluster] Kubernetes 실습

- ktdsEduCluster 접속 설정 변경
- 개인 Namespace 확인
- Sample app deploy
  - Deployment/Service
  - Scale Out
  - Ingress






# 3. Class2: ServiceMesh ( [가이드 문서 보기](./istio/ServiceMesh.md) )  



## 1) Service Mesh 란?



## 2) Istio 란?



## 3) [개인Cluster] 실습

- helm 설치
- helm 을 이용한 Istio 설치
- sample app sidecar inject
  - userlist application을 활용하여 istio sidecar inject 에 대한 이해도를 높인다.



## 4) [EduCluster] 실습

- sample app (bookinfo) install
  - bookinfo application 을 활용한 실습을 통해서 istio 를 이해한다.
  - bookinfo 는 온라인 서점의 단일 카탈로그 항목과 유사한 도서에 대한 정보를 표시하는 app 이다.
- Monitoring
  - Grafana
  - Kiali
  - Jaeger



## 5) [EduCluster] 실습(Traffic control)

- Traffic Shifting
  - 서비스별로 트래픽의 가중치를 조정하므로서 특정 버전에서 다른 버전으로 트래픽을 이동하는 방법을 제어할 수 있다.
- Request Routing
  - 여러 버전의 마이크로서비스로 동적으로 라우팅하는 방법을 확인할 수 있다.
- Fault Injection
  - application 의 복원력을 테스트하기 위해서 결함을 주입할 수 있다.
- Circuit Breaking
  - istio는 Connection pool 과  Load balancing pool 기반의 circuit breaking 기능을 제공한다.


