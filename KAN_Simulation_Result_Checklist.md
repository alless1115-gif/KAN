# KAN Simulation Result Checklist

## 목적

이 문서는 `vg-id` curve 기반 KAN simulation을 실행한 뒤,
결과가 좋은지, 애매한지, 나쁜지를 판단하기 위한 체크리스트다.

대상 코드:

- [GKAN_VGID_DeviceBasis_Simulation.py](C:\Users\jueun\OneDrive\바탕 화면\260327_랩미팅\GKAN code\Main Code Files\Main Codes\GKAN_VGID_DeviceBasis_Simulation.py)

주요 확인 대상:

- basis plot
- target vs prediction plot
- training loss plot
- coefficient 결과
- basis bank 데이터


## 1. 가장 중요한 판단 질문

simulation 결과를 볼 때 가장 먼저 아래 질문 하나로 생각하면 된다.

`서로 다른 control bias에서 나온 Vg-Id 곡선들이 조합되어 새로운 1D 함수를 잘 만들 수 있는가?`

이 질문에 대해 `예`에 가까우면 좋은 결과다.


## 2. 반드시 확인할 출력 파일

코드 실행 후 아래 결과를 확인한다.

### 그래프

- basis plot
- target / prediction plot
- loss plot

### 엑셀 파일

- `device_basis_fit_result.xlsx`

확인 시트:

- `fit_result`
- `coefficients`
- `basis_bank`


## 3. Basis Plot 판정 기준

### 확인 항목

- curve들이 서로 충분히 다른가
- 단순한 amplitude 차이만 있는가
- threshold shift가 있는가
- slope 변화가 있는가
- width 변화가 있는가
- peak 위치 차이가 있는가
- 입력 구간 전체를 커버하는가

### 좋은 결과

- 곡선들이 눈에 띄게 다르다
- 단순 위아래 scaling이 아니라 shape 차이가 있다
- 각 곡선이 입력축에서 다른 역할을 할 수 있어 보인다

### 애매한 결과

- 일부 곡선만 다르고 나머지는 매우 유사하다
- shape 차이가 작다
- 차이는 보이지만 반복 측정 노이즈와 구분이 어렵다

### 나쁜 결과

- 거의 모든 곡선이 같은 모양이다
- amplitude만 다르고 shape는 거의 같다
- basis 다양성이 매우 부족해 보인다


## 4. Target vs Prediction Plot 판정 기준

### 확인 항목

- prediction이 target의 전체 형태를 따라가는가
- peak 위치를 재현하는가
- peak 폭을 재현하는가
- peak 높이를 재현하는가
- 특정 구간만 맞고 나머지가 무너지지 않는가

### 좋은 결과

- target과 prediction이 전체적으로 잘 겹친다
- 주요 peak와 valley 구조를 따라간다
- 국소적인 오차가 있어도 전체 shape는 잘 맞는다

### 애매한 결과

- 전체 추세는 맞지만 세부 구조가 잘 안 맞는다
- 몇몇 peak는 맞고 몇몇 peak는 틀린다
- 일부 구간에서만 성능이 떨어진다

### 나쁜 결과

- 전체 shape를 거의 못 따라간다
- peak 위치가 크게 어긋난다
- prediction이 너무 평평하거나 지나치게 진동한다


## 5. Loss Plot 판정 기준

### 확인 항목

- 초반에 loss가 감소하는가
- 중간 이후 loss가 안정적으로 수렴하는가
- loss가 크게 흔들리거나 발산하지 않는가

### 좋은 결과

- 초반에 명확하게 감소한다
- 이후 점차 완만하게 줄어든다
- 큰 진동 없이 수렴한다

### 애매한 결과

- 감소는 하지만 느리다
- 중간에 plateau가 길다
- 약간의 진동이 있다

### 나쁜 결과

- 거의 줄지 않는다
- 초반부터 평평하다
- 큰 진동이나 발산이 보인다


## 6. Coefficient 판정 기준

엑셀 파일의 `coefficients` 시트를 본다.

### 확인 항목

- 실제로 사용된 basis 개수가 몇 개인가
- coefficient가 한두 basis에만 몰리는가
- 거의 모든 basis가 0에 가까운가

### 좋은 결과

- 여러 basis가 의미 있게 사용된다
- 특정 basis 하나에만 완전히 의존하지 않는다
- basis bank 전체가 어느 정도 정보 기여를 한다

### 애매한 결과

- 2~3개 basis만 주로 사용된다
- 나머지는 거의 작다
- basis 수가 과도하게 많을 수 있다

### 나쁜 결과

- 사실상 1개 basis만 사용된다
- 대부분 coefficient가 거의 0이다
- basis들이 서로 redundant할 가능성이 높다


## 7. Basis Bank 자체 판정 기준

엑셀 파일의 `basis_bank` 시트를 본다.

### 확인 항목

- basis 개수가 너무 적지 않은가
- 서로 지나치게 유사하지 않은가
- 이상치 curve가 섞여 있지 않은가
- 특정 basis만 지나치게 큰 값 범위를 갖지 않는가

### 좋은 결과

- 5개 이상 basis가 있고 서로 차별성이 있다
- 극단적인 이상치가 없다
- 전처리 후 비교 가능한 범위를 가진다

### 애매한 결과

- basis 수는 충분하지만 차이가 작다
- 일부 basis가 이상치처럼 보인다

### 나쁜 결과

- basis 수가 너무 적다
- 대부분 중복된 curve다
- 노이즈가 너무 큰 curve가 많이 섞여 있다


## 8. 한 번에 판정하는 빠른 체크

아래 4개 중 3개 이상이 만족되면 1차적으로 좋은 결과로 본다.

- basis 곡선들이 서로 다르다
- target fitting이 잘 된다
- loss가 안정적으로 감소한다
- coefficient가 여러 basis에 분산된다

아래 4개 중 2개 이상이면 추가 검증이 필요하다.

- 곡선 차이는 보이지만 작다
- fitting은 되지만 일부 구간이 약하다
- coefficient가 몇 개 basis에 몰린다
- 반복 측정 재현성을 아직 확인하지 못했다

아래 중 하나라도 강하게 보이면 나쁜 결과로 본다.

- basis 곡선이 거의 다 같다
- fitting이 전반적으로 안 된다
- loss가 거의 줄지 않는다
- coefficient가 사실상 1개 basis에만 몰린다


## 9. 좋은 결과 다음에 해야 할 일

좋은 결과가 나오면 아래 순서로 간다.

1. 같은 조건 반복 측정으로 재현성 확인
2. control bias 개수 증가
3. 다른 control variable로 확장
4. toy target 대신 실제 target function으로 교체
5. 더 큰 network 구조 검토


## 10. 애매한 결과일 때 해야 할 일

아래 항목을 먼저 점검한다.

- basis 개수가 너무 적지 않은가
- 정규화 방식이 맞는가
- 곡선들이 실제로 shape 차이를 가지는가
- sweep range가 적절한가
- control bias 구간이 너무 좁지 않은가
- 반복 측정 노이즈가 너무 크지 않은가


## 11. 나쁜 결과일 때 의심할 것

- 소자가 basis source로 적합하지 않을 수 있다
- control bias가 shape를 충분히 바꾸지 않을 수 있다
- 현재 선택한 sweep gate가 적절하지 않을 수 있다
- 다른 control variable이 더 중요할 수 있다
- 이 소자는 basis보다 coefficient source에 더 가까울 수 있다


## 12. 최종 한 줄 판정

### 좋은 결과

`서로 다른 bias 조건의 Vg-Id 곡선들이 basis로서 서로 다른 역할을 하고, 그 조합만으로 target 함수를 충분히 재구성할 수 있다.`

### 애매한 결과

`basis 후보는 보이지만 곡선 차이 또는 재현성이 충분한지 추가 검증이 필요하다.`

### 나쁜 결과

`현재 조건에서 얻은 Vg-Id curve family는 basis bank로 쓰기에는 다양성이나 재현성이 부족하다.`
