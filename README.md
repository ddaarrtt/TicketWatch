



<img src="https://capsule-render.vercel.app/api?type=waving&color=3F51B5&height=300&section=header&text=⚾TicketWatch⚾&fontSize=70&fontColor=FFFFFF&animation=fadeIn&width=1200" width="1200" />

## 🚩 개요
**야구 티켓팅 시스템의 모니터링 환경을 구축**하는 프로젝트입니다. <br>
부하 테스트를 통해 시스템의 한계를 파악하고, **장애 상황에 대비할 수 있는 안정적인 운영 환경**을 마련합니다. <br>
이를 기반으로 개선 방향을 분석합니다.


<br>

## 📍 Outline
1️⃣ [프로젝트 개요](#-개요)  
2️⃣ [목표](#-목표)  
3️⃣ [기술 스택](#-기술-스택)  
4️⃣ [아키텍처](%EF%B8%8F-아키텍처)  
5️⃣ [테스트 시나리오](#-테스트-시나리오)  
6️⃣ [성능 모니터링 및 개선 방향](#-성능-모니터링-및-개선-방향)  
7️⃣ [종합 결론 및 개선 요약](#-종합-결론-및-개선-요약)  
8️⃣ [Trouble Shooting](#-trouble-shooting)  

<br>
<br>


## 👨‍👩‍👧‍👦 Contributors
<br>

| <img src="https://github.com/kcs19.png" width="200px"> | <img src="https://github.com/HongChan1412.png" width="200px"> | <img src="https://github.com/letmeloveyou82.png" width="200px"> | <img src="https://github.com/nanahj.png" width="200px"> |
| :---: | :---: | :---: | :---: |
| [김창성](https://github.com/kcs19) | [나홍찬](https://github.com/HongChan1412) | [최윤정](https://github.com/letmeloveyou82) | [이현정](https://github.com/nanahj) |


<br>


<br>

## 🥅 목표

1. **Grafana & Prometheus 기반의 모니터링 서버 구축**
    - 실시간 메트릭 수집 및 시각화를 통해 시스템 상태를 직관적으로 파악할 수 있도록 구성합니다.
    
2. **Exporter 연동을 통한 애플리케이션 자원 모니터링**
    - 각 서버 및 애플리케이션에 Exporter를 설치하여 CPU, 메모리, 트래픽 등의 리소스 상태를 실시간으로 수집합니다.
    
3. **실전 시나리오 기반의 부하 테스트 수행**
    - 실제 티켓팅 상황을 반영한 HTTP 및 DB 부하 테스트를 설계 및 실행하여 시스템 내구성을 점검합니다.
        - 자리 조회 (GET /seats)
        - 자리 예약 (POST /book)

<br>

## **👨‍💻 기술 스택**

| 역할 | 종류 |
| --- | --- |
| 🗄️ 데이터베이스 | ![MySQL](https://img.shields.io/badge/MySQL-4479A1.svg?style=for-the-badge&logo=mysql&logoColor=white) |
| 🛠️ 백엔드 | ![Spring Boot](https://img.shields.io/badge/SpringBoot-6DB33F.svg?style=for-the-badge&logo=spring-boot&logoColor=white) |
| 👀 모니터링 | ![Grafana](https://img.shields.io/badge/Grafana-F46800.svg?style=for-the-badge&logo=grafana&logoColor=white) |
| 📊 데이터 수집 | ![Prometheus](https://img.shields.io/badge/Prometheus-E6522C.svg?style=for-the-badge&logo=prometheus&logoColor=white) |
| ⚡ 부하 테스트 | ![wrk](https://img.shields.io/badge/wrk-000000.svg?style=for-the-badge&logo=linux&logoColor=white) <br> ![sysbench](https://img.shields.io/badge/sysbench-007ACC.svg?style=for-the-badge&logo=linux&logoColor=white) |
| 💻 가상화 플랫폼 | ![VMware](https://img.shields.io/badge/VMware-607078.svg?style=for-the-badge&logo=vmware&logoColor=white) |



<br>

## **🛠️ 아키텍처**

![image](https://github.com/user-attachments/assets/8658e657-5f00-4bff-a1c8-278670507d35)


## 🧪 테스트 시나리오

### ✅ 테스트 개요
- HTTP 부하 생성 도구인 `wrk`를 사용하여 요청 시뮬레이션 수행

### 📍 Step 1. 경기 좌석 조회 (Read 트래픽)
- 다수 사용자가 자리를 조회하는 상황을 시뮬레이션

```bash
wrk -t4 -c50 -d30s http://<ip>/seats
```

### 📍 Step 2. 좌석 예매 (Write 트래픽)
- 동시에 여러 사용자가 같은 좌석을 예매 시도
- DB에 `INSERT`, `UPDATE`, `SELECT` 트랜잭션이 혼합되어 발생

```bash
wrk -t10 -c500 -d30s -s post_book.lua http://<ip>/book
```

#### 📄 post_book.lua 내용

```lua
wrk.method = "POST"
wrk.headers["Content-Type"] = "application/json"

request = function()
  local seatId = math.random(100, 120)
  local userId = "user" .. math.random(1, 5000)
  local body = string.format('{"seatId":%d,"userId":"%s"}', seatId, userId)
  return wrk.format("POST", "/book", nil, body)
end
```

<br>

## 📊 성능 모니터링 및 개선 방향

### 🖥️ 1. CPU 사용률
```bash
system_cpu_usage{application="$application", instance="$instance"}
```
![image](https://github.com/user-attachments/assets/ed8b3384-06c8-4a6c-ab86-1bb27a822e20)

- 부하 시작과 함께 CPU 사용률이 100%까지 급등 후 서서히 하락  
- **서버의 부하 감당 한계를 명확히 확인**

**개선 방향:** 병렬 처리, 캐싱, 로드밸런싱 도입



### 📝 2. 로그 발생량
```bash
increase(logback_events_total{application="$application", instance="$instance"}[1m])
```
![image](https://github.com/user-attachments/assets/8fae21f2-5fc3-455f-a0cf-96e5cad75f19)

- 로그는 `1 ops/m` 수준으로 일정하게 발생  
- 예외 발생 시 로그가 정상적으로 기록되어 **에러 발생 시점을 확인할 수 있음**

**개선 방향:** 에러 발생 시간 기반 상세 분석, 로그 레벨 및 내용 점검



### 💾 3. MySQL 네트워크 트래픽
```bash
rate(mysql_global_status_bytes_received{instance="$host"}[$interval]) or irate(mysql_global_status_bytes_received{instance="$host"}[5m])
```
![image](https://github.com/user-attachments/assets/b579fc0d-20cb-4115-a617-5c66dd385b50)


- 테스트 중 DB 수신 트래픽 약 `700kB/s` 발생  
- 이는 사용자 요청이 **DB까지 전달되어 처리되고 있음을 의미**

**개선 방향:** 쿼리 최적화 및 인덱스 적용 검토, 연결 풀 관리 및 DB 세션 튜닝



### 🌐 4. HTTP 응답 상태 분석
```bash
http_server_requests_seconds_count
```
![image](https://github.com/user-attachments/assets/c11e1d14-6051-4189-87a9-249531b16735)

- 200 OK가 대부분이지만, **504 Gateway Timeout** 및 **409 Conflict** 오류 다수 발생

**개선 방향:** 서버 타임아웃 설정 재조정, 중복 요청 방지 로직 도입, DB 락 전략 적용 (비관적/낙관적 락), 요청 큐 혹은 예약 로직 보완



<br>

## 📌 종합 결론 및 개선 요약

| 항목 | 문제 현상 | 원인 분석 | 개선 방안 |
|------|------------|------------|------------|
| **CPU** | 부하 시 100% 급등 | 병렬 처리 부족 | 멀티스레딩, 캐싱, 로드밸런싱 도입 |
| **로그** | 예외 발생 시 생성 | 정상 로깅 동작 | 에러 로그 분석, 레벨 조정 |
| **DB** | 트래픽 증가 | 요청 정상 처리 | 쿼리 최적화, 인덱스 검토 |
| **HTTP 응답** | 504, 409 오류 다수 | 동시 요청 충돌, 타임아웃 | 락 처리 전략, UX 보완, 서버 설정 튜닝 |

---

이번 테스트를 통해 실제 트래픽 상황에서 발생할 수 있는 병목 지점을 사전에 파악할 수 있었으며, 향후 성능 향상을 위한 명확한 개선 방향을 도출할 수 있었습니다.  


<br>

## 🚀 Trouble Shooting
### ❗ 문제 상황: wrk에서 timeout 발생, Grafana에서는 확인되지 않음
기존 Grafana Promql : `sum by (status) (rate(http_server_requests_seconds_count[1m]))`

![image](https://github.com/user-attachments/assets/f64c2fb6-ad8e-47c5-8440-59abca393099)


### 1️⃣ 자바 코드 수정: 504 응답을 명시적으로 반환하도록 처리

```java
    @GetMapping("/seats")
    public ResponseEntity<?> getSeats() {
        ExecutorService controllerExecutor = Executors.newSingleThreadExecutor();
        try {
            Future<List<Seat>> future = controllerExecutor.submit(() -> seatService.getAllSeatsParallel());
            List<Seat> seats = future.get(2, TimeUnit.SECONDS); // 2초 내 미응답 시 timeout 처리
            return ResponseEntity.ok(seats);
        } catch (TimeoutException e) {
            log.warn("/seats 요청에서 TimeoutException 발생: 처리 시간 초과");
            return ResponseEntity.status(HttpStatus.GATEWAY_TIMEOUT).body("조회 시간이 초과되었습니다.");
        } catch (Exception e) {
            log.error("/seats 요청 처리 중 예외 발생", e);
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("오류가 발생했습니다.");
        } finally {
            controllerExecutor.shutdown();
        }
    }
```

---

### 2️⃣ 로그 확인: 예외 처리 여부 확인

```bash
nohup java -jar ticketing-api-0.0.1-SNAPSHOT.jar > logs/app.log 2>&1 &
```

- Java 애플리케이션을 nohup 명령어로 백그라운드에서 실행하며 로그를 저장했습니다.

![image (15)](https://github.com/user-attachments/assets/ac4fac3d-e553-4b98-93f7-6d7efee56412)

- 로그에서 TimeoutException 발생 시 로그가 정상적으로 기록되었음을 확인했습니다.

### 3. Prometheus 지표 수집 확인 (`504` 상태코드 응답)

✅ Prometheus에서 상태코드 지표 직접 조회
    
    ```
    http://<IP>:9090
    ```
    
    ### 검색창에 메트릭 입력
    
    ```
    http_server_requests_seconds_count
    ```
    
- 여기서 504 상태 코드 응답도 정상적으로 수집되고 있음을 확인했습니다.

![image (16)](https://github.com/user-attachments/assets/09cd845c-d00e-4200-9c34-e0d0a57ba074)


- PromQL 쿼리를 수정 후 적용한 결과, Grafana에서도 504 응답을 포함한 모든 상태 코드가 시각화 되었습니다.
    
    ![image (17)](https://github.com/user-attachments/assets/185e51d7-e8b0-480e-964d-a78fc51ce611)



<img src="https://capsule-render.vercel.app/api?type=waving&color=3F51B5&height=150&section=footer" width="1000" />
