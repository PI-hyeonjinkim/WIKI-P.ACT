# 목포 물류장 운영 가이드

> 출처: memory/mokpo-deploy.md (2026-03-30)
> 마지막 갱신: 2026-03-30

---

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
- RTSP 출력: `nvv4l2h264enc` (HW 인코더) → mediamtx
- 라인 좌표: `(722,502)→(1294,588)` [1920×1080 기준]

---

## 운영 이슈

### GUI 비활성화 (전 장비 적용)
RAM 2.4GB 절감, Swap 56%→4%, PTZ 지연 해소.
```bash
sudo systemctl set-default multi-user.target && sudo reboot
```

### js-108 Fire 모델 비활성화
smoke 오감지 → `cameras.yaml`에서 `fire_model.enabled: false`

### RTSP HW 인코더 (nvv4l2h264enc)
CPU 인코더(x264enc) 사용 시 프레임 충돌. `edge_stream2.py`에만 적용.

---

## Grafana 모니터링

- URL: `http://10.10.30.105:3030` (ID: admin / PW: admin1234)
- `health-collector` Docker가 2.5분마다 수집 → InfluxDB RMS 버킷

---

## 원격 접속

목포 물류장은 EC2 프록시 → ZeroTier SSH 점프 필요:
```bash
ssh -i DNSKEY.pem ubuntu@api.paidevteam.com
# EC2에서 ZeroTier를 통해 내부망 접근
```
