# 🛒 JMeter : BlackFriday WAS StressTest  

"다가오는 **🛍️블랙 프라이데이🛍️**를 대비하여, 시스템 안정성과 성능을 검증하기 위한 **부하 테스트 시나리오 및 실행 프로젝트**"

날짜: 2024년 10월 8일

<br>

## 🔍 배경

AWS EC2 인스턴스에서 호스팅되는 SpringBoot 애플리케이션의 성능을 평가하기 위해 가상의 **"Black Friday Sale"** 시나리오를 바탕으로 부하 테스트를 진행했습니다. 예상되는 높은 트래픽 상황을 시뮬레이션하여 **JMeter를 활용해 시스템의 한계를 실험**했습니다.

이번 프로젝트의 목표는 피크 트래픽 하에서 **시스템의 안정성, 성능**, 그리고 **잠재적인 병목 현상**을 평가하는 것이었습니다.

<br>

## 🎯 주요 목표

1. **피크 시간대의 높은 트래픽(평소의 10배)을 가정한 상황에서 시스템의 안정성 검증**
   
2. **시스템 성능의 한계점 파악**
   
3. **잠재적인 병목 지점 및 장애 포인트 식별**

<br>

## 📍 테스트 방법

1. **도구**: Apache JMeter를 사용하여 부하 테스트 수행
2. **시각화**: Grafana를 활용하여 테스트 결과 및 시스템 메트릭 시각화 
3. **시나리오**: 가상의 쇼핑몰 트래픽 패턴을 시뮬레이션 (예: 상품 조회, 장바구니 추가, 결제 등)
4. **부하 증가**: 단계적으로 사용자 수를 증가시키며 시스템 반응 관찰


<br>

## 🛠 실습 환경 및 기술

1. **플랫폼**
   - AWS EC2 인스턴스

2. **부하 테스트 도구**
   - JMeter (시나리오 기반 부하 테스트 계획 작성)

3. **테스트 대상 API**
   - HTTP 요청 샘플러를 이용한 사용자 동작 시뮬레이션
     * 메인 페이지 (`GET /test`)

4. **테스트 구성**
   - **스레드 그룹**
     * 1,000명의 동시 접속 사용자 시뮬레이션
     * 1시간 동안 트래픽 유지
   - **타이머**
     * Gaussian Random Timer 사용
     * 평균 대기시간 3초, 편차 1초
     * 실제 사용자 행동과 유사한 대기시간 반영
   - **리스너**
     * 응답 시간 그래프
     * TPS(초당 트랜잭션 수)
     * CPU/메모리 사용률 모니터링
<br>

## 📋 프로젝트 과정

### Docker 설치

Linux 환경에서 도커를 설치하기 위해 apt 명령어를 사용합니다.

```bash
sudo apt-get update

#Docker를 설치하기 전에 필요한 패키지를 설치
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common

#Docker GPG 키 추가
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

#Docker 저장소 추가
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu lunar stable"

sudo apt-get update

#Docker 설치
sudo apt-get install docker-ce docker-ce-cli containerd.io

#Docker 설치확인
docker --version

#Dokcer 사용자 권한설정
sudo usermod -aG docker %USER
newgrp docker
groups
```

### Docker Hub에서 WAS image 받아오기

Docker Hub에서 SpringBoot 이미지를 받아옵니다.

```bash
docker run --name myapp -d -p 8999:8999 isshomin/test-spring:1.0
```

<br>

### Jmeter 설치

**Apache JMeter는 웹 애플리케이션의 성능 테스트 도구**로, 공식 웹사이트에서 다운로드 후 설치하였습니다. <br>

설치할 때 Java가 필요하므로 설치되어 있는지 확인해야 합니다.

```bash
wget https://dlcdn.apache.org//jmeter/binaries/apache-jmeter-5.6.3.tgz

tar -xzvf apache-jmeter-5.6.3.tgz

#Jmeter 실행
cd apache-jmeter-5.6.3/bin 
./jmeter.sh
```

<br> 

### EC2 보안그룹 인바운드 규칙 추가

JMeter가 실행될 EC2 인스턴스에 대한 접근을 허용하기 위해 보안 그룹의 인바운드 규칙을 설정합니다. **TCP 포트 8999 포트를 허용**하였습니다. <br>

(포트를 허용하지 않으면 ec2에 접속해도 무한 로딩이 되고, 접속이 되지 않는다)

![image](https://github.com/user-attachments/assets/10d2c413-ddee-4c6a-9e68-9825ab4a41ea)


<br>

### Jmeter 실행

```yaml
./jmeter
```

<br>

### 📅 테스트 계획 생성

JMeter GUI가 실행되면 구체적인 JMeter  테스트 계획을 생성합니다.

<br>

1. **쓰레드 그룹**을 생성한다. <br>
  
  ![image 1](https://github.com/user-attachments/assets/e66e8d3b-5b7a-45c9-8d05-9f46cccb83c8)
  **1분**동안 동시 사용자수 **1000명씩 초당 16명씩 요청**을 보내고 **5번 반복**하도록 설정하였다.

<br>

2. **HTTP Request Defaults**를 아래와 같이 설정한다.

![image 2](https://github.com/user-attachments/assets/9072f15b-fbdf-4f6a-953f-64c00a01f124)
- 스레드 그룹 우클릭 > "Add" > "Config Element" > "HTTP Request Defaults"
- Server Name or IP: [EC2 인스턴스 주소]
- Protocol: http (또는 https)
- Path : 부하 테스트를 진행할 api 경로

<br>

3. **타이머**를 추가한다.
    
    ![image 4](https://github.com/user-attachments/assets/b537bf79-2fd7-41c7-9fcf-c84c0e38f807)
   
    편차를 **1000ms(1초), 지연시간을 3000ms(3초)** 로 설정한다.
<br>

### 🔌 Jmeter 플러그인 추가

**JMeter의 기능을 확장**하기 위해 플러그인을 추가할 수 있습니다. 그래프로 부하테스트 결과를 보기 위해 **Jmeter 플러그인을 추가**하였습니다.

<br>

1. **JMeter 플러그인 매니저**를 다운로드한다.

```bash
ubuntu@ip-10-3-0-148:~/apache-jmeter-5.6.3/lib/ext$ wget https://jmeter-plugins.org/get/ -O jmeter-plugins-manager.jar

sudo mv jmeter-plugins-manager.jar /path/to/apache-jmeter/lib/ext/

./jmeter.sh
```
JMeter 플러그인 매니저 **JAR 파일을 다운로드**하고, 다운로드한 **Jar 파일을 JMeter의 lib/ext 디렉토리로 이동** 한다. <br>
그리고 JMeter를 다시 실행한다.

<br>

2. JMeter GUI가 열리면, **"Options" > "Plugins Manager** 선택하고 플러그인 매니저 창에서 **원하는 플러그인을 선택**하고  **"Apply Changes and Restart JMeter"** 버튼을 클릭한다.

   ![image 5](https://github.com/user-attachments/assets/7dfc9f51-d891-4484-9e5f-8385a8f2822f)

    
    **🌷 설치된 플러그인**
    
    ![image 6](https://github.com/user-attachments/assets/1b2c7ddc-ac50-4ae6-bee6-c27c653d8897)


    🪄 **추가한 플러그인**
    
    ```yaml
    3 Basic Graphs
    5 Additional Graphs
    Composite Graph
    Custom Thread Groups
    PerfMon (Server Agent)
    ```

<br>

  3. 설치 후 JMeter를 다시 시작한다.

<br>

  4. Test Plan 에 리스너를 추가할 때, **새로운 그래프 옵션**들을 볼 수 있다.

  ![%ED%99%94%EB%A9%B4_%EC%BA%A1%EC%B2%98_2024-10-08_160453](https://github.com/user-attachments/assets/85a78688-f90a-4e8a-ae8c-e155dc2904c1)


<br>

  5. 플러그인을 추가하여  테스트 실행 중 실시간으로 결과를 보고, 테스트 완료 후에도 결과를 분석할 수 있다.

  ![image 7](https://github.com/user-attachments/assets/20131032-f1f7-429c-92cd-50a4a92a5f3e)

<br>

### 🎦 테스트 실행

- "실행" 메뉴에서 "시작"을 선택하거나 녹색 실행 버튼을 클릭하여 테스트를 실행한다.

    
<br>

## 📈 프로젝트 결과

### **시나리오 1️⃣: 한계 부하 테스트 (Stress Testing Beyond Capacity)**

- **목적:** WAS의 최대 처리 한계를 파악하고, 한계를 초과할 때의 시스템 반응을 평가
- **설정**
    - **사용자 수:** 점진적으로 증가시켜 서버가 포화 상태에 도달할 때까지
    - **테스트 기간:** 일정한 부하를 유지하면서 Ramp-up 기간의 변동에 따른 성능 변화를 모니터링
    - **Ramp-up 기간**: 60초에서 시작하여 단계적으로 감소시키거나 늘림
- **주요 목표**
    - 서버가 안정적으로 처리할 수 있는 최적의 Ramp-up 기간 파악
    - 서버가 과부하 상태에 도달할 때의 처리량, 응답 시간, 오류 발생률 측정
    - 서버가 과부하 시점에서 어떻게 대응하는지 확인(예: 응답 속도 저하, 오류 발생, 서버 다운 등)
- **한계 부하 테스트 결과 (Stress Testing Results)**
    - Ramp-up 기간: **30초**
        
        
        | **Thread Group** | 1000 |
        | --- | --- |
        | **Ramp-up period** | 30 |
        | **Loop Count** | 5 |
      
        ![1000_30](https://github.com/user-attachments/assets/3b8eac11-b229-45e2-9810-d0259a67285e)

        
        - 서버는 초기 사용자 증가 시점에서 안정적으로 작동했으며, **일정한 처리량과 응답 시간**을 유지했습니다.
        - 서버 부하는 **점진적으로 증가했지만 포화 상태에는 도달**하지 않았습니다.

        <br>
        
    - Ramp-up 기간: **20초** 
        
        
        | **Thread Group** | 1000 |
        | --- | --- |
        | **Ramp-up period** | 20 |
        | **Loop Count** | 5 |
 
        ![20_1000](https://github.com/user-attachments/assets/075de733-6536-44fe-897b-e748dc4f5d42)

 
        - 짧은 시간 안에 부하가 급격히 증가하면서 서버는 **포화 상태**에 빠졌고, **응답 시간은 극단적으로 길어졌으며, 약 30%의 요청이 처리되지 못했습니다.**

  <br>
        
    - 결론
        
        
        | **Ramp-up 기간** | **초당 사용자 증가량** | **서버 상태** | **응답 시간** | **오류율** | **결론** |
        | --- | --- | --- | --- | --- | --- |
        | **1000명 / 30초 / 5초** | 초당 33명 | 안정적인 처리, 일정한 처리량 유지 | 안정적 | 0% | 서버가 원활하게 부하를 처리함, 포화 상태 도달하지 않음 |
        | **1000명 / 20초 / 5초** | 초당 50명 | 급격한 부하 증가로 서버 포화 상태 도달 | 극단적으로 길어짐 | 약 30% 요청 처리 실패 | 과부하로 서버가 다운됨, 오류 다수 발생, 서버 자원 부족  |
        
         => 최적의 Ramp-up 시간을 **30초 이상으로 설정하는 것이 서버의 안정성을 보장하는 데 효과적**이며, 그 이하로 줄이는 경우에는 성능 개선을 위해 **추가적인 서버 자원**이 필요하거나, **수평적 확장(autoscaling)** 등의 대책이 요구되는 걸 확인할 수 있었습니다.
        
    <br>

<hr>

### **시나리오 2️⃣: 피크 부하 테스트 (Peak Load Testing)**

- **목적:** 특정 시간대에 트래픽이 급증할 때 웹 애플리케이션 서버(WAS)에 사용자 수를 급격히 늘리는 부하 테스트를 진행.
- **설정:**
    - **사용자 수:** 급격히 증가하는 사용자 수 (예: 100명에서 10000명으로 단시간 내 증가).
    - **테스트 기간:** 급증 상황을 30분간 유지.
    - **테스트 계획:** 이벤트 발생 시나리오, 예: 블랙프라이데이 세일 등.
- **측정 항목:**
    - 시스템 응답 시간
    - 데이터베이스 응답 시간
    - 서버 자원 사용률
- **주요 목표**
    - 급격히 증가하는 트래픽을 서버가 얼마나 효과적으로 처리할 수 있는지 평가.
    - 사용자 경험에 영향을 미치는 응답 시간 변화 측정.
    - 급격한 트래픽 증가 시 서버 자원(CPU, 메모리, 네트워크)이 얼마나 빠르게 소모되는지 분석.
- **한계 부하 테스트 결과 (Stress Testing Results)**
    - 사용자 수 : **100명**
    - Ramp-up 기간: **100초**
        
        
        | **Thread Group** | 100 |
        | --- | --- |
        | **Ramp-up period** | 100 |
        | **Loop Count** | 5 |

      ![100_100](https://github.com/user-attachments/assets/38c8fdd9-bb7e-4726-bc03-900718d8a2fa)

        
        - 서버는 **안정적으로 부하를 처리**했고, 응답 시간과 자원 사용률에 큰 변화가 없었습니다.
        - **CPU와 메모리 사용률은 정상 범위** 내에서 동작했으며, 응답 시간 역시 빠르게 처리되었습니다.

        <br>
        
    - 사용자 수 : **100명**
    - Ramp-up 기간: **10초**
        
        
        | **Thread Group** | 100 |
        | --- | --- |
        | **Ramp-up period** | 10 |
        | **Loop Count** | 5 |
 
        
        ![100_10.gif](JMeter%20%E1%84%87%E1%85%AE%E1%84%92%E1%85%A1%E1%84%90%E1%85%A6%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%2011943624eb2b80d6987cd7228e85ced8/100_10.gif)
        
        - 사용자 수가 **짧은 시간 내에 빠르게 증가**했지만, 서버는 여전히 안정적으로 작동했습니다.
        - **응답 시간이 약간 증가**했으나, 사용자 경험에 큰 영향을 미칠 정도는 아니었습니다.
        - CPU와 메모리 사용률은 약간 상승했지만, **자원은 여전히 충분히 여유**가 있었습니다.

        <br>

    - 사용자 수 : **500명**
    - Ramp-up 기간: **10초**
      
      | **Thread Group** | 500 |
      | --- | --- |
      | **Ramp-up period** | 10 |
      | **Loop Count** | 5 |
      
      ![500_10](https://github.com/user-attachments/assets/71cb18d1-3422-4fbe-aa8c-7d5fe9f01cc6)
    
    
    - 서버 부하가 크게 증가하면서 **응답 시간이 눈에 띄게 늘어났고**, 일부 요청에서 **응답 지연**이 발생했습니다. 
    - **CPU 및 메모리 사용률이 80% 이상으로 상승**했고, 일부 요청에서 오류(약 5%)가 발생했습니다.

<br>

  - 사용자 수: **10000명**
  - Ramp-up 기간: **1초**
    
    
    | **Thread Group** | 10000 |
    | --- | --- |
    | **Ramp-up period** | 1 |
    | **Loop Count** | 5 |
  
    ![10000_1](https://github.com/user-attachments/assets/bf8a892d-d1eb-49d1-aad8-8503bbf497c2)

    
    - **서버는 급격한 사용자 증가로 인해 심각한 과부하 상태에 도달**했습니다.
    - CPU와 메모리 사용률은 100%에 가까워졌으며, 데이터베이스 응답 시간이 급격히 느려져 서버는 더 이상 **정상적인 서비스를 제공할 수 없는 상태**에 도달했습니다.
  
  <br>

- 결론
    
    
    | **테스트 단계** | **응답 시간 변화** | **오류율** | **CPU/메모리 사용률** | **데이터베이스 응답 시간** | **결론** |
    | --- | --- | --- | --- | --- | --- |
    | **100명 / 100초** | 응답 시간 변화 없음 | 0% | 정상 범위 (낮음) | 정상 | 안정적인 서버 상태, 성능 영향 없음 |
    | **100명 / 10초** | 약간의 응답 시간 증가 | 0% | 약간 증가 (여유 있음) | 약간 증가 | 서버 안정성 유지, 성능에 큰 문제 없음 |
    | **500명 / 10초** | 응답 시간 증가 | 약 5% 오류 발생 | 80% 이상으로 상승 | 응답 시간 눈에 띄게 증가 | 서버 부담 증가, 성능 저하 발생, 확장 또는 튜닝 필요 |
    | **10000명 / 1초** | 극단적인 응답 시간 증가 | 약 50% 이상 오류 발생 | CPU/메모리 사용률 100% 도달 | 매우 느려짐 | 서버 과부하, 서비스 불가 상태, 확장 및 자원 확보 필요 |
    
    => 대규모 트래픽이 예상되는 이벤트나 프로모션 상황에서는 **수평적 확장(autoscaling)** 이나 **캐싱 및 로드 밸런싱**을 통한 부하 분산이 필수적임을 확인할 수 있었습니다.
    
<br>

### 📊 Grafana 시각화

테스트 결과를 효과적으로 이해하고 분석하기 위해 **Grafana를 활용하여 시각화를 진행**했습니다.


![image 8](https://github.com/user-attachments/assets/7abbd9ac-c0bb-4226-9022-54b26f544209)

![image 9](https://github.com/user-attachments/assets/1b9b8361-cd2c-40f0-a291-8c89de72dfbf)


이러한 Grafana 시각화를 통해, 대시보드를 생성하여 **시스템 응답 시간, 오류율, CPU 사용률 등의 메트릭을 모니터링하고 분석**할 수 있었습니다. <br>

또한 부하 테스트 중 **실시간으로 성능을 모니터링**하고, **다양한 형태의 그래프와 차트**를 통해 데이터를 직관적으로 표현할 수 있었습니다.

<br>

## 💦 개선할 점

- JMeter 서버와 테스트 대상 웹 서버가 같은 EC2 인스턴스에서 실행되는 환경에서 테스트 했기 때문에 두 서버가 동일한 메모리를 공유했습니다. 이로 인해 **JMeter 서버와 웹 서버가 분리된 환경에 비해 정확한 성능 측정이 어려웠습니다**.😓

- 부하 테스트를 GET 요청만 사용하여 진행했기 때문에, 서버 리소스를 더 많이 소모하고 실제 부하 상황을 더 강하게 반영할 수 있는 **POST 요청에 대해서는 확인하지 못했습니다**.💊

- **단일 API 엔드포인트(GET /test)만을 테스트**하여 다양한 사용자 행동 패턴을 시뮬레이션하지 못했습니다. 향후 상품 검색, 장바구니 추가, 결제 등 다양한 API를 포함한 복합적 시나리오 구성이 필요합니다.😢

- **데이터베이스 연동 및 관련 성능 테스트가 부족했**습니다. 향후 데이터베이스 쿼리 성능, 연결 풀 설정 등을 포함한 종합적인 테스트를 진행하고자 합니다.🥑
  
- grafana 아쉬운 점 추가

- **테스트 실행과 결과 분석 과정이 수동**으로 이루어져 효율성이 떨어졌습니다. CI/CD 파이프라인에 부하 테스트를 통합하고, 자동화된 결과 분석 및 보고서 생성 시스템 구축을 하고자 합니다.⌛

  <br>
