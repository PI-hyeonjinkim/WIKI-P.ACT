# MQTT 재연결 버그 (부팅 후 PTZ/명령 미동작)

> 출처: raw/troubleshooting/2026-04-17_known-issues.md
> 마지막 갱신: 2026-04-17

---

## 증상
- 장비 재부팅 후 PTZ 명령, 설정 변경 등 모든 플랫폼 명령 미반응
- edge-settings 프로세스는 정상 실행 중
- `sudo ss -tnp state established | grep 1883` 결과 없음 (MQTT 연결 없음)

## 원인
`settings_handler.py`의 `start_mqtt()`에서 최초 `connect()` 실패 시
예외 catch 후 `_mqtt_client = None`으로 설정 → 이후 재시도 없음.
부팅 초기 네트워크 준비 전에 `connect()` 실패하면 영구 미연결 상태.

## 해결
`start_mqtt()`를 백그라운드 재시도 루프로 교체:
```python
def start_mqtt():
    global _mqtt_client
    _mqtt_client = mqtt.Client(mqtt.CallbackAPIVersion.VERSION2)
    _mqtt_client.reconnect_delay_set(min_delay=5, max_delay=60)
    _mqtt_client.loop_start()

    def _initial_connect():
        delay = 5
        while True:
            try:
                _mqtt_client.connect(BROKER_HOST, BROKER_PORT)
                return
            except Exception as e:
                time.sleep(delay)
                delay = min(delay * 2, 60)

    threading.Thread(target=_initial_connect, daemon=True).start()
```

## 즉각 조치
```bash
sudo systemctl restart edge-settings
# 재시작 후 MQTT 연결 확인
sudo ss -tnp state established | grep 1883
```

## 재발 방지
- MQTT client는 항상 `loop_start()` + 백그라운드 재연결 스레드 패턴
- `reconnect_delay_set`으로 paho-mqtt 자체 재연결도 활성화
