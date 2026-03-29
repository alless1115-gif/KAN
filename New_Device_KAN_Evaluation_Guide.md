# New Device KAN Evaluation Guide

## 목적

이 문서는 새로운 소자를 가지고 KAN simulation을 시작할 때,
무엇을 먼저 확인해야 하는지와 그 소자가 KAN에서 어떤 역할을 맡을 수 있는지
판단하는 기준을 정리한 문서다.

핵심은 기존 코드의 변수명(`memristor conductance`, `synapse`, `GMC peak current`)에
소자를 억지로 맞추는 것이 아니라,
새 소자가 KAN 안에서 어떤 수학적 역할을 할 수 있는지를 먼저 정의하는 것이다.


## 1. 가장 먼저 확인해야 하는 항목

### 1) 입력과 출력 정의

- 입력으로 무엇을 줄 수 있는가
  - voltage
  - pulse number
  - time
  - frequency
  - light intensity
- 출력으로 무엇을 읽는가
  - current
  - conductance
  - resistance
  - threshold shift
  - optical response

확인 포인트:

- 입력과 출력이 명확히 정의되어 있는가
- 같은 입력 조건에서 출력이 반복적으로 측정 가능한가
- KAN의 입력 `x`로 연결할 수 있는 물리량이 분명한가


### 2) 출력 범위와 level 구조

- 출력 dynamic range가 충분한가
- 구분 가능한 discrete level 수가 몇 개인가
- level spacing이 균일한가
- 특정 구간에 값이 과도하게 몰려 있지 않은가

확인 포인트:

- KAN weight 또는 coefficient 후보값으로 쓸 수 있을 정도로 값 분해능이 있는가
- 양자화 후에도 의미 있는 값 다양성이 남는가


### 3) 재현성과 변동성

- 같은 입력을 여러 번 넣었을 때 같은 출력이 나오는가
- cycle-to-cycle variation이 큰가
- device-to-device variation이 큰가
- 읽기 노이즈가 큰가

확인 포인트:

- 평균값만이 아니라 표준편차나 분산도 함께 봐야 한다
- simulation에 들어갈 값은 단일 곡선보다 분포 형태로 이해하는 것이 좋다


### 4) 시간 안정성

- retention이 충분한가
- 시간이 지나면서 drift가 얼마나 생기는가
- read disturb 또는 stress effect가 있는가

확인 포인트:

- 학습 직후 값만 맞는 소자인지
- 시간이 지나도 표현 상태가 유지되는 소자인지


### 5) 업데이트 가능성

- 원하는 방향으로 값 조절이 가능한가
- 점진적인 multi-level update가 가능한가
- 너무 binary하게만 바뀌지 않는가

확인 포인트:

- 학습 가능한 hardware weight처럼 쓰려면 단계적 조절 가능성이 중요하다


### 6) 음수/양수 표현 가능성

- 출력이 한 방향 값만 가지는가
- differential pair로 signed weight를 만들 수 있는가
- 0 근처 값을 안정적으로 표현할 수 있는가

확인 포인트:

- KAN의 선형 weight나 coefficient는 보통 signed representation이 필요하다


### 7) 출력 곡선의 형태

- 출력 그래프가 smooth한가
- local response shape를 가지는가
- center 이동, 폭 조절, amplitude 변화가 가능한가
- hysteresis 때문에 동일 입력에서도 경로 의존성이 큰가

확인 포인트:

- 이 곡선을 basis function 자체로 사용할 수 있는지 판단해야 한다


## 2. 소자가 KAN에서 맡을 수 있는 역할

새 소자는 보통 아래 세 역할 중 하나로 분류할 수 있다.

### A. Weight Level Provider

소자가 직접 basis를 만들지는 않지만,
KAN의 weight가 가질 수 있는 discrete value 집합을 제공하는 경우다.

예:

- conductance levels
- differential current levels
- resistance difference levels

이 경우 필요한 것:

- 구분 가능한 level 수
- signed weight 구현 가능성
- 양자화 후 재현성

이 경우 기존 코드에서 가장 가까운 대응:

- `synapse`
- `spline_coef`

즉, 기존 Gaussian-like basis는 유지하고,
소자는 weight candidate set 역할만 담당한다.


### B. Basis Function Source

소자의 실제 입력-출력 곡선 자체가 KAN의 basis function 역할을 할 수 있는 경우다.

예:

- 특정 입력 구간에서 봉우리 형태를 가지는 응답
- shift 가능한 localized response
- amplitude 또는 width 제어 가능한 비선형 응답

이 경우 필요한 것:

- 곡선이 충분히 smooth할 것
- 서로 다른 center 또는 조건에서 여러 개의 basis를 만들 수 있을 것
- basis들이 입력 구간을 충분히 커버할 것

이 경우 기존 코드에서 바뀌는 부분:

- `PrewiseRadialBasisFunction()`을 새 소자 응답 함수로 교체해야 한다


### C. Hybrid Role

소자가 level provider 역할도 하고 basis source 역할도 할 수 있는 경우다.

이 경우 장점:

- 더 직접적인 hardware-aware KAN 구현 가능

이 경우 단점:

- 모델링 복잡도 증가
- 소자 variation과 basis variation을 동시에 다뤄야 함

처음 평가 단계에서는 hybrid보다
weight 역할과 basis 역할을 분리해서 검증하는 것이 안전하다.


## 3. Weight형 소자인지 판단하는 기준

아래 조건을 대체로 만족하면,
그 소자는 먼저 weight형 소자로 평가하는 것이 맞다.

### Weight형 판단 기준

- 출력이 discrete level 집합으로 안정적으로 정리된다
- 각 level이 반복 측정에서 비교적 잘 유지된다
- level 간격이 너무 심하게 불균일하지 않다
- differential pair 또는 다른 방법으로 signed weight를 만들 수 있다
- 출력 곡선 자체를 basis로 쓰기보다는 값 자체가 더 안정적이다
- 입력에 따른 출력 응답이 다소 단순해도 무방하다

### Weight형으로 시작하는 것이 좋은 경우

- 소자의 가장 큰 장점이 multi-level memory 특성일 때
- 출력 곡선 shape는 특별하지 않지만 상태 저장 성능이 좋을 때
- 기존 KAN 구조를 크게 바꾸지 않고 새 소자를 넣고 싶을 때

### Weight형이면 처음 simulation에서 볼 것

- 양자화 후 1D function fitting 성능
- level 수에 따른 fitting 차이
- variation 반영 전/후 성능 변화
- drift 반영 전/후 성능 변화


## 4. Basis형 소자인지 판단하는 기준

아래 조건을 대체로 만족하면,
그 소자는 basis형 소자로 평가하는 것이 맞다.

### Basis형 판단 기준

- 입력-출력 곡선이 smooth하고 예측 가능하다
- localized response 또는 특정 shape를 가진다
- center shift, width tuning, amplitude tuning 중 하나 이상이 가능하다
- 여러 조건에서 측정한 곡선들이 서로 다른 basis 집합처럼 동작한다
- discrete level 특성보다 응답 곡선 shape 자체가 더 중요하다

### Basis형으로 시작하는 것이 좋은 경우

- 소자의 가장 큰 장점이 비선형 전달 특성일 때
- Gaussian-like basis보다 실제 소자 응답을 쓰는 것이 더 자연스러울 때
- KAN의 spline/basis 부분을 하드웨어 곡선으로 직접 대체하고 싶을 때

### Basis형이면 처음 simulation에서 볼 것

- basis shape만으로 단순 1D 함수 근사가 되는지
- basis 개수에 따른 표현력 변화
- center coverage가 전체 입력 구간을 덮는지
- basis variation이 성능에 미치는 영향


## 5. Weight형과 Basis형을 구분하는 빠른 질문

아래 질문에 답하면 초기 방향을 빠르게 잡을 수 있다.

### 질문 1

"이 소자의 가장 큰 장점은 많은 상태값을 안정적으로 저장하는 것인가,
아니면 독특한 비선형 응답 곡선을 만드는 것인가?"

- 저장 상태값이 강점이면 Weight형 가능성이 크다
- 곡선 shape가 강점이면 Basis형 가능성이 크다


### 질문 2

"출력값을 하나의 숫자 level로 요약하는 것이 자연스러운가,
아니면 입력 전 구간의 response curve로 봐야 의미가 살아나는가?"

- 숫자 level로 요약 가능하면 Weight형에 가깝다
- 곡선 전체가 중요하면 Basis형에 가깝다


### 질문 3

"소자 응답을 여러 center로 이동시키거나 여러 조건에서 basis set처럼 만들 수 있는가?"

- 가능하면 Basis형에 적합하다
- 어렵다면 Weight형으로 시작하는 것이 안전하다


## 6. 가장 처음 해보는 simulation 순서

새 소자를 평가할 때는 아래 순서가 가장 안전하다.

### Step 1. 소자 전달 특성 측정

- 입력-출력 관계 측정
- dynamic range 확인
- 반복 측정 variation 확인
- 시간 drift 확인


### Step 2. 소자 역할 분류

- weight형
- basis형
- hybrid형


### Step 3. 가장 단순한 1D simulation

- 복잡한 분류보다 먼저 1D function fitting
- noise 없는 조건부터 시작
- 그 다음 quantization 반영
- 그 다음 variation 반영
- 마지막으로 drift 반영


### Step 4. 그 다음에 확장

- pattern recognition
- time series
- PDE

복잡한 task는 소자 기본 특성이 확인된 후에 들어가는 것이 맞다.


## 7. 처음 성공 기준

처음부터 높은 accuracy를 보는 것이 아니다.

초기 성공 기준은 아래와 같다.

- 단순한 함수 근사가 된다
- 양자화 후에도 학습이 완전히 무너지지 않는다
- 반복 실행 시 결과 경향이 유지된다
- variation과 drift를 넣었을 때 성능 저하가 해석 가능한 수준이다


## 8. 실무적으로 가장 먼저 정리할 데이터

아래 데이터는 꼭 먼저 표로 정리해두는 것이 좋다.

- 입력 변수
- 출력 변수
- usable output range
- distinguishable level count
- mean and standard deviation
- cycle-to-cycle variation
- device-to-device variation
- drift over time
- hysteresis
- signed weight 가능 여부
- basis shape 활용 가능 여부


## 9. 결론

새 소자를 KAN simulation에 넣을 때 가장 먼저 해야 할 일은
기존 코드 변수명에 소자를 맞추는 것이 아니라,
그 소자가 아래 중 무엇인지 판단하는 것이다.

- weight level provider
- basis function source
- hybrid device

처음에는 대부분 1D regression으로 시작하는 것이 맞지만,
그 안에서 어떤 부분을 새 소자 특성으로 대체할지는
이 역할 분류가 끝난 뒤에 결정해야 한다.

즉, 첫 질문은
"이 소자가 memristor와 같은가?"가 아니라
"이 소자는 KAN에서 weight를 담당하는가, basis를 담당하는가?"여야 한다.
