#  < K8s Monitoring Setup >



# 1. 개요

* k3s로 구성된 Kubernetes 클러스터에서 Grafana를 사용하여 모니터링을 설정하고 관리하는 방법을 검토한다.



## 1) 아키텍처

![Integrate Prometheus and Grafana over Kubernetes Cluster](./K8sMonitoring.assets/1594668243636.png)





# 2. kube-prometheus-stack 설치

* `kube-prometheus-stack`은 Prometheus, Grafana, 그리고 필요한 메트릭 수집기를 포함하는 Helm 차트임

* kube-prometheus-stack Helm Chart 구성도

  * Prometheus
  * Grafana
  * 메트릭 수집기
    * kube-state-metrics
    * node-exporter

  



## 1) [참고] Helm 설치(설치되지 않은 경우)

```sh

$ curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash

```





## 2) prometheus CRD 설치



```sh


# CRD 설치
$
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/example/prometheus-operator-crd/monitoring.coreos.com_alertmanagers.yaml
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/example/prometheus-operator-crd/monitoring.coreos.com_podmonitors.yaml
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/example/prometheus-operator-crd/monitoring.coreos.com_probes.yaml
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/example/prometheus-operator-crd/monitoring.coreos.com_prometheuses.yaml
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/example/prometheus-operator-crd/monitoring.coreos.com_prometheusrules.yaml
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/example/prometheus-operator-crd/monitoring.coreos.com_servicemonitors.yaml
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/example/prometheus-operator-crd/monitoring.coreos.com_thanosrulers.yaml


customresourcedefinition.apiextensions.k8s.io/alertmanagers.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/podmonitors.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/probes.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/prometheuses.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/prometheusrules.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/servicemonitors.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/thanosrulers.monitoring.coreos.com serverside-applied



## 확인
$ kubectl get crds | grep monitoring.coreos.com
alertmanagers.monitoring.coreos.com        2024-06-08T11:20:41Z
podmonitors.monitoring.coreos.com          2024-06-08T11:20:42Z
probes.monitoring.coreos.com               2024-06-08T11:20:42Z
prometheuses.monitoring.coreos.com         2024-06-08T11:20:43Z
prometheusrules.monitoring.coreos.com      2024-06-08T11:20:44Z
servicemonitors.monitoring.coreos.com      2024-06-08T11:20:44Z
thanosrulers.monitoring.coreos.com         2024-06-08T11:20:45Z


```





## 2) Helm 저장소 추가 및 업데이트

```sh
$ 
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```





## 3) Helm values.yaml 확인

```sh
$ helm search repo kube-prometheus-stack
NAME                                            CHART VERSION   APP VERSION     DESCRIPTION
prometheus-community/kube-prometheus-stack      60.0.0          v0.74.0         kube-prometheus-stack collects Kubernetes manif...



# values 확인
$ helm show values prometheus-community/kube-prometheus-stack



$ mkdir -p ~/helm/charts
  cd ~/helm/charts


# chart download (fetch)
$ helm fetch prometheus-community/kube-prometheus-stack

$ ll 
-rw-r--r-- 1 ktdseduuser ktdseduuser 513880 Jun  8 11:21 kube-prometheus-stack-60.0.1.tgz

# 압축해지
$ tar -xzvf kube-prometheus-stack-60.0.1.tgz

$ cd ~/helm/charts/kube-prometheus-stack

# values.yaml 확인
$ vi values.yaml


```









## 4) kube-prometheus-stack 설치



### install

```sh
# NS 생성
$ kubectl create ns monitoring


# 4.217.252.117 IP를 본인VM IP로 변경 필요



# kube-prometheus-stack 설치

$ helm -n monitoring install prometheus prometheus-community/kube-prometheus-stack \
  --set alertmanager.enabled=false \
  --set grafana.ingress.enabled=true \
  --set grafana.ingress.hosts[0]=grafana.4.217.252.117.nip.io \
  --set prometheus.ingress.enabled=true \
  --set prometheus.ingress.hosts[0]=prometheus.4.217.252.117.nip.io \
  --dry-run=true


############
# 실제 설치시는 dry-run 을 제외하고 실행한다.


############
NAME: prometheus
LAST DEPLOYED: Sat Jun  8 11:27:08 2024
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace monitoring get pods -l "release=prometheus"



# 확인
$ helm -n monitoring ls
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
prometheus      monitoring      1               2024-06-08 11:27:08.328981885 +0000 UTC deployed        kube-prometheus-stack-60.0.1    v0.74.0


# 삭제시...
$ helm -n monitoring delete prometheus


```





### upgrade

```sh



$ helm -n monitoring ls
song@dio-bastion01:~/helm/charts/kube-prometheus-stack$ helm -n monitoring ls
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
prometheus      monitoring      1               2024-07-05 01:32:41.73515446 +0000 UTC  deployed        kube-prometheus-stack-61.2.0    v0.75.0



$ helm -n monitoring upgrade prometheus prometheus-community/prometheus \
  --set alertmanager.enabled=false \
  --set grafana.ingress.enabled=true \
  --set grafana.ingress.hosts[0]=grafana.diopro.duckdns.org \
  --set prometheus.ingress.enabled=true \
  --set prometheus.ingress.hosts[0]=prometheus.diopro.duckdns.org \
  --set prometheus.prometheusSpec.additionalScrapeConfigs='
    - job_name: "event_exporter"
      static_configs:
      - targets: ["event-exporter:9102"]'



Release "prometheus" has been upgraded. Happy Helming!
NAME: prometheus
LAST DEPLOYED: Fri Jul  5 05:37:41 2024
NAMESPACE: monitoring
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
The Prometheus server can be accessed via port 80 on the following DNS name from within your cluster:
prometheus-server.monitoring.svc.cluster.local


Get the Prometheus server URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace monitoring -l "app.kubernetes.io/name=prometheus,app.kubernetes.io/instance=prometheus" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace monitoring port-forward $POD_NAME 9090


#################################################################################
######   WARNING: Pod Security Policy has been disabled by default since    #####
######            it deprecated after k8s 1.25+. use                        #####
######            (index .Values "prometheus-node-exporter" "rbac"          #####
###### .          "pspEnabled") with (index .Values                         #####
######            "prometheus-node-exporter" "rbac" "pspAnnotations")       #####
######            in case you still need it.                                #####
#################################################################################


The Prometheus PushGateway can be accessed via port 9091 on the following DNS name from within your cluster:
prometheus-prometheus-pushgateway.monitoring.svc.cluster.local


Get the PushGateway URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace monitoring -l "app=prometheus-pushgateway,component=pushgateway" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace monitoring port-forward $POD_NAME 9091

For more information on running Prometheus, visit:
https://prometheus.io/








$ helm -n monitoring ls

NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
prometheus      monitoring      2               2024-07-05 05:37:41.398442516 +0000 UTC deployed        prometheus-25.22.0      v2.53.0



      
```







# 3. UI 접속

Helm 차트를 통해 Grafana가 설치되었으므로, 이를 구성하고 접근하는 방법을 설정해야 한다.



## 1) **Prometheus/Grafana  UI 확인**

domain 확인

```sh
# domain 확인
$ kubectl get ingress -n monitoring
NAME                                    CLASS     HOSTS                             ADDRESS    PORTS   AGE
prometheus-kube-prometheus-prometheus   traefik   prometheus.4.217.252.117.nip.io   10.0.0.9   80      46s
prometheus-grafana                      traefik   grafana.4.217.252.117.nip.io      10.0.0.9   80      46s


```



## 2) **Prometheus/Grafana UI 접속**

domain 확인

```sh


http://prometheus.4.217.252.117.nip.io/


http://grafana.4.217.252.117.nip.io/

```





## 3) **Grafana password 확인**

```sh

# Grafana password 확인
$ kubectl get secret -n monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

prom-operator

# ID / PASS
# admin / prom-operator

```





# 4. Prometheus 



## 1) 접속URL

```

http://prometheus.4.217.252.117.nip.io/
```



## 2) Target

* Prometheus가 모니터링하고 있는 모든 엔드포인트를 표기함
* Prometheus는 `prometheus.yml` 파일의 `scrape_configs` 섹션에서 설정된 대상들을 스크랩함

* 메뉴
  * Status > Target
* Targets의 주요 기능
  - **모니터링 상태 확인**
    - Prometheus가 대상의 메트릭을 정상적으로 수집하고 있는지 확인
    - 특정 대상이 `UP` 상태인지, `DOWN` 상태인지, 스크랩에 실패한 이유는 무엇인지 등을 파악
  - **문제 해결**
    - 어떤 대상에서 문제가 발생했는지, 스크랩에 실패한 이유는 무엇인지 확인
  - **라벨 및 메트릭 필터링**
    - 각 대상에 적용된 라벨을 통해 메트릭을 필터링하고 구체적인 데이터를 분석



## 3) Graph

* Prometheus에서 저장된 매트릭 데이터를 시각화 하여 보여준다.
* PromQL 을 직접 쿼리하여 결과를 확인할 수 있다.



### PromQL 예제

```sh
# 단순메트릭 조회
up

# 특정 매티릭 필터링
up{job="node-exporter"}
up{job="node-exporter", instance="10.0.0.5:9100"}


# container_memory_usage_bytes 예제
container_memory_usage_bytes


# 필터링
container_memory_usage_bytes{container="userlist"}
container_memory_usage_bytes{namespace="kube-system"}


```





# 5. Grafana



## 1) 접속URL

```

http://grafana.4.217.252.117.nip.io/

```



## 2) Compute Resource

### (1) Namespace (Pods)

* 메뉴 : Home > Dashboards > Kubernetes / Compute Resources / Namespace (Pods)
* POD 별 리소스(CPU/Memory)의 사용량을 확인 가능
* 특정 POD 를 클릭하면 POD 만 볼수 있는 별도의 모니터링 화면으로 이동됨

![image-20240607170545283](./K8sMonitoring.assets/image-20240607170545283.png)

* 본 매트릭 지표는 Prometheus 에서 지표를 집계하는 단위가 기준으로 그래프를 보여준다.
* 일반적으로 집계는 30초에서 1분단위로 집계를 한다.
* 그러므로 순간적으로 매트릭이 치솟는 경우는 집계내역에서 놓칠 수 있음을 고려해야 한다. 



### (2) Namespace (Workloads)

* 메뉴 : Home > Dashboards > Kubernetes / Compute Resources / Namespace (Pods)
* Workloads 별 리소스(CPU/Memory)의 사용량을 확인 가능
* 개별 POD가 아닌 POD 를 배포하는 Workload 단위로 리소스를 확인 함

![image-20240607170744281](./K8sMonitoring.assets/image-20240607170744281.png)

 



## 3) Node Exporter

### (1) Nodes

* 메뉴 : Home > Dashboards > Node Exporter / Nodes
* Node별 CPU, Memory, Disk 사용량을 확인한다.

![image-20240607171207952](./K8sMonitoring.assets/image-20240607171207952.png)

* 실제 운영 환경에서도 POD 들이 과도하게 Scheduling 될때가 자주 발생함.
* 이때 Node상태를 확인하는 중요한 모니터링이 된다.



### (2) Cluster

* 메뉴 : Home > Dashboards > Node Exporter / USE Method / Cluster
* Cluster 전체 관점에서 리소스(CPU, Memory, Disk) 사용량을 확인한다.





## 4) CoreDNS

* 메뉴 : Home > Dashboards > 
* CoreDNSCoreDNS는 Kubernetes의 기본 DNS 서버로, 클러스터 내 서비스 디스커버리 및 DNS 이름 해석을 담당
* 클러스터의 안정성과 네트워크 성능을 확인하기 위해 CoreDNS의 성능과 상태를 모니터링 수행

![image-20240607172013854](./K8sMonitoring.assets/image-20240607172013854.png)









# 6. Event exporter





## Ver1.0(google-containers)



```sh

$ cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: event-exporter
  namespace: monitoring
  labels:
    app: event-exporter
spec:
  replicas: 1
  selector:
    matchLabels:
      app: event-exporter
  template:
    metadata:
      labels:
        app: event-exporter
    spec:
      containers:
        - name: event-exporter
          image: gcr.io/google-containers/event-exporter:v0.3.0
          args:
            - "--source=kubernetes:https://kubernetes.default.svc"
            - "--sink=prometheus:https://prometheus-kube-prometheus-prometheus.monitoring.svc.cluster.local:9090"
          env:
            - name: KUBERNETES_SERVICE_HOST
              value: "kubernetes.default.svc"
            - name: KUBERNETES_SERVICE_PORT
              value: "443"
EOF



```





### clean up

```sh
$ kubectl -n monitoring get Deployment

$ kubectl -n monitoring  delete Deployment event-exporter

```









## Ver2.0(caicloud)



참고 : https://github.com/caicloud/event_exporter



### (1) deploy 파일 설정

```sh
$ mkdir -p /home/song/song/event-exporter
  cd /home/song/song/event-exporter


$ cat > deploy
--

--
```



```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: event-exporter
  labels:
    name: event-exporter

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    name: event-exporter
  name: event-exporter
subjects:
  - kind: ServiceAccount
    name: event-exporter
    namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: view

---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    name: event-exporter
  name: event-exporter
spec:
  replicas: 1
  revisionHistoryLimit: 2
  selector:
    matchLabels:
      app: event-exporter
  strategy:
    type: RollingUpdate
  template:
    metadata:
      annotations:
        prometheus.io/path: /metrics
        prometheus.io/port: '9102'
        prometheus.io/scrape: 'true'
      labels:
        app: event-exporter
    spec:
      containers:
        - name: event-exporter
          image: 'caicloud/event-exporter:v1.0.0'
          imagePullPolicy: Always
          args:
            - --eventType=Warning
            - --eventType=Normal
          ports:
            - containerPort: 9102
              name: http
          resources:
            limits:
              memory: 100Mi
            requests:
              memory: 40Mi
      serviceAccountName: event-exporter
      terminationGracePeriodSeconds: 30

---
apiVersion: v1
kind: Service
metadata:
  labels:
    name: event-exporter
  name: event-exporter
spec:
  ports:
    - name: http
      port: 9102
      targetPort: 9102
  selector:
    app: event-exporter
```





### (2) 배포

```sh
$ kubectl -n prometheus apply -f deploy.yaml

```





### (3) 확인

curl 수행가능한 특정 POD 내에서...

```sh
$ curl event-exporter:9102/metrics

or

$ curl event-exporter.prometheus.svc:9102/metrics
...

```





```

$ curl event-exporter.prometheus.svc:9102/metrics

# HELP event_exporter_build_info A metric with a constant '1' value labeled by version, branch,build_user,build_date and go_version from which event_exporter was built
# TYPE event_exporter_build_info gauge
event_exporter_build_info{branch="(HEAD",build_date="2020-11-18T02:59:59Z",build_user="Caicloud Authors",go_version="go1.13.6",version="b7605b3"} 1
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 1.48e-05
go_gc_duration_seconds{quantile="0.25"} 1.97e-05
go_gc_duration_seconds{quantile="0.5"} 2.79e-05
go_gc_duration_seconds{quantile="0.75"} 5.0399e-05
go_gc_duration_seconds{quantile="1"} 0.000132
go_gc_duration_seconds_sum 0.000671598
go_gc_duration_seconds_count 17
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 26
# HELP go_info Information about the Go environment.
# TYPE go_info gauge
go_info{version="go1.13.6"} 1
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.
# TYPE go_memstats_alloc_bytes gauge
go_memstats_alloc_bytes 3.7106e+06
# HELP go_memstats_alloc_bytes_total Total number of bytes allocated, even if freed.
# TYPE go_memstats_alloc_bytes_total counter
go_memstats_alloc_bytes_total 4.548e+07
# HELP go_memstats_buck_hash_sys_bytes Number of bytes used by the profiling bucket hash table.
# TYPE go_memstats_buck_hash_sys_bytes gauge
go_memstats_buck_hash_sys_bytes 1.458044e+06
# HELP go_memstats_frees_total Total number of frees.
# TYPE go_memstats_frees_total counter
go_memstats_frees_total 487616
# HELP go_memstats_gc_cpu_fraction The fraction of this program's available CPU time used by the GC since the program started.
# TYPE go_memstats_gc_cpu_fraction gauge
go_memstats_gc_cpu_fraction 5.063004821014084e-06
# HELP go_memstats_gc_sys_bytes Number of bytes used for garbage collection system metadata.
# TYPE go_memstats_gc_sys_bytes gauge
go_memstats_gc_sys_bytes 2.38592e+06
# HELP go_memstats_heap_alloc_bytes Number of heap bytes allocated and still in use.
# TYPE go_memstats_heap_alloc_bytes gauge
go_memstats_heap_alloc_bytes 3.7106e+06
# HELP go_memstats_heap_idle_bytes Number of heap bytes waiting to be used.
# TYPE go_memstats_heap_idle_bytes gauge
go_memstats_heap_idle_bytes 6.0956672e+07
# HELP go_memstats_heap_inuse_bytes Number of heap bytes that are in use.
# TYPE go_memstats_heap_inuse_bytes gauge
go_memstats_heap_inuse_bytes 5.464064e+06
# HELP go_memstats_heap_objects Number of allocated objects.
# TYPE go_memstats_heap_objects gauge
go_memstats_heap_objects 13773
# HELP go_memstats_heap_released_bytes Number of heap bytes released to OS.
# TYPE go_memstats_heap_released_bytes gauge
go_memstats_heap_released_bytes 5.9416576e+07
# HELP go_memstats_heap_sys_bytes Number of heap bytes obtained from system.
# TYPE go_memstats_heap_sys_bytes gauge
go_memstats_heap_sys_bytes 6.6420736e+07
# HELP go_memstats_last_gc_time_seconds Number of seconds since 1970 of last garbage collection.
# TYPE go_memstats_last_gc_time_seconds gauge
go_memstats_last_gc_time_seconds 1.7201897688595521e+09
# HELP go_memstats_lookups_total Total number of pointer lookups.
# TYPE go_memstats_lookups_total counter
go_memstats_lookups_total 0
# HELP go_memstats_mallocs_total Total number of mallocs.
# TYPE go_memstats_mallocs_total counter
go_memstats_mallocs_total 501389
# HELP go_memstats_mcache_inuse_bytes Number of bytes in use by mcache structures.
# TYPE go_memstats_mcache_inuse_bytes gauge
go_memstats_mcache_inuse_bytes 3472
# HELP go_memstats_mcache_sys_bytes Number of bytes used for mcache structures obtained from system.
# TYPE go_memstats_mcache_sys_bytes gauge
go_memstats_mcache_sys_bytes 16384
# HELP go_memstats_mspan_inuse_bytes Number of bytes in use by mspan structures.
# TYPE go_memstats_mspan_inuse_bytes gauge
go_memstats_mspan_inuse_bytes 64192
# HELP go_memstats_mspan_sys_bytes Number of bytes used for mspan structures obtained from system.
# TYPE go_memstats_mspan_sys_bytes gauge
go_memstats_mspan_sys_bytes 98304
# HELP go_memstats_next_gc_bytes Number of heap bytes when next garbage collection will take place.
# TYPE go_memstats_next_gc_bytes gauge
go_memstats_next_gc_bytes 6.693792e+06
# HELP go_memstats_other_sys_bytes Number of bytes used for other system allocations.
# TYPE go_memstats_other_sys_bytes gauge
go_memstats_other_sys_bytes 694652
# HELP go_memstats_stack_inuse_bytes Number of bytes in use by the stack allocator.
# TYPE go_memstats_stack_inuse_bytes gauge
go_memstats_stack_inuse_bytes 688128
# HELP go_memstats_stack_sys_bytes Number of bytes obtained from system for stack allocator.
# TYPE go_memstats_stack_sys_bytes gauge
go_memstats_stack_sys_bytes 688128
# HELP go_memstats_sys_bytes Number of bytes obtained from system.
# TYPE go_memstats_sys_bytes gauge
go_memstats_sys_bytes 7.1762168e+07
# HELP go_threads Number of OS threads created.
# TYPE go_threads gauge
go_threads 10
# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total counter
process_cpu_seconds_total 7.1
# HELP process_max_fds Maximum number of open file descriptors.
# TYPE process_max_fds gauge
process_max_fds 1.048576e+06
# HELP process_open_fds Number of open file descriptors.
# TYPE process_open_fds gauge
process_open_fds 10
# HELP process_resident_memory_bytes Resident memory size in bytes.
# TYPE process_resident_memory_bytes gauge
process_resident_memory_bytes 3.633152e+07
# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.
# TYPE process_start_time_seconds gauge
process_start_time_seconds 1.72018816313e+09
# HELP process_virtual_memory_bytes Virtual memory size in bytes.
# TYPE process_virtual_memory_bytes gauge
process_virtual_memory_bytes 1.37408512e+08
# HELP process_virtual_memory_max_bytes Maximum amount of virtual memory available in bytes.
# TYPE process_virtual_memory_max_bytes gauge
process_virtual_memory_max_bytes -1
# HELP promhttp_metric_handler_requests_in_flight Current number of scrapes being served.
# TYPE promhttp_metric_handler_requests_in_flight gauge
promhttp_metric_handler_requests_in_flight 1
# HELP promhttp_metric_handler_requests_total Total number of scrapes by HTTP status code.
# TYPE promhttp_metric_handler_requests_total counter
promhttp_metric_handler_requests_total{code="200"} 50
promhttp_metric_handler_requests_total{code="500"} 0
promhttp_metric_handler_requests_total{code="503"} 0




```







### (4) prometheus config 수정

```sh

scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9090']
    
  - job_name: event_exporter
    metrics_path: /metrics
    static_configs:
    - targets:
      - http://event-exporter:9102
    
```











### (9) clean up

```sh
$ kubectl -n prometheus get Deployment


$ cd /home/song/song/event-exporter
$ kubectl -n prometheus delete -f deploy.yaml

```









## Ver3.0







https://github.com/resmoio/kubernetes-event-exporter





### 참고



```sh


# monitoring 이 hardcoding 되어 있음.
$ kubectl -n monitoring apply -k https://github.com/resmoio/kubernetes-event-exporter.git

serviceaccount/event-exporter created
clusterrole.rbac.authorization.k8s.io/event-exporter created
clusterrolebinding.rbac.authorization.k8s.io/event-exporter created
configmap/event-exporter-cfg created
deployment.apps/event-exporter created



# 삭제시...
$ kubectl -n monitoring delete -k https://github.com/resmoio/kubernetes-event-exporter.git



```





## Ver4.0

https://pseonghoon.github.io/post/kubernetes-event-exporter/

https://github.com/bitnami/charts/tree/main/bitnami/kubernetes-event-exporter



### (1) install with helm

```sh

$ helm repo add bitnami https://charts.bitnami.com/bitnami

$ helm search repo kubernetes-event-exporter
NAME                                    CHART VERSION   APP VERSION     DESCRIPTION
bitnami/kubernetes-event-exporter       3.2.7           1.7.0           Kubernetes Event Exporter makes it easy to expo...


$ cd /helm/charts

$ helm fetch bitnami/kubernetes-event-exporter


# Namespace생성 
$ kubectl create ns event-exporter



$ cd ~/helm/charts/kubernetes-event-exporter


# install
$ helm --namespace monitoring upgrade event-exporter bitnami/kubernetes-event-exporter \
  --install \
  --set metrics.enabled=true \
  --set metrics.serviceMonitor.enabled=true




############################

  --set metrics.enabled=true \     # <--service 가 자동으로 만들어 진다.
  --version 3.2.7 \     
  --values values.yaml \
  
############################



# 확인
$ helm -n monitoring ls

NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
event-exporter  monitoring      1               2024-07-06 07:06:38.72388407 +0000 UTC  deployed        kubernetes-event-exporter-3.2.7 1.7.0
prometheus      monitoring      1               2024-07-05 10:50:55.593723208 +0000 UTC deployed        kube-prometheus-stack-61.2.0    v0.75.0



# 삭제시...
$ helm -n monitoring delete event-exporter


```



### (3) curl 확인

Curl 이 가능한 POD 에서...

```sh

$ curl http://event-exporter-kubernetes-event-exporter-metrics.monitoring.svc:2112/metrics


...

# HELP build_info A metric with a constant '1' value labeled by version, revision, branch, and goversion from which Kubernetes Event Exporter was built.
# TYPE build_info gauge
build_info{goarch="amd64",goos="linux",goversion="go1.21.12",revision="unknown",version="unknown"} 1
# HELP events_discarded The total number of events discarded because of being older than the maxEventAgeSeconds specified
# TYPE events_discarded counter
events_discarded 0
# HELP events_sent The total number of events processed
# TYPE events_sent counter
events_sent 65
# HELP go_build_info Build information about the main Go module.
# TYPE go_build_info gauge
go_build_info{checksum="",path="github.com/resmoio/kubernetes-event-exporter",version="(devel)"} 1
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 5.4302e-05
go_gc_duration_seconds{quantile="0.25"} 7.92e-05
go_gc_duration_seconds{quantile="0.5"} 9.54e-05
go_gc_duration_seconds{quantile="0.75"} 0.0001197
go_gc_duration_seconds{quantile="1"} 0.000296203
go_gc_duration_seconds_sum 0.002953313
go_gc_duration_seconds_count 27
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 24
# HELP go_info Information about the Go environment.
# TYPE go_info gauge
go_info{version="go1.21.12"} 1
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.
# TYPE go_memstats_alloc_bytes gauge
go_memstats_alloc_bytes 7.70976e+06
# HELP go_memstats_alloc_bytes_total Total number of bytes allocated, even if freed.
# TYPE go_memstats_alloc_bytes_total counter
go_memstats_alloc_bytes_total 5.4430144e+07
# HELP go_memstats_buck_hash_sys_bytes Number of bytes used by the profiling bucket hash table.
# TYPE go_memstats_buck_hash_sys_bytes gauge
go_memstats_buck_hash_sys_bytes 1.4706e+06
# HELP go_memstats_frees_total Total number of frees.
# TYPE go_memstats_frees_total counter
go_memstats_frees_total 269425
# HELP go_memstats_gc_sys_bytes Number of bytes used for garbage collection system metadata.
# TYPE go_memstats_gc_sys_bytes gauge
go_memstats_gc_sys_bytes 4.613096e+06
# HELP go_memstats_heap_alloc_bytes Number of heap bytes allocated and still in use.
# TYPE go_memstats_heap_alloc_bytes gauge
go_memstats_heap_alloc_bytes 7.70976e+06
# HELP go_memstats_heap_idle_bytes Number of heap bytes waiting to be used.
# TYPE go_memstats_heap_idle_bytes gauge
go_memstats_heap_idle_bytes 5.9392e+06
# HELP go_memstats_heap_inuse_bytes Number of heap bytes that are in use.
# TYPE go_memstats_heap_inuse_bytes gauge
go_memstats_heap_inuse_bytes 1.0084352e+07
# HELP go_memstats_heap_objects Number of allocated objects.
# TYPE go_memstats_heap_objects gauge
go_memstats_heap_objects 27174
# HELP go_memstats_heap_released_bytes Number of heap bytes released to OS.
# TYPE go_memstats_heap_released_bytes gauge
go_memstats_heap_released_bytes 5.390336e+06
# HELP go_memstats_heap_sys_bytes Number of heap bytes obtained from system.
# TYPE go_memstats_heap_sys_bytes gauge
go_memstats_heap_sys_bytes 1.6023552e+07
# HELP go_memstats_last_gc_time_seconds Number of seconds since 1970 of last garbage collection.
# TYPE go_memstats_last_gc_time_seconds gauge
go_memstats_last_gc_time_seconds 1.7202443624577742e+09
# HELP go_memstats_lookups_total Total number of pointer lookups.
# TYPE go_memstats_lookups_total counter
go_memstats_lookups_total 0
# HELP go_memstats_mallocs_total Total number of mallocs.
# TYPE go_memstats_mallocs_total counter
go_memstats_mallocs_total 296599
# HELP go_memstats_mcache_inuse_bytes Number of bytes in use by mcache structures.
# TYPE go_memstats_mcache_inuse_bytes gauge
go_memstats_mcache_inuse_bytes 2400
# HELP go_memstats_mcache_sys_bytes Number of bytes used for mcache structures obtained from system.
# TYPE go_memstats_mcache_sys_bytes gauge
go_memstats_mcache_sys_bytes 15600
# HELP go_memstats_mspan_inuse_bytes Number of bytes in use by mspan structures.
# TYPE go_memstats_mspan_inuse_bytes gauge
go_memstats_mspan_inuse_bytes 145320
# HELP go_memstats_mspan_sys_bytes Number of bytes used for mspan structures obtained from system.
# TYPE go_memstats_mspan_sys_bytes gauge
go_memstats_mspan_sys_bytes 195552
# HELP go_memstats_next_gc_bytes Number of heap bytes when next garbage collection will take place.
# TYPE go_memstats_next_gc_bytes gauge
go_memstats_next_gc_bytes 1.3092624e+07
# HELP go_memstats_other_sys_bytes Number of bytes used for other system allocations.
# TYPE go_memstats_other_sys_bytes gauge
go_memstats_other_sys_bytes 624584
# HELP go_memstats_stack_inuse_bytes Number of bytes in use by the stack allocator.
# TYPE go_memstats_stack_inuse_bytes gauge
go_memstats_stack_inuse_bytes 753664
# HELP go_memstats_stack_sys_bytes Number of bytes obtained from system for stack allocator.
# TYPE go_memstats_stack_sys_bytes gauge
go_memstats_stack_sys_bytes 753664
# HELP go_memstats_sys_bytes Number of bytes obtained from system.
# TYPE go_memstats_sys_bytes gauge
go_memstats_sys_bytes 2.3696648e+07
# HELP go_threads Number of OS threads created.
# TYPE go_threads gauge
go_threads 8
# HELP kube_api_read_cache_hits The total number of read requests served from cache when looking up object metadata
# TYPE kube_api_read_cache_hits counter
kube_api_read_cache_hits 31
# HELP kube_api_read_cache_misses The total number of read requests served from kube-apiserver when looking up object metadata
# TYPE kube_api_read_cache_misses counter
kube_api_read_cache_misses 34
# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total counter
process_cpu_seconds_total 1.3
# HELP process_max_fds Maximum number of open file descriptors.
# TYPE process_max_fds gauge
process_max_fds 1.048576e+06
# HELP process_open_fds Number of open file descriptors.
# TYPE process_open_fds gauge
process_open_fds 11
# HELP process_resident_memory_bytes Resident memory size in bytes.
# TYPE process_resident_memory_bytes gauge
process_resident_memory_bytes 6.0633088e+07
# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.
# TYPE process_start_time_seconds gauge
process_start_time_seconds 1.72024190565e+09
# HELP process_virtual_memory_bytes Virtual memory size in bytes.
# TYPE process_virtual_memory_bytes gauge
process_virtual_memory_bytes 1.330855936e+09
# HELP process_virtual_memory_max_bytes Maximum amount of virtual memory available in bytes.
# TYPE process_virtual_memory_max_bytes gauge
process_virtual_memory_max_bytes 1.8446744073709552e+19
# HELP send_event_errors The total number of send event errors
# TYPE send_event_errors counter
send_event_errors 0
# HELP watch_errors The total number of errors received from the informer
# TYPE watch_errors counter
watch_errors 0







```





### (9) clean up

```sh
$ kubectl -n prometheus get Deployment


$ cd /home/song/song/event-exporter
$ kubectl -n prometheus delete -f deploy.yaml



$ kubectl -n event-exporter delete svc event-exporter-svc
  
```















# 7. Prometheus install

prometheus stack 으로 설치하는 prometheus 에서는 scrap 을 추가할 수가 없다.

별도 Prometheus 를 설치하는 방법을 알아보자.







## 1) helm deploy

```sh
# repo추가
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
  helm repo list
  
$ helm repo update




# NS 생성
$ kubectl create ns prometheus





# 설치전 기설치여부 확인
$ helm -n prometheus list
NAME    NAMESPACE       REVISION        UPDATED STATUS  CHART   APP VERSION


$ helm search repo prometheus
...
prometheus-community/prometheus                         25.13.0         v2.49.1         Prometheus is a monitoring system and time seri...
...
prometheus-community/prometheus                         25.21.0         v2.52.0         Prometheus is a monitoring system and time seri...



NAME                                                    CHART VERSION   APP VERSION     DESCRIPTION
prometheus-community/prometheus                         25.22.0         v2.53.0         Prometheus is a monitoring system and time seri...
prometheus-community/prometheus-node-exporter           4.37.0          1.8.1           A Helm chart for prometheus node-exporter






# Fetch
$ mkdir -p ~/helm/charts/
  cd ~/helm/charts/


$ helm fetch prometheus-community/prometheus

$ ll
-rw-r--r-- 1 ktdseduuser ktdseduuser 59331 Jun  4 12:39 prometheus-22.6.2.tgz
-rw-r--r-- 1 ktdseduuser ktdseduuser 69825 Sep  2 16:34 prometheus-23.4.0.tgz
-rw-r--r-- 1 ubuntu ubuntu 75037 Feb 24 08:45 prometheus-25.13.0.tgz
-rw-r--r-- 1 song song 79422 Jun 14 17:17 prometheus-25.21.0.tgz
-rw-r--r-- 1 song song  79631 Jul  5 13:50 prometheus-25.22.0.tgz





$ tar -zxvf prometheus-25.22.0.tgz

$ cd ~/helm/charts/prometheus

# helm 실행 dry-run
$ helm -n prometheus install prometheus . \
  --set configmapReload.prometheus.enabled=true \
  --set server.enabled=true \
  --set server.ingress.enabled=true \
  --set server.ingress.hosts[0]=prometheus2.diopro.duckdns.org \
  --set server.persistentVolume.enabled=false \
  --set alertmanager.enabled=false \
  --set kube-state-metrics.enabled=false \
  --set prometheus-node-exporter.enabled=false \
  --set prometheus-pushgateway.enabled=false \
  --dry-run=true > 11.dry-run.yaml


  
#######
  --set server.image.repository=quay.io/prometheus/prometheus \
  --set server.namespaces[0]=kafka \

#######

#######





# helm 실행




# 확인
$ helm -n prometheus list
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
prometheus      kafka           1               2023-06-04 12:35:18.418462244 +0000 UTC deployed        prometheus-22.6.2       v2.44.0

NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
prometheus      kafka           1               2023-09-02 16:35:39.831272244 +0000 UTC deployed        prometheus-23.4.0       v2.46.0

NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
prometheus      kafka           1               2024-02-24 08:48:00.012127268 +0000 UTC deployed        prometheus-25.13.0      v2.49.1

NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
prometheus      kafka           1               2024-06-14 17:18:49.039512168 +0000 UTC deployed        prometheus-25.21.0      v2.52.0


NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
prometheus      prometheus      1               2024-07-05 13:54:59.775484 +0000 UTC    deployed        prometheus-25.22.0      v2.53.0



## 확인
$ helm -n prometheus status prometheus
$ helm -n prometheus get all prometheus


## 삭제시...
$ helm -n prometheus delete prometheus




```



### [Troble Shooting] CRB 추가 생성

- 권한 오류가 발생하여 확인하니 helm chart 에 의해서 당연히  설치되어야 할 권한이 생기지 않았다.
- helm chart 오류인듯 하다.
- 아래와 같이 수동으로 생성한다.

```sh
$ kubectl -n kafka apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    component: "server"
    app: prometheus
    release: prometheus
    chart: prometheus-15.10.1
    heritage: Helm
  name: prometheus-server
subjects:
  - kind: ServiceAccount
    name: prometheus-server
    namespace: kafka
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus-server
EOF


$ kubectl -n prometheus get ClusterRoleBinding | grep prometheus

prometheus-server                                        ClusterRole/prometheus-server                                        112s


```







## 3) prometheus 확인 

```sh
# pod 확인
$ kubectl -n prometheus get pod 
NAME                                          READY   STATUS    RESTARTS      AGE
prometheus-server-5b5d787f8d-rb8zz            1/1     Running   0             4m36s
---
prometheus-server-6676478584-cdqlc            2/2     Running   0          87s
---
prometheus-server-8476c8485-qn8cv             1/2     Running   0             18s
---
prometheus-server-58d88565d9-4wdh8   2/2     Running   0          2m40s





# pod log 확인
$ kubectl -n prometheus logs -f deploy/prometheus-server



# svc 확인
$ kubectl -n prometheus  get svc
NAME                                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                               AGE
...
prometheus-server                     ClusterIP   10.43.124.104   <none>        80/TCP                                117s
---
prometheus-server                     ClusterIP   10.43.3.91      <none>        80/TCP                                         118s
---
prometheus-server                     ClusterIP   10.43.156.58   <none>        80/TCP                                         39s




# ClusterRoleBinding 확인
$ kubectl -n prometheus  get ClusterRoleBinding prometheus-server

NAME                ROLE                            AGE
prometheus-server   ClusterRole/prometheus-server   4m28s


$ kubectl -n prometheus get ClusterRoleBinding | grep prometheus
prometheus-server                                        ClusterRole/prometheus-server                                        3m52s


```





## 4) ingress

helm  install 시 생성했으므로 추가로 생성할 필요 없음.  

````sh
$ cd ~/githubrepo/ktds-edu-kafka

$ cat ./kafka/strimzi/monitoring/21.prometheus-ingress.yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: prometheus-ingress
spec:
  ingressClassName: traefik
  rules:
  - host: "prometheus.kafka.20.249.174.177.nip.io"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: prometheus-server
            port:
              number: 80


$ kubectl -n kafka apply -f ./kafka/strimzi/monitoring/21.prometheus-ingress.yaml

````

- 확인
  - URL : http://prometheus.kafka.20.249.174.177.nip.io







# 8. events확인



```sh

# event확인 시간으로 소팅
$ kubectl -n yjsong get events --sort-by=.metadata.creationTimestamp


LAST SEEN   TYPE      REASON              OBJECT                           MESSAGE
38m         Normal    Killing             pod/userlist-9fbfc64bc-c2drh     Stopping container userlist
38m         Normal    SuccessfulCreate    replicaset/userlist-75574dc68d   Created pod: userlist-75574dc68d-nck9r
38m         Normal    ScalingReplicaSet   deployment/userlist              Scaled up replica set userlist-75574dc68d to 1
38m         Normal    Scheduled           pod/userlist-75574dc68d-nck9r    Successfully assigned yjsong/userlist-75574dc68d-nck9r to dio-master02
37m         Normal    Pulling             pod/userlist-75574dc68d-nck9r    Pulling image "ssongman/iuserlist:v1"
37m         Warning   Failed              pod/userlist-75574dc68d-nck9r    Error: ErrImagePull
37m         Warning   Failed              pod/userlist-75574dc68d-nck9r    Failed to pull image "ssongman/iuserlist:v1": failed to pull and unpack image "docker.io/ssongman/iuserlist:v1": failed to resolve reference "docker.io/ssongman/iuserlist:v1": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
38m         Warning   Failed              pod/userlist-75574dc68d-nck9r    Error: ImagePullBackOff
38m         Normal    BackOff             pod/userlist-75574dc68d-nck9r    Back-off pulling image "ssongman/iuserlist:v1"
37m         Normal    Scheduled           pod/userlist-8d74d58d8-tcslb     Successfully assigned yjsong/userlist-8d74d58d8-tcslb to dio-master03
37m         Normal    SuccessfulCreate    replicaset/userlist-8d74d58d8    Created pod: userlist-8d74d58d8-tcslb
37m         Normal    ScalingReplicaSet   deployment/userlist              Scaled up replica set userlist-8d74d58d8 to 1
37m         Normal    Pulled              pod/userlist-8d74d58d8-tcslb     Container image "ssongman/userlist:v1" already present on machine
37m         Normal    Created             pod/userlist-8d74d58d8-tcslb     Created container userlist
37m         Normal    Started             pod/userlist-8d74d58d8-tcslb     Started container userlist




### 파드 삭제
$ kubectl -n yjsong delete pod userlist-8d74d58d8-tcslb



# event확인
$ kubectl -n yjsong get events --sort-by=.metadata.creationTimestamp
LAST SEEN   TYPE      REASON                   OBJECT                           MESSAGE
39m         Normal    Killing                  pod/userlist-9fbfc64bc-c2drh     Stopping container userlist
38m         Normal    SuccessfulCreate         replicaset/userlist-75574dc68d   Created pod: userlist-75574dc68d-nck9r
38m         Normal    ScalingReplicaSet        deployment/userlist              Scaled up replica set userlist-75574dc68d to 1
38m         Normal    Scheduled                pod/userlist-75574dc68d-nck9r    Successfully assigned yjsong/userlist-75574dc68d-nck9r to dio-master02
38m         Normal    Pulling                  pod/userlist-75574dc68d-nck9r    Pulling image "ssongman/iuserlist:v1"
38m         Warning   Failed                   pod/userlist-75574dc68d-nck9r    Failed to pull image "ssongman/iuserlist:v1": failed to pull and unpack image "docker.io/ssongman/iuserlist:v1": failed to resolve reference "docker.io/ssongman/iuserlist:v1": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
38m         Warning   Failed                   pod/userlist-75574dc68d-nck9r    Error: ErrImagePull
38m         Normal    BackOff                  pod/userlist-75574dc68d-nck9r    Back-off pulling image "ssongman/iuserlist:v1"
38m         Warning   Failed                   pod/userlist-75574dc68d-nck9r    Error: ImagePullBackOff
38m         Normal    SuccessfulCreate         replicaset/userlist-8d74d58d8    Created pod: userlist-8d74d58d8-tcslb
38m         Normal    Scheduled                pod/userlist-8d74d58d8-tcslb     Successfully assigned yjsong/userlist-8d74d58d8-tcslb to dio-master03
38m         Normal    ScalingReplicaSet        deployment/userlist              Scaled up replica set userlist-8d74d58d8 to 1
38m         Normal    Started                  pod/userlist-8d74d58d8-tcslb     Started container userlist
38m         Normal    Created                  pod/userlist-8d74d58d8-tcslb     Created container userlist
38m         Normal    Pulled                   pod/userlist-8d74d58d8-tcslb     Container image "ssongman/userlist:v1" already present on machine
31s         Normal    Scheduled                pod/userlist-8d74d58d8-xl5xc     Successfully assigned yjsong/userlist-8d74d58d8-xl5xc to dio-master03
32s         Normal    SuccessfulCreate         replicaset/userlist-8d74d58d8    Created pod: userlist-8d74d58d8-xl5xc
32s         Normal    Killing                  pod/userlist-8d74d58d8-tcslb     Stopping container userlist
32s         Warning   FailedToUpdateEndpoint   endpoints/userlist-svc           Failed to update endpoint yjsong/userlist-svc: Operation cannot be fulfilled on endpoints "userlist-svc": the object has been modified; please apply your changes to the latest version and try again
31s         Normal    Pulled                   pod/userlist-8d74d58d8-xl5xc     Container image "ssongman/userlist:v1" already present on machine
31s         Normal    Created                  pod/userlist-8d74d58d8-xl5xc     Created container userlist
31s         Normal    Started                  pod/userlist-8d74d58d8-xl5xc     Started container userlist

```





# 11. elasticsearch kibana



### helm search

```sh


$ kubectl create ns elastic

$ helm search repo elasticsearch
NAME                                                    CHART VERSION   APP VERSION     DESCRIPTION
bitnami/elasticsearch                                   21.2.8          8.14.2          Elasticsearch is a distributed search and analy...
gitlab/fluentd-elasticsearch                            6.2.8           2.8.0           A Fluentd Helm chart for Kubernetes with Elasti...
prometheus-community/prometheus-elasticsearch-e...      6.0.0           v1.7.0          Elasticsearch stats exporter for Prometheus
bitnami/dataplatform-bp2                                12.0.5          1.0.1           DEPRECATED This Helm chart can be used for the ...
bitnami/kibana                                          11.2.10         8.14.2          Kibana is an open source, browser based analyti...
song@dio-bastion01:~$



$ cd ~/helm/charts

$ helm fetch bitnami/elasticsearch

```



### helm install

```sh



$ helm -n elastic install elasticsearch bitnami/elasticsearch \
    --set global.kibanaEnabled=true \
    --set master.persistence.enabled=false \
    --set master.replicaCount=1 \
    --set master.persistence.enabled=false \
    --set data.replicaCount=1 \
    --set data.persistence.enabled=false \
    --set coordinating.replicaCount=1 \
    --set kibana.persistence.enabled=false \
    --set kibana.ingress.enabled=true \
    --set kibana.ingress.hostname=kibana.diopro.duckdns.org \
    --set kibana.ingress.ingressClassName=traefik \
    --dry-run=true > 12.dry-run.yaml
    
    


###########################
    --set ingress.enabled=true \
    --set ingress.hostname=elasticsearch.diopro.duckdns.org
    --set ingress.ingressClassName=traefik \

###########################

$ diff 11.dry-run.yaml 12.dry-run.yaml

###########################


```





## kibana

```


http://kibana.diopro.duckdns.org






```





## event exporter setting



```




apiVersion: v1
data:
  config.yaml: |
    leaderElection: {}
    logFormat: pretty
    logLevel: debug
    receivers:
      - name: dump
        stdout: {}
      - name: "slack"
        slack:
          token: "xxxxxxxxxxxxxxxx"
          channel: "#kube-event"
          message: "Received a Kubernetes Event {{ .Message}}"
          username: "kube-event-exporter"
          fields:
            message: "{{ .Message }}"
            namespace: "{{ .Namespace }}"
            reason: "{{ .Reason }}"
            object: "{{ .Namespace }}"
      - name: "elasticsearch"
        elasticsearch:
          hosts:
            - "http://elasticsearch.elastic.svc:9200"
          username: "elastic"
          index: "kube-events"
          indexFormat: "kube-events-{2006-01-02}"
          useEventID: true
    route:
      routes:
      - match:
        - receiver: dump
      - match:
          - kind: "Pod|Deployment|ReplicaSet"
            receiver: "slack"
      - match:
          - receiver: "elasticsearch"
kind: ConfigMap
metadata:
  annotations:
    meta.helm.sh/release-name: event-exporter
    meta.helm.sh/release-namespace: monitoring
  creationTimestamp: "2024-07-06T07:06:39Z"
  labels:
    app.kubernetes.io/instance: event-exporter
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: kubernetes-event-exporter
    app.kubernetes.io/version: 1.7.0
    helm.sh/chart: kubernetes-event-exporter-3.2.7
  name: event-exporter-kubernetes-event-exporter
  namespace: monitoring
  resourceVersion: "10588650"
  uid: db43532d-aad0-4154-9a05-abc33323c700
  
  
  
```

