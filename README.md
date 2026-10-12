# multirotor2

CFD에서 로터 높이를 바꾸는 케이스를 위한 5채널 원통 유동장 공동 학습입니다.
원본 `jo-ban/multirotor`의 `claude/great-faraday-ijzn3f` 브랜치
`fe2148f5c7a88c19b6c9b4061cadfc8e9aa51366`에서 복사했습니다.

입력 순서: `L/D`, `sin(theta)`, `cos(theta)`, `z/R_D`, `h/RD`.
출력 순서: `ur`, `utheta`, `uz` (영역별 하나의 공동 U-Net).
`z/R_D`는 실제 CFD 격자 위치이며 `h/RD`는 지면 대비 로터 높이의 무차원 값입니다.
5번째 채널은 한 케이스 내에서 상수이며, CSV 값을 다시 RD로 나누지 않습니다.

## CSV

```csv
folder,RD,L_over_D,h/RD,ground_z,type,center_x,center_y
cases/case01,9.3101751,1.5,0.5,0,T,0,0
cases/case02,9.3101751,1.5,1.0,0,T,0,0
cases/case03,9.3101751,1.6,0.75,0,V,0,0
```

위 숫자는 예시입니다. 실제 CFD 폴더, 높이 및 지면 좌표를 기입하세요.
`h/RD`는 필수이며 양의 유한 값이어야 합니다. 별칭 `h_over_rd`, `height_over_rd`도 지원합니다.
`rotor_z` 열은 읽지 않습니다. 내부 물리 좌표는 `rotor_z = ground_z + (h/RD)*RD`로 계산합니다.
계산한 로터면은 실제 메시의 지면과 상단 사이에 있어야 합니다.
`ground_z`는 실제 OpenFOAM 좌표이며 임의로 0으로 바꾸지 않습니다.
RD는 생략 시 9.3101751 m이고 한 데이터셋의 RD는 동일해야 합니다.
`L_over_D`는 `L/D` 별칭도 지원합니다.
`type=V`는 검증, `type=-`는 학습 제외이며 나머지는 학습에 사용됩니다.
높이가 다른 케이스는 각각 다른 CFD 폴더로 등록하세요. CSV 변경만으로 CFD를 재계산하지 않습니다.

## Colab

[새 multirotor2 Colab 노트북](https://colab.research.google.com/drive/1V1iMVtgUQ8oBQ9gogshCkKIK4lyuy8Ee)

`aerodownwash12@gmail.com` 계정으로 Drive를 마운트하세요.
노트북 파일은 `multirotor2_colab.ipynb`입니다. 기존 노트북은 변경하지 않았습니다.
새 노트는 `jo-ban/multirotor2`의 `main`을 사용하고 결과를
`MyDrive/multirotor2_results/B(P4)`에 저장합니다.
설정 셀에서 `DRIVE_DATA`, `DRIVE_CSV`를 새 CFD 데이터와 h/RD CSV에 맞추세요.
`PREDICT_H_OVER_RD`를 지정하면 해당 높이로 예측하며 None이면 기준 케이스의 높이를 사용합니다.
노트북의 clone 셀은 공개 GitHub 저장소 `jo-ban/multirotor2`의 `main`을 사용합니다.

## 실행

```bash
pip install -r work_multirotor/requirements.txt
python work_multirotor/extract_nd.py --csv /data/cases.csv --data-root /data --output-root /data/dataset_zrd --sa-output-root /data/dataset_zrd_sa
python work_multirotor/train_nd.py --csv /data/cases.csv --data-root /data/dataset_zrd --sa-data-root /data/dataset_zrd_sa --model-dir /data/models_zrd --sa-model-dir /data/models_zrd_sa --epochs 100 --lambda-gradient 0.1
python work_multirotor/predict_nd.py --model-dir /data/models_zrd --sa-model-dir /data/models_zrd_sa --rd 9.3101751 --l-over-d 1.5 --h-over-rd 0.75 --ground-z 0 --z-max 32 --no-show --output /data/results/prediction.npz --plot-output /data/results/prediction.png
python work_multirotor/evaluate_error.py --csv /data/cases.csv --data-root /data/dataset_zrd --sa-data-root /data/dataset_zrd_sa --model-dir /data/models_zrd --sa-model-dir /data/models_zrd_sa --output-dir /data/results/evaluation --no-show
```

위 예시의 lambda_gradient=0.1, ground_z=0, z_max=32는 실행 예시이며 실제 조건을 사용하세요.
손실은 기존 속도 L1 + lambda_gradient × 공변 표면 기울기 L1입니다.
총 추력 30000 N과 기존 Vi 기반 속도 무차원화는 유지합니다.
FATO(2RD)와 SA(2.5RD)는 서로 다른 데이터셋과 모델을 사용합니다.
기존 4채널 가중치는 호환되지 않으며 높이별 실제 CFD 결과로 다시 학습해야 합니다.
학습 입력은 추출 NPZ에서 읽습니다. CSV 높이를 바꿨으면 캐시 검사를 거쳐 새 조건을 추출하세요.

## 전체 z범위 표시

추출, 학습, 예측은 지면부터 실제 메시 마지막 z격자까지 사용합니다.
예측 3D 원통, 예측 PNG, 추출 확인 PNG, CFD 비교 HTML/PNG 모두 RD 높이에서 자르지 않습니다.
검증 MAE/NMAE, 최대 오차와 최대 오차율도 전체 z격자의 원본 샘플에서 계산합니다.
비교 3D는 반응성을 위해 표시 격자만 축소하지만 첫/마지막 z점은 포함합니다.
저장 배열과 통계는 전체 원본 해상도를 유지합니다.
HTML은 Plotly를 포함하므로 내려받아 회전·확대·성분 및 FATO/SA 선택을 할 수 있습니다.

기존 저장 비교 파일을 전체 z범위로 다시 표시하려면:

```bash
python work_multirotor/evaluate_error.py --replot --output-dir /data/results/evaluation
python work_multirotor/visualize_prediction_3d.py --input /data/results/prediction.npz --output /data/results/prediction_3d.html
```
