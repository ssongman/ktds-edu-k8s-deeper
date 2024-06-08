



# 1. 개요

headless service 를 이용하여 POD들의 IP 를 취득하는 방법을 확인한다.

단, 클러스터 내부에서만 사용 가능하다.



# 2. Headless Service 활용한 IP 획득 분석

일반적으로 Service는 고유의 IP를 가지고 있으며 Load Balancer(Proxy) 역할을 수행한다.

그와 대비하여 Headless Servcie 아래와 같은 특징이 있다.

- IP 가 없으며 직접 POD 로 연결된다.
- Load Balancer 역할을 수행하지 못한다.
- POD 의 IP 를 확인 할 수 있다.



## 1) headless service 생성

아래와 같이 ClusterIP 가 None 기록하는 Service 가 Headless service 이다.

```yaml
kind: Service
apiVersion: v1
metadata:
  name: userlist-svc-hl
  namespace: song
spec:
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 8181
  selector:
    app: userlist
  clusterIP: None
  clusterIPs:
    - None
  type: ClusterIP
  sessionAffinity: None
```



## 2) nslookup 를 활용하여 IP 획득 사례

userlist-svc 로 연결가능한 pod 는 2개일때 일반서비스와 Headless Service가 어떻게 다르게 반응하는지 확인해보자.



- 일반 서비스 IP

```sh
$  nslookup userlist-svc.song.svc.cluster.local
Server:         172.30.0.10
Address:        172.30.0.10:53
 
Name:   userlist-svc.song.svc.cluster.local
Address: 172.30.99.19
```

일반 서비스는 Service 의 IP가 리턴된다.





- Headless Service IP

```sh
$ nslookup userlist-svc-hl.song.svc.cluster.local
Server:         172.30.0.10
Address:        172.30.0.10:53
 
 
Name:   userlist-svc-hl.song.svc.cluster.local
Address: 10.129.6.202
Name:   userlist-svc-hl.song.svc.cluster.local
Address: 10.129.6.207
```

Headless Service 의 경우 각 POD 의 IP 를 반환 받는다.



## 3) java Sample

Java 에서 nslookup 기능과 같이 도메인주소를 이용하여 IP 알아내는 방법을 살펴보자.

InetAddress getAllByName() 함수로 IP 목록을 알아 낼수 있다. 단, 동일 Cluster 내에서만 접근이 가능하다.

아래는 Googling 에서 찾아본 일부 Java 샘플이다.

### Java샘플1

참조링크: https://ery221.tistory.com/4



```java
import java.net.InetAddress;
import java.net.UnknownHostException;
import java.util.Scanner;
 
public class NSLookup {
     public static void main(String args[]) {
 
           Scanner scan = new Scanner(System.in);
           InetAddress inetaddr[] = null;
           System.out.print("주소를 입력하시오 : ");
           String str = scan.nextLine();
           try {
                inetaddr = InetAddress.getAllByName(str);
           } catch (UnknownHostException e) {
                e.printStackTrace();
           }
 
           for (int i = 0; i < inetaddr.length; i++) {
                System.out.println("getHostName = " + inetaddr[i].getHostName());
                System.out.println("getHostAddress = " + inetaddr[i].getHostAddress());
                System.out.println("toString = " + inetaddr[i].toString());
                System.out.println("--------------------------");
           }
     }
 
}
```

실행결과

```
주소를 입력하시오 : www.kpu.ac.kr
getHostName = www.kpu.ac.kr
getHostAddress = 210.93.48.95
toString = www.kpu.ac.kr/210.93.48.95
--------------------------
```

Replicas 가 N개이면 N개의 정보가 리턴 될 것이다. 



### Java샘플2

참조링크: https://needneo.tistory.com/205

```java
import java.net.InetAddress;
import java.net.UnknownHostException;
 
public class Main {
 
    public static void main(String args[]) throws UnknownHostException {
        InetAddress ipAddress = InetAddress.getByName("www.coupang.com");
 
        System.out.println("현재 아이피 : " + ipAddress.getHostAddress());
        System.out.println("현재 호스트명 : " + ipAddress.getHostName());
    }
}
```

실행결과

```
현재 아이피 : 23.201.37.176
현재 호스트명 : www.coupang.com
```