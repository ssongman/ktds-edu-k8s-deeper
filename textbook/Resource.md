

# Kubernetes 효율적인 리소스 관리




## 1. 리소스 관리 개념 이해



### Kubernetes에서의 리소스 관리란?

Kubernetes에서 리소스 관리란 클러스터 내의 CPU, 메모리와 같은 컴퓨팅 자원을 효율적으로 사용하도록 관리하는 것을 의미한다. 이를 통해 애플리케이션이 안정적이고 예측 가능하게 동작하도록 보장한다.



### 리소스 요청과 제한

- **리소스 요청 (Resource Requests)**
  - 컨테이너가 안정적으로 실행되기 위해 필요한 최소한의 리소스를 지정
  - requests.memory 값의 여유 메모리가 있는 노드를 선택하여 스케쥴링 함
  - Pod 실행될때 해당 값만큼 점유하지는 않으며 실제 실행되는 값과는 차이가 있음

- **리소스 제한 (Resource Limits)**
  - 컨테이너가 사용할 수 있는 최대 리소스를 지정
  - 컨테이너 내 메모리 점유를 제한하므로써 다른 컨테이너 및 다른 POD, Node 에 영향을 미치지 않도록 함




#### 예제

```sh

apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: demo-container
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"

```

- requests.memory가 64Mi 이므로 64Mi의 여유가 있는 Node 에 POD 를 실행 시킨다.
- limits.memory 128Mi 이므로 nginx Container 가 128Mi 보다 커지지 않도록 제한한다.
- 만약 이보다 많은 메모리를 사용하고 하는 경우 OOM 이 발생하고 Container 는 재시작된다.









# 2. limit / Request 확인



## 1)  Request 확인

Request memory 를 좀더 정확히 이해하기 위해서 




#### nginx sample

```sh

# nginx  POD 실행
$ cat <<EOF | ku apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: demo-container
    image: nginx 
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
EOF


```



메모리 실제 사용량 확인

```sh
# POD 
$ kubectl  top pod
NAME                       CPU(cores)   MEMORY(bytes)
resource-demo              0m           2Mi

```



* 64Mi 만큼의 여유 공간의 Node 를 검사후 POD 가 실행됨
* 실제 사용량은 2Mi 임





## 2) limit 확인

`limits.memory`로 설정된 값을 초과하면 컨테이너는 OOM(Out of Memory) 에러를 발생시키고, Kubernetes는 해당 컨테이너를 재시작 시킨다.

Limit  memory 를 좀더 정확히 이해하기 위해서 실제 실습을 수행해보자.



### 1. **Pod 정의 및 배포**

```sh

# python POD 실행
$ cat <<EOF | ku apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: demo-container
    image: python:alpine3.20
    args:
    - sleep
    - 365d
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
EOF


```



메모리 실제 사용량 확인

```sh
# POD 
$ ku top pod resource-demo
NAME            CPU(cores)   MEMORY(bytes)
resource-demo   0m           0Mi

```

사용량은 거의 없다.





### 2. 메모리 사용량 증가 스크립트 작성

메모리를 인위적으로 증가시키기 위한 Python 스크립트를 작성합니다. 이 스크립트는 큰 배열을 생성하여 메모리를 소모합니다.



`memory_hog.py` 파일을 작성합니다:

```sh


$ ku exec -it resource-demo -- sh


# PS1 변수 설정 (색상이 포함된 설정)
PS1='\[\e[32m\]\u@\h \[\e[33m\]\w\[\e[0m\]\$ '

$ mkdir -p ~/app/src
  cd ~/app/src


$ 
```





```python
import time

def memory_hog():
    a = []
    while True:
        a.append(' ' * 10**6)  # 1MB씩 메모리를 채움
        time.sleep(0.1)  # 0.1초 대기

if __name__ == "__main__":
    memory_hog()
```

### 3. 메모리 사용량 증가 스크립트를 Pod에서 실행

`kubectl exec` 명령을 사용하여 Pod 내에서 스크립트를 실행합니다.

먼저 Pod에 스크립트를 복사합니다:

```
kubectl cp memory_hog.py resource-demo:/memory_hog.py
```

그런 다음 Pod 내에서 스크립트를 실행합니다:

```
kubectl exec -it resource-demo -- python /memory_hog.py
```

### 4. OOM 발생 확인

몇 초 후, `kubectl describe pod resource-demo` 명령을 사용하여 OOM 이벤트를 확인할 수 있습니다:

```

kubectl describe pod resource-demo
```

`kubectl describe pod` 명령의 출력에는 다음과 같은 OOMKilled 메시지가 포함될 것입니다:

```
State:          Waiting
  Reason:       OOMKilled
Last State:     Terminated
  Reason:       OOMKilled
  Exit Code:    137
```

또는 `kubectl get events` 명령을 사용하여 클러스터 이벤트를 확인할 수도 있습니다:

```

kubectl get events --sort-by=.metadata.creationTimestamp
```

### 요약

위 단계를 통해, 컨테이너가 `limits.memory`로 설정된 값을 초과할 때 발생하는 OOM 상황을 실습할 수 있습니다. 이를 통해 수강생들은 Kubernetes의 리소스 관리 기능을 직접 경험하고 이해할 수 있게 됩니다.











### 리소스 쿼터

리소스 쿼터는 네임스페이스 단위로 리소스 사용량을 제한하여, 클러스터의 특정 네임스페이스가 전체 리소스를 독점하지 않도록 한다.



#### 예제

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota-demo
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: "8Gi"
    limits.cpu: "10"
    limits.memory: "16Gi"
```

------





## 2. 리소스 관리 실습



### 리소스 요청과 제한 설정

1. **리소스 요청과 제한 설정**

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: resource-demo
   spec:
     containers:
     - name: demo-container
       image: nginx
       resources:
         requests:
           memory: "64Mi"
           cpu: "250m"
         limits:
           memory: "128Mi"
           cpu: "500m"
   ```

   ```sh
   
   kubectl apply -f resource-demo.yaml
   ```

2. **리소스 쿼터 설정**

   ```yaml
   apiVersion: v1
   kind: ResourceQuota
   metadata:
     name: quota-demo
     namespace: default
   spec:
     hard:
       pods: "10"
       requests.cpu: "4"
       requests.memory: "8Gi"
       limits.cpu: "10"
       limits.memory: "16Gi"
   ```

   ```sh
   bash
   코드 복사
   kubectl apply -f quota-demo.yaml
   ```

### Horizontal Pod Autoscaler (HPA) 사용

HPA는 애플리케이션의 부하에 따라 자동으로 Pod의 수를 조절한다.

1. **HPA 설정 예제**

   ```yaml
   apiVersion: autoscaling/v1
   kind: HorizontalPodAutoscaler
   metadata:
     name: hpa-demo
   spec:
     scaleTargetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: demo-deployment
     minReplicas: 1
     maxReplicas: 10
     targetCPUUtilizationPercentage: 50
   ```

   ```sh
   
   kubectl apply -f hpa-demo.yaml
   ```

2. **Deployment 설정**

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: demo-deployment
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: demo
     template:
       metadata:
         labels:
           app: demo
       spec:
         containers:
         - name: demo-container
           image: nginx
           resources:
             requests:
               memory: "64Mi"
               cpu: "250m"
             limits:
               memory: "128Mi"
               cpu: "500m"
   ```

   ```sh
   
   kubectl apply -f deployment-demo.yaml
   ```

------



## 3. 리소스 관리 모니터링

### Metrics Server 설치

Metrics Server는 Kubernetes 클러스터 내의 리소스 사용량을 모니터링하는 데 사용된다.

1. **Metrics Server 설치**

   ```sh
   
   kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
   ```

2. **리소스 사용량 모니터링**

   ```sh
   kubectl top nodes
   kubectl top pods
   ```

------

### 요약

이번 강의를 통해 Kubernetes에서 리소스를 효율적으로 관리하는 방법을 이해하고, 실습을 통해 직접 설정하고 모니터링하는 방법을 학습했다. 

리소스 요청과 제한, 리소스 쿼터, 그리고 HPA를 사용하여 클러스터 리소스를 효과적으로 관리할 수 있다.