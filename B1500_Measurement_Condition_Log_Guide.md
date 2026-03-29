# B1500 Measurement Condition Log Guide

파일:

- [B1500_Measurement_Condition_Log_Template.csv](C:\Users\jueun\OneDrive\바탕 화면\260327_랩미팅\GKAN code\Main Code Files\Main Codes\B1500_Measurement_Condition_Log_Template.csv)

이 파일은 B1500에서 측정한 CSV 데이터가
시뮬레이션에서 어떤 basis curve에 대응되는지 정리하기 위한 엑셀용 템플릿이다.


## 어떻게 쓰는지

엑셀에서 CSV를 열고, 측정할 때마다 한 줄씩 채우면 된다.

예를 들어:

- `1번째 데이터 = Vd = 0.1 V 조건`
- `2번째 데이터 = Vd = 0.3 V 조건`
- `3번째 데이터 = BG = -10 V 조건`

같은 식으로 기록한다.


## 주요 열 설명

### `measurement_index`

- 전체 측정 조건 번호
- 1, 2, 3, 4처럼 순서대로 부여

### `basis_family`

- 어떤 종류의 basis set인지 구분
- 예:
  - `Vd_step`
  - `BG_step`
  - `SG2_step`
  - `SG3_step`

### `basis_label`

- simulation에 넣을 basis 이름
- 예:
  - `Id_Vd_0p1`
  - `Id_BG_m10`
  - `Id_SG2_40`

### `raw_data_order`

- B1500 raw 파일 안에서 몇 번째 데이터인지
- 예: `1`, `2`, `3`

### `device_name`

- 어떤 소자인지 구분
- 예: `Device_A`, `MoS2_A`, `SplitGate_01`

### `sweep_terminal`

- sweep한 단자 이름
- 예:
  - `VG1`
  - `VG2`
  - `VSG1`
  - `VSG2`

### `read_terminal`

- 읽은 전류 이름
- 보통 `ID`

### `control_variable_1`, `control_value_1`

- basis shape를 바꾸는 주 제어 변수
- 예:
  - `Vd`, `0.1`
  - `BG`, `-10`
  - `SG2`, `40`

### `control_variable_2`, `control_value_2`

- 추가로 고정하거나 함께 관리할 두 번째 제어 변수
- 처음에는 비워두는 것을 권장

### `vd_value`, `vbg_value`, `sg2_value`, `sg3_value`

- 실제 bias 값을 별도 열로 적는 칸
- 나중에 필터링하거나 정렬할 때 편하다

### `sweep_start`, `sweep_stop`, `sweep_step`

- sweep 조건 기록
- 예:
  - `-20`, `40`, `0.5`

### `sweep_direction`

- `forward` 또는 `reverse`

### `repeat_index`

- 같은 조건 반복 측정 번호
- 예: `1`, `2`, `3`

### `raw_csv_file`

- B1500에서 저장한 원본 파일명

### `raw_csv_column`

- 원본 CSV 안에서 실제 데이터 컬럼명
- 예:
  - `Data001`
  - `Data002`
  - `Id`

### `simulation_column`

- 최종 simulation용 CSV에서 사용할 컬럼명
- 이 이름을 [GKAN_VGID_DeviceBasis_Simulation.py](C:\Users\jueun\OneDrive\바탕 화면\260327_랩미팅\GKAN code\Main Code Files\Main Codes\GKAN_VGID_DeviceBasis_Simulation.py)의 `curve_columns`에 넣으면 된다

### `use_for_basis`

- basis로 실제 사용할지 여부
- `Y` 또는 `N`

### `notes`

- hysteresis 큼, 노이즈 큼, 재현성 낮음 같은 메모 기록


## 가장 추천하는 사용 방식

### 1차 실험

- `basis_family`를 하나만 선택
- 예: `Vd_step`
- 이 경우 `Vd`만 바꾸고 나머지는 고정

### 2차 실험

- `BG_step` 또는 `SG2_step` 추가
- 첫 번째 결과와 비교


## simulation 연결 방법

최종적으로 `use_for_basis = Y`인 행만 골라서
simulation용 CSV를 만들면 된다.

예:

- `Vg`
- `Id_Vd_0p1`
- `Id_Vd_0p3`
- `Id_Vd_0p5`
- `Id_Vd_1p0`
- `Id_Vd_2p0`

그리고 코드에서:

```python
curve_columns = [
    "Id_Vd_0p1",
    "Id_Vd_0p3",
    "Id_Vd_0p5",
    "Id_Vd_1p0",
    "Id_Vd_2p0",
]
```

처럼 연결하면 된다.


## 한 줄 요약

이 템플릿은 `B1500에서 몇 번째로 측정한 데이터가 어떤 bias 조건이고, 그게 simulation에서 어떤 basis curve 이름으로 들어가는지`를 정리하기 위한 표다.
