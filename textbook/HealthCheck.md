# < Kubernetes Health Check >





# 1. Kubernetes Health Check

## 1) Health Check란 무엇인가?

Health Check는 애플리케이션이 정상적으로 동작하고 있는지 주기적으로 확인하는 방법이다. 이를 통해 애플리케이션의 상태를 모니터링하고, 문제가 발생하면 자동으로 복구할 수 있다.





## 2) Kubernetes에서 Health Check의 중요성

Kubernetes는 컨테이너화된 애플리케이션을 관리하기 위한 오케스트레이션 도구로, Health Check를 통해 다음과 같은 이점을 제공한다:

- **자동 복구**: 문제가 있는 컨테이너를 자동으로 재시작하여 서비스의 가용성을 높인다.
- **트래픽 관리**: 문제가 있는 컨테이너로의 트래픽을 차단하여 사용자에게 영향을 주지 않도록 한다.
- **애플리케이션 상태 관리**: 애플리케이션의 상태를 지속적으로 모니터링하여 안정적인 서비스를 제공한다.

------





# 2. Health Check의 종류

## 1) Liveness Probe

Liveness Probe는 컨테이너가 살아 있는지 확인한다. 만약 Liveness Probe가 실패하면, Kubernetes는 해당 컨테이너를 재시작한다.

## 2) Readiness Probe

Readiness Probe는 컨테이너가 준비되었는지 확인한다. Readiness Probe가 실패하면, 해당 컨테이너로의 트래픽을 중단한다. 이는 애플리케이션이 초기화되거나 의존 서비스가 준비되지 않았을 때 유용하다.

## 3) Startup Probe

Startup Probe는 애플리케이션이 초기화되었는지 확인한다. 초기화 시간이 긴 애플리케이션에 유용하며, 설정된 시간 내에 성공하지 못하면 컨테이너를 재시작한다.



## 4) Probe 설정 항목별 설명

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: liveness-sample
spec:
  containers:
  - name: nginx
    image: nginx:latest
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 3
      timeoutSeconds: 1
      failureThreshold: 3
      successThreshold: 1
```



* **httpGet**: HTTP GET 요청을 통해 컨테이너의 특정 엔드포인트 상태를 확인한다.

```
httpGet:
  path: /
  port: 80
```

* **tcpSocket**: 지정한 포트로 TCP 연결을 시도하여 컨테이너의 상태를 확인한다.

```
tcpSocket:
  port: 80
```

* **exec**: 컨테이너 내에서 명령어를 실행하여 상태를 확인한다.

```
exec:
  command:
  - cat
  - /tmp/healthy
```

- **initialDelaySeconds**: 컨테이너 시작 후 처음 Probe를 실행하기 전 대기 시간을 설정한다. (기본값: 0)
- **periodSeconds**: Probe를 주기적으로 실행하는 간격을 설정한다. (기본값: 10)
- **timeoutSeconds**: Probe 실행 시 응답을 기다리는 최대 시간을 설정한다. (기본값 1s)
- **successThreshold**: Probe가 연속적으로 성공해야 하는 횟수를 설정한다. (기본값 1)
- **failureThreshold**: Probe가 연속적으로 실패해야 하는 횟수를 설정한다. (기본값 3)





# 3. Liveness Probe 실습

NGINX를 사용하여 간단한 liveness Probe를 설정하는 방법을 살펴보자.



## 1) Liveness Probe 기본

```sh

$ cat <<EOF | ku create -f -
apiVersion: v1
kind: Pod
metadata:
  name: liveness-sample
spec:
  containers:
  - name: nginx
    image: nginx:latest
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 3
EOF


$ ku get pod liveness-sample
NAME              READY   STATUS    RESTARTS   AGE
liveness-sample   1/1     Running   0          11s


```

* 위 yaml 은 probe 가 3초에 한번씩 check 를 수행
* 만약 localhost:80 수행문장이 연속으로 3번(default) 실패한다면 해당 Container 는 재기동됨



describe 명령으로 확인해 보자.

```sh

$ ku describe pod liveness-sample
...
Containers:
  nginx:
    Container ID:   containerd://3e3b456a7fd1a9906c8b66d42826b54af912d074ace1e3c6b4c9e20c259f3b2b
    Image:          nginx:latest
    Image ID:       docker.io/library/nginx@sha256:0f04e4f646a3f14bf31d8bc8d885b6c951fdcf42589d06845f64d18aec6a3c4d
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 07 Jun 2024 14:03:49 +0000
    Ready:          True
    Restart Count:  0
    Liveness:       http-get http://:80/ delay=3s timeout=1s period=3s #success=1 #failure=3
    Environment:    <none>


```



### Clean Up

```sh
$ ku delete pod liveness-sample

```







## 2) Liveness Probe 실패 사례

```yaml
$ cat <<EOF | ku create -f -
apiVersion: v1
kind: Pod
metadata:
  name: liveness-failure
spec:
  containers:
  - name: nginx
    image: nginx:latest
    livenessProbe:
      httpGet:
        path: /nonexistent
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 3
EOF


```

* 실패되는 사례를 보기 위해서 맞지 않는 path(/nonexistent) 를 표기



계속해서 실패되는

```sh

$ ku get pod liveness-failure
NAME               READY   STATUS    RESTARTS   AGE
liveness-failure   1/1     Running   0          11s


$ ku get pod liveness-failure
NAME               READY   STATUS    RESTARTS     AGE
liveness-failure   1/1     Running   2 (4s ago)   29s

# 

```

계속해서 재기동된다.



describe 명령으로 확인해 보자.

```sh
$ ku describe pod liveness-failure
...
Containers:
  nginx:
    ...
    Ready:          False
    Restart Count:  5
    Liveness:       http-get http://:80/nonexistent delay=3s timeout=1s period=3s #success=1 #failure=3
    ...
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       False
  ContainersReady             False
  PodScheduled                True


  Type     Reason     Age                    From               Message
  ----     ------     ----                   ----               -------
  Normal   Scheduled  3m19s                  default-scheduler  Successfully assigned user03/liveness-failure to ke-bastion03
  Normal   Pulled     3m18s                  kubelet            Successfully pulled image "nginx:latest" in 1.538s (1.538s including waiting)
  Normal   Pulled     3m6s                   kubelet            Successfully pulled image "nginx:latest" in 1.617s (1.617s including waiting)
  Normal   Pulled     2m54s                  kubelet            Successfully pulled image "nginx:latest" in 1.483s (1.483s including waiting)
  Normal   Created    2m54s (x3 over 3m18s)  kubelet            Created container nginx
  Normal   Started    2m54s (x3 over 3m18s)  kubelet            Started container nginx
  Warning  Unhealthy  2m44s (x9 over 3m14s)  kubelet            Liveness probe failed: HTTP probe failed with statuscode: 404
  Normal   Killing    2m44s (x3 over 3m8s)   kubelet            Container nginx failed liveness probe, will be restarted
  Normal   Pulling    2m43s (x4 over 3m19s)  kubelet            Pulling image "nginx:latest"


```

Liveness probe failed 로 인해서 계속 재기동 되고 있음을 확인할 수 있다.



### Clean Up

```sh

$ ku delete pod liveness-failure

```



## 3) command  health check 

command 를 이용해서 health check 하는 방법을 알아보자.

* 특정 파일을 cat 으로 check 

* 실패를 유발하기 위해 30초 후에 파일 삭제

```sh


$ cat <<EOF | ku create -f -
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
      failureThreshold: 3
EOF


```



30초안에는 문제가 없다.

```sh

$ ku get pod liveness-exec
NAME            READY   STATUS    RESTARTS   AGE
liveness-exec   1/1     Running   0          6s


$ ku describe pod liveness-exec
...
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  15s   default-scheduler  Successfully assigned user03/liveness-exec to ke-bastion03
  Normal  Pulling    15s   kubelet            Pulling image "registry.k8s.io/busybox"
  Normal  Pulled     14s   kubelet            Successfully pulled image "registry.k8s.io/busybox" in 1.382s (1.382s including waiting)
  Normal  Created    14s   kubelet            Created container liveness
  Normal  Started    14s   kubelet            Started container liveness



```



하지만 35초 이후부터는 probe가 파일이 없음을 인지한다.

실패를 3번 인식한 45초 이후부터는 Container 재기동 됨을 나타낸다. (Container liveness failed liveness probe, will be restarted)

```sh


$ ku get pod liveness-exec



$ ku describe pod liveness-exec
...
  Type     Reason     Age   From               Message
  ----     ------     ----  ----               -------
  Normal   Scheduled  40s   default-scheduler  Successfully assigned user03/liveness-exec to ke-bastion03
  Normal   Pulling    39s   kubelet            Pulling image "registry.k8s.io/busybox"
  Normal   Pulled     38s   kubelet            Successfully pulled image "registry.k8s.io/busybox" in 1.382s (1.382s including waiting)
  Normal   Created    38s   kubelet            Created container liveness
  Normal   Started    38s   kubelet            Started container liveness
  Warning  Unhealthy  5s    kubelet            Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory



  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  66s                default-scheduler  Successfully assigned user03/liveness-exec to ke-bastion03
  Normal   Pulling    66s                kubelet            Pulling image "registry.k8s.io/busybox"
  Normal   Pulled     65s                kubelet            Successfully pulled image "registry.k8s.io/busybox" in 1.382s (1.382s including waiting)
  Normal   Created    65s                kubelet            Created container liveness
  Normal   Started    65s                kubelet            Started container liveness
  Warning  Unhealthy  22s (x3 over 32s)  kubelet            Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
  Normal   Killing    22s                kubelet            Container liveness failed liveness probe, will be restarted


  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  80s                default-scheduler  Successfully assigned user03/liveness-exec to ke-bastion03
  Normal   Pulled     79s                kubelet            Successfully pulled image "registry.k8s.io/busybox" in 1.382s (1.382s including waiting)
  Warning  Unhealthy  36s (x3 over 46s)  kubelet            Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
  Normal   Killing    36s                kubelet            Container liveness failed liveness probe, will be restarted
  Normal   Pulling    5s (x2 over 80s)   kubelet            Pulling image "registry.k8s.io/busybox"
  Normal   Pulled     4s                 kubelet            Successfully pulled image "registry.k8s.io/busybox" in 1.502s (1.502s including waiting)
  Normal   Created    4s (x2 over 79s)   kubelet            Created container liveness
  Normal   Started    4s (x2 over 79s)   kubelet            Started container liveness

      
```





### Clean Up

```sh
$ ku delete pod liveness-exec

```





# 4. Readiness Probe 실습



```sh
$ cat <<EOF | ku create -f -
apiVersion: v1
kind: Pod
metadata:
  name: readiness-sample
spec:
  containers:
  - name: nginx
    image: nginx:latest
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 3
EOF


$ ku get pod readiness-sample
NAME               READY   STATUS    RESTARTS   AGE
readiness-sample   0/1     Running   0          5s


```

* 위 yaml 은 probe 가 3초에 한번씩 check 를 수행
* 만약 localhost:80 수행문장이 연속으로 3번(default)  실패한다면 해당 컨테이너로의 트래픽을 중단한다.



describe 명령으로 확인해 보자.

```sh
$ ku describe pod readiness-sample
...
Containers:
  nginx:
    ...
    Restart Count:  0
    Readiness:      http-get http://:80/ delay=3s timeout=1s period=3s #success=1 #failure=3

```



### Clean Up

```sh
$ ku delete pod readiness-sample
  ku delete svc readiness-sample-svc

```





# 5. Startup Probe 실습

```sh

$ cat <<EOF | ku create -f -
apiVersion: v1
kind: Pod
metadata:
  name: startup-sample
spec:
  containers:
  - name: nginx
    image: nginx:latest
      
    livenessProbe:
      httpGet:
        path: /
        port: 80
      failureThreshold: 1
      periodSeconds: 10
      
    startupProbe:
      httpGet:
        path: /
        port: 80
      failureThreshold: 30
      periodSeconds: 10
EOF


```

* startupProbe 설정으로 AP 시작을 완료하는 데 최대 5분(30 * 10 = 300초)의 준비시간이 있음.

* startupProbe 가 한 번 성공하면 livenessProbe 가 대신하여 컨테이너 Health Check 를 시작함.

* startupProbe 가 성공하지 못하면 컨테이너는 300초 후에 종료됨.

  

```sh

$ ku get pod startup-sample
NAME             READY   STATUS    RESTARTS   AGE
startup-sample   1/1     Running   0          15s


$ ku describe pod startup-sample
Containers:
  nginx:
    ...
    Restart Count:  0
    Liveness:       http-get http://:80/ delay=0s timeout=1s period=10s #success=1 #failure=1
    Startup:        http-get http://:80/ delay=0s timeout=1s period=10s #success=1 #failure=30


```



### Clean Up

```sh
$ ku delete pod startup-sample

```





# 6. 마무리

본과정을 통해 Kubernetes의 Health Check 기능을 이해하고 설정 및 모니터링하는 방법에 대해서 학습했다.

Health Check를 적절히 활용하여 안정적이고 고가용성의 애플리케이션을 운영할 수 있다.

