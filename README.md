# Carpybara — 프로토타입 로드맵

> **기준일**: 2026-04-16  
> **작성자**: Carpybara  
> **목적**: 프로토타입 완성까지의 구매 목록, 작업 순서, 타임라인을 한눈에 파악

---

## 현재 단계

```
[✅ 완료] 하드웨어 스펙 확정 & 부품 선정
     ↓
[🔄 진행 중] 프로토타입 준비 — 부품 주문 + PCB 설계
     ↓
[ ] 캐리어보드 PCB 제작 및 조립
     ↓
[ ] 펌웨어 개발
     ↓
[ ] EVT (기능 검증 테스트)
```

---

## 즉시 구매 목록 (지금 주문해야 하는 것)

### A. 디스플레이 — 오늘 주문

| 품목 | 구매처 | Part No | 수량 | 단가 | 배송 | 용도 |
|------|--------|---------|:----:|-----:|------|------|
| 2.8" IPS ILI9341 브레이크아웃 모듈 | Amazon | Hosyond **B0CMGF4Q68** | 1 | ~$15 | **1~2일** | 펌웨어 개발 (ESP32 DevKit에 바로 연결) |
| 2.8" IPS ILI9341 베어패널 | BuyDisplay.com | **ER-TFT028A2-4** | 3 | $6.35 | **~2주** (중국 직배) | 캐리어보드 장착 + 40MHz SPI 검증 |

> **왜 두 개를 사는가**: Hosyond는 즉시 도착 → 펌웨어 개발 즉시 시작. BuyDisplay 베어패널은 2주 배송 → 그 기간에 PCB 설계 병행. 둘 다 ILI9341 드라이버로 동일 → 펌웨어 코드 그대로 이식.

### B. 개발 환경

| 품목 | 구매처 | 비고 |
|------|--------|------|
| ESP32-WROVER-E-N16R8 DevKit | Amazon / Digi-Key | 프로토타입 MCU.

### C. 캐리어보드 PCB 부품 — PCB 설계 완료 후 발주

PCB 설계(EasyEDA Pro) → Gerber + BOM 생성 → JLCPCB에 PCB 제작 + SMT 일괄 발주.  
아래 부품은 JLCPCB SMT 서비스(LCSC 재고) 또는 Digi-Key로 조달:

| 품목 | Part No | 조달처 | 수량 |
|------|---------|--------|:----:|
| FPC 커넥터 50핀 0.5mm ZIF | BuyDisplay 데이터시트 기준 선정 | LCSC / Digi-Key | 1 |
| Tactile 스위치 (6×6mm SMD) | C&K **PTS645SM43SMTR92 LFS** | Newark / Onlinecomponents | 2 |
| 조도 센서 | Vishay **TEMT6000X01** | Digi-Key | 1 |
| 피에조 버저 (SMD) | FUET **FUET-9018** | LCSC (C391035) | 1 |
| 백라이트 MOSFET | AOS **AO3400A** | Digi-Key | 1 |
| USB-C 리셉터클 | GCT **USB4105-GF-A** | JLCPCB (C3020560) | 1 |
| ESD 보호 | ST **USBLC6-2SC6** | Digi-Key | 1 |
| USB-UART 브릿지 | WCH **CH340C** | LCSC | 1 |
| ESP32 DevKit 소켓 (38핀) | 2.54mm 핀소켓 × 2열 | LCSC | 1 set |
| 수동부품 (저항/캡/인덕터) | 0402/0805 표준 값 | JLCPCB LCSC 일괄 | 다수 |

---

## 남은 작업 순서 및 타임라인

```
Week 0 (지금, 4/16)
├── [구매] Hosyond B0CMGF4Q68 Amazon 주문
├── [구매] BuyDisplay ER-TFT028A2-4 × 3 주문
└── [확인] ESP32-WROVER DevKit / ELM327 재고 확인

Week 1 (4/17~4/20)
├── [도착] Hosyond 1~2일 내 도착
├── [개발] ESP32 DevKit + Hosyond 연결 → TFT_eSPI 동작 확인
├── [개발] 기본 캐릭터 렌더링 펌웨어 작성 시작
└── [설계] 캐리어보드 PCB 설계 시작 (EasyEDA Pro)
        기준: BuyDisplay ER-TFT028A2-4 FPC 50핀 0.5mm 스펙
        포함 부품: FPC 커넥터, 2-버튼, TEMT6000, 피에조,
                  백라이트 MOSFET, USB-C, CH340C, DevKit 38핀 소켓

Week 2 (4/21~4/27)
├── [설계] 캐리어보드 PCB 설계 완료
├── [발주] JLCPCB PCB 제작 + SMT 발주 (제작 1~2주)
└── [개발] 펌웨어 개발 계속 (Hosyond + DevKit)

Week 3~4 (4/28~5/10)
├── [도착] BuyDisplay ER-TFT028A2-4 베어패널 (~2주)
├── [도착] JLCPCB 캐리어보드 PCB
├── [검증] BuyDisplay 40MHz SPI 동작 실측
│         통과 → 양산 1st Source 확정
│         실패 → WINSTAR WF28JTYAJDNN0로 전환 (Digi-Key 재주문)
└── [조립] 캐리어보드 + BuyDisplay 패널 + 기타 부품 조립
        → 프로토타입 완성
```

---

## 완료된 결정 요약 (Trade Study 기반)

| 항목 | 결정 | 근거 |
|------|------|------|
| **MCU** | ESP32-WROVER-E-N16R8 (16MB Flash / 8MB PSRAM) | PSRAM으로 풀스크린 스프라이트 가능. 양산 시 WROOM 다운그레이드 검토 |
| **디스플레이** | 2.8" IPS ILI9341, 240×320, 4-wire SPI | Trade Study #2, #5 |
| **OBD2 (프로토)** | ELM327 BT (Veepeak VP11) | 기존 코드 재활용 |
| **OBD2 (양산)** | NXP TJA1051T/3 CAN 트랜시버 직결 | Trade Study #3 |
| **전원 (프로토)** | USB-C (DevKit 내장 LDO) | 별도 회로 불필요 |
| **전원 (양산)** | OBD2 12V → MPS MP2315GJ-Z Buck → 3.3V | 고효율 95%, 차량 전원 직결 |
| **입력** | 2-버튼 Tactile SMD (케이스 상단면) | 운전 중 직관성 우선 (Trade Study #6) |
| **조도 센서** | Vishay TEMT6000X01 (아날로그 ADC) | 백라이트 자동 조광 |
| **거치** | Beanbag + Magnet (MagSafe-style, N45SH) | 대시보드 무접착(R0) 필수 (Trade Study #4 Rev C) |
| **케이스 소재** | PC-ABS 블렌드 | 차량 대시 80°C+ 내열, ABS 단독 금지 |
| **PCB** | 4레이어, 약 60×80mm | SPI 40MHz 신호 무결성 + 안테나 keepout |

---

## 미결 이슈 / 리스크

| 우선순위 | 이슈 | 완화 방법 |
|:---:|------|---------|
| **P0** | BuyDisplay 40MHz SPI 동작 미검증 | 베어패널 도착 즉시 실측. 실패 시 WINSTAR WF28JTYAJDNN0(Digi-Key, $10~13)로 전환 |
| **P0** | 캐리어보드 PCB 설계 미완료 | EasyEDA Pro + easyeda-mcp로 설계 착수 예정 |
| **P1** | BuyDisplay 단일 공급처 (Digi-Key/Mouser 미입점) | 초도 재고 여유분 확보, WINSTAR 2nd Source 준비 |
| **P1** | OBD2 커넥터 (SAE J1962) 공급처 미결 | 양산 PCB 설계 전 결정 예정 |
| **P1** | 거치 마운트 세부 부품 미결 (자석/강판/베이스) | 케이스 설계 단계에서 결정 예정 |
| **P2** | 모든 부품 실물 검증 미진행 (현재 100% 데이터시트 기반) | EVT 단계에서 전수 검증 |
| **P2** | WROVER vs WROOM 다운그레이드 미결 | 프로토타입 PSRAM 사용량 실측 후 결정 |

---

## 주요 제약사항 (반드시 준수)

- 앞유리 흡착컵 거치 **금지** — 미국 28개 주 이상에서 불법
- 대시보드 접착/잔여물 **금지** (R0) — VHB·Dual Lock·Nano Gel 전부 불가
- 자석 등급 **N45SH 이상** (일반 N42는 SoCal 80°C+ 환경에서 감자화)
- 심박조율기/ICD 경고 라벨 — 제품·패키지·매뉴얼 **3곳 필수**
- ILI9488 디스플레이 **금지** — ESP32와 18-bit SPI 비호환
- ABS 단독 케이스 **금지** — PC-ABS 블렌드 필수

---

*세부 부품 스펙은 `Carpybara_Frozen_AVL.md` (Rev G) 참조.*  
*의사결정 히스토리는 `Trade_Studies/` 폴더 참조.*
