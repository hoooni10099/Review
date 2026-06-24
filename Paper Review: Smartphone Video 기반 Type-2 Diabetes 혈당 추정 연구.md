# Paper Review: Smartphone Video 기반 Type-2 Diabetes 혈당 추정 연구

## 논문 정보

* **Title**: *Estimation of Blood Glucose Level of Type-2 Diabetes Patients Using Smartphone Video through PCA-DA*
* **Authors**: Tauseef Tasin Chowdhury, Tahmin Mishma, Saeem Osman, Tanzilur Rahman
* **Conference**: NSysS 2019
* **DOI**: https://doi.org/10.1145/3362966.3362983
* **Keywords**: PPG, Smartphone Video, Blood Glucose Estimation, Signal Processing, PCR

---

## 1. 논문 한 줄 요약

이 논문은 **스마트폰 카메라로 손가락 끝 영상을 촬영하고, 영상의 밝기 변화에서 PPG 신호를 추출한 뒤, PPG의 형태학적 특징을 이용해 비침습적으로 혈당을 추정할 수 있는지 확인한 proof-of-concept 연구**이다.

---

## 2. 연구 배경

혈당 측정은 일반적으로 손끝에서 피를 뽑아 혈당계로 측정하는 침습적 방식이 많이 사용된다. 하지만 이 방식은 통증이 있고, 반복 측정이 불편하며, 시험지와 채혈 도구가 지속적으로 필요하다는 단점이 있다.

이 논문은 이러한 문제를 줄이기 위해 **스마트폰 카메라만으로 혈당을 추정할 수 있는 가능성**을 탐색한다. 스마트폰의 플래시와 카메라를 이용하면 손가락 끝의 혈류 변화에 따른 밝기 변화를 얻을 수 있고, 이를 PPG 신호로 변환할 수 있다.

핵심 아이디어는 다음과 같다.

```text
손가락 영상 촬영
→ RGB 프레임 추출
→ Red channel 평균값으로 PPG 파형 생성
→ 노이즈 및 baseline drift 제거
→ PPG peak 및 미분 특징 추출
→ PCR 회귀 모델로 혈당 예측
```

---

## 3. 연구 방법

### 3.1 데이터 수집

논문에서는 **iPhone 7 Plus**를 사용하여 손가락 끝 영상을 촬영했다.

* 측정 위치: 검지 손가락 끝
* 촬영 장치: iPhone 7 Plus
* 촬영 조건: 30 fps, 720p
* 측정 시간: 약 60초
* 피험자 수: 18명
* 연령 범위: 15~61세
* 각 피험자당 5회 측정
* Reference 혈당값: 상용 혈당계 사용

스마트폰의 카메라와 플래시 위에 손가락을 올려두면, 혈액량 변화에 따라 영상의 red channel intensity가 주기적으로 변한다. 논문은 이 밝기 변화를 평균내어 PPG 신호로 변환하였다.

---

### 3.2 PPG 신호 추출

이 연구에서는 별도의 ROI를 세밀하게 설정하지 않고, **전체 프레임의 red intensity 평균값**을 사용했다.

즉, 각 프레임마다 red channel의 평균 밝기를 계산하고, 시간에 따른 평균 밝기 배열을 PPG waveform으로 사용했다.

```text
Video frame
→ RGB frame
→ Red channel intensity
→ Frame-wise average
→ PPG waveform
```

이 방식은 구현이 단순하다는 장점이 있지만, 손가락 위치 변화, 압력 변화, 플래시 반사, motion artifact에 취약할 수 있다.

---

### 3.3 전처리

논문에서는 PPG 신호의 품질을 높이기 위해 두 가지 전처리 방법을 사용했다.

#### 1) Gaussian Filter

Gaussian smoothing을 사용하여 고주파 잡음을 줄였다.
스마트폰 영상 기반 PPG는 조명 변화, 카메라 센서 노이즈, 미세 움직임의 영향을 받기 때문에 smoothing 과정이 필요하다.

#### 2) ALS, Asymmetric Least Squares

ALS는 baseline correction에 사용되었다.
손가락을 카메라에 올려두는 과정에서 발생하는 motion interference나 baseline wandering을 줄이기 위한 목적이다.

정리하면, 논문의 전처리 흐름은 다음과 같다.

```text
Raw PPG
→ ALS baseline correction
→ Gaussian filtering
→ Clean PPG
```

---

### 3.4 특징 추출

전처리된 PPG 신호에서 다음과 같은 특징을 추출했다.

| Feature                    | 의미                                      |
| -------------------------- | --------------------------------------- |
| Systolic Peak              | PPG 파형의 주요 상승 피크                        |
| Diastolic Peak             | 수축기 이후 나타나는 보조 피크                       |
| DelT                       | systolic peak와 diastolic peak 사이의 시간 차이 |
| First Derivative Peaks     | PPG 1차 미분 신호의 피크                        |
| Second Derivative Features | PPG 파형의 곡률/가속도 성분 관련 특징                 |

특히 논문에서는 **미분 기반 특징**이 혈당 예측 성능 향상에 중요한 역할을 한 것으로 보고했다.

PPG 원신호보다 미분 신호를 사용하면 peak가 더 뚜렷해지고, 파형의 형태적 변화가 강조된다. 이는 혈당 변화가 직접적으로 빛의 세기를 바꾸기보다는 혈류, 혈관 탄성, 말초 순환 상태 등에 간접적으로 영향을 줄 수 있기 때문에 의미가 있다.

---

### 3.5 회귀 모델

논문에서는 혈당 예측 모델로 **PCR, Principal Component Regression**을 사용했다.

PCR은 다음과 같은 방식이다.

```text
여러 PPG feature
→ PCA로 주요 성분 추출
→ 추출된 principal components로 회귀 모델 학습
→ 혈당값 예측
```

PCR을 사용한 이유는 PPG 특징들 사이에 상관관계가 있을 수 있기 때문이다. PCA를 통해 중복 정보를 줄이고, 주요 성분만 사용해 회귀를 수행한다.

---

## 4. 주요 결과

논문에서 비교한 주요 결과는 다음과 같다.

| 입력 방식                  |           SEP |
| ---------------------- | ------------: |
| Raw PPG                | 약 29.17 mg/dL |
| ALS + Gaussian 전처리 PPG | 약 19.96 mg/dL |
| 전처리 + 미분 특징            | 약 18.30 mg/dL |
| DelT 특징                | 약 21.54 mg/dL |

가장 좋은 성능은 **전처리된 PPG에서 미분 기반 characteristic points를 추출하고 PCR에 입력한 경우**였다.

논문은 최종적으로 다음 결과를 보고했다.

* Best SEP: 약 **18.30~18.31 mg/dL**
* Clarke Error Grid:

  * Zone A: 82.6%
  * Zone B: 17.4%
  * Zone C/D/E: 0%

즉, 논문 기준에서는 예측 결과 대부분이 임상적으로 위험하지 않은 범위에 들어갔다고 해석할 수 있다.

---

## 5. 논문의 핵심 기여

이 논문의 핵심 기여는 다음과 같다.

### 5.1 스마트폰 기반 비침습 혈당 추정 가능성 제시

별도의 고가 장비가 아닌 스마트폰 카메라와 플래시만으로 PPG를 추출하고, 이를 혈당 추정에 활용했다는 점에서 접근성이 높다.

### 5.2 Raw PPG보다 전처리와 feature extraction이 중요함을 보임

Raw PPG를 그대로 사용하는 것보다, ALS와 Gaussian filter를 적용하고 미분 특징을 추출했을 때 예측 성능이 크게 좋아졌다.

이는 생체신호 분석에서 단순히 모델을 복잡하게 만드는 것보다, **신호 품질 개선과 특징 설계가 매우 중요하다**는 점을 보여준다.

### 5.3 작은 데이터셋에서 PCR 기반 접근을 사용

데이터 수가 많지 않은 상황에서 딥러닝 모델을 사용하는 대신, PCA와 회귀를 결합한 PCR을 사용했다. 이는 소규모 생체신호 데이터셋에서는 현실적인 접근이다.

---

## 6. 개인적으로 중요하게 본 점

이 논문은 “스마트폰으로 혈당을 정확히 측정할 수 있다”를 완전히 증명한 논문이라기보다는, **PPG waveform morphology가 혈당과 관련된 정보를 일부 포함할 가능성이 있다**는 것을 보여준 연구에 가깝다.

혈당은 PPG로 직접 측정되는 물질이 아니다.
스마트폰 카메라가 glucose molecule을 직접 감지하는 것이 아니라, 혈당 변화와 관련될 수 있는 말초 혈류, 혈관 반응, 혈액 점도, 피부 광학 특성 변화 등이 PPG 파형에 간접적으로 반영된다고 보는 것이 더 타당하다.

따라서 이 연구는 다음과 같이 해석하는 것이 적절하다.

```text
직접 혈당 센싱 연구
보다는
PPG 형태 특징 기반 혈당 간접 추정 연구
```

---

## 7. 한계점

### 7.1 피험자 수가 적음

피험자는 18명으로 매우 적다.
혈당 추정은 개인차가 큰 문제이기 때문에, 이 정도 데이터만으로 일반화 성능을 주장하기는 어렵다.

피부색, 손가락 두께, 혈압, 체온, 말초혈류, 식사 상태, 측정 압력 등이 모두 PPG 파형에 영향을 줄 수 있다.

---

### 7.2 Train/Test 분리 방식이 명확하지 않음

논문은 전체 88 trials 중 75%를 학습, 25%를 테스트에 사용했다고 설명한다.
하지만 피험자 단위로 분리했는지, trial 단위로 랜덤 분리했는지는 명확하지 않다.

만약 같은 피험자의 일부 trial이 train set에 들어가고, 다른 trial이 test set에 들어갔다면 모델 성능이 실제보다 좋게 보일 수 있다.

이 문제를 피하려면 다음과 같은 검증이 필요하다.

```text
Subject-wise split
또는
Leave-One-Subject-Out Validation
```

---

### 7.3 혈당 분포 범위가 충분히 설명되지 않음

Clarke Error Grid 결과는 좋아 보이지만, 실제 혈당값이 좁은 범위에 몰려 있다면 모델이 좋아 보일 수 있다.

예를 들어 대부분의 데이터가 정상 혈당 범위 근처에 있다면, 저혈당이나 고혈당 상황에서도 잘 동작하는지는 알 수 없다.

---

### 7.4 스마트폰 촬영 조건에 취약할 수 있음

스마트폰 PPG는 다음 요소에 민감하다.

* 손가락을 누르는 압력
* 플래시 밝기와 발열
* 카메라 자동 노출
* 자동 화이트밸런스
* 손가락 위치 변화
* 움직임
* 피부색과 손가락 두께

논문은 iPhone 7 Plus 하나로 실험했기 때문에, 다른 스마트폰에서도 같은 성능이 나올지는 추가 검증이 필요하다.

---

### 7.5 Reference 혈당계 자체의 오차

정답값으로 상용 혈당계를 사용했지만, 혈당계 자체에도 측정 오차가 있다.
비침습 추정 모델을 학습할 때 reference label에 오차가 있으면 모델 성능 평가도 불확실해질 수 있다.

---

### 7.6 용어와 방법 설명의 일관성 부족

논문 제목에는 PCA-DA가 들어가지만, 실제 방법 설명과 결과에서는 PCR이 중심적으로 사용된다.
또한 feature 설명에서 first derivative와 second derivative 관련 표현이 혼재되어 있어 재현성 측면에서 아쉬움이 있다.

---

## 8. 발전 가능한 방향

이 논문을 발전시키기 위해서는 다음과 같은 방향이 필요하다.

### 8.1 더 큰 데이터셋 확보

피험자 수를 늘리고, 다양한 조건에서 데이터를 수집해야 한다.

* 정상 혈당
* 저혈당
* 고혈당
* 식전/식후
* 다양한 연령
* 다양한 피부톤
* 다양한 스마트폰 기종

---

### 8.2 Subject-independent 검증

실제 적용 가능성을 보려면 같은 사람이 train과 test에 동시에 들어가면 안 된다.

권장 검증 방식은 다음과 같다.

```text
1. Subject-wise train/test split
2. Leave-One-Subject-Out Cross Validation
3. External validation dataset 평가
```

---

### 8.3 RGB 채널 및 추가 특징 활용

논문은 주로 red channel 평균값을 사용했다.
하지만 스마트폰 영상에는 RGB 채널 정보가 있으므로, 다음과 같은 특징을 추가로 사용할 수 있다.

* Red/Green/Blue channel별 AC/DC ratio
* RGB normalized intensity
* Channel ratio feature
* PPG amplitude variability
* Pulse interval variability
* Systolic upstroke time
* Diastolic decay time
* First derivative peak
* Second derivative APG feature

---

### 8.4 PCR 외의 모델 비교

PCR 외에도 다음 모델을 비교할 수 있다.

* PLSR
* SVR
* Random Forest
* XGBoost
* Gaussian Process Regression
* LightGBM
* 작은 MLP 모델

데이터가 적을 경우에는 복잡한 딥러닝보다 PCR, PLSR, Random Forest, SVR 같은 모델이 더 현실적일 수 있다.

---

### 8.5 평가 지표 다양화

SEP만으로는 모델 성능을 충분히 설명하기 어렵다.
다음 지표를 함께 사용하는 것이 좋다.

* MAE
* RMSE
* MARD
* Bland-Altman Plot
* Clarke Error Grid
* Parkes Error Grid
* Subject-wise error distribution

---

## 9. 내가 활용할 수 있는 아이디어

이 논문에서 바로 가져올 수 있는 가장 유용한 아이디어는 혈당 예측 자체보다는 **PPG 신호 처리와 feature extraction 방식**이다.

### 9.1 ALS baseline correction

rPPG나 fingertip PPG에서 baseline drift가 발생할 때 ALS를 전처리 후보로 사용할 수 있다.

활용 예시:

```text
Raw PPG / rPPG
→ ALS baseline correction
→ Bandpass filtering
→ Peak detection
→ HR 또는 SQI 계산
```

---

### 9.2 미분 기반 PPG feature

PPG의 1차 미분과 2차 미분은 파형의 형태적 변화를 더 잘 드러낼 수 있다.

활용 가능한 feature:

* First derivative peak amplitude
* Peak-to-peak interval
* Systolic upstroke slope
* Diastolic decay slope
* APG feature
* Peak sharpness
* Pulse morphology consistency

이러한 특징은 혈당 예측뿐 아니라 rPPG 품질 평가에도 사용할 수 있다.

---

### 9.3 SQI 설계에 활용

내 연구에서 rPPG와 radar를 융합할 때, 각 신호의 품질을 평가하는 SQI가 중요하다.
이 논문의 PPG morphology feature는 SQI 설계에 활용할 수 있다.

예시:

```text
rPPG signal
→ peak detection
→ derivative feature extraction
→ pulse morphology consistency 계산
→ SQI_rPPG 계산
→ Kalman Filter의 measurement noise R에 반영
```

즉, PPG의 피크와 미분 특징을 단순 예측 feature가 아니라 **신호 품질 평가 feature**로 사용할 수 있다.

---

## 10. 최종 정리

이 논문은 스마트폰 카메라 기반 PPG를 이용해 혈당을 추정하려는 초기 연구이다.
실험 규모가 작고 검증 방식에 한계가 있기 때문에, 실제 의료기기로 사용 가능하다고 보기는 어렵다.

하지만 다음 점에서 참고 가치가 있다.

* 스마트폰 영상에서 PPG를 추출하는 간단한 구조를 제시했다.
* Gaussian filter와 ALS를 이용한 전처리 흐름을 보여준다.
* Raw PPG보다 전처리와 미분 특징 추출이 성능을 크게 개선했다.
* 작은 데이터셋에서는 PCR 같은 전통적 회귀 모델도 실용적인 선택이 될 수 있다.
* PPG morphology feature는 혈당뿐 아니라 HR 추정, SQI 계산, rPPG 품질 평가에도 활용 가능하다.

개인적으로 이 논문은 혈당 예측 논문으로만 보기보다, **카메라 기반 PPG 신호에서 형태학적 특징을 추출하고 생체정보 추정에 활용한 사례**로 참고하는 것이 더 현실적이라고 생각한다.

---

## Takeaway

> 스마트폰 카메라 기반 PPG만으로 혈당을 안정적으로 측정할 수 있다고 단정하기는 어렵다.
> 그러나 PPG 전처리, baseline correction, 미분 기반 morphology feature extraction은 rPPG/PPG 생체신호 분석에서 충분히 활용 가치가 있다.
