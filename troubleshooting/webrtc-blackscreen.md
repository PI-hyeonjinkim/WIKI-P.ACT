# WebRTC 검은 화면 (Black Screen)

> 출처: raw/troubleshooting/2026-04-17_known-issues.md
> 마지막 갱신: 2026-04-17

---

## 케이스 A: mediamtx source 충돌 (wide 뷰만 black)

### 증상
- wide 뷰만 검은 화면, zoom 뷰는 정상
- ICE 상태: connected (네트워크 문제 아님)
- mediamtx paths API: `my_camera` webRTCSession 0개

### 원인
`mediamtx.yml`의 `my_camera` 경로가 `source: rpiCamera`로 설정됨.
mediamtx가 Pi Camera 직접 캡처 시도 + gst 파이프라인도 동일 경로에 RTSP push → 소스 충돌.

### 해결
```yaml
# /opt/mediamtx/mediamtx.yml
my_camera:
  source: publisher   # rpiCamera 아님
```
```bash
sudo systemctl restart mediamtx
```

### 재발 방지
gst 파이프라인이 RTSP push하는 경로는 항상 `source: publisher`.

---

## 케이스 B: coturn external-ip 누락 (외부망에서 black)

### 증상
- 외부 브라우저에서 ZeroTier 디바이스 볼 때 black screen
- ICE 실패 또는 connected인데도 영상 없음
- coturn 로그: `allocation timeout`
- 로컬 망 브라우저에서 같은 망 디바이스는 정상

### 원인
EC2 NAT 환경에서 `external-ip` 미설정 → coturn이 사설IP(172.31.36.71)를 릴레이 주소로 광고 → 외부 브라우저 연결 불가.

### 해결
```bash
# /etc/turnserver.conf 에 추가
external-ip=54.253.91.60/172.31.36.71

sudo systemctl restart coturn
```

### 재발 방지
EC2 coturn 설정 시 `external-ip=공인IP/사설IP` 필수.
AWS 보안그룹 UDP 49152-65535 인바운드 오픈 확인.

---

## 케이스 C: 재부팅 직후 black (WHEP 타임아웃)

### 증상
- 장비 재부팅 직후 모든 브라우저 black screen
- 5~10분 후 자연 복구
- 백엔드 로그: `WHEP request failed: timed out`

### 원인
Hailo AI 파이프라인 초기화(수분 소요) 동안 mediamtx WHEP 응답 불가.
기존 timeout 8초 → 초기화 중 timeout 발생.

### 해결
```python
# backend/app/services/mqtt_consumer.py
with urllib.request.urlopen(req, timeout=20) as resp:  # 8→20초
```
※ 2026-04-17 기준 코드 수정 완료, 플랫폼 배포 대기 중.

---

## 빠른 진단 순서

1. **zoom 되고 wide만 black?** → 케이스 A (mediamtx source)
2. **외부망에서만 black, 로컬망은 OK?** → 케이스 B (coturn external-ip)
3. **재부팅 직후만 black, 나중엔 OK?** → 케이스 C (WHEP timeout)
4. **ICE 상태 확인**: connected면 네트워크 OK → 스트림 소스 문제
5. **ICE failed?** → TURN 서버 또는 네트워크 문제
