[multirotor2: 가변 로터 높이 h/RD / 5채널]
CSV: folder,RD,L_over_D,h/RD,ground_z,type,center_x,center_y
h/RD는 지면 대비 로터 높이/RD. CSV의 rotor_z를 읽지 않음.
내부 rotor_z=ground_z+(h/RD)*RD. RD는 데이터셋 내 고정.
입력: L/D, sin(theta), cos(theta), z/R_D, h/RD.
출력: ur, utheta, uz 공동 모델. FATO 2RD / SA 2.5RD 별도 학습.
손실: 속도 L1 + lambda_gradient * 표면 기울기 L1.
지면부터 마지막 z격자까지 추출/학습/표시/검증. RD 높이 제한 없음.
자세한 실행 예시는 저장소 README.md 및 README_LINUX.md 참고.
