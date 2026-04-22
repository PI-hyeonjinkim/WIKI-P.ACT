# ONVIF Ghost Service — 카메라가 혼자 움직이는 문제

## 증상

카메라가 명령을 내리지 않았는데 혼자 왔다갔다 움직임. `onvif_srvd` 서비스를 stop하면 멈추고, start하면 다시 움직임.

## 원인

구버전 `onvif.service` 파일이 `Restart=always`로 시스템에 잔존하여, 신버전 `onvif_srvd.service`와 동시에 두 인스턴스가 실행됨.

- **구버전**: `--url rtsp://{ip}:8554/detection` (AI detection 스트림 URL)
- **신버전**: `--url rtsp://{ip}:8554/my_camera`

두 인스턴스가 `/home/ptz/onvif_srvd.pid` 파일을 두고 충돌하며 `Can't create pid file: Resource temporarily unavailable` 에러 반복 발생.

## 진단

```bash
ps aux | grep onvif_srvd | grep -v grep
# detection URL로 실행된 인스턴스가 보이면 이 문제
```

## 해결

```bash
sudo systemctl stop onvif
sudo systemctl disable onvif
sudo rm -f /etc/systemd/system/onvif.service
sudo pkill -9 -f "onvif_srvd.*detection"
sudo rm -f /home/ptz/onvif_srvd.pid
sudo systemctl restart onvif_srvd
```

## 재발 방지

`install.sh`에 구버전 `onvif.service` 자동 제거 코드 추가 완료 (2026-04-07). 신규 배포 시 자동 처리됨.

**기존 장비**는 위 진단 명령으로 수동 확인 필요.

> 출처: memory/onvif-ghost-service.md (2026-04-08)
