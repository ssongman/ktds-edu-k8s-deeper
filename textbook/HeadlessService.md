# < Headless Service >





# 1. 개요



Kubernetes에서 N개의 레플리카로 운영되는 AP가 있다고 가정하자.

일반적인 서비스 호출 방식인 로드 밸런싱을 통해서는 단일 Pod로만 요청이 전달된다.

즉, 한번의 호출로 특정할 수 없는 어느 한 POD에 연결된다.



하지만 AP의 모든 POD에 메시지를 보내야 하는 상황이라면 어떻게 해야 할까?

기존의 일반적인 서비스 방식으로는 모든 POD에 메시지를 전달하는 것은 불가능하다.

이런 경우에는 headless service 를 이용하여 POD들의 IP를 취득한후 해당 IP들로  통신하는 방안이 있을 수 있겠다.    

단, 이 방법은 POD들이 클러스터 내부 IP를 사용하기 때문에 클러스터 내부에서만 사용 가능하다.



Headless Service를 활용하여 POD들의 IP를 회득하는 방법에 대해서 알아보자.



# 2. Headless Service 활용한 POD IP 획득

일반적으로 Service는 고유의 IP를 가지고 있으며 Load Balancer(Proxy) 역할을 수행한다.

그와 대비하여 Headless Servcie 아래와 같은 특징이 있다.

- IP 가 없으며 직접 POD 로 연결된다.
- Load Balancer 역할을 수행하지 못한다.
- POD 의 IP 를 확인 할 수 있다.



## 1) 사전작업

### userlist app 준비

​	replicas 2 및 userlist-svc 준비

```sh 

# userlist app 실행
$ ku apply -k github.com/ssongman/userlist


# replicas 2 로 설정
$ ku scale deployment userlist --replicas=2

```



### curltest pod 준비

```sh

$ ku run curltest --image=curlimages/curl -- sleep 365d
```





## 2) headless service 생성

아래와 같이 ClusterIP 가 None 기록하는 Service 가 Headless service 이다.

```sh


$ aliase ku='kubectl -n user03'


$ cat <<EOF | ku apply -f -
kind: Service
apiVersion: v1
metadata:
  name: userlist-svc-hl
spec:
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 8181
  selector:
    app: userlist
  clusterIP: None
  type: ClusterIP
EOF

```





## 3) 확인

```sh

# POD들의 IP 확인
$ ku get pod -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
curltest                    1/1     Running   0          40h   10.42.0.10   ke-bastion03   <none>           <none>
userlist-9fbfc64bc-8pczv    1/1     Running   0          17h   10.42.0.17   ke-bastion03   <none>           <none>
userlist-9fbfc64bc-9vpwc    1/1     Running   0          17h   10.42.0.18   ke-bastion03   <none>           <none>


# SVC IP 확인
$ ku get svc
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
userlist-svc      ClusterIP   10.43.113.209   <none>        80/TCP    40h
userlist-svc-hl   ClusterIP   None            <none>        80/TCP    70m





# curl test 하기 위해서 curltest POD로 진입
$ ku exec -it curltest -- sh


## POD IP 로 Call
$ curl 10.42.0.17:8181/users/1
{"id":1,"name":"Marlon Wisozk","gender":"F","image":"/assets/image/cat1.jpg"}~ $


$ curl 10.42.0.18:8181/users/1
{"id":1,"name":"Miss Hollis Lehner","gender":"F","image":"/assets/image/cat1.jpg"}~ $



## Service 로 Call
$ curl userlist-svc/users/1
{"id":1,"name":"Marlon Wisozk","gender":"F","image":"/assets/image/cat1.jpg"}

$ curl userlist-svc/users/1
{"id":1,"name":"Miss Hollis Lehner","gender":"F","image":"/assets/image/cat1.jpg"}


CTRL + D

```





## 4) nslookup 를 활용하여 IP 획득 사례

userlist-svc 로 연결가능한 pod 는 2개일때 일반서비스와 Headless Service가 어떻게 다르게 반응하는지 확인해보자.



- 일반 서비스 IP

```sh
# nslookup 명령이 가능한 pod 내로 진입
$ ku exec -it curltest -- sh

$ nslookup userlist-svc.user03.svc.cluster.local
Server:         10.43.0.10
Address:        10.43.0.10:53


Name:   userlist-svc.user03.svc.cluster.local
Address: 10.43.113.209

```

일반 서비스는 Service 의 IP가 리턴된다.



- Headless Service IP

```sh
$ nslookup userlist-svc-hl.user03.svc.cluster.local
Server:         10.43.0.10
Address:        10.43.0.10:53


Name:   userlist-svc-hl.user03.svc.cluster.local
Address: 10.42.0.17
Name:   userlist-svc-hl.user03.svc.cluster.local
Address: 10.42.0.18

```

Headless Service 의 경우 각 POD 의 IP 를 반환 받는다.

AP내부에서 이렇게 반환 받은 IP 를 활용하여 직접 메세지를 보낼 수 있다.





# 3. java Sample

Java 에서 nslookup 기능과 같이 도메인주소를 이용하여 IP 알아내는 방법을 살펴보자.

InetAddress getAllByName() 함수로 IP 목록을 알아 낼수 있다. 

단, 동일 Cluster 내에서만 접근이 가능하다.  그러므로 Cluster 내부에서 직접 빌드하고 수행하는 테스트를 진행할 것이다.





## 1) java test pod 배포 

테스트 편의를 위해서 java Compile이 가능한 POD를 배포해보자.



### java test 배포

```sh

# javatest pod 배포
$ ku create deploy javatest --image=openjdk:17 -- sleep 365d


# javatest POD 내부로 진입
$ ku exec -it deploy/javatest -- bash


# PS1 변수 설정 (색상이 포함된 설정)
PS1='\[\e[32m\]\u@\h \[\e[33m\]\w\[\e[0m\]\$ '

```



### Hello world 소스 빌드 및 수행

javatest POD 내부에서 간단한 hello world 수준의 소스를 빌드하여 수행해보자.

```sh

$ mkdir -p ~/app/src
  cd ~/app/src


$ cat > Main.java
---
public class Main {
	public static void main(String[] args) {
		System.out.println("Hello, Nice Guy!");
	}
}
---

$ javac Main.java && java -cp . Main
Hello, Nice Guy!


```





## 2) java Sample



### Java Sample1



```java

$ cd ~/app/src

$ cat > NSLookup.java
---
import java.net.InetAddress;
import java.net.UnknownHostException;
import java.util.Scanner;
 
public class NSLookup {
     public static void main(String args[]) {
 
           Scanner scan = new Scanner(System.in);
           InetAddress inetaddr[] = null;
           System.out.print("▲ 주소입력 : ");
           String str = scan.nextLine();
           try {
                inetaddr = InetAddress.getAllByName(str);
           } catch (UnknownHostException e) {
                e.printStackTrace();
           }
 
           for (int i = 0; i < inetaddr.length; i++) {
                System.out.println("--------------------------");
                System.out.println("getHostName = " + inetaddr[i].getHostName());
                System.out.println("getHostAddress = " + inetaddr[i].getHostAddress());
                System.out.println("toString = " + inetaddr[i].toString());
                System.out.println("--------------------------");
           }
     }
}
---
    
```

실행결과

```sh

# 빌드
$ javac NSLookup.java

# 수행
$ java -cp . NSLookup

## 아래는 다양한 테스트 결과이니 참고하자.

▲ 주소입력 : www.kpu.ac.kr
--------------------------
getHostName = www.kpu.ac.kr
getHostAddress = 210.93.48.59
toString = www.kpu.ac.kr/210.93.48.59
--------------------------


▲ 주소입력 : google.com
--------------------------
getHostName = google.com
getHostAddress = 142.250.206.206
toString = google.com/142.250.206.206
--------------------------
--------------------------
getHostName = google.com
getHostAddress = 2404:6800:400a:805:0:0:0:200e
toString = google.com/2404:6800:400a:805:0:0:0:200e
--------------------------


▲ 주소입력 : userlist-svc
--------------------------
getHostName = userlist-svc
getHostAddress = 10.43.113.209
toString = userlist-svc/10.43.113.209
--------------------------

▲ 주소입력 : userlist-svc-hl
--------------------------
getHostName = userlist-svc-hl
getHostAddress = 10.42.0.17
toString = userlist-svc-hl/10.42.0.17
--------------------------
--------------------------
getHostName = userlist-svc-hl
getHostAddress = 10.42.0.18
toString = userlist-svc-hl/10.42.0.18
--------------------------

```

Replicas 가 N개이면 N개의 정보가 리턴 된다.









### Java Sample2

userlist-svc-hl 에서 IP 추출후 Rest API Call 수행하는 Samlple 이다.



```sh

$ cd ~/app/src

$
cat > RestApiGetExample.java
```



```java
import java.net.InetAddress;
import java.net.UnknownHostException;
import java.util.Scanner;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;

public class RestApiGetExample {
    private static final String SERVICE_NAME = "userlist-svc-hl";
    // private static final String SERVICE_NAME = "userlist-svc-hl.user03.svc.cluster.local";

    public static void main(String args[]) {

        Scanner scan = new Scanner(System.in);
        InetAddress inetaddr[] = null;
        // System.out.print("▲ 주소입력 : ");
        // String URL = scan.nextLine();

        try {
            inetaddr = InetAddress.getAllByName(SERVICE_NAME);
        } catch (UnknownHostException e) {
            e.printStackTrace();
        }

        for (int i = 0; i < inetaddr.length; i++) {
            System.out.println("--------------------------");
            System.out.println("getHostName = " + inetaddr[i].getHostName());
            System.out.println("getHostAddress = " + inetaddr[i].getHostAddress());
            System.out.println("toString = " + inetaddr[i].toString());

            try {
                String GET_URL = "http://" + inetaddr[i].getHostAddress() + ":8181/users/1";
            	System.out.println("GET_URL = " + GET_URL);
                System.out.println("Rest Call...");
                String response = sendGetRequest(GET_URL);
                System.out.println(response);
            } catch (Exception e) {
                e.printStackTrace();
            }
            System.out.println("--------------------------");
        }
    }

    private static String sendGetRequest(String getUrl) throws Exception {
        // URL 객체 생성
        URL url = new URL(getUrl);
        HttpURLConnection httpURLConnection = (HttpURLConnection) url.openConnection();

        // 요청 설정
        httpURLConnection.setRequestMethod("GET");

        // 응답 코드 확인
        int responseCode = httpURLConnection.getResponseCode();
        System.out.println("GET Response Code :: " + responseCode);

        // 응답 읽기
        if (responseCode == HttpURLConnection.HTTP_OK) { // 성공
            BufferedReader in = new BufferedReader(new InputStreamReader(httpURLConnection.getInputStream()));
            String inputLine;
            StringBuffer response = new StringBuffer();

            while ((inputLine = in.readLine()) != null) {
                response.append(inputLine);
            }
            in.close();

            // 반환
            return response.toString();
        } else {
            throw new RuntimeException("GET request not worked");
        }
    }
}
```

실행결과

```sh

# 빌드 & 수행
$
javac RestApiGetExample.java && java -cp . RestApiGetExample

--------------------------
getHostName = userlist-svc-hl
getHostAddress = 10.42.0.17
toString = userlist-svc-hl/10.42.0.17
Rest Call...
GET Response Code :: 200
{"id":1,"name":"Marlon Wisozk","gender":"F","image":"/assets/image/cat1.jpg"}
--------------------------
--------------------------
getHostName = userlist-svc-hl
getHostAddress = 10.42.0.18
toString = userlist-svc-hl/10.42.0.18
Rest Call...
GET Response Code :: 200
{"id":1,"name":"Miss Hollis Lehner","gender":"F","image":"/assets/image/cat1.jpg"}
--------------------------

```





## 3) Clean Up

```sh

$ ku delete deploy javatest

```







# 4. Python Sample





## 1) python test pod 배포 

테스트 편의를 위해서 java Compile이 가능한 POD를 배포해보자.

```sh
$ ku create deploy pythontest --image=python:alpine3.20 -- sleep 365d


# pythontest POD 내부로 진입
$ ku exec -it deploy/pythontest -- sh

# PS1 변수 설정 (색상이 포함된 설정)
PS1='\[\e[32m\]\u@\h \[\e[33m\]\w\[\e[0m\]\$ '

```



### Hello world 소스 빌드 및 수행

pythontest POD 내부에서 간단한 hello world 수준의 소스를 빌드하여 수행해보자.

```sh
$ mkdir -p ~/app/src
  cd ~/app/src


$ cat > hello_world.py
---
print("Hello, World!")
---

$ python hello_world.py

```





## 2) python Sample



```sh


$ cd ~/app/src

$ cat > get_ip_addresses.py

```



```python
import socket

def get_ip_addresses(domain_name):
    try:
        ip_addresses = socket.gethostbyname_ex(domain_name)[2]
        return ip_addresses
    except socket.gaierror as e:
        print(f"Error resolving {domain_name}: {e}")
        return []

if __name__ == "__main__":
    service_name = "userlist-svc-hl" 
    # service_name = "userlist-svc-hl.user03.svc.cluster.local" 
    ip_addresses = get_ip_addresses(service_name)

    if ip_addresses:
        print("Resolved IP addresses:")
        for ip in ip_addresses:
            print(ip)
    else:
        print(f"No IP addresses found for {service_name}")
        
```



```sh
## 수행
$ python get_ip_addresses.py

Resolved IP addresses:
10.42.0.17
10.42.0.18

```





## 3) Clean Up

```sh

$ ku delete deploy pythontest

```



