



<img src="https://capsule-render.vercel.app/api?type=waving&color=3F51B5&height=300&section=header&text=🎯TicketWatch🎯&fontSize=70&fontColor=FFFFFF&animation=fadeIn&width=1200" width="1200" />

# 🎫 TicketWatch
실시간 티켓 예매 시스템의 상태를 모니터링 환경을 구축하는 프로젝트입니다.



<br>

## 📍 Outline
- [1️⃣ Contributors](#1%EF%B8%8F⃣-contributors)
- [2️⃣ Contents](#2%EF%B8%8F⃣-contents)
- [3️⃣ Performance Optimization](#3%EF%B8%8F⃣-performance-optimization)
- [4️⃣ Trouble Shooting](#4%EF%B8%8F⃣-trouble-shooting)

<br>
<br>


## 👨‍👩‍👧‍👦 Contributors
<br>

| <img src="https://github.com/kcs19.png" width="200px"> | <img src="https://github.com/HongChan1412.png" width="200px"> | <img src="https://github.com/letmeloveyou82.png" width="200px"> | <img src="https://github.com/nanahj.png" width="200px"> |
| :---: | :---: | :---: | :---: |
| [김창성](https://github.com/kcs19) | [나홍찬](https://github.com/HongChan1412) | [최윤정](https://github.com/letmeloveyou82) | [이현정](https://github.com/nanahj) |


<br>


<br>

## 🚩개요
본 프로젝트는 **야구 티켓팅 시스템의 안정성을 확보하기 위한 모니터링 환경을 구축** 하는 것을 목표로 합니다.
부하 테스트를 통해 시스템의 한계를 파악하고, **장애 상황에 대비할 수 있는 안정적인 운영 환경**을 마련합니다.

<br>

## 🥅 목표

1. **Grafana & Prometheus 기반의 모니터링 서버 구축**
    → 실시간 메트릭 수집 및 시각화를 통해 시스템 상태를 직관적으로 파악할 수 있도록 구성합니다.
    
2. **Exporter 연동을 통한 애플리케이션 자원 모니터링**
    → 각 서버 및 애플리케이션에 Exporter를 설치하여 CPU, 메모리, 트래픽 등의 리소스 상태를 실시간으로 수집합니다.
    
3. **실전 시나리오 기반의 부하 테스트 수행**
    → 실제 티켓팅 상황을 반영한 HTTP 및 DB 부하 테스트를 설계 및 실행하여 시스템 내구성을 점검합니다.
    - 자리 조회 (GET /seats)
    - 자리 예약 (POST /book)

<br>

### **👨‍💻 기술 스택**

| 역할 | 종류 |
| --- | --- |
| 🗄️ 데이터베이스 | ![MySQL](https://img.shields.io/badge/MySQL-4479A1.svg?style=for-the-badge&logo=mysql&logoColor=white) |
| 🛠️ 백엔드 | ![Spring Boot](https://img.shields.io/badge/SpringBoot-6DB33F.svg?style=for-the-badge&logo=spring-boot&logoColor=white) |
| 👀 모니터링 | ![Grafana](https://img.shields.io/badge/Grafana-F46800.svg?style=for-the-badge&logo=grafana&logoColor=white) |
| 📊 데이터 수집 | ![Prometheus](https://img.shields.io/badge/Prometheus-E6522C.svg?style=for-the-badge&logo=prometheus&logoColor=white) |
| ⚡ 부하 테스트 | ![wrk](https://img.shields.io/badge/wrk-000000.svg?style=for-the-badge&logo=linux&logoColor=white) <br> ![sysbench](https://img.shields.io/badge/sysbench-007ACC.svg?style=for-the-badge&logo=linux&logoColor=white) |
| 💻 가상화 플랫폼 | ![VMware](https://img.shields.io/badge/VMware-607078.svg?style=for-the-badge&logo=vmware&logoColor=white) |



<br>

### 🎯 시나리오 흐름


#### ✅ Step 1. 경기 오픈 10분 전

- 사용자들이 앱에 접속 → `GET /seats` 호출
- **조회(Read)** 중심의 트래픽 발생  
- **점진적으로 부하 증가**

```bash
wrk -t4 -c50 -d30s http://<ip>:8080/seats
```


#### 🚀 Step 2. 경기 예매 시작! (D-day)

- 수천 건의 `POST /book` 요청 **동시 집중**
- 동일 좌석에 다수 사용자 예약 → **DB 충돌** 발생 가능
- DB에는 `INSERT`, `UPDATE`, `SELECT` 트랜잭션이 **복합적으로 작용**

```bash
wrk -t10 -c500 -d30s -s post_book.lua http://<ip>:8080/book
```

> ⚠️ 좌석 중복 예약이나 성능 저하를 테스트하기 위한 시나리오입니다.

##### 📄 post_book.lua (Lua 스크립트 예시)

```lua
-- POST 메서드 설정
wrk.method = "POST"
wrk.headers["Content-Type"] = "application/json"

-- 요청 본문 생성 함수
request = function()
  -- 랜덤한 seatId, userId 생성
  local seatId = math.random(100, 120)
  local userId = "user" .. math.random(1, 5000)

  -- JSON 형식 바디 구성
  local body = string.format('{"seatId":%d,"userId":"%s"}', seatId, userId)

  -- 최종 요청 반환
  return wrk.format(nil, "/book", nil, body)
end
```





<img src="https://capsule-render.vercel.app/api?type=waving&color=3F51B5&height=150&section=footer" width="1000" />
