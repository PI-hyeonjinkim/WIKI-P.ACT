# Device UI — 카메라 설정 웹 UI

> 출처: docs/edge/device-ui.md (2026-04-22)
> 마지막 갱신: 2026-04-22

---

## 개요

MK3pro 장비 자체에서 서빙하는 카메라 설정 UI.  
Next.js 정적 빌드 → 장비 배포 → `edge-settings`(포트 7778)가 서빙.

**접속**: `http://192.168.33.66:7778`

---

## 페이지 구성

| 경로 | 내용 |
|------|------|
| `/` | 메인 (카메라 상태, RTSP 프리뷰) |
| `/login` | 로그인 |
| `/change-password` | 비밀번호 변경 |
| `/thermal` | 열화상 설정 |

---

## 주요 컴포넌트

| 파일 | 역할 |
|------|------|
| `CameraSettings.tsx` | 해상도/FPS/밝기 파라미터 |
| `NetworkSettings.tsx` | IP/게이트웨이/DNS 설정 |
| `PTZControl.tsx` | PTZ 조이스틱 + 프리셋 |
| `SecuritySettings.tsx` | 비밀번호, 인증 |
| `SystemControl.tsx` | 재부팅, 서비스 재시작, 업데이트 |
| `WhepPlayer.tsx` | WebRTC 영상 재생 |

---

## API 연동

`settings_handler.py`(포트 7778) API 호출:

```
GET  /api/settings          카메라 설정 조회
POST /api/settings          설정 변경 (width, height, framerate ...)
POST /api/ai                AI ON/OFF { "enabled": true }
POST http://localhost:7777/move  PTZ 이동
```

---

## 빌드 및 배포

```bash
# 빌드 (로컬)
cd device-ui
npm run build       # → device-ui/out/ 생성

# 배포 (SCP)
scp -r device-ui/out/ ptz@192.168.33.66:/home/ptz/device-ui/
```

> ⚠️ `out/` 은 .gitignore 포함. 소스만 배포하면 동작 안 함 — 반드시 빌드 후 배포.
