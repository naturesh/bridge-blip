# LoRA-Bridge Instructblip 

- 특정 task에 맞게 instructblip 을 최적화 해보자


## LoRA-Bridge

LoRA Bridge는 LoRA 기술을 파인튜닝에 이용하지 않고 정보의 흐름을 유지하기 위한 하나의 다리로서 사용하는 아키텍쳐 입니다.

instructblip-flan-t5-xl 모델의 경우 language_model 로 flan-t5-xl 을 사용하고 있습니다. 하지만 이번 SCPC에서 주어진 과제는 문장 생성이 아닌 단순히 A,B,C,D 의 선지를 맞추면 되는 간단한 멀티모달을 만드는것이 였습니다. 따라서 저는 디코더의 문장 생성 능력은 불필요 하다고 생각 되어 최대한 디코더를 줄이기 위해 노력하였습니다. 

하지만 instructblip 의 좋은 성능은 좋은 language_model 의 영향도 있을것, 디코더에도 문장을 이해하는 능력이 있을것, 이라는 가정을 바탕으로 
디코더의 크기는 줄이면서 디코더의 문장 이해 능력은 유지하려고 노력하였습니다.

디코더의 문장 이해 능력을 유지하기 위해서는 디코더를 부분적으로 사용할 필요가 있다고 판단하여 총 24 레이어의 디코더 구조를 0, 3, 6, 9... 21 로 총 8개로 슬라이싱 하는 과정을 진행하였습니다. 

잘라진 디코더는 정보의 흐름이 끊긴 상태이기 때문에 이를 연결하기 위해서 남은 8개 레이어에 r=96, lora_alpha=192 의 비교적 큰 LoRA를 사용하여 정보의 흐름을 간이적으로 연결하고자 하였습니다. 



## Performance
- 아래 결과는 레이어 선택을 제외하고 모두 같은 조건에서 진행되었습니다. ( LoRA 적용 여부, 분류 헤드 등 .. ) 

아래 Smooth Loss Graph와 같이 0, 3, 6, 9... 구조를 이용하는 경우 초기 손실 감소 폭은 물론 평균적인 손실에 있어서도 더 안정적인 모습을 볼수 있었습니다. 

|레이어 선택|epoch|색상|
|:---|:---|:---|
|0,2,4,6|1|녹색|
|0,2,4,6,8,10,12,14|1|파란색|
|0,1,2,3,4,5,6,7|1|빨간색|
|0,3,6,9,12,15,18,21|1|하늘색|


<img width="700" height="450" alt="image" src="https://github.com/user-attachments/assets/9ce44e9c-d702-4d0b-8417-454cf54cdb77" />

# How to use 


### 사용 방법 

1. github zip 파일로 다운받은 후 bridgeblip 으로 폴더명을 변경한다.
2. bridgeblip 을 colab 에 폴더 업로드 한다.
    해당 경로가 되도록. `/content/drive/MyDrive/bridgeblip`

3. colab 런타임을 A100 으로 변경한다.
4. train.ipynb, torchscript.ipynb, torchscript_inference.ipynb 순으로 실행한다. 

<br>

- 본 프로젝트는 2025 7/26 기준. Python 3.11.13, A100 에서 실행되었습니다.

<br>

### 파일 실행 시 
- train.ipynb 실행시 checkpoint/weights-0.pt 로 가중치 파일이 생성 됨
- torchscript.ipynb 실행시 checkpoint/weights-0.pt 를 불러와 torchscript 로 변환 후 checkpoint/torchscript.pt 로 모델 저장
- torchscript_inference.ipynb checkpoint/torchscript.pt 모델을 불러와 추론 실행 - `추론파일`
- 추론파일 실행 시 submission.csv 가 생성됨
<br><br>
- torchscript.pt, weights-0.pt 가 있는 경우 checkpoint 폴더에 넣고 torchscript_inference.ipynb 실행 
- 단 제공하는 torchscript.pt, weights-0.pt 는 cuda 에서 학습되고 저장되었으므로 오류 없이 실행하기 위해서는 cuda 환경이 필요.
<br><br>

### 파일 정보
- train.ipynb : 모델을 학습시키는 노트북 입니다.
- torchscript.ipynb : 모델을 로드해서 torchscript 로 바꾸는 노트북 입니다.
- torchscript_inference.ipynb : 추론 코드 노트북 입니다.
- instructblip.config.json : huggingface 의 Salesforce/instructblip-flan-t5-xl config.json을 수정한 파일입니다. 수정 내역은 torchscript.ipynb 참고
<br><br>

- checkpoint : 체크포인트를 저장하는 폴더 입니다.
- competition : SCPC 대회 제공 파일 입니다.

### 주의사항 

각 노트북 실행 후 checkpoint에 저장이 되었는지 확인 필수.
런타임 빠르게 끊게 되면 제대로 저장이 안 될수도 있음

competition 데이터는 직접 다운로드받아서 넣어야 함.
