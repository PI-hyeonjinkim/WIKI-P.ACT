# 디바이스 인벤토리

> 출처: raw/devices/2026-04-17_device-inventory.md
> 마지막 갱신: 2026-04-17

---

## 플랫폼 서버

| 항목 | 값 |
|------|-----|
| 로컬 IP | 192.168.33.33 |
| SSH | `ssh lenovo@192.168.33.33` (pw: lenovo) |
| 역할 | Docker Compose 플랫폼 전체 운영 |
| 접속 확인 | `docker ps` |

---

## EC2 프록시

| 항목 | 값 |
|------|-----|
| 도메인 | api.paidevteam.com |
| 공인 IP | 54.253.91.60 |
| ZeroTier IP | 10.235.62.214 |
| SSH | `ssh -i DNSKEY.pem ubuntu@api.paidevteam.com` |
| 역할 | nginx 리버스 프록시, coturn TURN 서버 |

---

## 엣지 디바이스

### MK3-0f5d86bc — 줌 PTZ 카메라

| 항목 | 값 |
|------|-----|
| 플랫폼 등록명 | 줌무한(어안두줄) PTZ |
| 로컬 IP | 192.168.33.66 |
| ZeroTier | 없음 |
| SSH | `ssh ptz@192.168.33.66` (pw: ptz) |
| AI | 없음 |
| 특이사항 | 로컬 망에서만 접근 가능, 브라우저 직접 연결 (TURN 불필요) |

---

### MK3-005f607f — P.ACT01 (AI 카메라)

| 항목 | 값 |
|------|-----|
| 플랫폼 등록명 | P.ACT01 |
| ZeroTier IP | 10.235.62.222 |
| SSH | .33 경유 점프 필요 (아래 참고) |
| AI 모델 | yolov11m.hef (/home/ptz/models/) |
| 연결된 줌 카메라 | 192.168.1.68 |
| mediamtx 스트림 | my_camera (AI), detection_zoom (줌), detection |

**SSH 접속:**
```bash
# 플랫폼 서버(.33)를 통해 점프
ssh -J lenovo@192.168.33.33 ptz@10.235.62.222
```

**알려진 이슈:**
- wide 뷰 black screen → mediamtx.yml `source: publisher` 확인 [[webrtc-blackscreen]]
- 재부팅 후 PTZ 미동작 → MQTT 연결 확인 [[mqtt-reconnect]]

---

## 목포 물류장

| 항목 | 값 |
|------|-----|
| 관리 서버 | 10.235.62.251 (LENOVO / 1234) |
| 배포 현황 | 안전감지 14대, TP 입출차 js-124(10.10.30.124) |
| 특이사항 | GUI 비활성화 적용 (RAM 절감), RTSP HW 인코더 적용 |
