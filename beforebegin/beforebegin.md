# < 시작전에 >





# 1. 실습 환경 준비(개인PC)



## 1) mobaXterm 설치

Cloud VM에 접근하기 위해서는 터미널이 필요하다.

CMD / PowerShell / putty 와 같은 기본 터미널을 이용해도 되지만 좀더 많은 기능이 제공되는 MobaxTerm(free 버젼) 을 다운로드 하자.



- download 위치
  - 링크: https://download.mobatek.net/2312023031823706/MobaXterm_Installer_v23.1.zip

- mobaxterm 실행

![image-20220601194018844](beforebegin.assets/image-20220601194018844.png)





## 2) gitbash 설치

교육문서를 다운로드 받으려면 Git Command 가 필요하다. Windows 에서는 기본 제공되지 않아 별도 설치 해야 한다.

- 다운로드 주소 : https://github.com/git-for-windows/git/releases/download/v2.45.2.windows.1/Git-2.45.2-64-bit.exe
- 참조 링크 : https://git-scm.com/





## 3) Typora 설치

교육자료(MarkDown 문서)를 typora로 확인하기 위해 Typora를 설치한다. 

github site 를 직접 확인해도 되긴 하지만 각종 실습 자료를 직접 수정해야 하므로 가능한 Typora 를 이용하자.



### (1) Typora 설치

- 참고
  - 링크: https://typora.io/

- download 위치
  - 다운로드주소 : https://download.typora.io/windows/typora-setup-x64.exe

- Typora 실행



### (2) Typora 환경설정

원할한 실습을 위해 코드펜스 옵션을 아래와 같이 변경하자.

- 코드펜스 설정
  - 메뉴 : 파일 > 환경설정 > 마크다운 > 코드펜스
    - 코드펜스에서 줄번호 보이기 - check
    - 긴문장 자동 줄바꿈 : uncheck




![image-20220702154444773](beforebegin.assets/image-20220702154444773.png)

- 개요보기 설정
  - 메뉴 : 보기 > 개요
    - 개요 : check






# 2. 교육문서 Download

실습을 위해서 해당 자료를 download 하여 markdown 전용 viewer 인 Typora 로 오픈하여 실습에 참여하자.



## 1) 개인 PC에 교육자료 download

gitbash 실행후 command 명령어로 아래와 같이 디렉토리를 생성후 git clone 으로 download 하자.

```sh
## githubrepo directory 생성
$ mkdir -p /c/githubrepo

$ cd /c/githubrepo

$ git clone https://github.com/ssongman/ktds-edu-k8s-deeper.git
Cloning into 'ktds-edu-k8s-deeper'...
remote: Enumerating objects: 597, done.
remote: Counting objects: 100% (32/32), done.
remote: Compressing objects: 100% (12/12), done.
remote: Total 597 (delta 22), reused 28 (delta 20), pack-reused 565
Receiving objects: 100% (597/597), 3.85 MiB | 9.97 MiB/s, done.
Resolving deltas: 100% (326/326), done.



$ ll /c/githubrepo
drwxr-xr-x 1 송양종 197121 0 Jun  6 11:06 ktds-edu-k8s-deeper/

```



만약 교육중 자료가 변경(오타 변경 등의 사유로) 되어 다시 받아야 하는 경우 가 있을 경우 해당 위치에서 git pull 만 다시 받도록 하자.

```sh
$ cd /c/githubrepo/ktds-edu-k8s-deeper

$ git pull

```







## 2) Typora 로 readme.md 파일오픈

- typora 로 오픈
  - 파일열기(Ctrl + O)  후 아래 파일 오픈


```
## typora 에서 아래 파일 오픈

C:\githubrepo\ktds-edu-k8s-deeper\README.md
```







# 3. 실습 환경 준비(Cloud)

본 교육 과정에서의 모든 실습은 Cloud 에서 수행할 것이다.



## 1) 개인 VM 서버 주소 확인- ★

원할한 실습을 위해서 개인별 한개씩 VM 이 할당되어 있다.  해당 노드에 kubernetes 를 설치 및 다양한 실습을 진행할 것이다.

수강생별 개인 VM Server 접속 주소를 확인하자. 또한 KtdsEduCluster 에서 사용할 개인별 Namespace 를 확인하자.



### 개인 VM 서버 주소 확인

| 이름   | 팀           | Email | Namespace | VM  Server   | VM  Server IP |
| ------ | ------------ | ----- | --------- | ------------ | ------------- |
| 송양종 | AX성장전략팀 | 강사1 | user01    | ke-bastion01 | 4.217.232.226 |
| 송양종 | AX성장전략팀 | 강사2 | user02    | ke-bastion02 | 4.230.148.205 |
|        |              |       |           |              |               |
|        |              |       |           |              |               |
|        |              |       |           |              |               |
|        |              |       |           |              |               |
|        |              |       |           |              |               |
|        |              |       |           |              |               |
|        |              |       |           |              |               |
|        |              |       |           |              |               |
|        |              |       |           |              |               |
|        |              |       |           |              |               |
|        |              |       |           |              |               |



### 멀티클러스터 VM 확인

| 이름   | 팀              | Email                  | Namespace | VM  Server   | VM  Server IP | IP        |
| ------ | --------------- | ---------------------- | --------- | ------------ | ------------- | --------- |
| 송양종 | AX성장전략팀    | 강사1                  | user01    |              |               |           |
| 송양종 | AX성장전략팀    | 강사2                  | user02    |              |               |           |
| 송양종 | AX성장전략팀    | 강사3                  | user03    | ke-bastion04 | 20.41.84.246  | 10.0.0.12 |
| 정문경 | ICT  CoE팀      | mungyeong.jeong@kt.com | user11    | ke-bastion21 | 20.39.203.193 | 10.0.0.23 |
| 문예진 | 데이터DX개발팀  | yejin.moon@kt.com      | user12    | ke-bastion22 | 20.39.200.39  | 10.0.0.24 |
| 류경하 | 아키텍처팀      | kyungha.ryu@kt.com     | user13    | ke-bastion23 | 4.217.239.59  | 10.0.0.25 |
| 임성식 | ICIS  Tr 빌링팀 | sslim@kt.com           | user14    | ke-bastion24 | 4.230.1.184   | 10.0.0.8  |
| 김재현 | DX개발팀        | kim.db@kt.com          | user15    | ke-bastion25 | 4.217.233.116 | 10.0.0.9  |
| 백승연 | ICT  CoE팀      | seung_yeon.baek@kt.com | user16    | ke-bastion26 | 4.230.3.62    | 10.0.0.26 |
| 이승미 | 인프라DX개발팀  | seungmii.lee@kt.com    | user17    | ke-bastion27 | 4.217.236.165 | 10.0.0.27 |
|        |                 |                        | user18    | ke-bastion28 | 4.230.3.61    | 10.0.0.28 |
|        |                 |                        | user19    | ke-bastion29 | 4.230.2.86    | 10.0.0.29 |
|        |                 |                        | user20    | ke-bastion30 | 4.217.234.211 | 10.0.0.30 |





## 2) SSH (Mobaxterm) 실행

Mobaxterm 을 실행하여 VM 접속정보를 위한 신규 session 을 생성하자.

- 메뉴
  - session  : 상단 좌측아이콘 클릭

  - SSH : 팝업창 상단 아이콘 클릭



![image-20240609163324386](./beforebegin.assets/image-20240609163324386.png)



빨간색 영역을 주의해서 입력한후 접속하자.



- Romote host
  - 개인별로 접근 주소가 다르므로 위 수강생별  VM  Server IP 주소를 확인하자.
  - ex)  bastion03 : 4.217.xxx.117  (각자 자신 VM IP 를 입력해야 함)

- User
  - Specify username 에 Check
  - User : ktdseduuser입력
    - Password 는 별도 공지
  
- Port : 22



