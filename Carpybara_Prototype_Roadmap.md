# Carpybara — Frozen AVL (Approved Vendor List)

> Frozen AVL이란: 양산에 사용할 모든 부품의 **정확한 제조사, 모델번호, 패키지, 대체품**을 확정한 문서.  
> 이 문서에 없는 부품으로 교체하려면 반드시 HW owner의 서면 승인이 필요합니다.  
> **EMS(공장)에 이 문서를 BOM과 함께 제출합니다.**

---

## 문서 규칙

- **Status** 표기: CONFIRMED = 확정 / PENDING = 직접 확인 후 확정 예정 / TBD = 후보만 있고 미결정
- **Scope** 표기: **[P]** 프로토타입 전용 / **[M]** 양산 전용 / **[PM]** 프로토+양산 공용
- **1st Source**: 기본 사용 부품 (설계/검증 기준)
- **2nd Source**: 1st Source 수급 불가 시 사용할 대체 부품 (핀/패키지 호환 필수)
- **AEC-Q 등급**: Q100 (IC), Q101 (이산 반도체), Q200 (수동부품), — (해당 없음/Industrial grade)
- 2nd Source가 없는 부품은 **공급 리스크가 높음** → 최소 3개월분 완충 재고 확보

---

## 1. 메인 컨트롤러 [PM]

| 항목 | 내용 |
|------|------|
| **기능** | MCU + Wi-Fi + Bluetooth Classic + BLE |
| **Status** | CONFIRMED |

| 구분 | 제조사 | Part Number | 패키지 | Flash | PSRAM | AEC-Q | 비고 |
|------|--------|-------------|--------|-------|-------|-------|------|
| **1st Source (프로토+양산)** | Espressif | **ESP32-WROVER-E-N16R8** | Module (18×25.5mm) | 16MB | **8MB** | — | FCC ID: 2AC7Z-WROVER. PSRAM으로 풀스크린 스프라이트 가능 |
| **2nd Source (양산 다운그레이드 후보)** | Espressif | ESP32-WROOM-32E-N16 | Module (18×25.5mm) | 16MB | 없음 | — | WROVER와 핀 호환. Partial Update만 사용 시 WROOM으로 다운그레이드 가능 ($1~2 절감) |

> **프로토타입 전략 (Trade Study #1, Strategy A+)**: WROVER DevKit으로 개발 → 실제 메모리 사용량 측정 → PSRAM 불필요 시 양산에서 WROOM으로 다운그레이드. "오버스펙 프로토 → 데이터 기반 다운그레이드" 원칙.  
> **프로토타입 캐리어보드**: WROVER DevKit 소켓(38핀) 방식. 모듈 직접 납땜 아닌 탈착식.  
> **양산**: ESP32-WROVER-E 모듈 직접 SMT 납땜.

---

## 2. USB-UART 브릿지 [P]

| 항목 | 내용 |
|------|------|
| **기능** | USB-C를 통한 펌웨어 플래싱 + 시리얼 디버그 |
| **Status** | CONFIRMED |

> 프로토타입은 DevKit에 내장. 양산 PCB는 DevKit 없이 직접 실장 필요.

| 구분 | 제조사 | Part Number | 패키지 | AEC-Q | 비고 |
|------|--------|-------------|--------|-------|------|
| **1st Source** | WCH (Nanjing Qinheng) | CH340C | SOP-16 | — | USB 2.0, 외부 크리스탈 불필요 |
| **2nd Source** | WCH | CH340N | SOP-8 | — | 핀 수 적음, 풋프린트 다름 — 회로도 수정 필요 |

---

## 3. 디스플레이 [PM]

| 항목 | 내용 |
|------|------|
| **기능** | 캐릭터 애니메이션 출력 |
| **Status** | PENDING — 스펙 확정, 양산 공급업체 실측 검증 진행 중 |

**확정 스펙**: 2.8" IPS TFT / 드라이버 ILI9341 / 해상도 240×320 / 4-wire SPI / 동작 온도 -20~70°C  
- Trade Study #5에서 2.4" → 2.8"로 변경 확정 (거치 방식 변경에 따른 크기 제약 해제)
- 동작 온도 70°C 제한 → NTC 기반 4단계 SW 보호 로직으로 보완 (아래 표 참조)
- 제품 리스팅에 동작 온도 -20~70°C 명시 필수

**4단계 온도 보호 (NTC 기반)**:
| 단계 | NTC 온도 | 백라이트 | FPS | 히스테리시스 |
|------|---------|---------|-----|------------|
| Normal | < 50°C | 100% | 60 | — |
| Warn | 50~55°C | 70% | 60 | 5°C × 5초 |
| Throttle | 55~60°C | 40% | 30 | 5°C × 5초 |
| Degraded | 60~65°C | 20% | 15 | 5°C × 5초 |
| Safe | ≥ 65°C (3초) | OFF | Sleep | < 55°C × 5초 |

### 프로토타입 전용 [P]

| 구분 | 제조사/판매처 | Part Number | 폼팩터 | 드라이버 | 인터페이스 | 조달 경로 | 비고 |
|------|-------------|-------------|--------|---------|-----------|----------|------|
| **프로토 Display** | Hosyond (Amazon) | B0CMGF4Q68 | 브레이크아웃 모듈 (2.54mm 헤더 + FPC 확장 + microSD) | ILI9341V | 4-wire SPI | Amazon Prime (1-2일) | IPS 280 cd/m², 240×320, 용량형 터치, ESP32 예제 코드 포함. SW 개발 전용 — 양산 Part와 SW 호환 (ILI9341 공통) |

### 양산 후보 [M]

| 구분 | 제조사/공급처 | Part Number | 폼팩터 | 드라이버 | 인터페이스 | FPC | 동작 온도 | 밝기 | 단가(100개) | 비고 |
|------|-------------|-------------|--------|---------|-----------|-----|---------|------|------------|------|
| **1st Source (검증 중)** | BuyDisplay (EastRising) | ER-TFT028A2-4 | 베어 패널 | ILI9341 | 4-wire SPI | 50핀 0.5mm ZIF | -20~70°C | ~250-300 cd/m² | $6.35 | **직접 거래만** (Digi-Key/Mouser 미입점). **40MHz SPI 동작 미검증** — Teensy 포럼에 16MHz 권장 보고 존재. 샘플 실측 필요 |
| **1st Source 프로토 검증용** | BuyDisplay | ER-TFTM028-4 | 모듈(동일 패널 + PCB 브레이크아웃) | ILI9341 | 4-wire SPI | — | -20~70°C | ~250-300 cd/m² | ~$10 | ER-TFT028A2-4와 동일 패널. 40MHz SPI 검증 샘플로 1개 주문 (배송 2주) |
| **2nd Source (검증 필요)** | WINSTAR | WF28JTYAJDNN0 | 베어 패널 | ILI9341 | 4-wire SPI | ~40핀 0.5mm ZIF | -20~70°C | **500 cd/m²** | ~$10-13 | **Digi-Key 입점** (공급 안정). 차량 내 고휘도 유리. 단가 2배. FPC 핀아웃 1st와 비호환 → 드롭인 교체 불가 |

### 검토 결과 제외된 후보

| 제외 Part | 제외 사유 |
|----------|---------|
| ~~WINSTAR WF28ETLAJDNN0~~ | **TN 패널 + 8080 패러렐 전용** — IPS/SPI 요구사항 불만족. 올바른 IPS 버전은 WF28J**TYAJDNN0 ("J" variant) |
| ~~LCDWiki MSP2807~~ | **TN 패널**, 동작 온도 -20~60°C — 70°C 요구사항 미달. IPS 변형이라는 이전 기재는 오류 |

### 양산 디스플레이 확정 프로세스

```
1. BuyDisplay ER-TFTM028-4 샘플 1개 주문 (배송 2주)
2. ESP32 DevKit + TFT_eSPI로 40MHz SPI 동작 실측
   - 초기화 안정성
   - 풀스크린 갱신 fps
   - 애니메이션 렌더링 성능
3. 결과에 따라:
   - 통과 → BuyDisplay ER-TFT028A2-4 양산 1st Source 확정
   - 실패 → WINSTAR WF28JTYAJDNN0로 1st Source 전환 (단가 2배 수용)
4. 1st Source 확정 후 캐리어보드 PCB FPC 커넥터 설계 확정
```

### FPC 핀아웃 비호환 주의

BuyDisplay 50핀과 WINSTAR 40핀의 FPC 핀아웃은 **비호환**. PCB 설계 시 1st Source를 기준으로 커넥터 풋프린트 고정. 2nd Source(WINSTAR) 전환 시 PCB 재설계 또는 어댑터 보드 필요.

---

## 4. OBD2 통신 — CAN 직결 [M]

| 항목 | 내용 |
|------|------|
| **기능** | OBD2 CAN 버스 직결 통신 + OBD2 12V 전원 수급 |
| **Status** | CONFIRMED (Trade Study #3) — **양산 PCB에만 탑재. 프로토타입은 ELM327 BT 사용** |

| 구분 | 제조사 | Part Number | 패키지 | AEC-Q | 비고 |
|------|--------|-------------|--------|-------|------|
| **CAN 트랜시버 1st** | NXP | TJA1051T/3 | SOIC-8 | Q100 | 3.3V I/O. TXD Dominant Timeout 내장 — ESP32 크래시 시 CAN 버스 블로킹 자동 방지 |
| **CAN 트랜시버 2nd** | TI | SN65HVD230DR | SOIC-8 | Q100 | 3.3V 네이티브. Dominant Timeout 없음 — 외부 보호 회로 필요 |
| **CAN TVS 다이오드** | NXP | PESD1CAN | SOT-23 | Q101 | CAN 라인 ESD 보호 (~$0.10) |
| **CAN 종단 저항** | — | 120Ω 0402 | 0402 | Q200 | CANH-CANL 사이. 솔더 브릿지로 on/off 가능하게 설계 |

---

## 5. 전원 아키텍처 — 2-Stage Buck [PM]

### 5-0. 전체 아키텍처

```
OBD2 12V ──→ [보호 회로 §6] ──→ [Stage 1 Buck] ──→ 5V ──→ [Power MUX §7] ──→ 5V_COMMON
                                                                    ↑
USB-C 5V ──→ [Schottky] ─────────────────────────────→ 5V_USB ───┘
                                                                          │
                                                                          ↓
                                                      [Stage 2 Buck MP2315] → 3.3V_PERIPH
```

### 5-A. 프로토타입 전원

프로토타입 캐리어보드는 DevKit의 내장 USB-C + LDO를 사용. 별도 전원 회로 불필요.

### 5-B. Stage 1 Buck (12V → 5V) [PM]

| 항목 | 내용 |
|------|------|
| **기능** | OBD2 12V → 5V. Load Dump 65V(ISO 7637) 내성, 크랭킹 UVLO 4.3V |
| **Status** | CONFIRMED |

| 구분 | 제조사 | Part Number | 패키지 | Vin | Iout | UVLO | AEC-Q | LCSC |
|------|--------|-------------|--------|-----|------|------|-------|------|
| **1st Source** | TI | **TPS54360BQDDARQ1** | **HSOP-8 PowerPAD (DDA)** | 4.5~60V | 3.5A | 4.3V | **Q100** | C100891 |
| **2nd Source** | TI | TPS54341QDRCRQ1 | VSON-10 | 4.5~42V | 3.5A | 4.3V | Q100 | (확인 필요) |

> **2nd Source 주의**: TPS54341-Q1은 42V max로 TPS54360B 60V보다 낮으나, SMBJ20CA 2차 TVS 클램프 32V 기준 여유 10V로 Load Dump 대응 가능. Footprint는 VSON-10으로 HSOP-8과 다르므로 PCB 재설계 필요.

**Stage 1 관련 수동부품** [PM]:
| 부품 | 값 | 허용오차 | 패키지 | 용도 | LCSC |
|------|---|---------|--------|------|------|
| FB R_top | 100kΩ | **0.1%** | 0402 | Vout=4.99V 설정 | C97521 |
| FB R_bot | 19.1kΩ | **0.1%** | 0402 | Vout=4.99V 설정 | C160111 |
| Rt | 240kΩ | 1% | 0402 | Fsw=500kHz 설정 | C25812 |
| EN pull-up | 100kΩ | 1% | 0402 | EN pin to VIN | C25787 |
| L1 인덕터 | **15µH** | 2A 차폐 | 5×5mm | Stage 1 에너지 저장 | C128268 |
| Cout | 22µF/16V MLCC × 2 | X7R | 0805 | Stage 1 출력 안정화 | C45783 |
| Cout HF | 100nF + 10nF | X7R | 0402 | Stage 1 고주파 바이패스 | C1525, C1567 |

### 5-C. Stage 2 Buck (5V → 3.3V) [PM]

| 항목 | 내용 |
|------|------|
| **기능** | 5V → 3.3V_PERIPH (LCD, 센서, 버튼 등 주변장치용) |
| **Status** | CONFIRMED |

| 구분 | 제조사 | Part Number | 패키지 | Vin | Iout | Vref | AEC-Q | LCSC |
|------|--------|-------------|--------|-----|------|------|-------|------|
| **1st Source** | MPS | MP2315GJ-Z | TSOT23-8 | 4.5~24V | 1.5A | 0.8V | — (Industrial) | C15425 |
| **2nd Source** | TI | TLV62569DBVR | SOT-23-5 | 2.5~5.5V | 2A | 0.6V | — | C129255 |

**Stage 2 관련 수동부품** [PM]:
| 부품 | 값 | 허용오차 | 패키지 | 용도 | LCSC |
|------|---|---------|--------|------|------|
| FB R_top | 100kΩ | **0.1%** | 0402 | Vout=3.27V 설정 | C97521 |
| FB R_bot | **32.4kΩ** | **0.1%** | 0402 | Vout=3.27V 설정 | C131087 |
| L2 인덕터 | 4.7µH | 2A 차폐 | 4×4mm | Stage 2 에너지 저장 | C167229 |
| Cin | 22µF/10V MLCC | X7R | 0805 | Stage 2 입력 바이패스 | C159801 |
| Cout | 22µF/10V + 10µF/10V MLCC | X7R | 0805 | Stage 2 출력 안정화 | C159801, C19702 |
| Cout HF | 100nF | X7R | 0402 | 고주파 바이패스 | C1525 |

> ~~AMS1117-3.3 (LDO)~~ — 차량 고온 환경 발열 문제로 Rev A에서 제거됨.  
> ~~v3.0의 R_bot=22kΩ (4.44V 출력 오류)~~ — **32.4kΩ 0.1%로 수정 완료** (v3.0 자가검증).

---

## 6. 입력 보호 회로 [PM]

### 6-1. OBD2 12V 입력 보호 체인

보호 체인 순서: **eFuse → TVS 1차 (SMC) → ESD → Ferrite → TVS 2차 (SMB) → Ideal Diode → Stage 1 Buck**

| 구분 | 제조사 | Part Number | 패키지 | 기능 | AEC-Q | LCSC |
|------|--------|-------------|--------|------|-------|------|
| **eFuse** | TI | TPS25221-Q1 | SOT-23-6 | 과전류 100µs 차단 (1A 설정) | Q100 | C2987547 |
| **TVS 1차** | Littelfuse | SMCJ24CA | **SMC** | Load Dump 1차 클램프 (3kW, 양방향, Standoff 24V, Clamp 39V @ 1A) | Q101 | C92832 |
| **ESD 보호** | NXP | PESD2CAN-LX | SOT-23 | <1ns ESD 응답, ±15kV | Q101 | C35783 |
| **Ferrite** | Murata | BLM31PG601SN1 | 1206 | 600Ω@100MHz, 3A, 고주파 감쇄 | Q200 | C105160 |
| **TVS 2차** | Littelfuse | SMBJ20CA | **SMB** | 2차 클램프 (600W, 양방향, Standoff 20V, Clamp 32V @ 1A) — Stage 1 Abs Max 65V 보호 | Q101 | C24342 |
| **Ideal Diode Ctrl** | TI | LM74700-Q1 | SOT-23-6 | 역극성 이중 보호, 이상적 다이오드 (VF ~0V) | Q100 | C2662568 |
| **Pass PMOS** | Vishay | SQJ422EP-T1_GE3 | PowerPAK SO-8 | LM74700 쌍 (65V, 26A, Low Rds(on)) | Q101 | C197495 |

**TVS 패키지 선정 원칙 (위치별 에너지 기준)**:
| 위치 | 패키지 | 에너지 | 이유 |
|------|--------|--------|------|
| OBD2 1차 TVS (SMCJ24CA) | **SMC 필수** | 3kW peak | Load Dump 10~50J 흡수, SMB(600W)로는 부족 |
| OBD2 2차 TVS (SMBJ20CA) | SMB 적정 | 600W peak | 1차 통과분만 받음, SMB로 충분 |
| USB VBUS TVS (SMBJ5.0CA) | SMB 적정 | 600W peak | 5V 라인 ESD용 |

### 6-2. USB-C 5V 입력 보호 [P]

| 구분 | 제조사 | Part Number | 패키지 | 기능 | AEC-Q | LCSC |
|------|--------|-------------|--------|------|-------|------|
| **Schottky** | Diodes Inc. | SBR2U60P1 | SOD-123 | VF 0.25V @ 500mA, 저손실 | — | C236404 |
| **TVS USB VBUS** | Littelfuse | SMBJ5.0CA | SMB | USB VBUS ESD/Surge (600W, 양방향, Clamp 9.2V) | — | C24283 |
| **Ferrite GND** | Murata | BLM18PG471SN1 | 0603 | USB GND Loop 차단 (470Ω@100MHz) | — | C105161 |

### 6-3. USB-C 데이터 라인 ESD [P]

| 구분 | 제조사 | Part Number | 패키지 | AEC-Q |
|------|--------|-------------|--------|-------|
| **1st Source** | STMicroelectronics | USBLC6-2SC6 | SOT-23-6 | — |

---

## 7. Power MUX [PM]

| 항목 | 내용 |
|------|------|
| **기능** | USB 5V / OBD2 Stage1 5V 중 높은 쪽 자동 선택 |
| **Status** | CONFIRMED |

| 구분 | 제조사 | Part Number | 패키지 | Vin | AEC-Q | LCSC |
|------|--------|-------------|--------|-----|-------|------|
| **1st Source** | TI | LM66100 | SOT-23-6 | 1.5~5.5V | — | C1009826 |
| **2nd Source** | TI | TPS2116 | SOT-23-6 | 1.6~5.5V | — | C2686097 |

---

## 8. USB-C 커넥터 [P]

| 항목 | 내용 |
|------|------|
| **기능** | 펌웨어 플래싱 + 시리얼 디버그 (CH340C 경유). 프로토타입에서는 전원도 겸함. |
| **Status** | CONFIRMED |

| 구분 | 제조사 | Part Number | 비고 |
|------|--------|-------------|------|
| **USB-C 리셉터클** | GCT | USB4105-GF-A | 16핀, SMD, USB 2.0 (D+/D- 포함), 기구 고정탭, 5A, -40~85°C, JLCPCB C3020560 |

**USB-C 필수 회로** [P]:
| 부품 | 값 | 용도 |
|------|---|------|
| CC1 풀다운 저항 | 5.1kΩ | USB-C 전원 수신 식별 (필수) |
| CC2 풀다운 저항 | 5.1kΩ | USB-C 전원 수신 식별 (필수) |
| 입력 벌크캡 | 100µF / 10V | 인러시/케이블 탈착 서지 흡수 |

---

## 9. OBD2 입력 커넥터

### 9-A. 프로토타입 [P]

| 항목 | 사양 |
|------|------|
| PCB측 커넥터 | **JST B3B-XH-A** (3핀, 2.5mm 피치) |
| 상대 커넥터 | JST XHP-3 하우징 + SXH-001T-P0.6 컨택 |
| 케이블 | AWG22 3C, 500mm (12V / GND / Shield) |
| 핀 할당 | 1=12V, 2=GND, 3=Shield (케이스 프레임) |
| LCSC | C145992 |

### 9-B. 양산 [M]

| 항목 | 사양 |
|------|------|
| 커넥터 | **SAE J1962 Type A Male** (16핀 OBD2 표준) |
| 사용 핀 | 4(GND), 5(Sig GND), 6(CANH), 14(CANL), 16(+12V) |
| 공급처 | TBD (DVT 전 확정 필요) |
| PCB측 케이블 커넥터 | TBD — JST 또는 Molex 확정 필요 |

---

## 10. 사용자 입력 (2-버튼) [PM]

| 항목 | 내용 |
|------|------|
| **기능** | 캐릭터 모드 전환 (이전/다음) |
| **Status** | CONFIRMED (Rev F에서 로터리 엔코더 → 2-버튼으로 변경) |

### 배치

- **위치**: 케이스 **상단면** (대시보드 거치 시 사용자가 팔을 뻗었을 때 손가락이 자연스럽게 닿는 면)
- **레이아웃**: 가로 배치 (좌/우)
  - **버튼 A (좌)** = 이전 모드 (Prev)
  - **버튼 B (우)** = 다음 모드 (Next)
  - UX 표준: 서양 읽기 방향(좌→우 = 과거→미래)과 일치
- **중심 간격**: 15~20mm (운전 중 오조작 방지)
- **캡 크기**: 외부 10×10mm (장갑 착용 시 조작 가능)

### Tactile Switch

| 구분 | 제조사 | Part Number | 패키지 | 작동력 | 동작 온도 | 수명 | 단가 (100개) | AEC-Q | 비고 |
|------|--------|-------------|--------|-------|---------|------|-------------|-------|------|
| **1st Source** | C&K | PTS645SM43SMTR92 LFS | 6×6mm SMD 4-pad gull-wing | 160gf | -40~+85°C | 100,000 사이클 | ~$0.16 | — | KiCad 표준 풋프린트(`SW_SPST_PTS645`) 기준. Newark/Onlinecomponents 재고 |
| **2nd Source** | Würth Elektronik | 430182043816 (WS-TASV) | 6×6mm SMD 4-pad gull-wing | 160gf | -40~+85°C | **1,000,000 사이클** | $0.44 | — | 동일 풋프린트 (드롭인 교체 가능). DK 재고 안정. 10배 수명 |

### 버튼 캡

| 항목 | 스펙 |
|------|------|
| **소재** | ABS (케이스 PC-ABS와 통일감, 클릭감 명확) |
| **형상** | 정사각 또는 원형, 외경 10mm |
| **높이** | 케이스 표면 위로 0.5~1mm 돌출 |
| **색상** | TBD (케이스 색상과 함께 결정) |

### 디바운싱 회로

| 부품 | 값 | 패키지 | 수량 | 용도 |
|------|---|--------|------|------|
| 풀업 저항 | 10kΩ | 0402 | 2 | 버튼 A, B 각 1개 |
| 디바운스 캡 | 100nF | 0402 | 2 | 버튼 A, B 각 1개 (선택적 — SW 디바운스 대체 가능) |

### 펌웨어 기능 확장 여지

- **짧게 누름** (<500ms): 기본 동작 (이전/다음)
- **길게 누름** (>1s): 보조 기능 (예: 밝기 조절, 경고 해제)
- **두 버튼 동시**: 리셋/숨김 메뉴 등

---

## 11. 조도 센서 [PM]

| 항목 | 내용 |
|------|------|
| **기능** | LCD 백라이트 자동 조광 (Auto-dimming) |
| **Status** | CONFIRMED |

| 구분 | 제조사 | Part Number | 인터페이스 | 패키지 | AEC-Q | 비고 |
|------|--------|-------------|-----------|--------|-------|------|
| **1st Source** | Vishay Semiconductor Opto | TEMT6000X01 | 아날로그 (ADC → **GPIO15**) | 1206 (3216 Metric) | — | Active, Digi-Key 재고 9,010개 확인. 10K 단가 ~$0.50 |
| **2nd Source** | Vishay | VEML7700 | I2C | 옵토 LGA | — | 풋프린트·인터페이스 다름 — 회로/코드 변경 필요 |

> **v3.1 GPIO 변경**: GPIO34 (조도) → **GPIO15** (조도 ADC). GPIO34는 VIN 모니터로 재할당.

---

## 12. NTC 온도 센서 [PM]

| 항목 | 내용 |
|------|------|
| **기능** | LCD 백면 부착 온도 감지, 4단계 온도 보호 입력 (§3 참조) |
| **Status** | CONFIRMED |

| 구분 | 제조사 | Part Number | 패키지 | 저항 | 정확도 | AEC-Q | LCSC |
|------|--------|-------------|--------|------|--------|-------|------|
| **1st Source** | Murata | NCP18XH103F03RB | 0402 | 10kΩ @ 25°C | ±1% | — | C77014 |

**설치 방식**:
- 프로토타입: 플라잉 리드 (AWG30 TP + Shield, 50mm) + 3M 8810 열전도 테이프
- 양산: FPC 일체형 NTC (디스플레이 모듈 커넥터에 통합)
- 단선 감지: SW에서 ADC >95% or <5% 시 NTC_OPEN/SHORT 판정, safe fallback 진입

---

## 13. 피에조 버저 [PM]

| 항목 | 내용 |
|------|------|
| **기능** | 버튼 클릭 피드백 + 경고음 |
| **Status** | CONFIRMED |

| 구분 | 제조사 | Part Number | 구동 전압 | 크기 | 동작 온도 | 음압 | 단가 (1K) | LCSC |
|------|--------|-------------|---------|------|---------|------|---------|------|
| **1st Source** | FUET | FUET-9018 | 1~25V (정격 3V) | 9×9×2.1mm | -30~70°C | 65dB | $0.27 | C391035 |
| **2nd Source** | TDK | PS1240P02BT | 3~30V | ø12.2×6.5mm | -10~70°C | 70dB | $0.10 | C76871 (Through-hole) |

**선정 이유 (FUET-9018)**: SMD 타입으로 SMT 공정 통합 가능, GPIO 직접 구동 (5mA), 소형 (9×9×2.1mm), 3.3V 직접 구동 안정적.  
**2nd Source 주의**: PS1240P02BT는 Through-hole — 별도 삽입 공정 필요.

---

## 14. 백라이트 제어 MOSFET [PM]

| 항목 | 내용 |
|------|------|
| **기능** | PWM으로 LCD 백라이트 밝기 제어 |
| **Status** | CONFIRMED |

| 구분 | 제조사 | Part Number | 패키지 | Vgs(th) | AEC-Q | LCSC |
|------|--------|-------------|--------|---------|-------|------|
| **1st Source** | AOS | AO3400A | SOT-23 | ~1.4V | — | C20917 |
| **2nd Source** | Nexperia | 2N7002 | SOT-23 | ~1.8V | Q101 | C8545 |

**관련 수동부품**:
| 부품 | 값 | 용도 |
|------|---|------|
| 게이트 직렬 저항 | 33Ω 0402 | 스위칭 에지 제어, EMI 감소 |
| 게이트 풀다운 | 100kΩ 0402 | ESP32 부팅 중 백라이트 OFF 유지 |
| 백라이트 벌크캡 | 4.7µF 0603 | PWM 리플 흡수 |

---

## 15. PCB [PM]

| 항목 | 프로토 (v3.1) | 양산 |
|------|-------------|------|
| **레이어** | 4 | 4 |
| **크기** | **75×90mm** | TBD (케이스 확정 후) |
| **두께** | 1.6mm 표준 | 1.6mm |
| **표면처리** | **ENIG 권장** (차량 고온 환경) | ENIG 필수 |
| **최소 선폭/간격** | 6/6 mil | 6/6 mil |
| **비아** | 0.3mm | 0.3mm |
| **동박** | 1oz 전체 | 1oz 전체 |
| **스택업** | **L1=Signal+부품 / L2=GND solid / L3=Power+GND pour / L4=Signal+GND pour** | 동일 |
| **Status** | CONFIRMED | TBD (치수) |

**레이아웃 필수 규칙**:
- BT 안테나 Keep-out: 17×8mm, L1~L4 전부 구리 없음, 안테나 border 5mm 이상 돌출
- Buck SW 트레이스 10mm 이내, 루프 5mm² 이내
- ADC 신호 (GPIO34 VIN 모니터, GPIO35 NTC): Buck SW 노드에서 10mm 이상 이격
- Stage1 Buck HSOP-8 Thermal pad: 6개 via로 L2 GND plane 연결
- USB-C D+/D- 차동쌍 길이 매칭 (±5mil)

---

## 16. 케이스 및 거치 [PM]

| 항목 | 스펙 |
|------|------|
| **소재** | **PC-ABS 블렌드** (확정, ABS 단독 사용 불가) |
| **설치 방식** | **Beanbag + Magnet (MagSafe-style)** (확정, Trade Study #4 Rev C) |
| **색상** | TBD |
| **금형** | TBD (사출 금형 $3,000~$10,000 추정) |
| **Status** | 소재/설치 방식 CONFIRMED, 나머지 TBD |

### 16-A. 거치 구성 부품

**대시보드 원상보존 필수(R0)** — 어떤 영구 접착제도 대시 표면에 닿지 않음.

| 부위 | 제조사/후보 | Part Number | 스펙 | 비고 |
|------|----------|-------------|------|------|
| **자석 어레이 (베이스 측)** | totalElement 또는 K&J Magnetics | MagSafe-compatible ring (54×46×2.5mm) **또는** N42SH disc ×4 | **N45SH (150°C)** 권장, 최소 N42SH. Pull force 800~1100 gf | MagSafe 호환 ring이 검증된 옵션 |
| **강판 플레이트 (기기 측)** | 국내 SUS 벤더 TBD | SUS430 또는 동급 ferrous | 원형 50mm dia × 0.5mm | 케이스 바닥 포켓에 접착 매립 |
| **Beanbag 베이스** | TBD — EMS 액세서리 OEM 또는 Bracketron 계열 | 커스텀 (100×80×20mm, ~150g) | PU/PVC 외피, 비드 충전, 바닥 실리콘 논슬립 | 상면에 자석 어레이 인서트 |
| **패키지 경고 라벨** | 표준 | — | "심박조율기 보유자 주의" | 박스 라벨링 |

### 16-B. 케이스 설계 요구사항

- 바닥면에 원형 포켓 (Ø50mm × 3mm deep) — 강판 인서트 수납
- 자석(Beanbag 측)-PCB 안테나 중심 이격 **≥15mm** (WiFi/BT 간섭 방지)
- 케이스 바닥 평탄 — Beanbag 상면 자석과 정렬 보장

### 16-C. EVT 검증 항목

1. Pull force 측정 (spring gauge) — 기기 분리에 800~1100 gf 필요
2. 1G 급정거 진동 시뮬 — 분리 없음 확인
3. 여름 SoCal 대시 72시간 로깅 — 자력/Beanbag 상태
4. WiFi/BT RSSI 측정 (자석 유/무 비교) — 간섭 ≤3 dB
5. 곡면/경사 대시(5°, 15°, 30°) Beanbag 슬립 테스트

---

## 17. 기구 조립 부품 [PM]

| 부품 | 모델 | 용도 |
|------|------|------|
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
| A | 2026-04-09 | 초판 작성. LDO 제거, Buck 확정. 4레이어 PCB 확정. 송풍구 설치 확정 |
| B | 2026-04-10 | 디스플레이 2.5"→2.4" ILI9341 스펙 확정. USB-C USB4105-GF-A 변경. FUET-9018 확정. BOM 파일럿/양산 2-tier 추가 |
| C | 2026-04-14 | 거치 방식: 송풍구 클립 → 대시보드 VHB 접착. 디스플레이: 2.4" → 2.8" ILI9341 IPS. BuyDisplay ER-TFT028A2-4 1st Source |
| D | 2026-04-14 | MCU: WROOM → WROVER-E-N16R8 (PSRAM 8MB). OBD2 CAN 직결 부품 추가 (양산). 전원 섹션 프로토/양산 분리 |
| E | 2026-04-15 | 디스플레이 섹션 대폭 수정. WINSTAR 오류 정정(TN→IPS). LCDWiki MSP2807 제외. Hosyond B0CMGF4Q68 프로토 전용 추가. BuyDisplay 40MHz SPI 검증 프로세스 명시 |
| F | 2026-04-15 | 사용자 입력: 로터리 엔코더 → 2-버튼 (Trade Study #6). 케이스 상단면 가로 배치. C&K PTS645 1st Source 확정 |
| G | 2026-04-16 | 거치 방식 전면 재설계 (Trade Study #4 Rev C). VHB/Dual Lock → Beanbag + Magnet (MagSafe-style). R0 요건 확정. N45SH 자석, SUS430 강판, Beanbag 구성 추가 |
| **H** | **2026-04-16** | **팀원 Rev E 설계 반영 merge. ① 전원 아키텍처: 단일 MP2315 Buck → 2-stage Buck (Stage1 TPS54360BQDDARQ1 60V AEC-Q100 + Stage2 MP2315) — Load Dump ISO 7637 대응. ② 입력 보호 회로 섹션 신규 (§6): TPS25221-Q1 eFuse, SMCJ24CA TVS 1차 SMC, PESD2CAN-LX ESD, BLM31PG601SN1 Ferrite, SMBJ20CA TVS 2차 SMB, LM74700-Q1 + SQJ422EP PMOS Ideal Diode. ③ Power MUX 섹션 신규 (§7): LM66100 (2nd: TPS2116). ④ NTC 온도 센서 섹션 신규 (§12): Murata NCP18XH103F03RB 0402. ⑤ OBD2 프로토 커넥터 추가 (§9): JST B3B-XH-A. ⑥ PCB 크기 60×80mm → 75×90mm, ENIG 권장, 4레이어 stackup 명시 (§15). ⑦ Scope 표기 [P][M][PM] 전체 적용. ⑧ AEC-Q 컬럼 전체 추가. ⑨ Stage 2 FB R_bot 32.4kΩ 0.1% 수정. ⑩ GPIO15 조도 센서 할당 명시. ⑪ 4단계 온도 보호 테이블 §3에 명시. ⑫ TVS 패키지 선정 원칙 신규. ⑬ 충돌 항목 해결: 로터리 엔코더(Rev E §10) 폐기 → 2-버튼 유지, VHB 거치(Rev E §17) 폐기 → Beanbag+Magnet 유지.** |

---

*본 문서는 PCB 발주 및 EMS 계약 시 BOM과 함께 제출하는 공식 부품 승인 목록입니다.*
