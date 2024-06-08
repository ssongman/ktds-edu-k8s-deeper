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

* Headless Service 활용한 POD IP 획득
* Java Sample 확인
* Python Sample 확인



# K8s 멀티클러스터링 ([바로가기](./textbook/MultiCluster.md))

* 멀티 클러스터 개요 : 클러스터 확정 및 멀티 클러스터링 필요성
* 사례로 보는 멀티 클러스터 가 필요성 
* Clium 을 활용한 멀티 클러스터링 구조 및 실습



# Init Container ([바로가기](./textbook/InitContainer.md))

* 파일명 : InitContainer.md




