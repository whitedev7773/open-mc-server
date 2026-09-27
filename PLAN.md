# Open MC Server 프로젝트 기획

## 1. 프로젝트 개요

### 프로젝트명

**Open MC Server**

### 목표

Minecraft에서 별도의 서버 구축, 포트포워딩, IP 공유 없이 사용자가 자신의 싱글플레이 월드를 공개하고 다른 플레이어가 공개방 목록에서 해당 월드를 찾아 바로 참가할 수 있도록 한다.

기본 사용자 흐름은 다음과 같다.

```text
싱글플레이 월드 실행
→ 공개방 열기
→ 중앙 공개방 목록 등록
→ 다른 사용자가 방 검색
→ 참가 버튼 클릭
→ Relay를 통한 안전한 연결
→ 멀티플레이 시작
```

### 핵심 가치

1. **접근성**
   - 포트포워딩 불필요
   - 서버 구축 불필요
   - IP 직접 입력 불필요
   - 원클릭 참가

2. **발견성**
   - 공개방 목록 제공
   - 검색 및 필터
   - 태그 기반 탐색

3. **보안**
   - Host IP 보호
   - 참가자 IP 보호
   - Relay 기반 통신
   - 일회성 참가 인증
   - DDoS 및 Abuse 대응

---

# 2. 전체 시스템 구성

PublicRoom은 크게 다음 네 영역으로 구성한다.

```text
PublicRoom
├─ Minecraft Mod
├─ Control Backend
├─ Relay Network
└─ Management / Moderation
```

## Minecraft Mod

실제 사용자가 Minecraft 안에서 사용하는 클라이언트 기능이다.

주요 기능:

- 공개방 목록
- 공개방 검색
- 방 생성
- 방 참가
- 방 관리
- 플레이어 관리
- 연결 상태 표시
- 신고 및 차단
- 친구 및 초대 기능
- Integrated Server 연동

---

## Control Backend

게임 패킷을 전달하지 않고 서비스 상태를 관리한다.

주요 기능:

- 사용자 인증
- Minecraft 계정 확인
- 사용자 관리
- Room 생성 및 삭제
- Room 검색
- Room Heartbeat
- Join Ticket 발급
- Relay 서버 배정
- Rate Limit
- Abuse Detection
- 신고 및 제재

---

## Relay Network

Host와 참가자 간 Minecraft 트래픽을 중계한다.

```text
Host Minecraft
      │
      │ Encrypted Tunnel
      ▼
    Relay
      ▲
      │ Encrypted Tunnel
      │
Player Minecraft
```

PublicRoom의 핵심 네트워크 인프라이다.

주요 역할:

- Host IP 은닉
- 참가자 IP 은닉
- NAT 및 공유기 환경 우회
- 포트포워딩 제거
- 연결 수 제한
- Traffic Rate Limit
- 비정상 연결 차단
- Room별 Session 격리

---

## Management / Moderation

서비스 운영자가 사용하는 관리 시스템이다.

주요 기능:

- 신고 확인
- 사용자 제재
- 방 강제 종료
- Room 생성 제한
- Relay 사용 제한
- Abuse 탐지
- 서버 및 Relay 상태 확인
- 통계 및 Monitoring

---

# 3. Minecraft 사용자 기능

## 3.1 공개방 메뉴

Minecraft 메인 메뉴 또는 멀티플레이 메뉴에 PublicRoom 진입점을 추가한다.

```text
[싱글플레이]
[멀티플레이]
[공개방]
[설정]
```

공개방 화면 예시:

```text
┌─────────────────────────────────────┐
│ 공개방                              │
│                                     │
│ [검색...........................]    │
│                                     │
│ 기원의 야생               4 / 8     │
│ Survival · 1.21.x · 24 ms           │
│ #생존 #건축                [참가]    │
│                                     │
│ Skyblock 같이 해요        2 / 4     │
│ Skyblock · 1.21.x · 31 ms            │
│                            [참가]    │
└─────────────────────────────────────┘
```

지원 기능:

- 방 목록
- 검색
- 새로고침
- 정렬
- 필터
- Room 상세 정보
- 참가
- 즐겨찾기

---

# 4. 공개방 생성

싱글플레이 월드 실행 중 다음 메뉴를 제공한다.

```text
ESC
→ 공개방 열기
```

설정 가능한 항목:

```text
방 이름
방 설명
최대 인원

게임 모드
난이도

공개 범위
├─ 공개
├─ 비밀번호
├─ 친구 전용
├─ 초대 전용
└─ 비공개 링크

태그
├─ 생존
├─ 건축
├─ PvP
├─ 미니게임
└─ 기타
```

방을 생성하면 내부적으로 다음 과정이 수행된다.

```text
Integrated Server 준비
        ↓
Backend에 Room 생성 요청
        ↓
Relay 서버 할당
        ↓
Host Tunnel Token 발급
        ↓
Host → Relay Tunnel 연결
        ↓
Room 활성화
        ↓
공개방 목록 노출
```

사용자는 자신의 IP나 포트를 직접 설정하지 않는다.

---

# 5. 방 관리

Host는 공개방 생성 후 별도의 관리 화면을 사용할 수 있다.

```text
공개방 관리

기원의 야생

상태
● 공개 중

플레이어
4 / 8

접속자
- Giwon
- Steve
- Alex
- Player123

[플레이어 관리]
[공개 설정 변경]
[초대 코드]
[방 닫기]
```

플레이어 관리 기능:

- Kick
- Ban
- 신고
- 차단
- 권한 관리

향후 추가:

- OP 권한
- Whitelist
- 친구 등록
- Host 승인제

---

# 6. 참가 기능

사용자는 IP 주소를 입력하지 않는다.

```text
공개방 선택
→ 참가
```

실제 연결 과정:

```text
Client
  ↓
Backend에 Join 요청
  ↓
사용자 및 Room 검증
  ↓
Join Ticket 발급
  ↓
Relay 접속
  ↓
Ticket 검증
  ↓
Host Tunnel 연결
  ↓
Minecraft Session 시작
```

---

# 7. 네트워크 기본 정책

## Public Room은 Relay Only

공개방에서는 P2P Direct Connection을 사용하지 않는다.

```text
Host
 │
 │ outbound encrypted tunnel
 ▼
Relay
 ▲
 │ encrypted tunnel
 │
Player
```

공개방에서 사용하지 않는 기술:

- 직접 Port Forwarding
- UPnP
- NAT-PMP
- STUN 기반 직접 연결
- NAT Hole Punching
- ICE P2P Candidate 교환

이유는 Host IP 및 참가자 IP 노출 가능성을 제거하기 위해서이다.

---

# 8. 유명인 및 스트리머 IP 보호

PublicRoom의 핵심 보안 목표 중 하나이다.

참가자에게 다음 정보를 절대 전달하지 않는다.

```text
Host Public IPv4
Host IPv6
Host Local IP
Host Minecraft Port
```

참가자에게 전달되는 정보는 다음 정도로 제한한다.

```text
roomId
joinTicket
relayEndpoint
expiration
```

예:

```json
{
  "roomId": "rm_f39a2",
  "joinTicket": "...",
  "relay": "kr-03.relay.example.com",
  "expiresAt": 1790520134
}
```

참가자는 Relay의 주소만 확인할 수 있으며 Host의 실제 네트워크 주소는 알 수 없다.

---

# 9. Room 데이터 구조

예시:

```text
Room
├─ id
├─ hostId
├─ name
├─ description
├─ minecraftVersion
├─ publicRoomVersion
├─ currentPlayers
├─ maxPlayers
├─ gameMode
├─ tags
├─ relayRegion
├─ visibility
├─ createdAt
└─ lastHeartbeat
```

---

# 10. Public API 데이터 분리

Database 객체를 그대로 Client에 전달하지 않는다.

```text
Database Model
≠
Public API Model
```

예:

```text
PublicRoomInfo
├─ roomId
├─ roomName
├─ hostDisplayName
├─ playerCount
├─ maxPlayers
├─ minecraftVersion
├─ tags
└─ ping
```

다음과 같은 내부 정보는 포함하지 않는다.

```text
Host IP
Player IP
내부 Relay ID
Host Tunnel Token
DB 내부 ID
Email
내부 Abuse 정보
```

---

# 11. Room 생명주기

Room은 영구 객체가 아니라 Lease 형태로 관리한다.

정상 상태:

```text
CREATE
  ↓
ACTIVE
  ↓
CLOSING
  ↓
CLOSED
```

비정상 종료:

```text
ACTIVE
  ↓
UNHEALTHY
  ↓
EXPIRED
```

예시 정책:

```text
Heartbeat: 5초

15초 이상 미응답
→ UNHEALTHY

30초 이상 미응답
→ EXPIRED
→ Room 목록 제거
```

Minecraft가 강제 종료되더라도 유령 방이 계속 남지 않는다.

---

# 12. 사용자 인증

익명 사용자가 공개방을 무제한 생성하지 못하도록 Minecraft 계정 기반 인증을 사용한다.

기본 사용자 식별값:

```text
Minecraft UUID
```

구조:

```text
Minecraft Account
        ↓
Authentication
        ↓
Minecraft UUID
        ↓
PublicRoom User
```

초기 버전에서는 PublicRoom 자체 ID/PW 계정을 따로 만들지 않는 방향을 우선 고려한다.

---

# 13. Join Ticket

Room 참가 시 실제 연결 정보를 직접 전달하지 않고 일회성 Join Ticket을 발급한다.

```text
POST /rooms/{roomId}/join
```

응답:

```text
roomId
relay
joinTicket
expiresAt
```

Ticket에 포함되는 정보:

```text
roomId
playerId
relayId
expiration
nonce
signature
```

예상 유효시간:

```text
20~30초
```

Ticket은 기본적으로 한 번만 사용할 수 있다.

```text
첫 번째 사용
→ ACCEPT

동일 Ticket 재사용
→ DENY
```

---

# 14. Host Tunnel Token

Host 역시 임의의 Room으로 Relay를 등록할 수 없어야 한다.

방 생성 시 Backend에서:

```text
roomId
relayEndpoint
hostTunnelToken
```

을 발급한다.

Host는 다음 정보를 이용해서 Relay에 연결한다.

```text
roomId
hostTunnelToken
```

Relay는 Token을 검증한 뒤에만 해당 Room용 Host Tunnel을 생성한다.

---

# 15. 연결 종료 보안

정상적인 방 종료:

```text
Host: Close Room
      ↓
Room → CLOSING
      ↓
신규 Join 차단
      ↓
활성 Join Ticket 폐기
      ↓
Player Session 종료
      ↓
Relay Mapping 제거
      ↓
Host Tunnel 종료
      ↓
Room → CLOSED
```

비정상 종료:

```text
Minecraft Crash
또는
Internet Disconnect
      ↓
Host Tunnel Lost
      ↓
Relay 감지
      ↓
신규 Join 차단
      ↓
기존 Session 정리
      ↓
Room 자동 만료
```

종료된 Room의 Token과 Session은 재사용하지 않는다.

---

# 16. 공개방 Spam 방지

악의적인 사용자가 수천~수만 개의 Room을 생성하는 공격을 고려한다.

기본 정책:

- 인증된 사용자만 Room 생성
- Minecraft 계정 하나당 동시 공개방 1개
- Room 생성 Rate Limit
- 반복 생성 탐지
- 신고 및 Abuse History 반영

예:

```text
POST /rooms

계정 기준:
3회 / 10분

IP 기준:
보다 느슨한 제한 적용
```

Rate Limit은 단순 IP 기준이 아니라 다음 값을 조합한다.

```text
Account
Session
IP / IP Prefix
```

학교, 회사, 군부대, PC방, CGNAT 환경에서 여러 사용자가 하나의 공인 IP를 사용할 수 있기 때문이다.

---

# 17. Room Abuse Detection

내부적으로 사용자 및 Room 활동을 분석할 수 있다.

예:

```text
Room 생성 빈도
Room 평균 유지시간
신고 횟수
실제 접속자 수
동일 제목 반복
비정상적인 Join 실패율
```

내부 상태 예:

```text
NORMAL
SUSPICIOUS
RESTRICTED
BANNED
```

사용자에게 공개적인 점수로 표시할 필요는 없다.

---

# 18. DDoS 방어 구조

Control Plane과 Data Plane을 분리한다.

## Control Plane

```text
Authentication
Room API
Search
Join Ticket
Moderation
```

## Data Plane

```text
Minecraft Relay
```

구조:

```text
                    CDN / WAF
                        │
                   API Gateway
                        │
        ┌───────────────┼───────────────┐
        │               │               │
       Auth            Room           Abuse
        │               │               │
        └───────────────┼───────────────┘
                       DB
                      Redis


────────────────────────────────────

Host
 │
 ▼

Relay Cluster

 ▲
 │
Player
```

Relay가 공격당하더라도 인증이나 Room 검색 등의 Control Plane 서비스가 같이 중단되지 않도록 한다.

---

# 19. Origin 보호

Control Backend의 실제 서버 IP를 직접 노출하지 않는다.

```text
Internet
   ↓
CDN / WAF
   ↓
Load Balancer / API Gateway
   ↓
Backend
```

Backend Origin은 가능한 경우 CDN 또는 Load Balancer에서 오는 연결만 허용한다.

---

# 20. Relay Pool

Relay를 단일 서버로 구성하지 않는다.

예:

```text
KR
├─ KR-01
├─ KR-02
├─ KR-03
└─ KR-04

JP
├─ JP-01
└─ JP-02

SG
└─ SG-01
```

Room 생성 시 적절한 Relay를 할당한다.

```text
Room A → KR-02
Room B → KR-04
Room C → KR-01
```

향후 Relay 배정 기준:

- Host Latency
- Client 예상 Latency
- Relay CPU 부하
- Relay Memory
- Network Bandwidth
- 지역
- 장애 여부

---

# 21. Relay 자원 제한

Room 하나가 Relay의 자원을 무제한 소비할 수 없게 한다.

Room 및 Connection 기준 제한:

```text
최대 동시 연결
최대 Handshake 수
초당 신규 Connection
최대 Packet 크기
최대 Bandwidth
Idle Timeout
Handshake Timeout
```

예:

```text
Room Max Players = 8

Active Sessions = 8
Pending Handshakes = 최대 4
```

공격자가 수천 개의 Connection을 생성해도 Host의 Minecraft 서버까지 그대로 전달하지 않는다.

---

# 22. Minecraft 프로토콜 보호

Relay는 Minecraft 프로토콜을 지나치게 많이 이해하지 않는 구조를 우선한다.

기본 역할:

```text
Encrypted Stream Relay
```

검사 항목:

- Frame Size
- Connection State
- Timeout
- Bandwidth
- 비정상 Connection Pattern

Minecraft 프로토콜 전체를 Relay가 해석하도록 만들면 버전 호환성과 보안 유지 비용이 크게 증가한다.

---

# 23. Host Connection Guard

Relay를 통과한 트래픽도 Integrated Server 앞에서 추가적으로 검증할 수 있다.

```text
Player
 ↓
Relay
 ↓
Connection Guard
 ↓
Minecraft Integrated Server
```

검증 대상:

- 비정상 Packet
- Packet Spam
- Oversized Payload
- Invalid Handshake
- 비정상 State Transition
- Custom Payload 크기

---

# 24. PublicRoom 자체 Protocol

Custom Payload는 별도 Namespace를 사용한다.

예:

```text
publicroom:hello
publicroom:capabilities
publicroom:session
publicroom:status
```

모든 Packet Handler에서 다음 검증을 수행한다.

```text
Length Check
Range Check
Type Check
Permission Check
State Check
```

Client가 전달하는 값은 신뢰하지 않는다.

---

# 25. 방 검색

MVP 검색 기능:

- 방 이름
- Host 이름
- 태그

필터:

- Minecraft 버전
- 게임 모드
- 인원수
- 지역
- 비밀번호 여부

정렬:

- 최근 생성
- 플레이어 수
- Ping

향후 추가:

- 즐겨찾기
- 최근 참가
- 친구가 참가한 방
- 추천

---

# 26. Ping 표시

Host IP를 직접 Ping하지 않는다.

Client와 Relay 사이의 Latency를 측정한다.

```text
Client
↓
KR Relay
22 ms
```

Host 또한 Relay 기준 Latency를 측정한다.

```text
Host
↓
KR Relay
14 ms
```

이를 기반으로 연결 품질을 추정한다.

표시 방법:

```text
22 ms
```

또는

```text
좋음
보통
나쁨
```

---

# 27. 친구 및 비공개 방

후속 버전에서 Social 기능을 추가한다.

지원 가능한 Visibility:

```text
PUBLIC
UNLISTED
FRIENDS
INVITE_ONLY
PASSWORD
```

## PUBLIC

공개 목록에 노출된다.

## UNLISTED

검색 목록에는 표시되지 않고 Room Code를 통해서만 참가한다.

## FRIENDS

Host의 친구만 참가할 수 있다.

## INVITE_ONLY

Host의 초대 또는 승인이 필요하다.

## PASSWORD

비밀번호를 알아야 참가할 수 있다.

---

# 28. Room Code

Discord 등의 외부 서비스로 쉽게 공유할 수 있도록 Room Code 기능을 제공한다.

예:

```text
AB7D-K2MQ
```

Room Code는 서버 주소가 아니다.

```text
Room Code
    ↓
Backend
    ↓
Room 확인
    ↓
Join Ticket 발급
    ↓
Relay
```

Room Code가 외부에 노출되더라도 Host IP는 노출되지 않는다.

---

# 29. 친구 기능

후속 기능:

- 친구 요청
- 친구 수락
- 친구 삭제
- 친구 목록
- Online 상태
- 친구가 플레이 중인 Room
- 친구 전용 공개방
- 직접 초대

---

# 30. 신고 및 차단

사용자 기능:

- Room 신고
- Player 신고
- Player 차단
- Room 숨기기

Host 기능:

- Kick
- Temporary Ban
- Permanent Room Ban

운영자 기능:

- 신고 목록
- 사용자 일시 정지
- 사용자 영구 정지
- Room 강제 종료
- Room 생성 권한 제한
- Relay 사용 제한

---

# 31. Minecraft Mod 호환성

초기 버전에서는 지원 범위를 좁힌다.

우선 지원:

```text
Fabric
PublicRoom Mod
Vanilla-compatible environment
```

추후 Host의 다음 정보를 제공한다.

```text
Minecraft Version
Mod Loader
Loader Version
Required Mod List
Mod Hash
```

접속 전에 호환성 검사 가능:

```text
접속할 수 없습니다.

필요한 모드:
- Create
- Fabric API
```

---

# 32. 개인정보 보호

서버에서 장기 보관하지 않을 데이터:

```text
Raw IP
Join Ticket
Host Tunnel Token
Relay Session Token
Session Mapping
```

보안 목적의 IP Log 역시 필요한 기간만 보관한다.

일반 통계는 가능한 경우 익명화된 집계 데이터로 전환한다.

예:

```text
지역별 사용자 수
평균 Room 유지시간
평균 플레이어 수
Relay 사용량
평균 Session 시간
```

---

# 33. 로그 정책

필요한 정보만 기록한다.

보안 로그 예:

```text
Timestamp
Anonymous / Internal User ID
Action Type
Result
Relay Region
Abuse Flag
```

민감한 값:

```text
Token
Password
Raw Authentication Credential
Full Packet Payload
```

등은 일반 로그에 기록하지 않는다.

---

# 34. 장애 처리

다음 상황을 고려한다.

## Relay 장애

```text
Relay 장애 감지
→ 해당 Relay 신규 Room 할당 중단
→ Backend 상태 갱신
→ 새로운 Relay 할당
```

향후에는 기존 Session Migration도 검토할 수 있다.

## Backend 장애

이미 연결된 게임 Session은 가능한 경우 즉시 종료되지 않도록 Control Plane과 Data Plane을 분리한다.

## Host Crash

```text
Host Tunnel 종료
→ Room UNHEALTHY
→ 신규 Join 중단
→ Room 만료
```

---

# 35. 프로젝트 저장소 구조

권장 Monorepo 구조:

```text
publicroom/
│
├─ mod/
│  └─ Minecraft Fabric Mod
│
├─ api/
│  └─ Control Backend
│
├─ relay/
│  └─ Relay Server
│
├─ protocol/
│  └─ Shared Protocol Definition
│
├─ web/
│  └─ Admin Dashboard
│
├─ infrastructure/
│  └─ Docker / Deployment / IaC
│
├─ tests/
│
└─ docs/
```

---

# 36. 기술 스택 후보

## Minecraft Mod

```text
Java
Fabric
```

## Control Backend

```text
TypeScript
Fastify
```

또는:

```text
Go
```

## Database

```text
PostgreSQL
```

## Ephemeral State

```text
Redis
```

용도:

- Room Presence
- Heartbeat
- Rate Limit
- Join Ticket Nonce
- Session 상태

## Relay

후보:

```text
Rust
```

또는

```text
Go
```

Relay에서는 대량의 동시 Connection과 네트워크 I/O가 중요하다.

---

# 37. API 기능

예시 API:

```text
POST   /auth/session

GET    /rooms
POST   /rooms
GET    /rooms/{roomId}
DELETE /rooms/{roomId}

POST   /rooms/{roomId}/heartbeat
POST   /rooms/{roomId}/join

POST   /rooms/{roomId}/report
POST   /users/{userId}/report

POST   /room-codes/{code}/join
```

향후:

```text
GET    /friends
POST   /friends/request
POST   /friends/accept

POST   /rooms/{roomId}/invite
```

---

# 38. 개발 단계

## Phase 0 — Prototype

목표:

- Fabric Mod 기본 구조
- UI 삽입
- Integrated Server 테스트
- Minecraft 자동 접속 테스트

---

## Phase 1 — PublicRoom UI

구현:

- 공개방 메뉴
- Room 리스트
- Room 상세 화면
- Room 생성 화면
- 가짜 데이터 기반 UI

이 단계에서는 실제 Backend 연결 없이 UI를 완성한다.

---

## Phase 2 — Room Backend

구현:

- Room REST API
- PostgreSQL
- Redis
- Room 생성
- Room 삭제
- Heartbeat
- Room 검색

---

## Phase 3 — Relay Prototype

구현:

```text
Host
→ Relay
→ Client
```

Minecraft TCP Stream을 Relay를 통해 전달하는 기능을 먼저 검증한다.

---

## Phase 4 — One-click Join

구현:

- Join Ticket
- Relay Session
- 자동 Minecraft 연결
- Room 선택 → 참가 흐름 완성

---

## Phase 5 — Security MVP

구현:

- Minecraft 계정 인증
- Relay Only 정책
- Host Tunnel Token
- Join Ticket Replay 방지
- Rate Limit
- Room 생성 제한
- Session Timeout

---

## Phase 6 — Public Alpha

구현:

- 공개 Room 운영
- 검색
- 태그
- Kick
- Ban
- Report
- Abuse 관리
- 관리자 Dashboard

---

## Phase 7 — Relay Cluster

구현:

- Multi Relay
- 지역별 Relay
- Relay 상태 Monitoring
- 자동 Relay 선택
- 장애 서버 제외

---

## Phase 8 — Social

구현:

- 친구
- 초대
- Room Code
- Friends Only
- Invite Only
- Password Room

---

## Phase 9 — Compatibility

구현:

- Mod 정보 확인
- Mod List Hash
- Loader 확인
- Version Compatibility 검사

---

## Phase 10 — Production

구현:

- Auto Scaling
- Multi-region
- DDoS Protection
- Abuse Detection
- Monitoring
- Alert
- 관리자 운영 도구
- 장애 자동 복구

---

# 39. PublicRoom 0.1 MVP

첫 공개 테스트 버전에 포함할 기능:

## Minecraft Mod

- 공개방 메뉴
- Room 목록
- Room 생성
- Room 종료
- Room 참가
- Room 상세 정보

## Networking

- Integrated Server 공개
- Relay Only 연결
- Host Outbound Tunnel
- 원클릭 참가

## Security

- Host IP 보호
- 참가자 IP 보호
- Join Ticket
- Host Tunnel Token
- Ticket 재사용 방지
- Connection Timeout
- 기본 Traffic Limit

## Backend

- 사용자 인증
- Room API
- Heartbeat
- Room 자동 만료
- Relay 할당

## Abuse Protection

- 계정당 공개 Room 1개
- Room 생성 Rate Limit
- Join Rate Limit

## Basic Moderation

- Kick
- Report

## Information

- 현재 플레이어 수
- 최대 플레이어 수
- Minecraft 버전
- Ping

---

# 40. PublicRoom 0.2

추가 기능:

- 방 검색
- Room 태그
- 정렬 및 필터
- 비밀번호 Room
- UNLISTED Room
- Room Code
- 즐겨찾기
- 최근 참가
- Relay 지역 선택

---

# 41. PublicRoom 0.3

Social 기능:

- 친구
- 친구 요청
- 친구 초대
- Friends Only Room
- Invite Only Room
- Host 참가 승인
- 사용자 차단
- 강화된 Moderation

---

# 42. PublicRoom 0.4

호환성 및 편의 기능:

- Mod 호환성 검사
- Required Mod 표시
- Minecraft Loader 검사
- Relay 자동 지역 선택
- Room 설정 Preset
- Room 재생성 편의 기능

---

# 43. PublicRoom 1.0

Production 수준 목표:

- Multi-region Relay Cluster
- Auto Scaling
- DDoS Protection
- Abuse Scoring
- Monitoring
- Alerting
- 운영자 Dashboard
- 자동 장애 대응
- 개인정보 최소 보존
- 안정적인 업데이트 시스템
- Relay Load Balancing
- Room Discovery 최적화

---

# 44. 보안 핵심 원칙

PublicRoom은 다음 원칙을 프로젝트 전체에서 유지한다.

1. 공개방에서는 P2P를 사용하지 않는다.
2. Host는 외부에 Port를 열지 않는다.
3. 참가자는 Host IP를 전달받지 않는다.
4. Host 역시 참가자 IP를 직접 알 필요가 없다.
5. 모든 참가에는 단기 Join Ticket을 사용한다.
6. Ticket은 일회성으로 사용한다.
7. Host와 Relay 연결에도 별도의 인증 Token을 사용한다.
8. Room 생성은 인증된 Minecraft 계정만 허용한다.
9. Rate Limit은 Account, Session, IP를 복합적으로 사용한다.
10. Control Plane과 Relay Data Plane을 분리한다.
11. Relay에서 공격 Traffic을 Host에 도달하기 전에 차단한다.
12. Room 종료 후 모든 Token과 Session을 폐기한다.
13. Database Model을 그대로 Public API로 노출하지 않는다.
14. Raw IP와 Token 등 민감정보는 최소한으로 저장한다.
15. Minecraft Client에서 전달되는 데이터는 신뢰하지 않는다.

---

# 45. 프로젝트 우선순위

개발 및 제품 결정의 우선순위는 다음과 같이 설정한다.

```text
1. IP Privacy
2. 안전한 Network Architecture
3. 안정적인 연결
4. 간편한 Room 생성 및 참가
5. 공개방 Discovery
6. Abuse / Spam / DDoS 대응
7. Moderation
8. Social 기능
9. Mod Compatibility
10. 추천 및 고급 기능
```

---

# 46. 최종 제품 정의

> **PublicRoom은 Minecraft 싱글플레이 월드를 포트포워딩 없이 공개하고, 중앙 공개방 목록에서 다른 사용자가 해당 월드를 찾아 원클릭으로 참가할 수 있도록 하는 멀티플레이 플랫폼이다. 공개방의 모든 네트워크 연결은 Relay를 통해 중계하여 Host와 참가자의 실제 IP를 서로 노출하지 않으며, 인증·일회성 Join Ticket·Rate Limit·Room Lifecycle·Abuse Protection을 통해 공개 서비스 환경에서 발생할 수 있는 보안 위협을 최소화한다.**

핵심 구조는 다음 세 요소를 중심으로 유지한다.

```text
Minecraft Mod
      +
Room Directory / Control Backend
      +
Privacy Relay Network
```

이 세 요소를 먼저 안정적으로 완성한 후 친구, 초대, Mod 호환성, 추천 시스템 등의 부가 기능을 확장한다.
