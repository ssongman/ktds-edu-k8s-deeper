

### Kubernetes 효율적인 리소스 관리

------

## 아젠다

1. 리소스 관리 개념 이해
   - Kubernetes에서의 리소스 관리란?
   - 리소스 요청과 제한
   - 리소스 쿼터
2. 리소스 관리 실습
   - 리소스 요청과 제한 설정
   - 리소스 쿼터 설정
   - Horizontal Pod Autoscaler (HPA) 사용
3. 리소스 관리 모니터링
   - Metrics Server 설치
   - 리소스 사용 모니터링

------

## 1. 리소스 관리 개념 이해

### Kubernetes에서의 리소스 관리란?

Kubernetes에서 리소스 관리란 클러스터 내의 CPU, 메모리와 같은 컴퓨팅 자원을 효율적으로 사용하도록 관리하는 것을 의미합니다. 이를 통해 애플리케이션이 안정적이고 예측 가능하게 동작하도록 보장합니다.

### 리소스 요청과 제한

- **리소스 요청 (Resource Requests)**: 컨테이너가 안정적으로 실행되기 위해 필요한 최소한의 리소스를 지정합니다.
- **리소스 제한 (Resource Limits)**: 컨테이너가 사용할 수 있는 최대 리소스를 지정하여 다른 컨테이너에 영향을 미치지 않도록 합니다.

#### 예제

```
yaml코드 복사apiVersion: v1
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

### 리소스 쿼터

리소스 쿼터는 네임스페이스 단위로 리소스 사용량을 제한하여, 클러스터의 특정 네임스페이스가 전체 리소스를 독점하지 않도록 합니다.

#### 예제

```
yaml코드 복사apiVersion: v1
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

   ```
   yaml코드 복사apiVersion: v1
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

   ```
   bash
   코드 복사
   kubectl apply -f resource-demo.yaml
   ```

2. **리소스 쿼터 설정**

   ```
   yaml코드 복사apiVersion: v1
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

   ```
   bash
   코드 복사
   kubectl apply -f quota-demo.yaml
   ```

### Horizontal Pod Autoscaler (HPA) 사용

HPA는 애플리케이션의 부하에 따라 자동으로 Pod의 수를 조절합니다.

1. **HPA 설정 예제**

   ```
   yaml코드 복사apiVersion: autoscaling/v1
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

   ```
   bash
   코드 복사
   kubectl apply -f hpa-demo.yaml
   ```

2. **Deployment 설정**

   ```
   yaml코드 복사apiVersion: apps/v1
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

   ```
   bash
   코드 복사
   kubectl apply -f deployment-demo.yaml
   ```

------

## 3. 리소스 관리 모니터링

### Metrics Server 설치

Metrics Server는 Kubernetes 클러스터 내의 리소스 사용량을 모니터링하는 데 사용됩니다.

1. **Metrics Server 설치**

   ```
   bash
   코드 복사
   kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
   ```

2. **리소스 사용량 모니터링**

   ```
   bash코드 복사kubectl top nodes
   kubectl top pods
   ```

------

### 요약

이번 강의를 통해 Kubernetes에서 리소스를 효율적으로 관리하는 방법을 이해하고, 실습을 통해 직접 설정하고 모니터링하는 방법을 배웠습니다. 리소스 요청과 제한, 리소스 쿼터, 그리고 HPA를 사용하여 클러스터 리소스를 효과적으로 관리할 수 있습니다.