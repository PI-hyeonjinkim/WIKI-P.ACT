# EC2 nginx WebSocket 미설정으로 영상/MQTT 연결 불가

> 출처: raw/troubleshooting/2026-04-21_ec2-nginx-websocket-mqtt-tunnel.md

---

## 증상

- 플랫폼 Live 화면 영상 미출력
- 브라우저 콘솔: `MQTT 연결 타임아웃` (8초)
- 백엔드 로그: `GET /ws/tunnel/{device_id} 404` 반복
- **로컬망(192.168.33.33)에서는 수 주간 정상 동작**, 외부 접속(paidevteam.com) 시 문제

## 원인

EC2 nginx `api.paidevteam.com` 블록에 WebSocket 경로 3가지가 누락됨.

| 경로 | 목적 | 누락 이유 |
|------|------|-----------|
| `/mqtt` | 브라우저 → Mosquitto :9883 (MQTT WebSocket) | 처음부터 미설정 |
| `/ws/` | 엣지 → 플랫폼 WS 역방향 터널 | 2026-04-06 ws_tunnel 추가 시 nginx 업데이트 누락 |
| `http2` 제거 | WebSocket Upgrade 충돌 방지 | nginx 1.18.0은 HTTP/2 + WebSocket 동시 처리 불가 |

> nginx 1.18.0에서 `listen 443 ssl http2` 활성화 시, 브라우저가 HTTP/2로 협상 후 WebSocket upgrade 시도 → 502 반환 (RFC 8441 미지원)

## 해결

```bash
# EC2: /etc/nginx/sites-enabled/paidevteam.conf 수정

# 1. api.paidevteam.com 블록에 추가
location /mqtt {
    proxy_pass http://10.235.62.211:9883;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_read_timeout 86400;
}

location /ws/ {
    proxy_pass http://10.235.62.211:8080;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_read_timeout 86400;
}

# 2. 전체 443 블록에서 http2 제거
sudo sed -i 's/listen 443 ssl http2;/listen 443 ssl;/g' /etc/nginx/sites-enabled/*.conf

# 3. 적용
sudo nginx -t && sudo systemctl reload nginx
```

## 재발 방지

- **백엔드에 WebSocket 엔드포인트 추가 시** → EC2 nginx도 반드시 동시 업데이트
- **내부망 테스트만으로는 nginx 프록시 문제 미검출** → 외부 도메인으로도 확인 필요
- **nginx 1.18.0 + http2**: WebSocket 경로에서 502 발생 시 http2 제거 먼저 시도

### 플랫폼 WebSocket 경로 목록

| nginx 경로 | 프록시 대상 | 용도 |
|-----------|------------|------|
| `/mqtt` | 10.235.62.211:9883 | 브라우저 MQTT (WebRTCPlayer 시그널링) |
| `/ws/tunnel/{id}` | 10.235.62.211:8080 | 엣지↔플랫폼 역방향 HTTP 터널 |
