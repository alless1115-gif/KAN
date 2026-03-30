# IV 학습 코드 설명

이 문서는 현재 추가된 `GKAN_IV_Contact_Study.py` 코드가 무엇을 하는지, 왜 이렇게 구성했는지, 그리고 `top contact` 기준으로 어떻게 사용하는지를 정리한 설명 파일이다.

## 1. 이 코드의 목적

이 코드는 `top contact`, `bottom contact`, `NTC` 같은 실측 I-V 데이터를 이용해서 다음 두 가지를 순서대로 수행하기 위해 만들었다.

1. 각 I-V curve의 peak 위치와 fitting 가능 영역을 찾는다.
2. 그 curve들을 basis function처럼 사용해서 기존 GKAN의 1D regression과 같은 형태의 학습을 수행한다.

즉, 이 코드는 "실측 소자 곡선을 basis bank로 볼 수 있는가?"를 먼저 확인하고, 그 다음 "이 basis bank로 기존 GKAN과 같은 fitting 문제를 풀 수 있는가?"를 확인하는 역할을 한다.

## 2. 기존 GKAN과의 차이

기존 `GKAN_1DRG.py`에서는 출력이 아래처럼 계산된다.

```text
output = base_output + spline_output
```

- `base_output`: memristor conductance에 해당하는 residual/base term
- `spline_output`: Gaussian-like basis function들의 합

즉 기존 구조는 개념적으로 다음과 같다.

```text
y(Vg) = base(Vg) + gaussian_basis(Vg)
```

반면 지금 소자에서는 `Vs`를 바꾸면 peak의 높이나 모양이 바뀌므로, Gaussian을 직접 생성하기보다 실측 curve 자체를 basis로 사용하는 것이 더 자연스럽다.

그래서 현재 IV 학습 코드는 아래 구조를 사용한다.

```text
y(Vg) = sum_i a_i * I_measured(Vg ; Vs_i) + b
```

즉,

- `Vs_i`별 실측 curve 하나하나가 basis
- 학습되는 값은 각 basis의 coefficient `a_i`
- 필요하면 나중에 여기에 기존 memristor base term을 다시 추가할 수 있음

현재 단계에서는 "문제는 기존 GKAN과 동일하게 1D fitting으로 두되, basis만 실측 curve로 바꾼 버전"이라고 이해하면 된다.

## 3. 코드 전체 흐름

현재 메인 코드는 `GKAN_IV_Contact_Study.py`이다.

전체 파이프라인은 다음 순서로 진행된다.

1. manifest CSV를 읽는다.
2. manifest가 가리키는 raw CSV에서 I-V curve를 불러온다.
3. 각 curve의 peak를 찾는다.
4. peak를 기준으로 left/right fitting window를 찾는다.
5. `|dVg|`와 `|dVg|^2` 중 어느 쪽이 더 잘 맞는지 비교한다.
6. 선택된 curve들로 basis bank를 만든다.
7. basis coefficient만 학습하는 1D regression을 수행한다.
8. 결과를 엑셀과 그림으로 저장한다.

## 4. 입력 파일 구조

코드는 크게 두 종류의 입력을 사용한다.

### 4-1. manifest CSV

예:

- `Top_Contact_Condition_Log_Example.csv`
- `NTC_AAT_B1500_Condition_Log_Template.csv`
- `Advanced_AAT_B1500_Condition_Log_Template.csv`

manifest는 "어떤 raw CSV 파일의 어떤 column을 basis로 쓸 것인지"를 설명하는 매핑 테이블이다.

중요 column은 아래와 같다.

- `basis_label`: curve 이름
- `raw_csv_file`: 실제 raw data 파일명
- `raw_csv_column`: raw CSV 안의 전류 column명
- `raw_axis_column`: raw CSV 안의 sweep axis column명. 보통 `Vg`
- `sweep_start`, `sweep_stop`, `sweep_step`: 축 column이 없을 때 sweep axis 복원에 사용
- `use_for_basis`: `Y`인 curve만 basis로 사용

### 4-2. raw CSV

예를 들어 아래와 같은 형태를 가정한다.

```csv
Vg,Data001,Data002,Data003
-15,1.2E-9,1.5E-9,1.8E-9
-14.9,1.3E-9,1.6E-9,1.9E-9
...
```

여기서

- `Vg`: sweep axis
- `Data001`, `Data002`, `Data003`: 서로 다른 `Vs` 조건에서의 전류

## 5. fitting은 어떻게 판단하는가

이 코드에서 fitting은 "peak 근처 곡선이 어떤 함수형을 따르는가"를 판단하는 과정이다.

### 5-1. peak 탐색

먼저 각 curve에서 peak를 찾는다.

- 기본 설정은 `|I|`가 최대인 지점을 peak로 사용
- 그래서 `Vpeak`, `Ipeak`가 결정된다

### 5-2. left/right를 분리해서 분석

contact 소자는 비대칭일 수 있으므로 peak의 왼쪽과 오른쪽을 따로 본다.

- left side
- right side

각 side에 대해 별도로 fitting window와 transform을 선택한다.

### 5-3. 비교하는 두 가지 축

peak에서 떨어진 정도를 다음처럼 정의한다.

```text
dVg = |Vg - Vpeak|
```

그리고 전류는 peak 대비 감소량으로 바꾼다.

```text
r = |I| / |Ipeak|
y = -log(r)
```

이제 아래 두 가지를 비교한다.

```text
y vs |dVg|
y vs |dVg|^2
```

이 의미는 다음과 같다.

- `y`가 `|dVg|^2`에 대해 더 직선이면 Gaussian형에 가깝다.
- `y`가 `|dVg|`에 대해 더 직선이면 exponential형에 가깝다.

### 5-4. fitting window 선택

peak 바로 근처의 작은 구간부터 시작해서, 바깥쪽으로 점을 하나씩 늘려가며 선형성을 검사한다.

즉 각 side에 대해 다음을 반복한다.

1. peak 포함 5개 점
2. peak 포함 6개 점
3. peak 포함 7개 점
4. ...

각 window에 대해 선형 fitting을 수행하고 `R^2`, `RMSE`를 계산한다.

### 5-5. 최종 선택 기준

기본 기준은 다음과 같다.

1. `R^2 >= 0.995`를 만족하는 후보를 우선 선택
2. 그중 가장 넓은 window를 선택
3. 같으면 `R^2`가 더 높은 쪽을 선택
4. threshold를 만족하는 후보가 없으면 전체 후보 중 `R^2`가 가장 좋은 것을 선택

그래서 최종적으로 각 curve에 대해 다음이 저장된다.

- peak 위치
- left에서 가장 잘 맞는 transform
- left fitting window
- right에서 가장 잘 맞는 transform
- right fitting window

## 6. `curve_summary` 시트 의미

결과 엑셀의 `curve_summary` 시트에는 curve별 요약 정보가 저장된다.

대표 column은 아래와 같다.

- `dataset_name`: 데이터셋 이름
- `basis_label`: curve 이름
- `peak_axis`: peak가 생긴 sweep axis 값
- `peak_current`: peak 전류
- `left_best_transform`: left side에서 더 잘 맞는 축
- `left_best_r2`: left side의 최종 선형성
- `left_window_start`, `left_window_end`: left side fitting 구간
- `right_best_transform`: right side에서 더 잘 맞는 축
- `right_best_r2`: right side의 최종 선형성
- `right_window_start`, `right_window_end`: right side fitting 구간

해석 예시는 다음과 같다.

```text
left_best_transform = |dVg|^2
right_best_transform = |dVg|
```

이 경우 왼쪽 꼬리는 Gaussian-like, 오른쪽 꼬리는 exponential-like로 보는 것이 더 적절하다는 뜻이다.

## 7. basis bank는 어떻게 만들어지는가

fitting 결과를 얻은 다음에는 각 curve를 공통 축으로 보간해서 basis bank를 만든다.

이때 두 가지 방식이 가능하다.

### 7-1. `crop-mode = full`

curve 전체를 basis로 사용한다.

### 7-2. `crop-mode = fit_union`

left/right에서 선택된 fitting window의 합집합만 basis로 사용한다.

즉 peak 근처에서 "의미 있는 영역"만 잘라서 basis로 사용하는 방식이다.

현재 기본값은 `fit_union`이다.

## 8. 학습은 어떻게 진행되는가

학습 단계에서는 각 curve를 하나의 basis function으로 본다.

구조는 다음과 같다.

```text
prediction = basis_response @ coeff + bias
```

여기서

- `basis_response`: 입력 `Vg`에서 각 basis curve가 가지는 값
- `coeff`: 학습되는 coefficient
- `bias`: 상수항

즉 현재 코드는 "measured curve basis linear combination" 구조다.

이 학습은 기존 GKAN의 1D fitting 문제와 비교하기 위해 만들어진 것이고, 아직 memristor base term은 다시 붙이지 않았다. 지금은 먼저 contact basis만으로도 fitting이 되는지 확인하는 단계다.

## 9. `benchmark_summary` 시트 의미

학습 결과는 `benchmark_summary` 시트에 저장된다.

현재는 여러 개의 target function에 대해 같은 방식으로 fitting을 수행한다.

예:

- `gaussian_mixture`
- `skewed_peak`
- `quadratic_to_gaussian`
- `n_shape`

대표 column은 아래와 같다.

- `dataset_name`: 어떤 basis bank를 사용했는가
- `target_name`: 어떤 target function을 맞추었는가
- `final_mse`: 최종 MSE
- `final_r2`: 최종 R²
- `epochs`: 학습 epoch 수
- `learning_rate`: learning rate
- `weight_decay`: weight decay
- `n_basis`: basis 개수

해석은 간단하다.

- `final_mse`가 작을수록 좋다
- `final_r2`가 1에 가까울수록 좋다

만약 `top`, `bottom`, `NTC`를 같은 target에 대해 비교하면, 어떤 소자 basis가 어떤 유형의 문제에 더 유리한지 볼 수 있다.

특히 `n_shape` target에서 `NTC`가 유리한지 보는 것이 현재 연구 방향과 잘 맞는다.

## 10. 저장되는 결과 파일

예를 들어 `top contact`만 실행하면 아래 형태로 저장된다.

```text
top_contact_outputs/
  top_contact/
    top_contact_analysis.xlsx
    curve_overview.png
    basis_bank.png
    gaussian_mixture_fit.png
    skewed_peak_fit.png
    quadratic_to_gaussian_fit.png
    n_shape_fit.png
```

엑셀 파일 안에는 아래 시트가 들어간다.

- `curve_summary`
- `side_fit_records`
- `basis_bank`
- `benchmark_summary`
- 각 target별 coefficient 시트

여러 dataset을 같이 실행하면 비교용 `benchmark_comparison.xlsx`도 추가로 저장된다.

## 11. top contact만 먼저 진행하는 권장 절차

현재 연구 흐름상 가장 추천하는 순서는 다음과 같다.

1. `top contact`의 raw CSV 준비
2. manifest CSV에서 `Vs`별 curve 매핑
3. `curve_summary`에서 peak 위치와 `|dVg|` vs `|dVg|^2` 판단
4. `fit_union` 기준으로 basis bank 생성
5. `benchmark_summary`에서 학습 성능 확인
6. 그 후 동일한 방식으로 `bottom contact` 수행
7. 마지막으로 `NTC`를 넣어 `n_shape`나 shape-tunable 문제에서 더 유리한지 비교

## 12. 실행 예시

예시 명령은 아래와 같다.

```powershell
python GKAN_IV_Contact_Study.py `
  --manifest Top_Contact_Condition_Log_Example.csv `
  --output-dir top_contact_outputs `
  --powers 1 2 `
  --crop-mode fit_union `
  --basis-normalization peak `
  --epochs 2000
```

## 13. 지금 코드의 한계와 다음 단계

현재 코드는 다음 목적에 맞춰 단순화되어 있다.

- peak/fitting 구간 확인
- 실측 curve를 basis로 한 1D regression 가능성 확인

아직 포함하지 않은 것은 아래와 같다.

- 기존 GKAN의 memristor base term 재결합
- `Vs` 자체를 입력으로 받는 2D 모델
- 실측 target curve를 바로 regression target으로 넣는 구조

따라서 지금은 "연구 방향을 빠르게 검증하는 1차 분석 코드"로 보면 된다.

다음 단계에서는 결과에 따라 아래 중 하나로 확장할 수 있다.

1. `top/bottom` basis만으로 충분하면 현재 구조 유지
2. memristor residual이 필요하면 기존 `base_output` 추가
3. `Vs` dependence를 더 직접적으로 반영하려면 `y(Vg, Vs)` 형태의 2D 모델로 확장

## 14. 한 줄 요약

현재 `IV 학습 코드`는 실측 `top/bottom/NTC` I-V curve를 basis function처럼 사용해서,

- peak 주변의 fitting 영역이 `|dVg|`형인지 `|dVg|^2`형인지 먼저 판단하고
- 그 basis bank로 기존 GKAN과 같은 1D fitting 문제를 푸는 코드

라고 보면 된다.
