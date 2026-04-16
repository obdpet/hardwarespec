# Carpybara — Frozen AVL Rev E (Approved Vendor List)

> Frozen AVL이란: 양산에 사용할 모든 부품의 **정확한 제조사, 모델번호, 패키지, 대체품**을 확정한 문서.  
> 이 문서에 없는 부품으로 교체하려면 반드시 HW owner의 서면 승인이 필요합니다.  
> **EMS(공장)에 이 문서를 BOM과 함께 제출합니다.**

**Rev E 주요 변경**: 캐리어보드 v3.1 설계 반영 (2-stage 전원 아키텍처, AEC-Q100 보호 소자 완전 명시, 프로토타입 전원 오기재 수정).  
**관련 문서**: 프로토타입 캐리어보드 v3.1, SW v1.1

---

## 문서 규칙

- **Status** 표기: CONFIRMED = 확정 / PENDING = 직접 확인 후 확정 예정 / TBD = 후보만 있고 미결정
- **Scope** 표기: **[P]** 프로토타입 전용 / **[M]** 양산 전용 / **[PM]** 프로토+양산 공용
- **1st Source**: 기본 사용 부품 (설계/검증 기준)
- **2nd Source**: 1st Source 수급 불가 시 사용할 대체 부품 (핀/패키지 호환 필수)
- **AEC-Q 등급**: Q100 (IC), Q101 (이산 반도체), Q200 (수동부품), — (해당 없음/Industrial grade)

---

## 1. 메인 컨트롤러 [PM]

| 항목 | 내용 |
|------|------|
| **기능** | MCU + Wi-Fi + Bluetooth Classic + BLE |
| **Status** | CONFIRMED |

| 구분 | 제조사 | Part Number | 패키지 | Flash | PSRAM | AEC-Q | 비고 |
|---|---|---|---|---|---|---|---|
| **1st Source (프로토+양산)** | Espressif | ESP32-WROVER-E-N16R8 | Module (18×25.5mm) | 16MB | 8MB | — | FCC ID: 2AC7Z-WROVER. PSRAM으로 풀스크린 스프라이트 가능 |
| **2nd Source (양산 다운그레이드)** | Espressif | ESP32-WROOM-32E-N16 | Module (18×25.5mm) | 16MB | 없음 | — | WROVER와 핀 호환. Partial Update만 사용 시 다운그레이드 가능 ($1~2 절감) |

> **프로토타입**: WROVER DevKit 소켓(38핀) 방식 (탈착식). 모듈 직접 납땜 아님.  
> **양산**: ESP32-WROVER-E 모듈 직접 SMT 납땜.

---

## 2. USB-UART 브릿지 [P] 

> 프로토타입은 DevKit에 내장. 양산 PCB는 DevKit 없이 직접 실장 필요.

| 구분 | 제조사 | Part Number | 패키지 | AEC-Q | 비고 |
|---|---|---|---|---|---|
| **1st Source** | WCH (Nanjing Qinheng) | CH340C | SOP-16 | — | USB 2.0, 외부 크리스탈 불필요 |
| **2nd Source** | WCH | CH340N | SOP-8 | — | 핀 수 적음, 풋프린트 다름 — 회로도 수정 필요 |

---

## 3. 디스플레이 [PM]

| 항목 | 내용 |
|------|------|
| **기능** | 캐릭터 애니메이션 출력 |
| **Status** | PENDING — 스펙 확정, 공급업체 최종 선정 필요 |

**확정 스펙**: 2.8" IPS TFT / 드라이버 ILI9341 / 해상도 240×320 / 4-wire SPI / 동작 온도 **-20~70°C**  
**⚠️ Known Risk**: 70°C 상한은 여름 직사광선 주차 시 초과 가능. 캐리어보드 v3.1의 4단계 온도 보호가 65°C에서 Safe 모드 진입하여 보호. 사용자 매뉴얼에 dashmat 사용 권장 명시 필요. (v3.1 Known Risk R6 참조)

| 구분 | 제조사/공급처 | Part Number | 크기 | 드라이버 | 인터페이스 | 동작 온도 | 비고 |
|---|---|---|---|---|---|---|---|
| **1st Source (후보)** | BuyDisplay | ER-TFT028A2-4 | 2.8" IPS | ILI9341 | SPI | -20~70°C | IPS 데이터시트 명기. Mouser/Digi-Key 유통 |
| **2nd Source (후보)** | LCDWiki | 2.8" IPS ILI9341 | 2.8" IPS | ILI9341 | SPI | -20~70°C | MSP2807 IPS variant. FPC 핀 호환 확인 필요 |
| **비상 조달** | WINSTAR | WF28ETLAJDNN0 | 2.8" IPS | ILI9341V | SPI | -20~70°C | 산업용 500cd/m². 소량 긴급 조달용 |

**온도 보호 로직 (캐리어보드 v3.1 기준)**:
| 단계 | NTC 온도 | 백라이트 | FPS | 히스테리시스 |
|---|---|---|---|---|
| Normal | < 50°C | 100% | 60 | — |
| Warn | 50~55°C | 70% | 60 | 5°C × 5초 |
| Throttle | 55~60°C | 40% | 30 | 5°C × 5초 |
| Degraded | 60~65°C | 20% | 15 | 5°C × 5초 |
| Safe | ≥ 65°C (3초) | OFF | Sleep | < 55°C × 5초 |

> ~~"70°C 초과 시 백라이트 차단" (AVL Rev D의 이진 ON/OFF 기술)~~ → **4단계 상태 머신으로 업데이트** (v3.1)

---

## 4. OBD2 통신 — CAN 직결 [M]

> **프로토타입은 ELM327 BT(Veepeak VP11)로 OBD2 통신.** CAN 직결은 양산 PCB에만 탑재.

| 구분 | 제조사 | Part Number | 패키지 | AEC-Q | 비고 |
|---|---|---|---|---|---|
| **CAN 트랜시버 1st** | NXP | TJA1051T/3 | SOIC-8 | Q100 | 3.3V I/O. TXD Dominant Timeout 내장 — ESP32 크래시 시 CAN 버스 블로킹 자동 방지 |
| **CAN 트랜시버 2nd** | TI | SN65HVD230DR | SOIC-8 | Q100 | 3.3V 네이티브. Dominant Timeout 없음 — 외부 보호 회로 필요 |
| **CAN TVS 다이오드** | NXP | PESD1CAN | SOT-23 | Q101 | CAN 라인 ESD 보호 (~$0.10) |
| **CAN 종단 저항** | — | 120Ω 0402 | 0402 | Q200 | CANH-CANL 사이. 솔더 브릿지로 on/off 가능 |

---

## 5. 전원 아키텍처 — 2-Stage Buck (v3.1 기준)

> ~~"프로토타입 = DevKit 내장 USB-C + LDO / 양산 = OBD2 12V 단일 Buck"~~ (AVL Rev D의 기술 오류)  
> **v2.4부터 프로토타입도 OBD2 12V + USB-C dual input 구조 채택. v3.1에서 2-stage Buck으로 재설계.**

### 5-1. 전체 아키텍처 [PM]

```
OBD2 12V ──→ [보호 회로] ──→ [Stage 1 Buck] ──→ 5V ──→ [Power MUX] ──→ 5V_COMMON
                                                              ↑
USB-C 5V ──→ [Schottky] ───────────────────────────→ 5V_USB ──┘
                                                                   │
                                                                   ↓
                                               [Stage 2 Buck MP2315] → 3.3V_PERIPH
```

### 5-2. Stage 1 Buck (12V → 5V) [PM]

| 항목 | 내용 |
|---|---|
| **기능** | OBD2 12V → 5V, Load Dump 65V(ISO 7637) 내성, 크랭킹 UVLO 4.3V |
| **Status** | CONFIRMED (v3.1에서 확정) |

| 구분 | 제조사 | Part Number | 패키지 | Vin | Iout | UVLO | AEC-Q | LCSC |
|---|---|---|---|---|---|---|---|---|
| **1st Source** | TI | **TPS54360BQDDARQ1** | **HSOP-8 PowerPAD (DDA)** | 4.5~60V | 3.5A | 4.3V | **Q100** | C100891 |
| **2nd Source** | TI | TPS54341QDRCRQ1 | VSON-10 | 4.5~42V | 3.5A | 4.3V | Q100 | (확인 필요) |

> **2nd Source 주의**: TPS54341-Q1은 42V max로 TPS54360B의 60V보다 낮으나, SMBJ20CA 2차 TVS 클램프 32V 기준 여유 10V로 Load Dump 대응 가능. Footprint는 VSON-10으로 HSOP-8과 다르므로 PCB 재설계 필요.

**Stage 1 관련 수동부품** [PM]:
| 부품 | 값 | 허용오차 | 패키지 | 용도 | LCSC |
|---|---|---|---|---|---|
| FB R_top | 100kΩ | **0.1%** | 0402 | Vout=4.99V 설정 | C97521 |
| FB R_bot | 19.1kΩ | **0.1%** | 0402 | Vout=4.99V 설정 | C160111 |
| Rt | 240kΩ | 1% | 0402 | Fsw=500kHz 설정 | C25812 |
| EN pull-up | 100kΩ | 1% | 0402 | EN pin to VIN | C25787 |
| L1 인덕터 | **15µH** | 2A 차폐 | 5×5mm | Stage 1 | C128268 |
| Cout | 22µF/16V MLCC × 2 | X7R | 0805 | Stage 1 출력 | C45783 |
| Cout HF | 100nF + 10nF | X7R | 0402 | Stage 1 고주파 | C1525, C1567 |

### 5-3. Stage 2 Buck (5V → 3.3V) [PM]

| 항목 | 내용 |
|---|---|
| **기능** | 5V → 3.3V_PERIPH (LCD, 센서, 엔코더 등 주변장치용) |
| **Status** | CONFIRMED |

| 구분 | 제조사 | Part Number | 패키지 | Vin | Iout | Vref | AEC-Q | LCSC |
|---|---|---|---|---|---|---|---|---|
| **1st Source** | MPS | MP2315GJ-Z | TSOT23-8 | 4.5~24V | 1.5A | 0.8V | — (Industrial) | C15425 |
| **2nd Source** | TI | TLV62569DBVR | SOT-23-5 | 2.5~5.5V | 2A | 0.6V | — | C129255 |

**Stage 2 관련 수동부품** [PM]:
| 부품 | 값 | 허용오차 | 패키지 | 용도 | LCSC |
|---|---|---|---|---|---|
| **FB R_top** | **100kΩ** | **0.1%** | 0402 | **Vout=3.27V 정확 설정 (v3.0 재계산)** | C97521 |
| **FB R_bot** | **32.4kΩ** | **0.1%** | 0402 | **Vout=3.27V 정확 설정 (v3.0 재계산)** | C131087 |
| L2 인덕터 | 4.7µH | 2A 차폐 | 4×4mm | Stage 2 | C167229 |
| Cin | 22µF/10V MLCC | X7R | 0805 | Stage 2 입력 | C159801 |
| Cout | 22µF/10V + 10µF/10V MLCC | X7R | 0805 | Stage 2 출력 | C159801, C19702 |
| Cout HF | 100nF | X7R | 0402 | 고주파 바이패스 | C1525 |

> ~~AMS1117-3.3 (LDO)~~ — AVL Rev A에서 제거됨 (차량 고온 환경 발열).  
> ~~v3.0의 R_bot=22kΩ (4.44V 출력 오류)~~ — v3.0 자가검증에서 발견, **32.4kΩ 0.1%로 수정 완료**.

---

## 6. 입력 보호 회로 (v3.1 신규) [PM]

> v2.4까지 AVL에 누락됐던 차량 환경 보호 소자들을 v3.1 기준으로 전부 명시.

### 6-1. OBD2 12V 입력 보호 체인

보호 체인 순서: **PPTC/eFuse → TVS 1차 (SMC) → ESD → Ferrite → TVS 2차 (SMB) → Ideal Diode → Stage 1 Buck**

| 구분 | 제조사 | Part Number | 패키지 | 기능 | AEC-Q | LCSC |
|---|---|---|---|---|---|---|
| **eFuse** | TI | TPS25221-Q1 | SOT-23-6 | 과전류 100µs 차단 (1A 설정) | Q100 | C2987547 |
| **TVS 1차** | Littelfuse | SMCJ24CA | **SMC** | Load Dump 1차 클램프 (3kW, 양방향, Standoff 24V, Clamp 39V @ 1A) | Q101 | C92832 |
| **ESD 보호** | NXP | PESD2CAN-LX | SOT-23 | <1ns ESD 응답, ±15kV | Q101 | C35783 |
| **Ferrite** | Murata | BLM31PG601SN1 | 1206 | 600Ω@100MHz, 3A, 고주파 감쇄 | Q200 | C105160 |
| **TVS 2차** | Littelfuse | SMBJ20CA | **SMB** | 2차 클램프 (600W, 양방향, Standoff 20V, Clamp 32V @ 1A) — Stage 1 Abs Max 65V 보호 | Q101 | C24342 |
| **Ideal Diode Ctrl** | TI | LM74700-Q1 | SOT-23-6 | 역극성 2중 보호, 이상적 다이오드 (VF ~0V) | Q100 | C2662568 |
| **Pass PMOS** | Vishay | SQJ422EP-T1_GE3 | PowerPAK SO-8 | LM74700 쌍 (65V, 26A, Low Rds(on)) | Q101 | C197495 |

### 6-2. USB-C 5V 입력 보호 [P]

양산형은 USB-C 없음 (플래싱 전용 테스트 포트만 예정).

| 구분 | 제조사 | Part Number | 패키지 | 기능 | AEC-Q | LCSC |
|---|---|---|---|---|---|---|
| **Schottky** | Diodes Inc. | SBR2U60P1 | SOD-123 | VF 0.25V @ 500mA, 저손실 | — | C236404 |
| **TVS USB VBUS** | Littelfuse | SMBJ5.0CA | **SMB** | USB VBUS ESD/Surge (600W, 양방향, Clamp 9.2V) | — | C24283 |
| **Ferrite GND** | Murata | BLM18PG471SN1 | 0603 | USB GND Loop 차단 (470Ω@100MHz) | — | C105161 |

### 6-3. USB-C 데이터 라인 ESD [P]

| 구분 | 제조사 | Part Number | 패키지 | 비고 |
|---|---|---|---|---|
| **1st Source** | STMicroelectronics | USBLC6-2SC6 | SOT-23-6 | USB 2.0 D+/D- TVS 다이오드 (DevKit 내장) |

### 6-4. TVS 패키지 선정 원칙 (v3.1 신규 명시)

**"전부 SMC 확정"이 아니라 위치별 에너지에 맞춰 선정**:

| 위치 | 패키지 | 에너지 | 이유 |
|---|---|---|---|
| OBD2 1차 TVS (SMCJ24CA) | **SMC 필수** | 3kW peak | Load Dump 10~50J 흡수, SMB(600W)로는 부족 |
| OBD2 2차 TVS (SMBJ20CA) | SMB 적정 | 600W peak | 1차 통과분만 받음, SMB로 충분 (SMC는 오버스펙) |
| USB VBUS TVS (SMBJ5.0CA) | SMB 적정 | 600W peak | 5V 라인 ESD용, SMC는 보드 공간 낭비 |

---

## 7. Power MUX [PM]

| 항목 | 내용 |
|---|---|
| **기능** | USB 5V / OBD2 Stage1 5V 중 높은 쪽 자동 선택 (OBD2 우선) |
| **Status** | CONFIRMED (v3.1) |

| 구분 | 제조사 | Part Number | 패키지 | Vin | AEC-Q | LCSC |
|---|---|---|---|---|---|---|
| **1st Source** | TI | LM66100 | SOT-23-6 | 1.5~5.5V | — | C1009826 |
| **2nd Source** | TI | TPS2116 | SOT-23-6 | 1.6~5.5V | — | C2686097 |

---

## 8. USB-C 커넥터 + 관련 부품 [P]

| 구분 | 제조사 | Part Number | 비고 |
|---|---|---|---|
| **USB-C 리셉터클** | GCT | USB4105-GF-A | 16핀 SMD, USB 2.0, 기구 고정탭, 5A, -40~85°C, JLCPCB C3020560 |

**USB-C 필수 회로** [P]:
| 부품 | 값 | 용도 |
|---|---|---|
| CC1 풀다운 저항 | 5.1kΩ | USB-C 전원 수신 식별 (필수) |
| CC2 풀다운 저항 | 5.1kΩ | USB-C 전원 수신 식별 (필수) |
| 입력 벌크캡 | 100µF / 10V | 인러시/케이블 탈착 서지 흡수 |

---

## 9. OBD2 입력 커넥터

### 9-1. 프로토타입 [P]

| 항목 | 사양 |
|---|---|
| PCB측 커넥터 | **JST B3B-XH-A** (3핀, 2.5mm 피치) |
| 상대 커넥터 | JST XHP-3 하우징 + SXH-001T-P0.6 컨택 |
| 케이블 | AWG22 3C, 500mm (12V / GND / Shield) |
| 핀 할당 | 1=12V, 2=GND, 3=Shield (케이스 프레임) |
| LCSC | C145992 |

### 9-2. 양산 [M]

| 항목 | 사양 |
|---|---|
| 커넥터 | **SAE J1962 Type A Male** (16핀 OBD2 표준) |
| 사용 핀 | 4(GND), 5(Sig GND), 6(CANH), 14(CANL), 16(+12V) |
| 공급처 | TBD (DVT 전 확정 필요) |
| PCB측 케이블 커넥터 | TBD — JST 또는 Molex 확정 필요 |

---

## 10. 로터리 엔코더 [PM]

| 구분 | 제조사 | Part Number | 수명 | 비고 |
|---|---|---|---|---|
| **1st Source** | Alps Alpine | EC11E18244A5 | 30,000 사이클 | 수직 실장, 푸시 버튼 통합. Digi-Key 재고/단가 DVT 전 확인 |
| **2nd Source** | Bourns | PEC11R-4215F-S0024 | 30,000+ 사이클 | Digi-Key/Mouser 재고 풍부 |

**디바운싱/ESD 회로** [PM]:
| 부품 | 값 | 패키지 | 수량 | 용도 |
|---|---|---|---|---|
| 디바운스 캡 | 100nF | 0402 | 2 | 채널 A, B 각 1개 |
| 풀업 저항 | 10kΩ | 0402 | 3 | 채널 A, B, 스위치 각 1개 |
| ESD TVS | PESD5V0U2BT | SOT-23 | 1 | 채널 A/B 2선 보호 |

---

## 11. 사이드 버튼 [PM]

| 구분 | 제조사 | Part Number | 패키지 | 수량 |
|---|---|---|---|---|
| **1st Source** | — | 6×6mm SMD 택트 스위치 | 6×6mm | 2 | LCSC C318884 |
| **ESD 보호** | NXP | PESD5V0U2BT | SOT-23 | 1 | 2채널 동시 보호 |

---

## 12. 조도 센서 [PM]

| 구분 | 제조사 | Part Number | 인터페이스 | 패키지 | 비고 |
|---|---|---|---|---|---|
| **1st Source** | Vishay Semiconductor Opto | TEMT6000X01 | 아날로그 (ADC → GPIO15 v3.1 변경) | 1206 (3216 Metric) | Active, 10K 단가 ~$0.50 |
| **2nd Source** | Vishay | VEML7700 | I2C | 옵토 LGA | 회로/코드 변경 필요 |

> **v3.1 GPIO 변경**: GPIO34 (조도) → **GPIO15** (조도 ADC로 이동). GPIO34는 VIN 모니터로 재할당됨. SW v1.1에 반영.

---

## 13. NTC 온도 센서 [PM]

| 항목 | 내용 |
|---|---|
| **기능** | LCD 백면 부착 온도 감지, 4단계 온도 보호 입력 |
| **Status** | CONFIRMED |

| 구분 | 제조사 | Part Number | 패키지 | 저항 | 정확도 | LCSC |
|---|---|---|---|---|---|---|
| **1st Source** | Murata | NCP18XH103F03RB | 0402 | 10kΩ @ 25°C | ±1% | C77014 |

**설치 방식 (v3.1)**:
- 프로토타입: 플라잉 리드 (AWG30 TP + Shield, 50mm) + 3M 8810 열전도 테이프
- 양산: FPC 일체형 NTC (디스플레이 모듈 커넥터에 통합)
- 단선 감지: SW에서 ADC >95% or <5% 시 NTC_OPEN/SHORT 판정, safe fallback 진입

---

## 14. 피에조 버저 [PM]

| 구분 | 제조사 | Part Number | 구동 전압 | 크기 | 동작 온도 | LCSC |
|---|---|---|---|---|---|---|
| **1st Source** | FUET | FUET-9018 | 1~25V (정격 3V) | 9×9×2.1mm | -30~70°C | C391035 |
| **2nd Source** | TDK | PS1240P02BT | 3~30V | ø12.2×6.5mm | -10~70°C | C76871 (Through-hole) |

---

## 15. 백라이트 제어 MOSFET [PM]

| 구분 | 제조사 | Part Number | 패키지 | Vgs(th) | AEC-Q | LCSC |
|---|---|---|---|---|---|---|
| **1st Source** | AOS | AO3400A | SOT-23 | ~1.4V | — | C20917 |
| **2nd Source** | Nexperia | 2N7002 | SOT-23 | ~1.8V | Q101 | C8545 |

**관련 수동부품**:
| 부품 | 값 | 용도 |
|---|---|---|
| 게이트 직렬 저항 | 33Ω 0402 | 스위칭 에지 제어, EMI 감소 |
| 게이트 풀다운 | 100kΩ 0402 | ESP32 부팅 중 백라이트 OFF 유지 |
| 백라이트 벌크캡 | 4.7µF 0603 | PWM 리플 흡수 |

---

## 16. PCB [PM]

| 항목 | 프로토 (v3.1) | 양산 |
|---|---|---|
| **레이어** | 4 | 4 |
| **크기** | **75×90mm** (v3.0 70×90mm → +5mm HSOP-8 여유) | TBD (케이스 확정 후) |
| **두께** | 1.6mm 표준 | 1.6mm |
| **표면처리** | **ENIG 권장** (차량 고온 환경) | ENIG 필수 |
| **최소 선폭/간격** | 6/6 mil | 6/6 mil |
| **비아** | 0.3mm | 0.3mm |
| **동박** | 1oz 전체 | 1oz 전체 |
| **스택업** | **L1=Signal+부품 / L2=GND solid / L3=Power+GND pour / L4=Signal+GND pour** (v3.1 강화) | 동일 |

**v3.1 레이아웃 필수 규칙**:
- BT 안테나 Keep-out: **17×8mm, L1~L4 전부 구리 없음**, 안테나 boarder 5mm 이상 돌출
- Buck SW 트레이스 10mm 이내, 루프 5mm² 이내
- ADC 신호 (GPIO34 VIN 모니터, GPIO35 NTC): Buck SW 노드에서 10mm 이상 이격
- Stage1 Buck HSOP-8 Thermal pad: 6개 via로 L2 GND plane 연결
- USB-C D+/D- 차동쌍 길이 매칭 (±5mil)

---

## 17. 케이스 [PM]

| 항목 | 사양 |
|---|---|
| **소재** | PC-ABS 블렌드 (CONFIRMED, ABS 단독 사용 불가) |
| **설치 방식** | 대시보드 VHB 접착 (CONFIRMED, Trade Study #4) |
| **VHB 접착제** | **3M VHB GPH-060GF** (자동차 인테리어 인증, -40~149°C) |
| **색상** | TBD |
| **금형** | TBD (사출 금형 $3,000~$10,000 추정) |

---

## 18. 기구 조립 부품 [PM]

| 부품 | 모델 | 용도 |
|---|---|---|
| VHB 접착 테이프 | 3M VHB GPH-060GF | 케이스 → 대시보드 부착 |
| NTC 열전도 테이프 | 3M 8810 | NTC → LCD 백면 부착 |
| NTC 플라잉 리드 | AWG30 TP + Shield | 프로토 전용, 양산은 FPC 일체형 |
| M2 스탠드오프 | 나일론 5mm | DevKit 고정 (프로토 전용) |
| Loctite 222 | 나사 풀림 방지제 | 스탠드오프 고정 |

---

## AVL 변경 절차

```
부품 변경이 필요한 경우:

1. 변경 요청서 작성 (변경 이유, 대체 부품 스펙 비교)
2. HW owner 서면 승인
3. 대체 부품으로 기능 검증 (최소 5개 샘플)
4. AVL 문서 업데이트 + Revision 번호 변경
5. EMS에 업데이트된 AVL 전달

※ 승인 없는 부품 교체는 전량 반품 사유
```

---

## 변경 이력

| Rev | 날짜 | 변경 내용 |
|-----|------|---------|
| A | 2026-04-09 | 초판. LDO 제거, Buck 확정. 4레이어 PCB 확정. 송풍구 설치 확정 |
| B | 2026-04-10 | 디스플레이 2.5"→2.4" ILI9341 확정. USB-C USB4105-GF-A 변경. FUET-9018 확정. BOM 파일럿/양산 2-tier 추가 |
| C | 2026-04-14 | 거치 방식: 송풍구 클립 → 대시보드 VHB 접착. 디스플레이: 2.4" → 2.8" ILI9341 IPS. BuyDisplay ER-TFT028A2-4로 변경 |
| D | 2026-04-14 | MCU: WROOM → WROVER-E-N16R8 (PSRAM 8MB). OBD2 CAN 직결 부품 추가 (양산). 전원 섹션 프로토/양산 분리. 프로토타입 전략 명시 |
| **E** | **2026-04-15** | **캐리어보드 v3.1 반영. ① 전원 아키텍처 단일 MP2315 Buck → 2-stage Buck (Stage1 TPS54360BQDDARQ1 + Stage2 MP2315) 완전 재작성. ② 입력 보호 회로 섹션 신규 추가: TPS25221-Q1 (eFuse), SMCJ24CA (1차 TVS SMC), PESD2CAN-LX (ESD), BLM31PG601SN1 (Ferrite), SMBJ20CA (2차 TVS SMB), LM74700-Q1 + SQJ422EP PMOS (Ideal Diode). ③ Power MUX LM66100 섹션 신규. ④ 프로토타입 전원 기술 오류 정정: "DevKit USB-C + LDO 사용" → "v2.4부터 OBD2 12V + USB-C dual input". ⑤ 디스플레이 온도 보호 "이진 ON/OFF" → "4단계 상태 머신" 수정. ⑥ Stage 2 FB 저항 32.4kΩ 0.1% 명시 (v3.0의 22kΩ 4.44V 오류 수정본 반영). ⑦ TVS 패키지 선정 원칙 신규: "전부 SMC" 과잉 요구 대신 위치별 에너지 기준. ⑧ PCB 크기 60×80mm → 75×90mm (v3.1 기준). ⑨ Scope 표기 [P][M][PM] 신규 도입. ⑩ OBD2 양산 커넥터 SAE J1962 Type A Male 명시, PCB측 케이블 커넥터 TBD 표시. ⑪ 2nd source 전반 정비 (TPS54341-Q1, TPS2116 등 명시). ⑫ 각 IC에 AEC-Q 등급 컬럼 추가.** |

---

*본 문서는 PCB 발주 및 EMS 계약 시 BOM과 함께 제출하는 공식 부품 승인 목록입니다.*  
*Rev E 기준 — 캐리어보드 v3.1, SW v1.1과 동기됨.*
