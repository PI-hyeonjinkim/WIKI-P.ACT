# 신규 MK3pro 장비 설치 트러블슈팅

> 출처: raw/troubleshooting/2026-04-22_신규장비-설치-전체과정.md
> 마지막 갱신: 2026-04-22 (ZeroTier 10.235.62.165 신규 장비 설치 실증)

---

## 1. SIM 라우터 환경 — DNS 실패

## 증상
`curl`, `apt-get update` 실패. 네트워크 연결은 되지만 이름 해석 불가.

## 원인
모바일 라우터(192.168.1.1)가 DNS 서버로 등록되지만 실제 DNS 쿼리에 응답 안 함.

## 해결
```bash
CON=$(nmcli -t -f NAME con show --active | head -1)
nmcli con mod "$CON" ipv4.dns "8.8.8.8 1.1.1.1" ipv4.ignore-auto-dns yes
nmcli con up "$CON"
```

## 재발 방지
install.sh 실행 전 `curl -s https://api.paidevteam.com/health` 로 확인. 실패 시 DNS 수동 설정.

---

## 2. WHEP 404 — 부팅 직후 스트림 없음

## 증상
```json
{"error":"no stream is available on path 'my_camera'"}
```
장비 등록 직후 또는 재부팅 후 지속 발생.

## 원인
`mediamtx.yml`의 `my_camera` 기본 source가 `publisher` → edge-settings 서비스 기동 전에는 아무도 publish 안 함.

## 해결
`edge/mediamtx.yml` 수정 (2026-04-22 MinIO 반영 완료):
```yaml
paths:
  my_camera:
    source: rpiCamera   # publisher 아님!
```

## 재발 방지
- `my_camera` 섹션에 `rpiCameraWidth/Height/FPS` 파라미터 넣지 말 것 — AI ON(source=publisher) 상태에서도 mediamtx가 rpiCamera를 시도해 카메라 점유 충돌 발생
- 해당 파라미터는 global defaults 섹션에만 설정

---

## 3. GStreamer 도구 미설치

## 증상
`gst-launch-1.0: command not found` — AI 파이프라인 시작 실패.

## 원인
신규 Raspberry Pi 이미지에 GStreamer 기본 미설치.

## 해결
```bash
sudo apt-get install -y \
  gstreamer1.0-tools \
  gstreamer1.0-plugins-base \
  gstreamer1.0-plugins-good \
  gstreamer1.0-plugins-bad \
  gstreamer1.0-plugins-ugly \   # x264enc 포함 — 없으면 파이프라인 실패
  gstreamer1.0-rtsp
```

## 재발 방지
install.sh section 3에 반영 완료 (2026-04-22).

---

## 4. Hailo TAPPAS 미설치

## 증상
`hailonet: no such element` — GStreamer가 Hailo NPU 플러그인 못 찾음.
`HAILO_OUT_OF_PHYSICAL_DEVICES(74)` — /dev/hailo0 없음.

## 원인
신규 장비에 `hailo-all` 메타패키지 미설치.

## 해결
```bash
sudo apt-get install -y hailo-all
sudo modprobe hailo_pci

# /dev/hailo0 여전히 없으면:
sudo rmmod hailo_pci && sudo modprobe hailo_pci
ls /dev/hailo0
```

## 재발 방지
install.sh section 3에 반영 완료. 설치 후 자동 modprobe.

---

## 5. libhiredis / libredis_export.so 미설치

## 증상
`libhiredis.so.0.14: cannot open shared object file`
`libredis_export.so: no such file`

## 원인
- `libhiredis0.14`: Redis C 클라이언트, hailo-all에 미포함
- `libredis_export.so`: 커스텀 GStreamer Redis 플러그인, 표준 TAPPAS 미포함

## 해결
```bash
# libhiredis
sudo apt-get install -y libhiredis0.14

# libredis_export.so
REDIS_SO="/usr/lib/aarch64-linux-gnu/hailo/tappas/post_processes/libredis_export.so"
curl -fsSL "https://api.paidevteam.com/firmware/edge-sw/libredis_export.so" \
  -o /tmp/libredis_export.so
sudo mkdir -p "$(dirname "$REDIS_SO")"
sudo install -m 644 /tmp/libredis_export.so "$REDIS_SO"
```

## 재발 방지
install.sh section 3(libhiredis) / section 5(libredis_export.so) 반영 완료 (2026-04-22).

---

## 6. 카메라 점유 충돌

## 증상
`Pipeline handler in use by another process`

## 원인
mediamtx 재시작 또는 이전 파이프라인 비정상 종료 후 rpicam-vid zombie 잔존.

## 해결
```bash
sudo pkill -9 -f rpicam
sudo pkill -9 -f gst-launch
sudo fuser /dev/video0        # 잔존 프로세스 확인
sudo systemctl restart mediamtx
```

---

## 7. AI ON 후 mediamtx source 전환 순서

mediamtx가 `rpiCamera`로 카메라를 잡고 있는 상태에서 파이프라인 시작하면 충돌.

## 올바른 순서
```bash
# 1. mediamtx가 카메라 해제하도록 source 전환
curl -s -X PATCH http://localhost:9997/v3/config/paths/patch/my_camera \
  -H "Content-Type: application/json" \
  -d '{"source":"publisher"}'
sleep 2

# 2. 파이프라인 시작 (rpicam-vid → GStreamer → RTSP push)
bash /home/ptz/edge/wide_detection_fixed.sh &
```

---

## 8. 장비 인증 비밀번호 형식

## 증상
Device UI 로그인 `"Invalid password"`.

## 원인
비밀번호는 CPU serial 전체가 아닌 DEVICE_ID의 마지막 세그먼트:
```python
# settings_handler.py
serial = DEVICE_ID.split("-")[-1]
# "MK3-6d1c9f59" → 비밀번호 = "6d1c9f59"
```

---

## 9. Data Output MQTT Live Test 연결 실패

## 증상
Live Test Connect 버튼 클릭 → `Connection error`.

## 원인 A — WebSocket 포트 오류
`ws://broker:1883/mqtt` 시도 → 1883은 TCP MQTT 포트, WebSocket 불가.

| 브로커 | 올바른 WebSocket URL |
|--------|---------------------|
| paidevteam.com (내부) | `wss://api.paidevteam.com/mqtt` (EC2 nginx `/mqtt` 경로) |
| HiveMQ Cloud | `wss://{broker}:8884/mqtt` |
| 일반 외부 브로커 | `ws://{broker}:9883/mqtt` |

mosquitto WebSocket 포트: **9883** (TCP MQTT는 1883).

## 원인 B — Username/Password 입력 필드 없음
HiveMQ Cloud는 인증 필수. 2026-04-22 UI에 Username/Password 필드 추가 완료.

## 수정된 파일
`platform/frontend/src/app/(dashboard)/devices/[id]/page.tsx` — 연결 URL 로직 + Username/Password 입력 필드

---

## 10. Data Output — 연결 됐지만 메시지 없음

## 증상
Live Test "connected" 상태인데 메시지가 수신 안 됨.

## 원인
Data Output 이벤트 전체 흐름:
```
wide_detection_fixed.sh (gst-launch + libredis_export.so)
  → Redis pub/sub: detections:wide   [include_classes:["person"], 사람 있을 때만]
    → people_counter.py              ← people_count.enabled=false면 실행 안 됨
      → Redis pub/sub: events:MK3-xxxx
        → event_dispatcher.py        ← events:MK3-xxxx 구독
          → HiveMQ / 외부 MQTT
```

`people_count.enabled: false`이면 중간 브리지(people_counter.py)가 없어 이벤트가 전달 안 됨.

## 토픽 구조
엣지 발행: `{topic_prefix}/{device_id}/{model_name}/{event_type}`
예: `mk3pro/events/MK3-6d1c9f59/yolov11s/people_count`

Live Test 구독: `{topic_prefix}/{device_id}/#` (와일드카드)

## 해결
플랫폼 UI에서 피플카운팅 활성화 → people_counter.py 기동 → 이벤트 흐름 완성.

> ⚠️ 향후 개선: 피플카운팅 OFF여도 raw detection 이벤트를 `events:MK3-xxxx`에 발행하도록 파이프라인 확장 필요 (미구현).
