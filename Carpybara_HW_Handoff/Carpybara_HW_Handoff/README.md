# Carpybara — 하드웨어 핸드오버 패키지

> **용도**: 프로젝트에 새로 합류하는 HW 엔지니어/PCB 설계자/펌웨어 개발자/EMS에게 현 시점의 하드웨어 설계 상태를 넘겨주기 위한 문서 묶음.
> **패키지 버전**: 2026-04-16 기준
> **기준 문서**: Frozen AVL Rev H

---

## 이 패키지는 무엇인가

Carpybara는 ESP32 기반 차량용 OBD2 캐릭터 애니메이션 대시보드 장치입니다.
- 목표 소매가: $50 (Amazon US)
- 현 단계: **프로토타입 착수 직전** (캐리어보드 PCB 설계 시작 전)

이 핸드오버 패키지는 지금까지 확정된 모든 하드웨어 결정과 그 근거를 담고 있습니다. **이 문서만 읽으면 프로젝트의 HW 맥락을 전부 파악할 수 있어야 합니다.**

---

## 파일 구성

```
Carpybara_HW_Handoff/
├── README.md                          ← 이 문서 (인덱스 + 읽는 순서)
├── Carpybara_Prototype_Roadmap.md     ← ★ 진행 현황 + 구매 목록 + 타임라인 (한눈에 보기)
├── Carpybara_HW_Spec_Final.md         ← 전체 설계 가이드 + 양산 프로세스
├── Carpybara_Frozen_AVL.md            ← 부품 승인 목록 (Rev H) ★유일한 부품 기준
├── Carpybara_Data_Logic.md            ← OBD2 데이터 처리 로직 (펌웨어용)
└── Trade_Studies/                     ← 주요 의사결정 히스토리
    ├── Trade_Study_01_Platform.md     ← MCU: ESP32-WROVER 선정 근거
    ├── Trade_Study_02_Display.md      ← 디스플레이 드라이버: ILI9341 선정 근거 (superseded)
    ├── Trade_Study_03_OBD2.md         ← OBD2 통신: CAN 직결 선정 근거
    ├── Trade_Study_04_Mount.md        ← 거치: Beanbag + Magnet (MagSafe-style) 선정 근거 (Rev C, 최신)
    └── Trade_Study_05_Display_Size.md ← 디스플레이 크기: 2.8" 선정 근거
```

---

## 문서별 역할

| 문서 | 답하는 질문 | 권한/용도 |
|------|-----------|----------|
| **Frozen AVL Rev H** | "어느 제조사의 어느 부품을 쓰는가?" | **부품 승인의 유일한 기준 문서**. EMS/공장에 BOM과 함께 제출. 여기 없는 부품으로 교체하려면 HW owner 서면 승인 필요 |
| **HW Spec Final** | "왜 이렇게 설계했는가? 양산까지 뭘 해야 하는가?" | 전체 HW 설계 가이드. PCB 체크리스트, 인증 계획(FCC SDoC, Prop 65), BOM 원가 분석, 신뢰성 테스트, 관세 리스크 전략 포함 |
| **Data Logic** | "OBD2 어느 PID를 어떻게 파싱하는가?" | 펌웨어 개발자용. 데이터 흐름, 캐릭터 모드 매핑 |
| **Trade Studies** | "왜 이 부품/크기/거치를 골랐는가?" | 의사결정 히스토리. "이거 다시 검토해볼까?" 했을 때 **바퀴 재발명 방지용** |

---

## 역할별 필수 읽기 순서

### 새로 합류한 HW 엔지니어 (Full Onboarding)
1. 이 README 전부
2. **HW Spec Final** → 전체 그림 파악
3. **Frozen AVL Rev H** → 확정된 부품 숙지
4. **Trade Study 01, 03, 04, 05** → 주요 결정 근거 (02는 05에 의해 superseded)

### PCB 설계자 (캐리어보드 작업)
1. 이 README 전부
2. **Frozen AVL Rev H 전체** — 풋프린트/패키지/핀 수 확인
3. **HW Spec Final 섹션 STEP 3** — PCB 설계 체크리스트 (ESD, 테스트 포인트, 안테나 keepout, 디커플링)
4. **Trade Study 03 (OBD2)** — 양산 PCB에 CAN 직결 추가 계획 이해

### 펌웨어 개발자
1. 이 README 전부
2. **Data Logic** — OBD2 PID 파싱, 캐릭터 모드 로직
3. **Frozen AVL 섹션 1, 3, 7, 8** — MCU/디스플레이/2-버튼/조도센서 핀 할당
4. 프로토타입 디스플레이는 **Hosyond B0CMGF4Q68** (ILI9341 공통이라 양산 Part 변경 시 핀 번호만 수정)

### EMS/구매팀
1. **Frozen AVL Rev H** — 이것만 던져주면 됨
2. (BOM 파일은 별도)

---

## 현 시점 확정 스펙 (Quick Reference)

### 프로토타입
- **MCU**: ESP32-WROVER-E-N16R8 DevKit (38핀 소켓 방식)
- **디스플레이**: Hosyond B0CMGF4Q68 (Amazon Prime, 2.8" IPS ILI9341V, 브레이크아웃)
- **입력**: 2-버튼 (C&K PTS645, 케이스 상단면 좌=이전/우=다음)
- **OBD2**: ELM327 BT (Veepeak VP11)
- **전원**: USB-C (DevKit 내장)

### 양산
- **MCU**: ESP32-WROVER-E-N16R8 (또는 WROOM-32E로 다운그레이드, 메모리 실측 후 결정)
- **디스플레이**: BuyDisplay ER-TFT028A2-4 (1st, 검증 중) / WINSTAR WF28JTYAJDNN0 (2nd)
- **입력**: 2-버튼 Tactile SMD — 1st: C&K PTS645SM43SMTR92 LFS / 2nd: Würth 430182043816
- **OBD2**: TWAI + NXP TJA1051T/3 (CAN 직결)
- **전원**: OBD2 12V → MPS MP2315GJ-Z Buck → 3.3V
- **거치**: **Beanbag + Magnet (MagSafe-style)** — 자석 등급 N45SH (또는 min N42SH), 850±50 gf / 대시보드 무접착 (R0)
- **케이스**: PC-ABS 블렌드 / **버튼 캡 ABS 10×10mm** / **백면 Ø50×3mm 자석 포켓 + 안테나 15mm keepout**
- **PCB**: 4레이어, 약 60×80mm

---

## ★ 열려있는 이슈 (Open Items)

후임자가 이어서 진행해야 할 작업:

### P0 — 진행 중
- [ ] **디스플레이 주문 (오늘)** — 2종 동시 주문으로 프로토 속도 최대화
  - **Hosyond B0CMGF4Q68** (Amazon Prime, 1~2일) → ESP32 DevKit 연결, 펌웨어 개발 즉시 시작
  - **BuyDisplay ER-TFT028A2-4 × 3** (BuyDisplay.com, ~2주) → 캐리어보드 장착 + 40MHz SPI 검증
  - 두 디스플레이 모두 ILI9341 드라이버 → 펌웨어 코드 동일
  - BuyDisplay 40MHz 통과 → 양산 1st Source 확정 / 실패 → WINSTAR WF28JTYAJDNN0 전환
- [ ] **프로토타입 캐리어보드 PCB 설계** (디스플레이 주문과 병행 착수)
  - BuyDisplay ER-TFT028A2-4 FPC 50핀 0.5mm 스펙 기준으로 설계
  - EasyEDA Pro + `teileelektronik/easyeda-mcp` 사용 예정
  - 포함: 디스플레이 FPC 커넥터, 2-버튼(Tactile SMD), TEMT6000, 피에조, 백라이트 MOSFET, USB-C, CH340C, DevKit 38핀 소켓
  - 미포함: CAN 트랜시버, OBD2 커넥터, 12V Buck (양산 PCB에서 추가)

### P1 — 공급처/부품 미결
- [ ] **OBD2 커넥터 (차량측)** 공급처 선정 — SAE J1962 Type A Male 16핀 (AVL 섹션 4)
- [ ] **PCB측 케이블 커넥터** 선정 — JST 또는 Molex
- [ ] **12V 입력 보호 회로** 설계 — 역전압/서지 보호 부품 (AVL 섹션 5-B)
- [ ] **거치: 자석·스틸 디스크·Beanbag 공급처 선정** — N45SH 링 자석(K&J Magnetics or totalElement), SUS430 Ø50×0.5mm 디스크, 맞춤 Beanbag 100×80×20mm ~150g (AVL 섹션 12)

### P2 — 물리 검증 미실행
- [ ] 모든 부품 **실물 동작 검증 (EVT)** — 현재까지 100% 데이터시트 기반
- [ ] 2nd Source **드롭인 호환성 실측** — 상당수가 회로/풋프린트 변경 필요 (AVL 참조)
- [ ] BuyDisplay 단일 공급처 리스크 헷지 (Digi-Key/Mouser 미입점)
- [ ] **거치 EVT** — (a) 풀포스 측정 800-1100 gf, (b) 1G 브레이킹·코너링 쇼크 유지, (c) 72h SoCal 한낮 로깅 (감자화/미끌림), (d) 2.4 GHz RSSI 자석 유무 비교 (15mm keepout 검증), (e) Beanbag 텍스처/가죽/러버 대시 미끌림 실차 테스트

### P3 — 케이스/후속
- [ ] 케이스 색상 결정
- [ ] 케이스 금형 발주 (사출 금형 $3K~$10K)
- [ ] PCB 정확한 치수 확정 (케이스 설계 완료 후)
- [ ] **자석 포켓 인서트 방식 확정** — 오버몰딩 vs 스냅-인 vs 접착 (금형 복잡도/단가 트레이드오프)

---

## 중요 제약 사항 (반드시 지켜야 할 것)

### 부품 교체 금지 원칙
**AVL(Rev H)에 없는 부품으로 교체 불가**. 교체 필요 시:
1. 변경 요청서 (변경 이유 + 대체 부품 스펙 비교)
2. HW owner 서면 승인
3. 최소 5개 샘플로 기능 검증
4. AVL Revision 업데이트
5. EMS에 업데이트된 AVL 전달

### 기술 제약
- **ILI9488 사용 금지** — 18-bit SPI only, ESP32와 비호환
- **WROOM 사용 전 실측 필수** — SRAM 520KB, 풀스크린 스프라이트 불가 (프레임버퍼 153KB)
- **ABS 단독 케이스 금지** — 차량 대시보드 80°C+ 도달, PC-ABS 블렌드 필수
- **앞유리 흡착컵 거치 금지** — 미국 28개+ 주에서 불법
- **거치 R0 (무접착/무잔여)** — 대시보드에 접착제/스티커/잔여물 금지. VHB·Dual Lock(VHB 베이스)·Nano Gel·Command Strip 전부 불가. Beanbag 패시브 무게 + 자석 방식만 승인 (Trade Study #4 Rev C).
- **자석 등급** — N45SH (MagSafe-grade) 또는 최소 N42SH. 일반 N42 금지 (내열 80°C, SoCal 대시 환경에서 감자화 위험).
- **자석 안전 경고 라벨** — 심박조율기/ICD 사용자 대상 경고 의무 (제품·패키지·매뉴얼 3곳).

### 시장/인증
- 1차 시장: **미국 (Irvine, CA 기반)** → FCC SDoC 최우선
- 제품 리스팅에 **동작 온도 -20~70°C 명시 필수**
- California Prop 65 대응 — 소재 RoHS 적합성 서류 확보

---

## 자주 묻는 질문

**Q. 프로토타입 디스플레이(Hosyond)와 양산 디스플레이(BuyDisplay)가 다른데 괜찮은가?**
A. 네. 둘 다 ILI9341 드라이버 공통. 펌웨어는 TFT_eSPI 라이브러리 동일, `User_Setup.h`의 핀 번호만 변경하면 됨. SW 이전 비용 ≈ 0.

**Q. WROVER와 WROOM 중 양산에 뭘 쓰나?**
A. **WROVER로 개발 → 실측 후 PSRAM 불필요 확인 시 WROOM 다운그레이드**. 프로토 단계에서 메모리 사용량 측정 후 결정. 핀/코드 100% 호환이라 후반 전환 가능 (Trade Study #1, Strategy A+).

**Q. 양산 PCB와 프로토타입 PCB는 같은가?**
A. **다름**. 프로토타입은 WROVER DevKit 소켓 + USB 전원 + ELM327 BT. 양산은 WROVER 직납 + OBD2 12V Buck + CAN 직결. 프로토타입 캐리어보드 완성 후 양산 PCB를 새로 설계함.

**Q. Trade Study #2는 왜 superseded인가?**
A. #2에서 2.4" ILI9341로 결정했으나, #4(거치 방식이 송풍구 → 대시보드)로 바뀌면서 크기 제약이 해제됨. #5에서 2.8"로 재결정. #2의 드라이버 IC(ILI9341) 결정은 여전히 유효.

**Q. AVL Rev E에서 뭐가 바뀌었나?**
A. 디스플레이 섹션 대폭 수정. WINSTAR WF28ETLAJDNN0(실제 TN/패러렐)을 WF28JTYAJDNN0(IPS/SPI)로 교체. LCDWiki MSP2807(TN, -20~60°C)을 후보에서 제외. Hosyond B0CMGF4Q68을 프로토타입 전용으로 추가. BuyDisplay 40MHz SPI 이슈 검증 프로세스 명시. 상세는 AVL 변경 이력 참조.

**Q. AVL Rev F에서 뭐가 바뀌었나?**
A. **사용자 입력 변경**: 로터리 엔코더(Alps EC11E18244A5) → 2-버튼(C&K PTS645SM43SMTR92 LFS × 2). 근거: 운전 중 직관성 및 조작 안전성 우위. 배치: 케이스 상단면 가로(좌=이전/A, 우=다음/B). 2nd Source: Würth 430182043816 (동일 풋프린트, 1M 사이클). 버튼 캡 ABS 10×10mm. 피에조 버저 기능 "다이얼 피드백" → "버튼 클릭 피드백". 상세는 AVL 섹션 7 + 변경 이력 참조.

**Q. AVL Rev G에서 뭐가 바뀌었나?**
A. **거치 방식 전면 재설계** (Trade Study #4 Rev C). VHB 접착 → **Beanbag + Magnet, MagSafe-style**. R0 요건(무접착/무잔여) 도입. 구성: N45SH 자석 링 + SUS430 50mm 디스크 + 맞춤 Beanbag. 상세는 AVL 변경 이력 및 Trade Study #4 Rev C 참조.

**Q. AVL Rev H에서 뭐가 바뀌었나?**
A. **팀원 캐리어보드 v3.1 설계 반영 merge.** ① 전원 아키텍처: 단일 MP2315 Buck → **2-stage Buck** (Stage1 TPS54360B 60V AEC-Q100 → Stage2 MP2315) — Load Dump ISO 7637 대응. ② **입력 보호 체인 신규**: TPS25221-Q1 eFuse + SMCJ24CA TVS SMC + PESD2CAN-LX ESD + BLM31PG601SN1 Ferrite + SMBJ20CA TVS SMB + LM74700-Q1 Ideal Diode. ③ **Power MUX 신규**: LM66100 (USB-C/OBD2 자동 선택). ④ **NTC 온도 센서 신규**: Murata NCP18XH103F03RB (LCD 후면, 4단계 열보호). ⑤ OBD2 프로토 커넥터: JST B3B-XH-A 추가. ⑥ PCB: 60×80mm → 75×90mm, ENIG, 4레이어 stackup 명시. ⑦ 충돌 해결: 로터리 엔코더·VHB 거치(팀원 안) 폐기 → 2-버튼·Beanbag+Magnet 유지. 상세는 AVL 변경 이력 참조.

**Q. 왜 양산 PCB에서 자석/스틸 플레이트를 BOM에 올리지 않는가?**
A. 자석과 SUS430 스틸 디스크는 **케이스 어셈블리 BOM**에 속함(인서트 or 포켓 스냅-인). PCB BOM과 분리. 안테나 간섭 방지용 15mm keepout은 PCB 설계 시점에 반드시 반영.

---

## 연락 및 참고

- **핵심 MCP 서버 (PCB 설계용)**: `teileelektronik/easyeda-mcp` (EasyEDA Pro 제어)
- **PCB 제조**: JLCPCB (EVT/DVT) → 멕시코 EMS (성장기)
- **부품 조달**: Digi-Key/Mouser (EVT용), LCSC (양산용)
- **프로젝트 메모리**: `/Users/junwon/Global PBL Program/project/carpybara_ws/CLAUDE.md`

---

*본 패키지는 2026-04-16 시점 Frozen AVL Rev H 기준으로 작성되었습니다. AVL이 Revision 업데이트될 경우 이 README도 함께 갱신해주세요.*
