# http-keepalive-lab

HTTP Keep-Alive, HTTP/2 Multiplexing, HTTP/3 QUIC의 성능 차이를 실험으로 검증한다.

## 배경

"Keep-Alive는 켜져 있으니까"라고 넘기지만:
- Keep-Alive ON/OFF에서 TCP 3-way handshake 비용이 실제로 얼마인지?
- HTTP/1.1 Keep-Alive는 Head-of-Line Blocking이 있다 — 실제로 얼마나 영향?
- HTTP/2 multiplexing이 동일 연결에서 다중 요청을 병렬 처리한다는데 실제 차이는?
- HTTP/3(QUIC)은 UDP 기반으로 0-RTT 연결이 가능한데 체감 차이는?

## 가설

| # | 가설 | 근거 |
|---|------|------|
| H1 | Keep-Alive OFF는 ON보다 고부하에서 30%+ 느릴 것 | 매 요청마다 TCP handshake(1.5 RTT) 추가 |
| H2 | HTTP/2는 HTTP/1.1보다 소규모 요청 다수에서 유리할 것 | 단일 연결에서 스트림 멀티플렉싱 |
| H3 | HTTP/3(QUIC)은 패킷 로스 환경에서 HTTP/2보다 월등할 것 | 스트림별 독립 흐름 제어, HoL blocking 없음 |
| H4 | LAN 환경에서는 HTTP/1.1과 HTTP/2 차이가 미미할 것 | RTT가 <1ms이면 handshake 비용 무시 가능 |

## 실험 설계

### 서버
- Go net/http (HTTP/1.1, HTTP/2 기본 지원)
- Caddy 또는 nginx (HTTP/3 QUIC 지원)

### 시나리오
1. HTTP/1.1 Keep-Alive ON — 기본
2. HTTP/1.1 Keep-Alive OFF — `Connection: close` 헤더
3. HTTP/2 (TLS 필수)
4. HTTP/3 QUIC (UDP)

### 측정
- k6로 throughput, latency 측정
- tcpdump로 TCP handshake 횟수 캡처
- 패킷 로스 시뮬레이션 (tc netem)으로 QUIC 이점 확인

## 실험 환경

- 서버: Docker (2 CPU, 512MB RAM)
- 클라이언트: 별도 PC에서 k6/curl
- 네트워크: LAN + tc netem으로 지연/로스 시뮬레이션
