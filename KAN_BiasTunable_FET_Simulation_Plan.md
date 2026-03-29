# KAN Simulation Plan for Bias-Tunable FET Devices

## 목적

이 문서는 메모리 기능이 없는 FET 계열 소자에서,
`Vg-Id` transfer curve의 shape가 `Vd`, back-gate, 또는 추가 split-gate bias에 따라 달라지는 경우를
KAN simulation으로 연결하기 위한 1차 계획안이다.

핵심 가정은 아래와 같다.

- 메모리 state를 저장하는 소자가 아니다
- `drain bias` 또는 `additional gate bias`에 따라 Gaussian 형태 또는 유사 Gaussian 형태가 바뀐다
- 따라서 소자는 `weight memory`가 아니라 `bias-tunable basis source`로 다루는 것이 맞다


아래 슬라이드들이 이번 방향과 직접적으로 연결된다.

- `slide 6`: `Vd dependency`, `Slope adjustment`
- `slide 10`: `Back-Gate Effect on Reversal Voltage Biasing`
- `slide 11`: `Split-Gate Dual-Gate TFT with Reversal Voltage Biasing`, `Shape-tunable characteristics`
- `slide 26~28`: `SG1/SG2/SG3 sweep`, `SG step`
- `slide 31~32`: `MoS2`, `Vd = 0.5 ~ 5 V`, `Back-gate step`, `Sigma Ctrl`

즉 PPT의 소자들은 공통적으로 아래 구조를 가진다.

- 하나의 gate는 sweep input으로 사용
- 다른 단자(`Vd`, `BG`, `SG2`, `SG3`)는 curve shape를 바꾸는 control knob 역할
- control knob를 step으로 바꾸면 서로 다른 `Vg-Id` curve family가 생성됨

이 구조는 KAN에서 각 curve를 basis function으로 사용하는 방향과 잘 맞는다.


## 이번 1차 simulation의 개념적 매핑

### KAN 쪽 변수 매핑

- 입력축 `x`
  - sweep하는 gate voltage
  - 예: `VG1`, `VG2`, `VSG2`

- basis function `phi_i(x)`
  - 특정 control bias 조건에서 측정한 `Id(x)` curve
  - 예: `Vd = 0.1 V`, `0.2 V`, `0.5 V`
  - 예: `BG = -10 V`, `0 V`, `10 V`
  - 예: `SG2 = 20 V`, `30 V`, `40 V`

- coefficient `c_i`
  - simulation에서 학습되는 값
  - 처음 단계에서는 소자에서 직접 뽑지 않고 모델이 학습

즉 이번 방향은:

`실측 curve = basis`

`학습 coefficient = 각 basis의 기여도`


## 첫 번째로 추천하는 실험 구조

처음부터 `Vd`, `BG`, `SG2`, `SG3`를 다 섞지 않는다.
첫 번째 simulation은 반드시 아래처럼 간단하게 시작한다.

### 원칙

- sweep variable은 1개만
- control variable도 1개만
- 나머지 bias는 모두 고정

### 추천 시작안 A

- sweep variable: `Gate 1` 또는 `Split-Gate 1`
- control variable: `Vd`

이 경우 얻는 데이터:

- `Id(VG1 | Vd = a)`
- `Id(VG1 | Vd = b)`
- `Id(VG1 | Vd = c)`

### 추천 시작안 B

- sweep variable: `Gate 1` 또는 `Split-Gate 2`
- control variable: `Back-gate`

이 경우 얻는 데이터:

- `Id(VG1 | VBG = a)`
- `Id(VG1 | VBG = b)`
- `Id(VG1 | VBG = c)`

### 추천 시작안 C

- sweep variable: `SG1`
- control variable: `SG2` 또는 `SG3`

이 경우 얻는 데이터:

- `Id(VSG1 | VSG2 = a)`
- `Id(VSG1 | VSG2 = b)`
- `Id(VSG1 | VSG2 = c)`


## 가장 먼저 고를 control variable

아래 기준으로 하나를 먼저 고른다.

### 1순위 기준

- 곡선 shape 변화가 가장 크다
- 반복 측정 시 재현성이 좋다
- 측정이 간단하다
- 해석이 쉽다

### 추천 우선순위

1. `Vd`
2. `Back-gate`
3. `추가 split-gate`

이 우선순위는 측정 난이도와 해석 단순성을 기준으로 한 것이다.
실제 소자에서 `BG` 또는 `SG2`가 훨씬 더 큰 shape tuning을 만든다면 그 변수를 먼저 써도 된다.


## 1차 측정 계획

### Step 1. Sweep gate 결정

PPT의 소자 구조상 보통 아래 둘 중 하나가 적절하다.

- `SG1 sweep`
- `SG2 sweep`

처음에는 shape 변화가 가장 명확하게 보이는 축 하나만 선택한다.

### Step 2. Control bias 결정

예시:

- `Vd = 0.1, 0.3, 0.5, 1.0, 2.0 V`
- `VBG = -10, -5, 0, 5, 10 V`
- `VSG2 = 20, 30, 40, 50 V`

처음에는 5개 정도 조건이면 충분하다.

### Step 3. 나머지 변수 고정

- source grounding 여부 고정
- sweep range 고정
- sweep speed 고정
- sweep direction 고정
- temperature / light condition 고정

### Step 4. 반복 측정

같은 control bias 조건에서 3회 정도 반복 측정한다.

목적:

- 재현성 확인
- 평균 curve와 분산 확인
- basis 후보 제외 기준 설정


## 측정 데이터 정리 형식

simulation 입력 파일은 아래처럼 준비한다.

```csv
Vg,Id_ctrl_1,Id_ctrl_2,Id_ctrl_3,Id_ctrl_4,Id_ctrl_5
-20,...
-19.5,...
...
40,...
```

실제 컬럼명은 이렇게 쓰는 것이 더 좋다.

### Vd 기반 예시

```csv
Vg,Id_Vd_0p1,Id_Vd_0p3,Id_Vd_0p5,Id_Vd_1p0,Id_Vd_2p0
```

### Back-gate 기반 예시

```csv
Vg,Id_BG_m10,Id_BG_m5,Id_BG_0,Id_BG_5,Id_BG_10
```

### Split-gate 기반 예시

```csv
Vg,Id_SG2_20,Id_SG2_30,Id_SG2_40,Id_SG2_50
```


## Code 실행 계획

사용 파일:

- [GKAN_VGID_DeviceBasis_Simulation.py](C:\Users\jueun\OneDrive\바탕 화면\260327_랩미팅\GKAN code\Main Code Files\Main Codes\GKAN_VGID_DeviceBasis_Simulation.py)

해야 할 수정:

- `data_path`
- `vg_col`
- `curve_columns`

예시:

```python
data_path = "split_gate_vd_basis.csv"
vg_col = "Vg"
curve_columns = [
    "Id_Vd_0p1",
    "Id_Vd_0p3",
    "Id_Vd_0p5",
    "Id_Vd_1p0",
    "Id_Vd_2p0",
]
```

이렇게 하면 각 control bias 조건에서 측정한 curve가 basis bank로 자동 등록된다.


## 1차 simulation에서 확인할 것

### 1. Basis plot 확인

- curve들이 서로 충분히 다른가
- 단순 amplitude scaling만 있는가
- threshold / slope / width / peak position 변화가 보이는가

### 2. Fitting 결과 확인

- 단순 target function을 basis 조합으로 근사할 수 있는가
- loss가 안정적으로 감소하는가
- 특정 basis 몇 개만 과도하게 쓰이지 않는가

### 3. Coefficient 확인

- 모든 basis가 완전히 redundant하지 않은가
- 효과적인 basis 수가 몇 개인가


## 1차 판정 기준

### basis source로 유망한 경우

- control bias에 따라 curve shape가 의미 있게 달라진다
- 그 차이가 반복 측정 분산보다 충분히 크다
- target fitting이 어느 정도 된다
- basis 몇 개의 조합만으로도 다양한 출력이 나온다

### basis source로 약한 경우

- curve들이 거의 같은 shape의 단순 배수다
- control bias에 따른 차이가 매우 작다
- 반복 측정 분산이 조건 간 차이보다 크다
- fitting에서 사실상 한두 basis만 의미 있게 사용된다


## 2차 확장 계획

1차가 잘 되면 아래 순서로 확장한다.

1. 같은 축에서 control bias 개수 증가
2. 두 번째 control variable 추가
3. 실제 target function으로 대체
4. 단일 layer가 아닌 더 큰 network 구조 검토

예:

- 1차: `SG1 sweep + Vd step`
- 2차: `SG1 sweep + BG step`
- 3차: `SG1 sweep + (Vd, BG)` 비교
- 4차: `SG1`, `SG2`, `SG3` 각각의 basis family 비교


## 바로 실행 가능한 실무 To-do

### 이번 주 1차 목표

1. PPT 소자 중 하나를 고른다
2. sweep gate 1개를 정한다
3. control bias 1개를 정한다
4. control condition 5개를 정한다
5. `Vg-Id` curve family를 CSV로 만든다
6. basis simulation 1회 실행한다

### 가장 추천하는 시작점

PPT 내용상 가장 해석이 쉬운 시작점은 아래 둘 중 하나다.

1. `split-gate sweep + Vd step`
2. `split-gate sweep + back-gate step`

이 두 경우는 PPT에서 이미 shape tuning 근거가 직접 보이고,
basis-family 관점으로 연결하기 쉽다.


## 결론

이번 소자군은 memory weight device로 접근하기보다,
`bias-controlled transfer curve family`를 basis bank로 쓰는 방향이 적절하다.

따라서 첫 simulation은 아래 구조로 진행한다.

- 입력축: 하나의 sweep gate voltage
- basis 생성축: `Vd` 또는 `BG` 또는 추가 gate bias
- 출력: `Id`

즉, control bias를 바꿔 얻은 여러 개의 `Vg-Id` curve를 basis function 집합으로 사용하고,
그 basis들의 조합으로 target function을 재구성하는지 확인하면 된다.
