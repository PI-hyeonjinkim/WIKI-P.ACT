# WebRTC 스트리밍 파이프라인

> 출처: raw/decisions/2026-04-17_platform-architecture.md
> 마지막 갱신: 2026-04-17

---

## 전체 흐름

```
[카메라 센서]
    │ raw frames (pipe)
    ▼
[gst-launch-1.0 + Hailo NPU]
    │ AI 처리 (감지, 오버레이)
    │ x264enc → RTSP push
    ▼
[mediamtx :8554/my_camera]  ← source: publisher 필수
    │ WHEP (HTTP POST /my_camera/whep)
    ▼
[FastAPI backend MQTT consumer]
    │ SDP offer (MQTT webrtc/{id}/offer)
    │ SDP answer (MQTT webrtc/{id}/answer)
    ▼
[브라우저 RTCPeerConnection]
    │ ICE (직접 or TURN 릴레이)
    ▼
[영상 재생]
```

---

## WHEP 시그널링 상세

1. 브라우저 → MQTT: `webrtc/{device_id}/offer` (SDP offer + session_id)
2. 백엔드 MQTT consumer: `POST http://{device_ip}:8889/{stream_path}/whep` (timeout: 20초)
3. mediamtx → 백엔드: SDP answer
4. 백엔드 → MQTT: `webrtc/{device_id}/answer` (SDP answer + session_id)
5. 브라우저: setRemoteDescription → ICE 시작

---

## mediamtx 경로 설정 (P.ACT01)

| 경로 | source | 내용 | 상태 |
|------|--------|------|------|
| my_camera | **publisher** | gst+Hailo AI 처리 영상 | 정상 |
| detection_zoom | rtsp://admin:EdgePTZ%232025@192.168.1.68/stream1 | 줌 카메라 직접 소스 | 정상 |
| detection | publisher | 감지 전용 | — |

> ⚠️ `my_camera`를 `source: rpiCamera`로 설정하면 gst RTSP push와 충돌 → black screen
> 상세: [[mediamtx-source-conflict]]

---

## 프론트엔드 스트림 경로 선택

```typescript
// devices/[id]/page.tsx
streamPath={activeCamera === "zoom" ? "detection_zoom" : "my_camera"}
```
- wide(기본): `my_camera` — AI 처리 영상
- zoom: `detection_zoom` — 줌 카메라 직접 영상

---

## ICE 연결

mediamtx ICE 설정 (mediamtx.yml):
```yaml
webrtcAdditionalHosts: [10.235.62.222]   # ZeroTier IP 명시
webrtcICEServers2:
  - url: stun:api.paidevteam.com:3478
  - url: turn:api.paidevteam.com:3478
    username: AUTH_SECRET
    password: pact_turn_2026
    clientOnly: false
```
`username: AUTH_SECRET` 이면 mediamtx가 자동으로 HMAC 기반 time-limited 크리덴셜 생성.

---

## 온라인 상태 판단

- Redis key: `device:status:{device_id}` TTL 120초
- MQTT 수신마다 갱신
- 프론트 polling: dashboard 30초, devices 페이지 수동 새로고침
