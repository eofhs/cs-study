# [Wireshark] 패킷 캡슐화 계층과 HTTP/3(QUIC)

## 1. 계층 구조 이해 (캡슐화)
와이어샤크의 **Protocol Hierarchy(프로토콜 계층 통계)**는 데이터가 전송될 때 포장되는 계층 순서를 보여준다.

* **Ethernet II (L2 / Data Link)**: 로컬 네트워크 장비 식별 (MAC 주소)
* **IPv4 (L3 / Network)**: 통신 대상 컴퓨터 식별 (IP 주소)
* **UDP (L4 / Transport)**: 컴퓨터 내 실행 중인 대상 프로그램/포트 식별 (Port 443 등)
* **QUIC IETF (L7 / App & Security)**: 실제 암호화된 웹 트래픽 (HTTP/3 데이터 + TLS 1.3 암호화)

---

## 2. 왜 TCP/TLS가 아닌 UDP/QUIC인가?
Chrome 등 최신 브라우저와 Google 서비스는 기존 HTTP/1.1·HTTP/2 대신 성능이 향상된 **HTTP/3**을 기본으로 사용한다.

| 비교 항목 | 전통적인 HTTPS (HTTP/1.1, HTTP/2) | 최신 HTTPS (HTTP/3) |
| :--- | :--- | :--- |
| **스택 구조** | `IPv4` → `TCP` → `TLS` → `HTTP` | `IPv4` → `UDP` → `QUIC` |
| **전송 프로토콜** | TCP (3-Way Handshake 필요) | UDP (연결 수립 오버헤드 없음) |
| **암호화 방식** | 별도 TLS 핸드셰이크 진행 | QUIC 엔진 내부에 TLS 1.3 내장 (1-RTT / 0-RTT) |
| **특징** | 지연 시간(Latency) 상대적 큼 | 접속 속도 빠름, 패킷 손실 시 병목 최소화 |

---

## 3. 와이어샤크 핵심 디스플레이 필터

### 1) 프로토콜별 기본 필터
* **일반 TLS/HTTPS**: `tls` 또는 `tcp.port == 443`
* **HTTP/3(QUIC)**: `quic`

### 2) 도메인(SNI) 필터링
* **TLS 도메인 검색**: `tls.handshake.extensions_server_name contains "google"`
* **QUIC 도메인 검색**: `quic.crypto.sni contains "google"`
* **통합 검색 (TLS + QUIC 동시 적용)**:
  ```text
  tls.handshake.extensions_server_name contains "google" || quic.crypto.sni contains "google"
