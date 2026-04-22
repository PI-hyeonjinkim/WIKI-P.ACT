# EC2 프록시 및 네트워킹

> 출처: raw/decisions/2026-04-17_platform-architecture.md, docs/aws-proxy/ (2026-04-22)
> 마지막 갱신: 2026-04-22

---

## EC2 서버 정보

| 항목 | 값 |
|------|-----|
| Public IP | 54.253.91.60 |
| ZeroTier IP | 10.235.62.214 |
| 플랫폼 서버 ZeroTier IP | 10.235.62.211 |
| 접속 | `ssh -i DNSKEY.pem ubuntu@api.paidevteam.com` |

---

## ZeroTier

원격 엣지 디바이스와 플랫폼을 연결하는 사설 overlay 네트워크.
- 네트워크 대역: 10.235.62.x
- 로컬 디바이스(.66, .45)는 ZeroTier 미사용 → 로컬 망에서만 접근 가능

---

## nginx 서브도메인 라우팅

| 도메인 | 목적지 | 용도 |
|--------|--------|------|
| `api.paidevteam.com` | ZeroTier → 10.235.62.211:8080 | FastAPI 백엔드 |
| `wiki.paidevteam.com` | ZeroTier → 10.235.62.211:4000 | Quartz 위키 |
| `grafana.paidevteam.com` | ZeroTier → 10.235.62.211:3001 | Grafana |
| `minio.paidevteam.com` | ZeroTier → 10.235.62.211:9001 | MinIO Console |
| `s3.paidevteam.com` | ZeroTier → 10.235.62.211:9000 | MinIO S3 API |
| `mlflow.paidevteam.com` | ZeroTier → 10.235.62.211:5000 | MLflow |
| `prom.paidevteam.com` | ZeroTier → 10.235.62.211:9090 | Prometheus |

### MQTT TCP 프록시 (포트 1883)

nginx stream 블록에서 TCP 프록시:
```nginx
stream {
    server {
        listen 1883;
        proxy_pass 10.235.62.211:1883;
    }
}
```

> ⚠️ mosquitto에 브리지 추가 절대 금지 → 루프 발생 (실제 사고 경험)

### 펌웨어 서빙

```nginx
location /firmware/ {
    proxy_pass http://10.235.62.211:9000/firmware/;
}
```

---

## Route 53 DNS 레코드

| 도메인 | 타입 | 목적지 | 용도 |
|--------|------|--------|------|
| `paidevteam.com` | A | 54.253.91.60 | 루트 |
| `api.paidevteam.com` | A | 54.253.91.60 | API + MQTT |
| `wiki.paidevteam.com` | A | 54.253.91.60 | 위키 |
| `grafana.paidevteam.com` | A | 54.253.91.60 | Grafana |
| `minio.paidevteam.com` | A | 54.253.91.60 | MinIO Console |
| `s3.paidevteam.com` | A | 54.253.91.60 | MinIO S3 |
| `mlflow.paidevteam.com` | A | 54.253.91.60 | MLflow |
| `prom.paidevteam.com` | A | 54.253.91.60 | Prometheus |
| `bridge.paidevteam.com` | A | 54.253.91.60 | ~~deprecated~~ |

**TTL**: 300s

---

## TURN 서버 (coturn)

WebRTC NAT 통과 실패 시 미디어 릴레이.

```
# /etc/turnserver.conf 핵심 설정
listening-port=3478
realm=api.paidevteam.com
use-auth-secret
static-auth-secret=pact_turn_2026
external-ip=54.253.91.60/172.31.36.71   ← EC2 NAT 필수
min-port=49152
max-port=65535
```

> ⚠️ `external-ip` 없으면 릴레이 주소가 사설 IP로 광고 → 외부 연결 불가

인증: HMAC-SHA1 time-limited (백엔드 `TURN_SECRET=pact_turn_2026`으로 생성)

```bash
# coturn 관리
sudo systemctl restart coturn
sudo journalctl -u coturn -n 20
```

---

## 브라우저 → 디바이스 연결 경로

| 디바이스 위치 | 브라우저 위치 | ICE 경로 |
|--------------|--------------|----------|
| 로컬 망(192.168.33.x) | 같은 로컬 망 | 직접 연결 |
| ZeroTier(10.235.62.x) | 외부 | TURN 릴레이 필수 |
| ZeroTier(10.235.62.x) | ZeroTier 내부 | 직접 연결 가능 |
