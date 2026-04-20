# 서비스 관리 및 진단

> 출처: raw/decisions/2026-04-17_operations.md
> 마지막 갱신: 2026-04-17

---

## 엣지 디바이스 서비스 재시작

```bash
sudo systemctl restart mediamtx          # 스트리밍 서버
sudo systemctl restart edge-settings     # 메인 설정/MQTT
sudo systemctl restart edge-ptz          # PTZ API
sudo systemctl restart edge-healthcheck
```

---

## 서비스 로그 확인

```bash
journalctl -u edge-settings -n 50 --no-pager
journalctl -u mediamtx -n 50 --no-pager
journalctl -u edge-ptz -n 50 --no-pager
```

> ⚠️ journald 영구 저장이 안 된 경우 재부팅 후 로그 소실
> 활성화: `sudo sed -i 's/^#*Storage=.*/Storage=persistent/' /etc/systemd/journald.conf && sudo systemctl restart systemd-journald`

---

## 스트림 상태 확인

```bash
# mediamtx paths (엣지 디바이스)
curl -s http://localhost:9997/v3/paths/list

# 확인 포인트:
# - ready: true → 스트림 활성
# - source.type: rtspSession → gst RTSP push 정상
# - readers 배열 → webRTCSession 있으면 브라우저 연결 중
```

---

## 부팅 후 진단 체크리스트

```bash
# 1. MQTT 연결 확인
sudo ss -tnp state established | grep 1883
# 결과 없으면 → sudo systemctl restart edge-settings

# 2. 서비스 상태
systemctl is-active edge-ptz edge-settings mediamtx

# 3. 스트림 상태
curl -s http://localhost:9997/v3/paths/list | python3 -c \
  "import sys,json; [print(f\"{i['name']} ready={i['ready']}\") for i in json.load(sys.stdin)['items']]"

# 4. 설정 파일 확인
cat /home/ptz/edge/settings.json
```

---

## 플랫폼 서버 관리

```bash
# 컨테이너 상태
docker ps

# 백엔드 로그
docker logs platform-backend --tail 50

# 전체 재시작
docker compose -p platform up -d

# 특정 서비스만 재빌드
docker compose -p platform up -d --build backend
```

---

## coturn 관리 (EC2)

```bash
sudo systemctl restart coturn
sudo systemctl status coturn
sudo journalctl -u coturn -n 20

# 설정 파일
sudo cat /etc/turnserver.conf
```

---

## 디바이스 온라인 상태 확인

```bash
# Redis에서 직접 확인
docker exec platform-redis redis-cli get device:status:MK3-005f607f
# 값 있으면 online, 없으면 offline (TTL 120초)
```
