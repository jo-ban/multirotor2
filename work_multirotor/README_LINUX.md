# multirotor2 Linux 실행

입력 5채널: L/D, sin(theta), cos(theta), z/R_D, h/RD. 출력 3채널: ur, utheta, uz.
CSV와 전체 실행 예시는 저장소 루트의 [README](../README.md)를 참고하세요.

```bash
cd work_multirotor
bash rotor.sh setup
bash rotor.sh check
bash rotor.sh extract --csv /data/cases.csv --data-root /data
bash rotor.sh train --csv /data/cases.csv --epochs 100 --lambda-gradient 0.1
bash rotor.sh evaluate --csv /data/cases.csv
bash rotor.sh predict --rd 9.3101751 --l-over-d 1.5 --h-over-rd 0.75 --ground-z 0 --z-max 32 --no-show
```

숫자와 경로는 예시입니다. 실제 h/RD, ground_z, 메시 z_max 및 손실 가중치를 사용하세요.
CSV는 rotor_z 대신 h/RD를 받습니다. 내부 rotor_z=ground_z+(h/RD)*RD입니다.
--h-over-rd, --ground-z, --z-max는 예측 필수 인자입니다.
Linux GUI에서는 --no-show를 빼면 TkAgg 창을 엽니다. python3-tk와 디스플레이 연결이 필요합니다.
원통은 마지막 z격자점까지 표시됩니다. FATO/SA는 각각 별도 데이터셋과 모델입니다.
기존 4채널 모델은 새 5채널 입력과 호환되지 않습니다.
