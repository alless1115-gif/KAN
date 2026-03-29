# AAT Device Measurement and CSV Setup for KAN Simulation

## 목적

이 문서는 PPT 안의 두 가지 AAT 계열 소자군에 대해,
KAN simulation을 위한 1차 측정 계획과 CSV 정리 방식을 분리해서 정리한 문서다.

대상 소자군:

- `NTC-AAT transistor` 계열
- `Advanced AAT based Gaussian` 계열


## 1. 공통 원칙

두 소자군 모두 메모리 상태를 저장하는 방향보다,
`bias-tunable transfer curve family`를 basis bank로 쓰는 방향이 적절하다.

즉 simulation 관점의 공통 구조는 아래와 같다.

- 입력축 `x`: sweep하는 gate voltage
- basis curve `phi_i(x)`: 특정 bias 조건에서 측정한 `Id(x)` curve
- coefficient: simulation에서 학습되는 값

처음에는 반드시 아래처럼 단순하게 간다.

- sweep variable 1개만 선택
- control variable 1개만 선택
- 나머지 bias는 고정


## 2. NTC-AAT transistor 계열

### PPT 대응 구간

- 슬라이드 15~22

### PPT에서 읽히는 핵심 변수

- `LS`, `RS` split gate
- `BG step`
- `Drain bias`
- `Forward / Reverse sweep`

### 추천 1차 측정 구조

#### 추천안 A

- sweep variable: `LS sweep`
- control variable: `BG step`
- 고정 변수: `RS`, `Vd`, sweep range, sweep speed

#### 추천안 B

- sweep variable: `RS sweep`
- control variable: `BG step`
- 고정 변수: `LS`, `Vd`, sweep range, sweep speed

#### 추천안 C

- sweep variable: `LS` 또는 `RS`
- control variable: `Vd step`
- 고정 변수: `BG`, 반대쪽 split gate

### 가장 추천하는 시작점

- `LS sweep + BG step`

이유:

- PPT에서 `BG step`과 `NTC / AAT transition`의 관계가 직접적으로 보인다
- basis family를 만들기 쉽다
- 나중에 `RS modulation`과도 비교하기 좋다

### NTC-AAT 1차 측정 과정

1. 측정할 소자 하나를 고른다
2. sweep gate를 `LS` 또는 `RS` 중 하나로 정한다
3. control variable을 `BG`로 정한다
4. `BG`를 5개 정도 값으로 정한다
5. 각 `BG` 조건에서 같은 sweep range로 `Id-Vsweep`를 측정한다
6. 같은 조건을 2~3회 반복한다
7. raw CSV와 condition log를 같이 정리한다

### NTC-AAT 측정 변수 예시

- `sweep terminal`: `LS`
- `read terminal`: `ID`
- `control variable`: `BG`
- `BG values`: `30, 20, 10, 0, -10 V`
- `fixed RS bias`: `0 V`
- `fixed Vd`: `10 V` 또는 `30 V`
- `sweep range`: `10 V -> -50 V` 또는 실제 실험 범위

### NTC-AAT simulation용 basis naming 예시

- `Id_BG_30`
- `Id_BG_20`
- `Id_BG_10`
- `Id_BG_0`
- `Id_BG_m10`


## 3. Advanced AAT based Gaussian 계열

### PPT 대응 구간

- 슬라이드 23~28

### PPT에서 읽히는 핵심 변수

- `SG1 sweep`
- `SG2 sweep`
- `SG3 sweep`
- `SG step`
- `n mode / p mode AAT`

### 추천 1차 측정 구조

#### 추천안 A

- sweep variable: `SG1`
- control variable: `SG2 step`
- 고정 변수: `SG3`, `Vd`, sweep range

#### 추천안 B

- sweep variable: `SG2`
- control variable: `SG3 step`
- 고정 변수: `SG1`, `Vd`, sweep range

#### 추천안 C

- sweep variable: `SG3`
- control variable: `SG2 step`
- 고정 변수: `SG1`, `Vd`, sweep range

### 가장 추천하는 시작점

- `SG1 sweep + SG2 step`

이유:

- PPT 슬라이드 26에서 바로 대응되는 구조가 보인다
- `SG1`을 입력축으로 놓고 `SG2`가 basis shape control knob가 되는지 보기 쉽다
- 후속으로 `SG3 step`과 비교하기 좋다

### Advanced AAT 1차 측정 과정

1. 측정할 소자 하나를 고른다
2. sweep gate를 `SG1`로 정한다
3. control variable을 `SG2`로 정한다
4. `SG2` bias를 4~5개 값으로 정한다
5. 각 `SG2` 조건에서 같은 `SG1` sweep range로 `Id-SG1`를 측정한다
6. 같은 조건을 2~3회 반복한다
7. raw CSV와 condition log를 같이 정리한다

### Advanced AAT 측정 변수 예시

- `sweep terminal`: `SG1`
- `read terminal`: `ID`
- `control variable`: `SG2`
- `SG2 values`: `20, 30, 40, 50 V`
- `fixed SG3 bias`: `40 V`
- `fixed Vd`: `5 V`
- `sweep range`: `-20 V -> 40 V` 또는 실제 실험 범위

### Advanced AAT simulation용 basis naming 예시

- `Id_SG2_20`
- `Id_SG2_30`
- `Id_SG2_40`
- `Id_SG2_50`


## 4. CSV는 두 종류로 관리하는 것이 좋다

각 소자군마다 CSV는 두 종류를 추천한다.

### 1) Condition log CSV

용도:

- B1500에서 몇 번째 데이터가 어떤 조건인지 기록
- raw CSV와 simulation 컬럼명을 연결

### 2) Simulation input CSV

용도:

- 실제 시뮬레이션 코드에 넣는 basis bank 파일
- 첫 열은 sweep axis
- 나머지 열은 basis curves


## 5. 파일 구성

이번에 함께 만든 파일:

- [NTC_AAT_B1500_Condition_Log_Template.csv](C:\Users\jueun\OneDrive\바탕 화면\260327_랩미팅\GKAN code\Main Code Files\Main Codes\NTC_AAT_B1500_Condition_Log_Template.csv)
- [NTC_AAT_Simulation_Input_Template.csv](C:\Users\jueun\OneDrive\바탕 화면\260327_랩미팅\GKAN code\Main Code Files\Main Codes\NTC_AAT_Simulation_Input_Template.csv)
- [Advanced_AAT_B1500_Condition_Log_Template.csv](C:\Users\jueun\OneDrive\바탕 화면\260327_랩미팅\GKAN code\Main Code Files\Main Codes\Advanced_AAT_B1500_Condition_Log_Template.csv)
- [Advanced_AAT_Simulation_Input_Template.csv](C:\Users\jueun\OneDrive\바탕 화면\260327_랩미팅\GKAN code\Main Code Files\Main Codes\Advanced_AAT_Simulation_Input_Template.csv)


## 6. 코드 연결 방법

시뮬레이션용 CSV가 준비되면
[GKAN_VGID_DeviceBasis_Simulation.py](C:\Users\jueun\OneDrive\바탕 화면\260327_랩미팅\GKAN code\Main Code Files\Main Codes\GKAN_VGID_DeviceBasis_Simulation.py)
에서 아래만 바꾸면 된다.

### NTC-AAT 예시

```python
data_path = "NTC_AAT_Simulation_Input.csv"
vg_col = "LS"
curve_columns = [
    "Id_BG_30",
    "Id_BG_20",
    "Id_BG_10",
    "Id_BG_0",
    "Id_BG_m10",
]
```

### Advanced AAT 예시

```python
data_path = "Advanced_AAT_Simulation_Input.csv"
vg_col = "SG1"
curve_columns = [
    "Id_SG2_20",
    "Id_SG2_30",
    "Id_SG2_40",
    "Id_SG2_50",
]
```


## 7. 바로 실행할 때 권장 순서

1. NTC-AAT와 Advanced AAT를 섞지 않는다
2. 각 소자군마다 별도 CSV를 만든다
3. 각 소자군에서 control variable은 하나만 먼저 쓴다
4. basis bank를 만든 뒤 각각 따로 simulation을 돌린다
5. 결과를 비교해서 어느 소자군이 basis source로 더 유리한지 판단한다


## 8. 한 줄 요약

- `NTC-AAT`: `LS/RS sweep + BG step` 구조로 시작
- `Advanced AAT`: `SG1 sweep + SG2 step` 구조로 시작
- 두 소자군 모두 `condition log CSV`와 `simulation input CSV`를 분리해서 관리
