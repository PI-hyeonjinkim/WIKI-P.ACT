# 목포 물류장 운영 가이드

> 마지막 업데이트: 2026-03-30

## 배포 현황

### 안전관제 (detection.py) — 14대

| 장비 | IP | camid | 카메라명 | Zone |
|------|-----|-------|---------|------|
| js-107 | 10.10.30.107 | 11 | MK01 | A1-2 |
| js-108 | 10.10.30.108 | 12 | MK02 | A1-5 |
| js-109 | 10.10.30.109 | 13 | MK03 | A1-8 |
| js-114 | 10.10.30.114 | 22 | MK07 | A1-27 |
| js-117 | 10.10.30.117 | 31 | MK11 | A1-53 |
| js-118 | 10.10.30.118 | 32 | MK10 | A1-50 |
| js-119 | 10.10.30.119 | 33 | MK09 | A1-47 |
| js-122 | 10.10.30.122 | 42 | MK13 | A1-19 |
| js-127 | 10.10.30.127 | 51 | MK17 | B1-1 |
| js-128 | 10.10.30.128 | 52 | MK18 | B1-3 |
| js-130 | 10.10.30.130 | 61 | MK19 | B1-4 |
| js-134 | 10.10.30.134 | 72 | MK22 | A2-3 |
| js-135 | 10.10.30.135 | 81 | MK23 | A2-6 |
| js-136 | 10.10.30.136 | 82 | MK24 | A2-10 |

### TP 입출차 분석 (edge_stream2.py) — 1대

- **js-124** (10.10.30.124)
- RTSP 소스: `rtsp://admin:admin1234!@10.10.30.123:554/live&channel=1&stream=0.sdp?real_stream`
- 4K 수신 → GStreamer 1920×1080 리사이즈 → 분석
- RTSP 출력: `nvv4l2h264enc` (HW 인코더) → mediamtx 8554/8889
- 라인 좌표: `(722,502)→(1294,588)` [1920×1080 기준]
- 출력: 텔레그램 알림 + REST API (`/tp_in`, `/tp_out`) + InfluxDB (`vehicle_events`, `BLOCK`)

### 기타 장비

| 장비 | 역할 |
|------|------|
| js-125, js-126 | 입출차 영상 송출 (streaming.py) |
| .105 PC | 차량번호인식 OCR (Docker aimedia/gpuacc_5070:0.1, 포트 9090) |
| 모든 Jetson | PTZ 제어 API (main.py, 포트 7777) |

## 운영 이슈 및 해결책

### GUI 비활성화 (전 장비 적용 완료)

RAM 2.4GB 절감, Swap 56%→4%, PTZ 지연 해소.

```bash
# 비활성화
sudo systemctl set-default multi-user.target && sudo reboot

# 복구 (필요 시)
sudo systemctl isolate graphical.target
```

### js-108 Fire 모델 비활성화

smoke 오감지 문제. `cameras.yaml`에서 `fire_model.enabled: false` 설정됨.

### RTSP 입출력 동시 사용 시 CPU 인코더 문제

CPU 인코더(x264enc) 사용 시 프레임 충돌 → `nvv4l2h264enc` (HW 인코더)로 해결.
`edge_stream2.py`에만 적용 (detection.py는 CSI 카메라라 무관).

## Grafana 모니터링

- URL: `http://10.10.30.105:3030`
- ID/PW: `admin` / `admin1234`
- 대시보드: 안전감지 관제 + InfluxDB Data Explorer
- `health-collector` Docker 컨테이너가 2.5분마다 수집 → InfluxDB RMS 버킷

## 원격 접속 방법

목포 물류장은 EC2 프록시를 통한 SSH 점프 필요:

```bash
ssh -i /c/Users/User/DNSKEY.pem ubuntu@api.paidevteam.com
# EC2에서 ZeroTier를 통해 내부망 접근
```

또는 Claude Code `ssh` 스킬 사용.

## 핵심 파일 경로 (로컬)

- `C:\Users\User\Desktop\진출입점검\` — 프로젝트 루트
- `edge_stream2.py` — TP 입출차 최신본 (HW 인코더 + 리사이즈)
- `remote.py` — SSH 원격 관리 도구
- `cameras.yaml` — 장비별 설정

> 출처: memory/mokpo-deploy.md (2026-03-30)
