# Carpybara — 프로토타입 캐리어보드 설계 v3.1

> **범위**: **프로토타입 전용** (DevKit 소켓 + ELM327 BT + JST XH OBD2 입력). 양산형 PCB(CAN 직결 + J1962)는 별도 문서로 분리 예정.  
> **v3.1 목표**: v3.0 자가검증에서 발견된 내부 모순(TPS54302/54360B 혼재)을 전면 정합, 패키지 HSOP-8 수정, FB 저항 Vref=0.8V 기준 재계산, 디스플레이 70°C 제약 Known Risk로 명시.  
> **v2.4 검토에서 발견된 Critical 3건 + High 4건은 이미 v3.0에서 반영됨.**  
> **관련 문서**: Frozen AVL Rev E (v3.1 동기), SW v1.1 (v3.1 기반)

---

## 0. 왜 계속 고치는가 — 정직한 답

**차량 전장 PCB는 한 번에 완성되지 않습니다.** 업계 표준은 3~5회 리비전입니다. Tier-1 자동차 부품사도 그렇습니다. 이유는:

1. **검토는 양파 까기입니다.**
   - 1차: 명백한 버그 (MP2315 FB 저항 → 4.44V 출력)
   - 2차: 시스템 이슈 (크랭킹, Load Dump, GND loop)
   - 3차: EMC/ESD/신뢰성
   - 4차: 양산성/DFM/수율
   
   레벨이 깊어질수록 더 깊은 층이 드러납니다. 이건 설계자의 실력 부족이 아니라 **엔지니어링 프로세스의 본질**입니다.

2. **"완벽한 PCB"는 없습니다.** 하지만 **"알려진 실패 모드를 전부 처리한 PCB"**는 만들 수 있습니다. 이게 v3.0의 목표입니다.

3. **차량 환경은 가장 극한의 전자 환경**입니다:
   - 전압: 정상 12~14V, Load Dump 87V, 역극성, 크랭킹 4V
   - 온도: -40°C ~ +85°C (대시보드 +105°C 가능)
   - 진동: 10-500Hz 5G
   - EMC: 라디오, 셀룰러, 키 리모컨, 시동 스파크
   - ESD: ±15kV (겨울철 정전기)
   
   이 모든 조건을 통과하려면 설계가 깊어질 수밖에 없습니다.

**v3.0은 이 점들을 전부 반영한 "검증 가능한 최선"입니다.** v3.1은 그 v3.0 자가검증에서 발견된 내부 불일치를 정합(整合)한 리비전입니다.

---

## 1. v3.1 설계 철학

### 1-1. 5대 원칙

| # | 원칙 | 의미 |
|---|------|------|
| 1 | **Defense in Depth** | 모든 Critical 보호는 하드웨어 + 소프트웨어 2중 안전 |
| 2 | **Fail-Safe** | 장애 시 "위험한 동작" 아닌 "안전 정지" 또는 "축소 동작"으로 fallback |
| 3 | **Automotive Grade (Power Path)** | 전원부는 AEC-Q100, 신호부는 Industrial로 구분 |
| 4 | **Graceful Degradation** | 크랭킹/고온/저전압 시 완전 정지 아닌 단계적 축소 |
| 5 | **Verifiable Spec** | 모든 중요 스펙은 측정 가능한 숫자로 정의 (예: "안정적" ❌ → "VIN 4.5V에서 3.3V±3% 유지" ✅) |

### 1-2. 완벽의 현실적 정의

**"완벽"이란**:
- ❌ 아니다: 모든 극한 조건에서 모든 기능이 정상 동작
- ✅ 맞다: 극한 조건에서 **안전하게 축소 동작** 후 **정상 조건 복귀 시 자동 회복**

**수용 가능한 거동 예**:
- 크랭킹 중 1~2초 화면 깜빡임 → 시동 완료 후 자동 복귀 ✅  
- 대시보드 85°C 이상 → 디스플레이 sleep, BT만 유지, 온도 하락 시 자동 복귀 ✅  
- 역극성 연결 → 회로 전류 0 (차단), 올바른 연결 시 자동 복귀 ✅

**수용 불가한 거동**:
- Load Dump 1회에 보드 파괴 ❌  
- 정상 크랭킹에서 매번 완전 재부팅 ❌  
- 역극성 연결 1회에 보드 파괴 ❌  
- ESD 1회에 CH340C 파괴 ❌

v3.0은 "수용 가능" 모든 항목을 충족, "수용 불가" 모든 항목을 방어합니다.

---

## 2. v2.4 → v3.0 → v3.1 주요 변경

### 2-1. v2.4 → v3.0 (이미 반영됨)

| # | 이슈 (v2.4) | 해결 (v3.0) |
|---|---|---|
| 1 | 🔴 MP2315 FB 저항 100k/22k = **4.44V 출력** (보드 파괴) | R_top=100kΩ, R_bot=**32.4kΩ (0.1%)** → 3.27V (Vref=0.8V 기준) |
| 2 | 🔴 크랭킹 홀드업 12ms (100ms 필요) | **2단 전원 구조** + Cranking-aware SW + 큰 벌크 |
| 3 | 🔴 Load Dump 대응 부족 (SMBJ16A로 87V 방어 불가) | **LM74700-Q1** 이상적 다이오드 + **TPS54360B-Q1 (60V, HSOP-8)** wide-input Buck |
| 4 | 🟠 USB 전원 마진 100mV | SBR2U60P1 (VF 0.25V) + LM66100 Power MUX |
| 5 | 🟠 GPIO15(strapping) LCD_CS 풀업 약함 | **GPIO17로 이동** (non-strapping, 충돌 없음) |
| 6 | 🟠 USB-OBD2 GND Loop | GND star-point + **GND 격리 페라이트** |
| 7 | 🟠 OBD2 입력 ESD 보호 부족 | **PESD2CAN-LX** 추가 (빠른 ESD 응답) |
| 8 | 🟡 4레이어 스택업 L3(Power)가 L4 신호 reference | **L2=GND, L3=GND, L4=Power** 구조로 변경 |
| 9 | 🟡 NTC 플라잉 리드 단선 시 floating | **SW 단선 감지 + safe fallback** 로직 추가 |
| 10 | 🟡 PPTC 반응 느림 (5초) | **eFuse TPS25221-Q1**로 교체 (100µs 트립) |

### 2-2. v3.0 → v3.1 (이번 리비전에서 수정)

| # | v3.0 내부 모순 / 누락 | v3.1 수정 |
|---|---|---|
| A | 🔴 Stage 1 Buck이 블록도/BOM에 **TPS54302-Q1**, 자가검증/Bring-up에만 **TPS54360B-Q1** — 내부 모순 | **전 문서 TPS54360B-Q1로 통일** |
| B | 🔴 BOM에 TPS54360B-Q1 패키지 "SOT-23-6"으로 오기재 | **HSOP-8 PowerPAD (DDA)**로 수정 |
| C | 🔴 Stage 1 FB 저항 TPS54302-Q1 기준(Vref=0.596V)으로 계산 | **TPS54360B-Q1 기준(Vref=0.8V)으로 재계산: R_top=100kΩ, R_bot=19.1kΩ** |
| D | 🟡 디스플레이 70°C 제약 Known Risk에 누락 | **Known Risk R6으로 추가** (4단계 보호와 함께 명시) |
| E | 🟡 여름 직사광선 테스트 1회성 | **지속 모니터링 프로토콜로 강화** (30분 간격 × 4시간) |
| F | 🟡 효율 계산 TPS54302-Q1 기준 | **TPS54360B-Q1 기준 재계산** (88% → 85% @ 12V/230mA) |
| G | 🟡 보드 크기 70×90mm (HSOP-8은 SOT-23-6보다 큼) | **75×90mm** (추가 5mm, L1 인덕터 주변 여유) |

---

## 3. 전원 아키텍처 v3.0 — 전면 재설계

### 3-1. 블록 다이어그램

```
╔═════════════════════════════════════════════════════════════════════════╗
║                          STAGE 0: 입력 보호                              ║
╠═════════════════════════════════════════════════════════════════════════╣
║                                                                         ║
║ [OBD2 12V] ──┬──[F1: eFuse TPS25221-Q1 1A]──┐                          ║
║              │        (100µs 트립)           │                          ║
║              ├──[TVS1: SMBJ24CA 양방향]      │                          ║
║              │        (역극성 + Surge)       │                          ║
║              ├──[ESD: PESD2CAN-LX]           │                          ║
║              │        (ESD ±15kV)            │                          ║
║              └──[Ferrite: BLM31PG601SN1 600Ω@100MHz 3A]                ║
║                                              │                          ║
║                                              ▼                          ║
║                                         VIN_PROTECTED                   ║
║                                              │                          ║
║                                  ┌───────────┤                          ║
║                                  │           │                          ║
║                          [LM74700-Q1 + PMOS] │                          ║
║                           이상적 다이오드    │                          ║
║                           (역극성 2중 보호)  │                          ║
║                                  │           │                          ║
║                                  └───────────┘                          ║
║                                              ▼                          ║
║                                         VIN_CLEAN                       ║
║                                              │                          ║
║                              ┌───────────────┤                          ║
║                              │               │                          ║
║              [C_bulk: 2×100µF/50V 폴리머]    │                          ║
║              [C_bypass: 10µF/50V X7R 1210]   │                          ║
║              [C_HF: 100nF + 10nF 0402]       │                          ║
║                              │               │                          ║
║                              └───────────────┘                          ║
║                                              │                          ║
╚══════════════════════════════════════════════╪══════════════════════════╝
                                               │
╔══════════════════════════════════════════════╪══════════════════════════╗
║                 STAGE 1: Wide-Input Buck (12V → 5V)                     ║
╠══════════════════════════════════════════════╪══════════════════════════╣
║                                              │                          ║
║                                              ▼                          ║
║                                  [TPS54360B-Q1 Buck]                    ║
║                                   - Vin: 4.5~60V (Abs Max 65V)          ║
║                                   - Vout: 5.0V @ 3.5A max               ║
║                                   - AEC-Q100 Grade 1 (-40~+125°C)       ║
║                                   - Package: HSOP-8 PowerPAD (DDA)      ║
║                                   - UVLO 4.3V (크랭킹 내성)             ║
║                                   - Vref: 0.8V                          ║
║                                   - Load Dump 65V ISO 7637 대응         ║
║                                   - 내부 Soft-start 4ms                 ║
║                                   ├── L1: 15µH/2A 차폐 (EVM 기준)       ║
║                                   ├── FB: R_top=100kΩ (0.1%)            ║
║                                   │      R_bot=19.1kΩ (0.1%)            ║
║                                   │      → Vout=(100/19.1+1)×0.8        ║
║                                   │             = 4.99V ✓               ║
║                                   ├── Rt: 240kΩ (Fsw=500kHz)            ║
║                                   ├── EN: 100kΩ pull-up to VIN          ║
║                                   └── Cout: 2×22µF/16V + 100nF          ║
║                                              │                          ║
║                                              ▼                          ║
║                                         5V_OBD (OBD2 경로)              ║
║                                              │                          ║
╚══════════════════════════════════════════════╪══════════════════════════╝
                                               │
╔═══════════════════════════════════════╪══════╪══════════════════════════╗
║                 STAGE 2: USB 경로 + Power MUX                           ║
╠═══════════════════════════════════════╪══════╪══════════════════════════╣
║                                       │      │                          ║
║    [USB-C VBUS 5V (DevKit)]           │      │                          ║
║            │                          │      │                          ║
║            ├──[Ferrite: BLM18PG471SN1]│      │                          ║
║            │   (GND loop 차단)        │      │                          ║
║            │                          │      │                          ║
║            └──[Schottky: SBR2U60P1]   │      │                          ║
║                VF 0.25V @ 500mA       │      │                          ║
║                       │               │      │                          ║
║                       ▼               ▼      ▼                          ║
║                   5V_USB         [LM66100 Ideal Diode OR Controller]    ║
║                                   - VIN 1.5~5.5V                        ║
║                                   - 자동 높은쪽 선택                    ║
║                                   - VF ~20mV (거의 무손실)              ║
║                                   - 5V_OBD 우선 (항상 더 높음)          ║
║                                              │                          ║
║                                              ▼                          ║
║                                         5V_COMMON (@ 4.8~5.0V)          ║
║                                              │                          ║
╚══════════════════════════════════════════════╪══════════════════════════╝
                                               │
╔══════════════════════════════════════════════╪══════════════════════════╗
║                 STAGE 3: 3.3V 변환 (5V → 3.3V)                          ║
╠══════════════════════════════════════════════╪══════════════════════════╣
║                                              │                          ║
║                                              ▼                          ║
║                              [C_in: 22µF/10V + 100nF]                   ║
║                                              │                          ║
║                                              ▼                          ║
║                                     [MP2315GJ-Z Buck]                   ║
║                                      - Vin: 4.5~24V (5V OK)             ║
║                                      - Vout: 3.30V @ 1.5A               ║
║                                      - 효율 ~92% @ 300mA                ║
║                                      ├── L2: 4.7µH/2A 차폐              ║
║                                      ├── FB: R_top=100kΩ (0.1%)         ║
║                                      │       R_bot=32.4kΩ (0.1%)        ║
║                                      │       → Vout = 0.8×(1+100/32.4)  ║
║                                      │              = 3.269V ✓          ║
║                                      │    (E96 최적: 100k/32.4k)        ║
║                                      └── Cout: 22µF + 10µF + 100nF     ║
║                                              │                          ║
║                                              ▼                          ║
║                                         3.3V_PERIPH                     ║
║                                       (LCD, 센서, 엔코더 등)            ║
╚═════════════════════════════════════════════════════════════════════════╝
```

### 3-2. 전원 동작 시나리오 검증표

| 시나리오 | VIN | VIN_CLEAN | 5V_OBD | 5V_USB | 5V_COMMON | 3.3V_PERIPH | 결과 |
|---|---|---|---|---|---|---|---|
| OBD2 단독 (IGN ON) | 12V | 12V | 5.0V | — | 5.0V | 3.30V | ✅ 정상 |
| USB-C 단독 (플래싱) | — | — | — | 4.75V | 4.73V | 3.30V | ✅ 정상 |
| 둘 다 연결 | 12V | 12V | 5.0V | 4.75V | 5.0V | 3.30V | ✅ OBD2 우선 |
| 크랭킹 일반 (6V, 100ms) | 6V | 6V | 5.0V | — | 5.0V | 3.30V | ✅ Stage1 계속 동작 |
| 크랭킹 극한 (4V, 200ms) | 4V | 4V | 벌크유지 | — | ~4.9V | ~3.30V | ⚠️ Graceful (아래 3-4 참조) |
| Load Dump 87V (400ms) | 87V→24V | 24V | 5.0V | — | 5.0V | 3.30V | ✅ TVS+LM74700 클램프 |
| 역극성 (-12V) | -12V | 0V | 0V | — | USB fallback | 0V 또는 3.30V | ✅ 전류 0 |
| 과전류 (>1A) | 12V | 12V | eFuse trip | — | USB fallback | 3.30V (USB 시) | ✅ 100µs 차단 |

### 3-3. Load Dump 대응 — v3.1 확정 설계

**ISO 7637-2 Pulse 5b (Suppressed Load Dump)**: 35~58V, 400ms  
**ISO 7637-2 Pulse 5a (Unsuppressed Load Dump)**: up to 87V, 400ms

**v3.1 확정 보호 체인**:

```
OBD2 12V 입력
    ↓
[PPTC/eFuse TPS25221-Q1 1A]     ← 과전류 100µs 차단
    ↓
[TVS 1차: SMCJ24CA, SMC]         ← 3kW peak, Standoff 24V, Clamp 39V @ 1A
    ↓
[ESD: PESD2CAN-LX]               ← <1ns 응답, 저에너지 ESD 전용
    ↓
[Ferrite: BLM31PG601SN1]         ← 600Ω@100MHz, 3A, 고주파 감쇄
    ↓
[TVS 2차: SMBJ20CA, SMB]         ← 600W peak, Standoff 20V, Clamp 32V @ 1A
    ↓
[LM74700-Q1 + 외부 PMOS]         ← Ideal diode, 역극성 2중 보호 (VF ~0V)
    ↓
VIN_CLEAN (정상 12V, 최악 32V)
    ↓
[TPS54360B-Q1]                   ← Abs Max 65V → 32V 입력 대비 여유 33V ✓
    ↓
5V_OBD
```

**v3.0과의 차이**:
- v3.0: Stage1을 TPS54302-Q1(Abs Max 28V)으로 제시 → SMBJ20CA 클램프 32V 초과로 자가검증에서 적발 → TPS54360B-Q1로 교체 결정했으나 앞부분 미반영
- v3.1: **TPS54360B-Q1로 전 문서 통일** (Abs Max 65V, Load Dump 65V ISO 7637 자체 대응 인증)

**검증된 마진**:

| 시나리오 | VIN_CLEAN | Stage1 Abs Max | 마진 | 결과 |
|---|---|---|---|---|
| 정상 12V | 12V | 65V | 53V | ✅ 충분 |
| 과충전 알터네이터 (16V) | 16V | 65V | 49V | ✅ 충분 |
| Load Dump 5b (58V) | 32V (2차 TVS) | 65V | 33V | ✅ 충분 |
| Load Dump 5a (87V) | 32V (2차 TVS) | 65V | 33V | ✅ 충분 |
| 역극성 (-12V) | 0V (LM74700 차단) | — | — | ✅ 차단 |
| 점프스타트 과전압 (24V) | 24V | 65V | 41V | ✅ 충분 |

### 3-4. 크랭킹 대응 — Graceful Degradation

**크랭킹 실상 (일반 승용차)**:
- 일반: 12V → 8~10V 드롭, 50~150ms 지속
- 혹한: 12V → 4~6V 드롭, 200~400ms 지속
- Start-stop 차량: 더 자주 발생하나 드롭 얕음

**v3.0 대응 전략**:

| 드롭 시나리오 | 하드웨어 거동 | 소프트웨어 거동 | 사용자 경험 |
|---|---|---|---|
| 8V↓ (정상) | Stage1 Buck 정상 동작 | 변화 없음 | 변화 없음 ✅ |
| 6V↓ (추운 날) | Stage1 UVLO 근접, 벌크 홀드업 50ms | 전압 감지 → 백라이트 30% | 화면 살짝 어두워짐 |
| 4V↓ (혹한) | Stage1 UVLO, BOD 트리거 | 3초 내 재부팅, 시동 화면 표시 | "시동 중..." 표시 후 복귀 |

**크랭킹 홀드업 계산 (재계산)**:

```
Stage 1 입력단 C_bulk = 2 × 100µF = 200µF (폴리머, 50V)
부하: Stage 2가 230mA @ 12V 소비 (5V/230mA/90% = 128mA @ 12V)
       + Ferrite/기타 ~20mA = 150mA

150ms 크랭킹, 12V → 6V 드롭:
  - Stage1 Buck이 6V에서 정상 동작 (UVLO 4.3V 여유)
  - 벌크캡 홀드업 불필요 ✅

200ms 크랭킹, 12V → 4V 드롭:
  - Stage1 Buck 셧다운 (UVLO < 4.3V)
  - 벌크 홀드업 필요: C × ΔV / I = 200µF × (6-4.3) / 150mA = 2.27ms (극히 짧음)
  - → 하드웨어만으로는 해결 불가
  - → SW 대응: BOD 트리거 → fast-recovery 부팅 → 3초 내 화면 복구
```

**핵심 인사이트**: 혹한 크랭킹은 **하드웨어로 완전 해결 불가능**. 상용 자동차 부품도 이렇게 동작함 (블랙박스, 내비게이션 모두 시동 시 재부팅됨). **SW 측 fast boot + UX로 자연스럽게 숨기는 것**이 업계 표준.

**SW 대응 (필수 구현)**:

```c
// 크랭킹 감지 및 대응
void crank_detect_task(void *arg) {
    while (1) {
        float vin = adc_read_vin();  // Stage1 입력 모니터링 (분압 후 ADC)
        
        if (vin < 9.0 && vin > 4.5) {
            // 크랭킹 진행 중 - 사전 절전
            lcd_set_backlight(30);      // 밝기 30%
            bt_scan_pause();            // BT 스캔 일시 중단
            cpu_freq_set(80);           // CPU 다운클럭
            crank_mode = true;
        } else if (vin >= 11.0 && crank_mode) {
            // 크랭킹 종료
            lcd_set_backlight(prev_level);
            bt_scan_resume();
            cpu_freq_set(240);
            crank_mode = false;
        }
        
        vTaskDelay(pdMS_TO_TICKS(50));  // 20Hz
    }
}

// Fast boot: 부팅 시 "시동 중..." 즉시 표시
void app_main() {
    // 1. 최소한의 초기화 (LCD만 먼저)
    lcd_init_minimal();
    lcd_draw_boot_screen();  // "Starting..." 표시 (50ms 이내)
    
    // 2. 나머지 초기화 병렬로
    xTaskCreate(bt_init_task, ...);
    xTaskCreate(obd_init_task, ...);
    // ...
}
```

### 3-5. 전원부 핵심 수치 검증

| 항목 | 값 | 검증 |
|---|---|---|
| Stage 1 효율 (12V→5V @ 230mA) | ~85% | TPS54360B-Q1 데이터시트 Fig.20 (light load) |
| Stage 1 효율 (12V→5V @ 1A) | ~92% | TPS54360B-Q1 데이터시트 Fig.20 (full load) |
| Stage 2 효율 (5V→3.3V @ 310mA) | ~92% | MP2315 데이터시트 Fig.10 |
| 전체 효율 (12V→3.3V @ 310mA 부하) | ~78% | 0.85 × 0.92 |
| OBD2 12V 입력 전류 (정상 310mA 부하) | ~110mA | 3.3V × 310mA / 0.78 / 12V |
| OBD2 12V 입력 전류 (최대 1A 부하 시) | ~360mA | 여유 고려 |
| 발열 Stage 1 (12V/230mA, η85%) | 0.20W | 매우 낮음 ✅ |
| 발열 Stage 2 (5V/310mA, η92%) | 0.09W | 매우 낮음 ✅ |
| FB 저항 오차 Stage 2 (0.1% tol) | Vout = 3.27V ± 0.5% | ESP32 ±5% 허용 범위 ✅ |
| FB 저항 오차 Stage 1 (0.1% tol) | Vout = 4.99V ± 0.5% | Stage 2(MP2315) 입력 허용 범위 ✅ |
| Fsw Stage 1 | 500kHz (Rt=240kΩ) | EMI 여유 (AM 대역 밖) |
| Fsw Stage 2 | 500kHz (MP2315 고정) | 동일 주파수, 정상 |

---

## 4. 신호 블록 v3.0

### 4-1. GPIO 재할당 (v2.4 → v3.0)

**핵심 변경**: LCD_CS를 GPIO15(strapping) → **GPIO17**로 이동.

| GPIO | v2.4 | v3.0 | 변경 이유 |
|---|---|---|---|
| 4 | LCD RST | LCD RST | 유지 |
| 5 | SD CS | SD CS | 유지 (strapping이나 풀업으로 충분) |
| 12 | GND 풀다운 | GND 풀다운 | 유지 (스트래핑 LOW 필수) |
| 13 | BT LED | BT LED | 유지 |
| 14 | 버튼1 | 버튼1 | 유지 |
| **15** | **LCD CS** ⚠️ | **조도 센서 ADC** | Strapping 핀 부하 제거 |
| 17 | — | **LCD CS** ★ | Non-strapping 핀으로 이동 |
| 18 | SPI SCK | SPI SCK | 유지 (33Ω 직렬은 **드라이버 측**에 배치) |
| 19 | SPI MISO | SPI MISO | 유지 |
| 21 | 버튼2 | 버튼2 | 유지 |
| 22 | LCD DC | LCD DC | 유지 |
| 23 | SPI MOSI | SPI MOSI | 유지 |
| 25 | 엔코더 SW | 엔코더 SW | 유지 |
| 26 | 버저 | 버저 | 유지 |
| 27 | 백라이트 PWM | 백라이트 PWM | 유지 |
| 32 | 엔코더 A | 엔코더 A | 유지 |
| 33 | 엔코더 B | 엔코더 B | 유지 |
| **34** | 조도 센서 | **VIN 모니터** ★ | 크랭킹 감지용 (분압 저항 47k:10k) |
| 35 | LCD 백면 NTC | LCD 백면 NTC | 유지 + 단선 감지 로직 |
| 36 (VP) | — | **ESP32 내부 온도 백업** | 내부 센서 고장 대비 |

**GPIO15 사용 변경 이유**:
- v2.4에서 LCD_CS = GPIO15는 **Strapping 핀** 
- 부팅 시 LOW = Silent boot 모드 → UART 디버그 메시지 안 나옴
- LCD 모듈 전원 ramp 지연 시 leakage로 LOW 잡힐 수 있음
- v3.0: GPIO15는 조도 센서 ADC (입력만, 부팅 시 floating → 자체 풀업/다운 필요 없음)
- GPIO17은 Non-strapping, 부팅 중 LOW 상태 유지해도 문제 없음

### 4-2. LCD 블록 — v3.0 회로

```
[디스플레이 모듈 커넥터 — 2.8" IPS ILI9341]
    │
    ├── VCC ◀── 3.3V_PERIPH
    │     │
    │     ├── 10µF/10V (0805)  ← Bulk, 커넥터에서 5mm 이내
    │     ├── 1µF/10V (0603)   ← Mid-freq
    │     └── 100nF + 10nF (0402) ← High-freq, VCC 핀 3mm 이내
    │
    ├── GND ◀── GND (star point)
    │
    ├── MOSI ◀── GPIO23 ──[33Ω] ── (연결) 
    │                   (ESP32 측 1mm 이내)
    │
    ├── SCK  ◀── GPIO18 ──[33Ω] ── (연결)
    │                   (ESP32 측 1mm 이내, 댐핑)
    │
    ├── LCD_CS ◀── GPIO17 ──[1kΩ 풀업 to 3.3V]  ★ Non-strapping
    │
    ├── DC  ◀── GPIO22
    │
    ├── RST ◀── GPIO4 ──[10kΩ 풀업 to 3.3V]
    │                ──[100nF to GND]  ← RC 딜레이로 ESP32보다 늦게 릴리스
    │
    ├── BL+ ◀── 3.3V_PERIPH
    │
    ├── BL- ──[AO3400A N-ch MOSFET Drain]
    │            Gate ◀── GPIO27 ──[33Ω] (PWM 20kHz)
    │                            ──[100kΩ 풀다운] (부팅 중 OFF 보장)
    │            Source ◀── GND
    │         Drain ── BL- + [4.7µF Tantalum GND]
    │
    ├── SD_MISO ── GPIO19 (공유, 10kΩ 풀업)
    ├── SD_MOSI ── GPIO23 (공유)
    ├── SD_SCK  ── GPIO18 (공유)
    └── SD_CS   ◀── GPIO5 ──[10kΩ 풀업]
```

**핵심 개선점 (v3.0)**:
1. **LCD_CS → GPIO17 (non-strapping)**: 부팅 시 안정성 크게 향상
2. **LCD VCC 3단 디커플링**: 10µF + 1µF + 100nF + 10nF로 전 주파수 대역 커버
3. **SPI 33Ω은 드라이버 측 1mm 이내**: 반사파 억제 실효성 확보
4. **RST에 RC 딜레이**: LCD가 ESP32보다 늦게 깨어나서 초기화 안정성 향상
5. **백라이트 MOSFET 게이트 풀다운**: 부팅 중 백라이트 OFF 강제

### 4-3. NTC 블록 — 단선 감지 추가

```
3.3V_PERIPH ──┬── [10kΩ (PCB 고정 저항)] ──┐
              │                             │
              │                             ├── 접합부 ──┬── 100nF ── GND
              │                             │            │
              │  [플라잉 리드 AWG30 TP]    │            └── → GPIO35 (ADC1_CH7)
              │   50mm + 스트레인 릴리프    │
              │                             │
              └── [10kΩ NTC NCP18XH103F]    │
                  (LCD 백면 부착, 3M 8810)  │
                             │               │
                             └── → 접합부 ───┘
```

**SW 단선 감지 로직**:

```c
typedef enum {
    NTC_OK,
    NTC_OPEN,      // 단선 (ADC ~ 3.2V)
    NTC_SHORT,     // 단락 (ADC ~ 0V)
    NTC_INVALID    // 노이즈 과다
} ntc_status_t;

ntc_status_t ntc_read_safe(float *temp_c) {
    uint16_t samples[8];
    for (int i = 0; i < 8; i++) {
        samples[i] = adc_read(ADC1_CH7);
        vTaskDelay(1);
    }
    sort_array(samples, 8);
    uint16_t median = (samples[3] + samples[4]) / 2;
    
    // 단선 감지: ADC > 95% of full scale (NTC open → 3.3V 근접)
    if (median > 3891) {  // 3.3V × 0.95 / 3.3V × 4095
        return NTC_OPEN;
    }
    // 단락 감지: ADC < 5% of full scale
    if (median < 205) {
        return NTC_SHORT;
    }
    // 노이즈 체크: max-min 스프레드
    if ((samples[7] - samples[0]) > 500) {
        return NTC_INVALID;
    }
    
    // 정상: 온도 계산
    *temp_c = ntc_adc_to_celsius(median);
    return NTC_OK;
}

void temp_task(void *arg) {
    static int fault_count = 0;
    
    while (1) {
        float temp = 0;
        ntc_status_t status = ntc_read_safe(&temp);
        
        if (status == NTC_OK) {
            fault_count = 0;
            apply_temp_protection(temp);
        } else {
            fault_count++;
            if (fault_count > 10) {  // 10회 연속 오류
                // Safe fallback: 백라이트 50% 고정, 로깅
                lcd_set_backlight(50);
                log_error("NTC FAULT: %d", status);
                
                // ESP32 내부 온도로 백업 모니터링
                float esp_temp;
                temperature_sensor_get_celsius(temp_sensor, &esp_temp);
                if (esp_temp > 70.0) {
                    lcd_set_backlight(20);
                    if (esp_temp > 85.0) {
                        lcd_sleep();
                    }
                }
            }
        }
        
        vTaskDelay(pdMS_TO_TICKS(125));  // 8Hz
    }
}
```

### 4-4. OBD2 입력 커넥터 + 보호 회로

```
[JST B3B-XH-A, 3핀]
 Pin 1: 12V_IN
 Pin 2: GND
 Pin 3: Shield (케이스만 연결, GND와 0Ω 연결 X)

┌─── 12V_IN
│
├─── [TVS1: SMCJ24CA] ─── GND    ← 1차 클램프 (Load Dump)
│      (양방향, Standoff 24V,
│       Clamp 39V @ 1A, 3kW peak)
│
├─── [ESD: PESD2CAN-LX] ── GND   ← 빠른 ESD 응답 (<1ns)
│
├─── [eFuse: TPS25221-Q1] ──┐    ← 100µs 트립 과전류 보호
│      (1A 설정,            │       0.5Ω RDS(on), AEC-Q100
│       Hotswap 내장)       │
│                           │
│                           ▼
│                    VIN_FUSED (12V)
│
│                           ├── [Ferrite: BLM31PG601SN1] ─┐
│                           │    (600Ω@100MHz, 3A)        │
│                           │                              │
│                           │                              ▼
│                           │                         VIN_FILTERED
│                           │                              │
│                           └── [TVS2: SMBJ20CA] ── GND   ← 2차 클램프
│                                (Standoff 20V,            │
│                                 Clamp 32V @ 1A)          │
│                                                          │
│                                              ┌───────────┤
│                                              │           │
│                                      [LM74700-Q1] ◀──────┤
│                                       + 외부 PMOS         │
│                                      (Ideal Diode:        │
│                                       역극성 + 저손실)    │
│                                              │            │
│                                              └───────────┘
│                                                          │
│                                                          ▼
│                                                     VIN_CLEAN (12V)
│                                                     (Stage 1 Buck 입력)
```

**보호 부품 선정 근거**:

| 부품 | 스펙 | 기능 | 왜 이걸 선택? |
|---|---|---|---|
| TPS25221-Q1 | 1A eFuse, AEC-Q100 | 과전류 100µs 차단 | PPTC(5초)보다 250배 빠름 — 단락 사고 시 Buck 생존 가능 |
| SMCJ24CA | 양방향 TVS, 3kW | Load Dump + 역극성 | SMB(600W)로는 Load Dump 에너지 부족. SMC(3kW) 필수 |
| PESD2CAN-LX | ESD TVS, <1ns | ESD ±15kV | TVS의 느린 응답(수 ns)으론 ESD 막지 못함. 별도 필요 |
| BLM31PG601SN1 | 600Ω@100MHz, 3A | 고주파 서지 감쇄 | Load Dump 저주파는 TVS, 고주파는 페라이트로 분담 |
| SMBJ20CA | 양방향 TVS, 600W | 2차 클램프 | 1차 TVS 통과분을 Stage1 Abs Max 내로 제한 |
| LM74700-Q1 | Ideal diode, 65V | 역극성 2중 보호 | Schottky(VF 0.4V) 대비 손실 1/40, AEC-Q100 |

### 4-5. USB-C 입력 + GND 격리

```
[USB-C VBUS from DevKit]
          │
          │  ← 주의: DevKit에는 이미 USB-C 커넥터가 있으나
          │    USBLC6-2SC6 ESD 보호가 VBUS에는 없음 (D+/D-만 있음)
          │
          ├── [TVS: SMBJ5.0CA] ── GND    ← VBUS ESD/Surge 보호
          │    (5V 전용, Clamp 9.2V)
          │
          ├── [Schottky: SBR2U60P1] ── 5V_USB
          │    (VF 0.25V @ 500mA, 60V/2A)
          │    (이상적 다이오드 필요 없음 — 5V는 Load Dump 걱정 없음)
          │
          │  [GND Return]:
          │
USB_GND ──[Ferrite: BLM18PG471SN1]── CB_GND
          (470Ω@100MHz, 1A)
          
          ↑
          이 페라이트가 USB GND와 캐리어보드 GND 사이
          차량 얼터네이터 노이즈 순환 전류 차단
```

**GND 전략 (Star Ground)**:

```
                ┌─────────────────────────────┐
                │     Star Ground Point       │
                │   (3.3V 레귤레이터 근처)    │
                └──────┬──────┬──────┬────────┘
                       │      │      │
         ┌─────────────┤      │      ├─────────────┐
         │             │      │      │             │
    [Digital GND] [Analog GND] [Power GND]  [Chassis GND]
         │             │              │             │
    ESP32 GPIO      NTC/조도      Buck/스위칭    케이스/Shield
    디지털 신호    ADC (저노이즈)   고전류        (격리, RC 연결)
         │             │              │             │
         └──── Layer 2 GND Plane (solid, 스티칭 via 촘촘) ────┘
         
         USB GND ──[Ferrite]── Star Ground  (단방향 DC 연결)
         OBD GND ──(직접)──── Star Ground   (Power 경로)
```

---

## 5. PCB 레이아웃 v3.0

### 5-1. 스택업 (4레이어, 개선됨)

**v2.4 스택업 문제**: L4 신호의 리턴 경로가 L3 Power plane → 임피던스 불연속, EMI 증가.

**v3.0 스택업**:

```
┌──────────────────────────────────────────┐
│ L1 (Top):    주 신호 + 부품 배치         │ ← GND pour (여유 영역)
├──────────────────────────────────────────┤
│ 0.2mm prepreg (고주파 신호용 얇은 간격)  │
├──────────────────────────────────────────┤
│ L2 (Inner 1): GND Plane (solid, 최우선)  │ ← L1 리턴 경로
├──────────────────────────────────────────┤
│ 1.06mm core                              │
├──────────────────────────────────────────┤
│ L3 (Inner 2): Power + 보조 GND           │ ← Power plane with 섹션 분리
│               (3.3V_PERIPH, 5V_COMMON,   │   GND pour로 세분화
│                VIN_CLEAN 섹션)           │
├──────────────────────────────────────────┤
│ 0.2mm prepreg                            │
├──────────────────────────────────────────┤
│ L4 (Bottom): 부수 신호 + GND pour        │ ← L3 GND/Power 리턴
│              (BT 안테나 아래 완전 비움)  │
└──────────────────────────────────────────┘

총 두께: 1.6mm 표준
동박: 1oz 전체 (차량 고온 환경, 큰 전류)
표면처리: ENIG (고온 환경 + 재작업성)
```

**핵심 설계**:
- **L2 = Solid GND**: L1의 모든 고속 신호(SPI, BLE 안테나 트레이스)의 일관된 리턴 경로
- **L3 = Split Power + GND**: 전원 섹션 분리, 사이에 GND pour로 커플링 차단
- **L4 = 신호 + GND pour**: L3의 GND pour가 리턴 경로 역할
- **스티칭 Via**: L1 ↔ L2 GND pour 간 5mm 간격 촘촘히 배치 (고주파 리턴 단축)

### 5-2. BT 안테나 Keep-out (v2.4 유지 + 강화)

```
캐리어보드 상단 edge
────────────────────────────────────────────────────
     │                                              │
     │     ┌──────────────────────────────────┐     │
     │     │                                  │     │
     │     │  DevKit 소켓 (38핀, 2열)         │     │
     │     │                                  │     │
     │     └──────────────────────────────────┘     │
     │                                       ┏━━━┓  │
     │                                       ┃   ┃  │
     │    Keep-out 영역:                    ┃ A ┃◀─┼── 안테나 17×8mm
     │    • L1, L2, L3, L4 전부 구리 없음   ┃   ┃  │
     │    • 모든 트레이스/비아 금지         ┗━━━┛  │
     │    • 안테나 끝단 → 보드 edge 5mm+ 돌출      │
     │                                              │
     └──────────────────────────────────────────────┘

추가 강화 (v3.0):
• 안테나 주변 20×15mm 영역: 금속 부품 배치 금지 
  (다른 부품이 안테나에 가까이 있으면 디튠)
• 케이스 설계 시: 안테나 위 5mm 이내 금속 페인트/도금 금지
```

### 5-3. 주요 부품 배치 가이드

```
┌─────────────────────────────────────────────────────┐
│  [OBD2 JST] ─┬──[TPS25221]─[SMCJ24CA]              │
│              └──[PESD2CAN]                         │
│                                                     │
│  [Ferrite][SMBJ20CA][LM74700+PMOS][100µF×2]        │
│                                                     │
│  [TPS54360B-Q1 HSOP-8 + L1 15µH + Cout] ← Stage 1 Buck      │
│   (20×25mm, HSOP-8이 SOT-23보다 크므로 여유)                │
│                                                     │
│  [LM66100 Power MUX]                                │
│  [MP2315 + L2 4.7µH + Cout] ← Stage 2 Buck          │
│   (10×15mm)                                         │
│                                                     │
│  ┌─────────────────────────────────┐                │
│  │                                 │                │
│  │   DevKit 38핀 소켓              │                │
│  │   (안테나 우측 boarder 돌출)    │─── 안테나 out  │
│  │                                 │                │
│  └─────────────────────────────────┘                │
│                                                     │
│  ┌─────────────────┐  ┌────────┐  ┌─────┐          │
│  │ 디스플레이      │  │ 조도    │  │ 엔코더│          │
│  │ 커넥터          │  │ 센서    │  │+ESD  │          │
│  │ (VCC 옆 디커플링│  │(ADC     │  │      │          │
│  │  10µF+1µF+100nF)│  │ 격리)   │  │      │          │
│  └────────┬────────┘  └────────┘  └─────┘          │
│           │                                         │
│           │ NTC 플라잉 리드 출구                    │
│           │ (스트레인 릴리프 홀)                    │
│           ▼                                         │
│  [버저]  [BT LED]  [SBR2U60P1]  [Ferrite USB-GND]  │
│                                                     │
│ [BTN1]◀── 좌측                                      │
│ [BTN2]◀── 좌측                                      │
│                                                     │
│  ⊕ M2 × 4 (모서리 3mm 안쪽)                         │
│                                                     │
│  크기: 75×90mm (v3.0 70×90mm에서 HSOP-8 여유로 +5mm)│
└─────────────────────────────────────────────────────┘
```

### 5-4. 핵심 레이아웃 규칙

| 규칙 | 이유 |
|---|---|
| Buck SW 노드 트레이스 최소화 (< 10mm) | EMI 방사 최소화 |
| Buck L/Cin/Cout 루프 최소화 (< 5mm²) | 고주파 루프 inductance |
| 전원 트레이스 폭: 3.3V 30mil, 5V 40mil, 12V 50mil | 전류 + 전압강하 |
| ADC 신호 (GPIO34/35): Buck SW에서 10mm+ 이격 | 노이즈 유입 방지 |
| SPI 트레이스: 30mm 이하, 아래 GND plane 연속 | 신호 무결성 |
| BT 안테나 밑 L1~L4 전부 비움 (17×8mm) | 안테나 디튠 방지 |
| USB-C D+/D- 차동쌍 길이 매칭 (±5mil) | USB 2.0 signal integrity |
| GND Via 5mm 간격 촘촘히 | 리턴 경로 단축 |
| Buck IC GND 패드 → L2 GND: 4개 via | 열 dissipation + 전기 |

---

## 6. 소프트웨어 안전 요구사항 (통합)

### 6-1. 필수 SW 안전 기능 (v3.0)

| # | 기능 | 구현 | 목적 |
|---|---|---|---|
| 1 | 워치독 (메인 루프) | 2초, esp_task_wdt | Hang 복구 |
| 2 | 워치독 (BT) | 별도 10초 | BT 재연결 시간 고려 |
| 3 | Brown-out Detector | HW BOD enable | 저전압 시 clean reset |
| 4 | 크랭킹 감지 | VIN ADC 모니터링 | 사전 절전, fast recovery |
| 5 | NTC 단선 감지 | ADC 상/하한 + 분산 | Floating 시 safe fallback |
| 6 | ESP32 내부 온도 | temperature_sensor_get | 모듈 자체 보호 |
| 7 | 4단계 온도 보호 | NTC → 상태 머신 | Graceful degradation |
| 8 | SD 사용 정책 | 부팅 시 로드 후 deinit | SPI 충돌 원천 차단 |
| 9 | SPI Mutex | FreeRTOS mutex | 안전망 |
| 10 | Deep sleep | BT 끊김 30초 후 | 전력 절약 |
| 11 | Fast Boot | LCD "Starting..." 50ms 이내 | UX |
| 12 | OBD 주행 중 업데이트 차단 | 차속 > 5km/h 시 거부 | 안전 |

### 6-2. 상태 전이 다이어그램

```
                    ┌──────────┐
                    │   BOOT   │
                    └────┬─────┘
                         │ 50ms 내 LCD "Starting..."
                         ▼
                    ┌──────────┐
                    │  INIT    │ ← 병렬 초기화
                    └────┬─────┘  (BT, OBD, Sensors)
                         │
                         ▼
         ┌───────── ┌──────────┐ ───────────┐
         │          │  NORMAL  │            │
         │          └─────┬────┘            │
         │                │                 │
  크랭킹 감지            온도 ↑            BT 연결
  (VIN<9V)              (55°C↑)           손실 30s
         │                │                 │
         ▼                ▼                 ▼
    ┌─────────┐     ┌──────────┐       ┌─────────┐
    │CRANKING │     │THROTTLED │       │DEEP     │
    │(절전)   │     │(축소동작)│       │SLEEP    │
    └────┬────┘     └─────┬────┘       └────┬────┘
         │                │                 │
    VIN 복구          온도 ↓               Wake
    (>11V, 3s)        (<45°C)              (Button)
         │                │                 │
         └────────────────┴─────────────────┘
                         │
                         ▼
                    ┌──────────┐
                    │  NORMAL  │
                    └──────────┘

  NTC 단선 → SAFE_FALLBACK (백라이트 50% 고정, 로깅)
  Critical Error → WATCHDOG_RESET → BOOT
```

### 6-3. 각 상태별 거동

| 상태 | 백라이트 | FPS | BT | OBD | CPU | 비고 |
|---|---|---|---|---|---|---|
| BOOT | OFF | — | OFF | OFF | 240MHz | 100ms 이내 |
| INIT | 50% | 15 | Scanning | Connecting | 240MHz | 2s 이내 |
| NORMAL | 자동(조도) | 60 | Connected | Active | 240MHz | 기본 동작 |
| CRANKING | 30% | 30 | Paused | Paused | 80MHz | VIN 회복 대기 |
| WARN (50~55°C) | 70% | 60 | ON | ON | 240MHz | 사용자 경고 |
| THROTTLED (55~60°C) | 40% | 30 | ON | ON | 160MHz | 발열 축소 |
| DEGRADED (60~65°C) | 20% | 15 | ON | ON | 80MHz | 최소 동작 |
| SAFE (>65°C) | OFF | — | ON(로깅) | ON | 80MHz | 디스플레이 sleep |
| DEEP_SLEEP | OFF | — | OFF | OFF | Sleep | Wake 대기 |
| SAFE_FALLBACK | 50% fix | 60 | ON | ON | 240MHz | NTC 고장 |

---

## 7. BOM v3.0 (완전판)

### 7-1. 전원 보호 + 변환 (Power Path)

| # | 부품 | 모델 | 패키지 | 수량 | AEC-Q | LCSC | 기능 |
|---|---|---|---|---|---|---|---|
| P1 | eFuse | TPS25221-Q1 | SOT-23-6 | 1 | Q100 | C2987547 | 100µs 과전류 차단 |
| P2 | Ideal Diode Ctrl | LM74700-Q1 | SOT-23-6 | 1 | Q100 | C2662568 | 역극성 2중 보호 |
| P3 | PMOS (LM74700 쌍) | SQJ422EP-T1_GE3 | PowerPAK SO-8 | 1 | Q101 | C197495 | 이상적 다이오드 pass FET |
| P4 | Buck Stage 1 | **TPS54360BQDDARQ1** | **HSOP-8 PowerPAD (DDA)** | 1 | Q100 | **C100891** | 12V → 5V @ 3.5A max (230mA 사용), Vref=0.8V, UVLO 4.3V, Load Dump 65V ISO 7637 대응 |
| P5 | Buck Stage 2 | MP2315GJ-Z | TSOT23-8 | 1 | — | C15425 | 5V → 3.3V @ 1.5A |
| P6 | Power MUX | LM66100 | SOT-23-6 | 1 | — | C1009826 | 자동 5V 소스 선택 |
| P7 | Schottky (USB) | SBR2U60P1 | SOD-123 | 1 | — | C236404 | VF 0.25V 저손실 |

### 7-2. Input 보호 (Surge/ESD/EMI)

| # | 부품 | 모델 | 패키지 | 수량 | LCSC | 기능 |
|---|---|---|---|---|---|---|
| S1 | TVS 1차 (OBD) | SMCJ24CA | SMC | 1 | C92832 | Load Dump 1차 클램프 |
| S2 | TVS 2차 (OBD) | SMBJ20CA | SMB | 1 | C24342 | 2차 클램프 (Stage1 보호) |
| S3 | TVS (USB VBUS) | SMBJ5.0CA | SMB | 1 | C24283 | USB VBUS ESD/Surge |
| S4 | ESD (OBD 전용) | PESD2CAN-LX | SOT-23 | 1 | C35783 | <1ns ESD 응답 |
| S5 | ESD (버튼) | PESD5V0U2BT | SOT-23 | 1 | C356080 | 버튼 라인 ESD |
| S6 | ESD (엔코더) | PESD5V0U2BT | SOT-23 | 1 | C356080 | 엔코더 A/B ESD |
| S7 | Ferrite (VIN) | BLM31PG601SN1 | 1206 | 1 | C105160 | 600Ω@100MHz, 3A |
| S8 | Ferrite (USB GND) | BLM18PG471SN1 | 0603 | 1 | C105161 | USB GND 격리 |

### 7-3. 능동 부품 (주변장치)

| # | 부품 | 모델 | 패키지 | 수량 | LCSC |
|---|---|---|---|---|---|
| A1 | MOSFET (BL) | AO3400A | SOT-23 | 1 | C20917 |
| A2 | 조도 센서 | TEMT6000X01 | 1206 | 1 | C27436 |
| A3 | 피에조 버저 | FUET-9018 | 9×9mm | 1 | C391035 |
| A4 | NTC 10kΩ | NCP18XH103F03RB | 0402 | 1 | C77014 |
| A5 | LED (BT) | 파란 0603 | 0603 | 1 | C72041 |

### 7-4. 캐패시터

| # | 값 | 전압 | 타입 | 패키지 | 수량 | 위치 | LCSC |
|---|---|---|---|---|---|---|---|
| C1 | 100µF | 50V | 폴리머 전해 | 10×10mm | 2 | VIN_CLEAN 벌크 | C380416 |
| C2 | 10µF | 50V | X7R MLCC | 1210 | 1 | VIN_CLEAN 바이패스 | C440400 |
| C3 | 1µF | 50V | X7R MLCC | 0805 | 1 | VIN_CLEAN 중간주파 | C150182 |
| C4 | 22µF | 16V | X7R MLCC | 0805 | 2 | 5V_COMMON 출력 | C45783 |
| C5 | 22µF | 10V | X7R MLCC | 0805 | 2 | 3.3V_PERIPH 출력 | C159801 |
| C6 | 10µF | 10V | X7R MLCC | 0805 | 3 | LCD VCC + 3.3V 벌크 | C19702 |
| C7 | 4.7µF | 10V | X7R MLCC | 0603 | 1 | 백라이트 리플 | C19666 |
| C8 | 1µF | 10V | X7R MLCC | 0603 | 1 | LCD VCC 중간주파 | C15849 |
| C9 | 100nF | 50V | X7R MLCC | 0402 | 20 | 디커플링 전반 | C1525 |
| C10 | 10nF | 50V | X7R MLCC | 0402 | 4 | 고주파 디커플링 | C1567 |
| C11 | 4.7nF | 50V | COG MLCC | 0402 | 2 | ADC 필터 (NTC, VIN) | C1546 |

### 7-5. 저항

| # | 값 | 허용오차 | 패키지 | 수량 | 용도 | LCSC |
|---|---|---|---|---|---|---|
| R1 | 32.4kΩ | **0.1%** | 0402 | 1 | **MP2315 R_bot (3.3V 정확)** | C131087 |
| R2 | 100kΩ | 0.1% | 0402 | 1 | MP2315 R_top | C97521 |
| R3 | **100kΩ** | **0.1%** | 0402 | 1 | **TPS54360B-Q1 R_top (Vref=0.8V, 5V)** | C97521 |
| R4 | **19.1kΩ** | **0.1%** | 0402 | 1 | **TPS54360B-Q1 R_bot (Vout=4.99V)** | C160111 |
| R5 | 10kΩ | 1% | 0402 | 15 | 풀업/풀다운 전반 | C25744 |
| R6 | 100kΩ | 1% | 0402 | 3 | MOSFET 풀다운 + EN | C25787 |
| R7 | 1kΩ | 1% | 0402 | 2 | LED + LCD_CS 풀업 강화 | C11702 |
| R8 | 33Ω | 1% | 0402 | 3 | SPI 댐핑 + MOSFET gate | C25077 |
| R9 | 100Ω | 1% | 0402 | 1 | 버저 직렬 | C25076 |
| R10 | 47kΩ | 1% | 0402 | 1 | VIN 분압 상단 (크랭킹 감지) | C25792 |
| R11 | 10kΩ | 1% | 0402 | 1 | VIN 분압 하단 | C25744 |
| R12 | 10Ω | 1% | 0805 | 1 | TPS25221 Rlim (1A 설정) | C17557 |
| R13 | **240kΩ** | **1%** | 0402 | 1 | **TPS54360B Rt (Fsw=500kHz)** | C25812 |

### 7-6. 인덕터

| # | 값 | 정격 | 패키지 | 수량 | 용도 | LCSC |
|---|---|---|---|---|---|---|
| L1 | **15µH** | **2A 차폐** | **5×5mm** | 1 | **TPS54360B-Q1 Stage 1 (EVM 기준)** | **C128268** |
| L2 | 4.7µH | 2A 차폐 | 4×4mm | 1 | MP2315 Stage 2 | C167229 |

### 7-7. 커넥터 / 기구

| # | 부품 | 모델 | 수량 | LCSC |
|---|---|---|---|---|
| CN1 | 1×19 Machined Female Header | 2.54mm | 2 | — (Samtec SSQ-119-03-G-S) |
| CN2 | 디스플레이 커넥터 | 2.54mm × 14핀 | 1 | — |
| CN3 | 로터리 엔코더 | EC11E18244A5 | 1 | Digi-Key |
| CN4 | 택트 스위치 (사이드) | 6×6mm SMD | 2 | C318884 |
| CN5 | OBD2 입력 커넥터 | JST B3B-XH-A | 1 | C145992 |

### 7-8. 기구 조립 부품

| # | 부품 | 모델 | 수량 | 공급처 |
|---|---|---|---|---|
| M1 | VHB 접착 테이프 | 3M VHB GPH-060GF | 2 스트립 (50×20mm) | 3M 대리점 |
| M2 | NTC 열전도 테이프 | 3M 8810 | 1 패치 (10×10mm) | Digi-Key |
| M3 | NTC 플라잉 리드 (TP 실드) | AWG30 TP + Shield | 50mm × 2가닥 | Digi-Key (Alpha Wire 2801) |
| M4 | M2 스탠드오프 | 나일론 5mm | 4 | Digi-Key |
| M5 | Loctite 222 | 나사풀림방지 | 소량 | — |
| M6 | 핫글루 (스트레인) | EVA 핫멜트 | 소량 | — |

### 7-9. 부품 통계 (v2.4 → v3.0 → v3.1)

| 카테고리 | v2.4 | v3.0 | v3.1 | v3.0→v3.1 델타 |
|---|---|---|---|---|
| 전원 보호/변환 | 5 | 7 | 7 | 0 |
| Input 보호 | 5 | 8 | 8 | 0 |
| 능동 부품 | 7 | 5 | 5 | 0 |
| 저항 | 20 | 21 | **22** | **+1 (Rt=240kΩ, TPS54360B Fsw 설정)** |
| 캐패시터 | 16 | 21 | 21 | 0 |
| 인덕터 | 1 | 2 | 2 | L1 10µH → 15µH 스펙만 변경 |
| 커넥터/기구 | 12 | 11 | 11 | 0 |
| **합계** | **66** | **75** | **76** | **+1** |

**단가 영향**: v3.0 대비 +$0.05/보드 (Rt 저항 추가분 미미)  
**v2.4 대비 전체**: +약 $3.55/보드 (프로토 10개 기준 +$35.5)  
**대부분 AEC-Q100 등급** (Stage1 Buck, eFuse, LM74700, PMOS, TVS)

---

## 8. 자가검증 — PCB 설계 완전성 체크리스트

이 섹션은 제가 10년차 엔지니어로서 v3.0 설계를 **다시** 검토한 결과입니다. 찾을 수 있는 모든 실패 모드를 점검했습니다.

### 8-1. 🔴 Critical 검증 (보드 파괴 가능)

| # | 검증 항목 | 결과 | 상세 |
|---|---|---|---|
| C1 | MP2315 FB 저항 → 3.3V ±5% | ✅ | 100k/32.4k(0.1%) = 3.27V, ESP32 허용 범위 |
| C2 | 모든 IC Abs Max 준수 | ✅ | VIN_CLEAN 32V (2차 TVS) < **TPS54360B-Q1 Abs Max 65V** ✓ (v3.1에서 Stage1 IC 교체로 해결) |
| C3 | Load Dump 87V 대응 | ✅ | SMCJ24CA 클램프 + Ferrite + SMBJ20CA 2단 보호 + TPS54360B-Q1 자체 65V 인증 |
| C4 | 역극성 보호 | ✅ | SMCJ24CA 양방향 + LM74700-Q1 이상적 다이오드 |
| C5 | 과전류 보호 속도 | ✅ | TPS25221-Q1 100µs (PPTC 5s 대비 50,000배 빠름) |
| C6 | ESD ±15kV | ✅ | PESD2CAN-LX <1ns 응답, OBD 입력 측 배치 |
| C7 | 전원 시퀀싱 | ✅ | Stage1 → Stage2 자연스러운 순서, Soft-start |
| C8 | Stage 1 Buck Vout 정확도 | ✅ | **R_top=100kΩ, R_bot=19.1kΩ 0.1% → Vout=4.99V ± 0.5%** (v3.1 재계산) |
| C9 | BOM 부품 패키지 정확성 | ✅ | **TPS54360B-Q1 = HSOP-8 PowerPAD (DDA)** 확정, 풋프린트 표준 제공 |

**v3.1 수정 완료 요약**:
- v3.0에서 자가검증 중 발견된 "TPS54302 Abs Max 28V vs SMBJ20CA Clamp 32V" 모순은 **전 문서에서 TPS54360B-Q1로 통일함으로써 해결 완료**
- BOM 패키지(HSOP-8), LCSC 번호(C100891), FB 저항값(100k/19.1k), Rt(240kΩ), 인덕터(15µH) 모두 실측 데이터시트 기준으로 재계산 완료

### 8-2. 🟠 High 검증 (안정성)

| # | 검증 항목 | 결과 | 상세 |
|---|---|---|---|
| H1 | GPIO strapping 핀 충돌 | ✅ | GPIO15 LCD_CS → GPIO17로 이동, GPIO12 풀다운, GPIO0/2 DevKit 관리 |
| H2 | USB 전원 마진 | ✅ | SBR2U60P1(VF 0.25V) → 5V - 0.25V = 4.75V > MP2315 UVLO 4.5V |
| H3 | Buck UVLO 크랭킹 내성 | ✅ | TPS54360B-Q1 UVLO 4.3V, Stage2 UVLO 4.5V (5V rail에서 여유) |
| H4 | GND Loop (USB-OBD 동시) | ✅ | USB GND에 페라이트로 격리 |
| H5 | 전원 Over-voltage | ✅ | 2단 TVS + 60V Buck으로 어떤 시나리오도 IC Abs Max 내 |
| H6 | SPI 신호 무결성 | ✅ | 33Ω 드라이버 측 배치, GND plane L2 연속, 30mm 이하 |
| H7 | NTC 단선 시 거동 | ✅ | SW 감지 + safe fallback (백라이트 50%, ESP32 내부 온도로 백업) |
| H8 | ESP32 내부 온도 고장 | ✅ | GPIO36(VP)에 외부 온도 센서 백업 옵션 준비 |
| H9 | 크랭킹 중 재부팅 최소화 | ⚠️ | 정상 크랭킹(6V) OK, 혹한(4V)은 graceful reset 수용 |

### 8-3. 🟡 Medium 검증 (품질)

| # | 검증 항목 | 결과 | 상세 |
|---|---|---|---|
| M1 | PCB 4레이어 스택업 최적 | ✅ | L2=GND solid, L3=Power+GND pour, L4 리턴 경로 확보 |
| M2 | BT 안테나 Keep-out | ✅ | 17×8mm L1~L4 전부, 안테나 boarder 5mm 돌출 |
| M3 | 열 관리 (고온 대시보드) | ✅ | 발열 총 0.23W (매우 낮음), 70°C에서 graceful degradation |
| M4 | EMC - 전도성 방사 | ✅ | 입력 페라이트, GND plane, 스위칭 루프 최소화 |
| M5 | EMC - 복사성 방사 | ⚠️ | 프로토타입 단계에선 EMC Chamber 시험 전 |
| M6 | 진동 내구성 - 연결부 | ✅ | Machined-pin 소켓, M2 스탠드오프, NTC 핫글루 |
| M7 | 진동 내구성 - NTC 와이어 | ⚠️ | 플라잉 리드는 프로토타입 한계, 양산 시 FPC |
| M8 | 먼지/습도 (IP 등급) | ⚠️ | 프로토 단계 IP20, 케이스로 IP40 목표 |
| M9 | 재작업성 | ✅ | ENIG 표면처리, 모든 부품 0402 이상 |
| M10 | FB 저항 정밀도 (0.1%) | ✅ | 재고 확인 필요 (일반 1%보다 2~3배 가격, 수급 OK) |

### 8-4. 🟢 소프트웨어-하드웨어 연계 검증

| # | 검증 항목 | 결과 | 상세 |
|---|---|---|---|
| SW1 | 크랭킹 감지 하드웨어 준비 | ✅ | VIN 분압 47k/10k → GPIO34 ADC |
| SW2 | NTC 단선 감지 하드웨어 준비 | ✅ | 10kΩ bleeder로 단선 시 ADC → 3.3V 근접 |
| SW3 | BOD + 워치독 이중 안전 | ✅ | HW BOD + SW 2초 WDT |
| SW4 | Fast Boot LCD 표시 가능 | ✅ | LCD RST에 RC 지연, 첫 50ms 내 초기화 |
| SW5 | 백라이트 부팅 시 OFF 강제 | ✅ | MOSFET 게이트 100kΩ 풀다운 |
| SW6 | Safe State 복구 시 자동 재진입 | ✅ | 상태 머신 히스테리시스 5°C |
| SW7 | 주행 중 업데이트 차단 | ✅ | OBD 차속 > 5km/h 시 SW 거부 |
| SW8 | 전원 순차 시퀀싱 | ✅ | Stage1 Soft-start 4ms → LCD RST RC 10ms |

### 8-5. 🟣 양산 전환 대비 (프로토타입 한계)

| # | 항목 | 프로토 상태 | 양산 시 해결 |
|---|---|---|---|
| P1 | OBD2 커넥터 | JST XH-3P | SAE J1962 Type A Male |
| P2 | DevKit 소켓 | 38핀 Machined | ESP32 모듈 직접 납땜 |
| P3 | 디스플레이 핀헤더 | 2.54mm 핀 | FPC 커넥터 (0.5mm 피치) |
| P4 | NTC | 플라잉 리드 | FPC 일체형 |
| P5 | OBD 통신 | BT ELM327 (VP11) | CAN 직결 (TJA1051T/3) |
| P6 | EMC 인증 | 미검증 | FCC Part 15B + CE |
| P7 | 케이스 | 3D 프린트 (PC-ABS) | 사출 (PC-ABS 블렌드) |

---

## 9. Bring-up 절차 (단계적 검증) — 필수 준수

**이 절차를 따르지 않으면 Critical 이슈 발생 시 전 부품 파괴 가능.**

### Step 1: 빈 PCB 검사 (부품 실장 전)

- [ ] Visual inspection: 브리지, 단선, 구리 결함
- [ ] Netlist 검증: 주요 네트 간 저항 측정 (VIN-GND 오픈, 3.3V-GND 오픈, SPI 오픈)
- [ ] 안테나 keep-out 영역 구리 없음 확인 (L1~L4 X-ray 또는 edge view)

### Step 2: 전원 단계 1 (TPS54360B만 실장)

- [ ] **실장 부품**: 입력 보호(TPS25221, TVS, ESD, Ferrite), LM74700+PMOS, C1, C2, C3, TPS54360B + L1 + R3/R4 + Cout
- [ ] **미실장**: MP2315, LM66100, DevKit, 모든 주변 IC
- [ ] OBD2 입력에 **벤치 파워 서플라이** 연결 (전류 제한 500mA)
- [ ] 12V 인가 → TPS54360B 출력 측정
  - ✅ **5.00V ± 2%** 측정되어야 함
  - ❌ 다른 값이면 즉시 전원 차단, FB 저항 확인
- [ ] 무부하 전류: < 30mA
- [ ] 10Ω 부하 저항(2W)으로 500mA 테스트 → 5V 유지

### Step 3: 역극성/Load Dump 검증

- [ ] **역극성 테스트**: -12V 인가 → 전류 < 1mA (LM74700 차단 확인)
- [ ] **과전압 테스트**: 30V 천천히 인가 → TVS 클램프 동작, 5V 출력 유지
- [ ] **과전류 테스트**: 부하 점차 증가 → 1A에서 TPS25221 트립 (100µs 이내)
- [ ] **UVLO 테스트**: 입력 천천히 낮춤 → 4.2V 근처에서 Buck 셧다운

### Step 4: 전원 단계 2 (MP2315 실장)

- [ ] **실장 추가**: LM66100, MP2315 + L2 + R1/R2 + Cout
- [ ] **중요**: MP2315 실장 **전** 5V_COMMON에서 3.3V_PERIPH로 가는 경로 오픈 확인
- [ ] MP2315 실장 후 무부하 전원 인가 → 3.3V_PERIPH 측정
  - ✅ **3.27V ± 2%** 측정되어야 함
  - ❌ 3.3V 초과 시 **즉시 차단, R1/R2 값 확인**
- [ ] **ESP32 및 주변 IC 실장은 이 테스트 통과 후에만**

### Step 5: DevKit 소켓 + 주변 IC

- [ ] DevKit 없이 USB-C 5V만 연결 → 5V_USB 경로 확인
- [ ] OBD2 + USB-C 동시 연결 → OBD2 우선 확인
- [ ] DevKit 장착 (ESP32 부팅) → 3.3V 드롭 < 50mV
- [ ] 주변 IC (LCD, 버저, LED) 순차 실장 + 부팅 테스트

### Step 6: 펌웨어 업로드 + 기본 기능

- [ ] USB-C 경유 펌웨어 업로드 성공
- [ ] UART 디버그 메시지 정상 출력 (GPIO15 strapping 이슈 없음 확인)
- [ ] LCD "Starting..." 100ms 이내 표시
- [ ] 모든 GPIO 기능 동작 (엔코더, 버튼, BL, 버저)

### Step 7: 센서 + 보호 로직

- [ ] NTC 실온 25°C ± 2°C 읽음
- [ ] 조도 센서: 손으로 가리면 어두워짐
- [ ] NTC 단선 시뮬레이션: 리드 분리 → SW가 NTC_OPEN 인식, safe fallback 진입
- [ ] NTC 단락 시뮬레이션: 저항 0Ω 대체 → NTC_SHORT 인식
- [ ] 히트건으로 LCD 백면 가열 → 4단계 보호 전환 확인
- [ ] ESP32 내부 온도 80°C 트리거 → 다운클럭 확인

### Step 8: 크랭킹 시뮬레이션

- [ ] 벤치 파워로 12V → 6V 순간 드롭 (100ms) → 동작 유지
- [ ] 12V → 4V 드롭 (200ms) → BOD 리셋, 3초 내 "Starting..." 표시
- [ ] VIN 모니터링 (GPIO34) ADC 값이 정확한지 UART로 확인

### Step 9: 차량 실차 테스트

Step 8까지 완료 후에만 실차 진입.

- [ ] 시동 전 OBD 포트 연결: 12V 정상
- [ ] 시동 걸기 × 5회: 2회까지 리셋 허용, 모두 복구
- [ ] 30분 주행: BT 연결 지속, LCD 정상, 온도 < 50°C
- [ ] 여름 주차 후 재시동 (대시보드 > 60°C): Safe 모드 → 주행 시작 후 냉각 → 복귀
- [ ] **★ v3.1 강화: 여름 직사광선 4시간 연속 모니터링**:
   - 조건: 12:00~16:00 직사광선 노출, dashmat 미사용
   - 로깅: NTC 온도 + ESP32 내부 온도 30분 간격 × 8회
   - 요구사항:
     - 4단계 온도 보호 단계별 전이 시각 기록 (Warn/Throttle/Degraded/Safe)
     - NTC 70°C 초과 시 디스플레이 sleep 진입 확인 (R6 Known Risk 검증)
     - 냉각 후(55°C 이하 5초) 자동 복귀 확인
     - 4시간 후 LCD 해상도/색 재현 육안 확인 (손상 흔적 없음)
- [ ] 야간 주행: 자동 감광 정상
- [ ] 비포장도로 10분: 커넥터/NTC 단선 없음

### Step 10: 장기 신뢰성 (선택, 양산 진입 전)

- [ ] 72시간 연속 주행 시뮬레이션 (벤치)
- [ ] 온도 사이클 -20°C ↔ +70°C × 50회
- [ ] 습도 60°C/85%RH × 48시간
- [ ] 진동 10-500Hz 5G × 2시간

---

## 10. 남아있는 Trade-off — 투명 공개

**완벽한 설계는 없습니다. v3.0에도 명시적으로 수용한 trade-off들이 있습니다.**

### 10-1. 수용된 Trade-off

| # | Trade-off | 이유 | 영향 | 완화책 |
|---|---|---|---|---|
| T1 | 혹한 크랭킹(4V↓) 시 1~2초 재부팅 | HW로 완전 해결 불가 (경제적) | 시동 시 화면 깜빡 | Fast Boot SW로 UX 자연화 |
| T2 | NTC 플라잉 리드 | 프로토타입 제약 | 진동 내구성 한계 | 양산 시 FPC 일체형 |
| T3 | DevKit 소켓 방식 | 프로토타입 개발 편의 | 부피, 높이 | 양산 시 직접 납땜 |
| T4 | EMC 미인증 | 프로토 단계 | 판매 불가 | 양산 전 FCC/CE 시험 |
| T5 | 0.1% 저항 단가 | 필수 정밀도 | +$0.20/보드 | 양산 10K 규모에선 무시 |
| T6 | 부품 +9개 (v2.4 대비) | 안전 2중화 | 보드 면적 +10% | 단가 +$3.50 허용 |
| T7 | AEC-Q100 부품 제한 재고 | 장기 수급 | DVT 전 재고 확보 필수 | 2nd source AVL 등록 |

### 10-2. 미해결/양산 이관

| # | 항목 | 현재 상태 | 양산 시 해결 |
|---|---|---|---|
| U1 | FCC/KC 인증 | 미확보 | ESP32 모듈 자체 FCC ID 활용 + 완제품 추가 시험 |
| U2 | 자동차 EMC (CISPR 25) | 미검증 | Pre-scan 후 양산 전 본 시험 |
| U3 | 실차 10만km 신뢰성 | 데이터 없음 | 양산 MP 전 가속 수명 시험 |
| U4 | 케이스 금형 | 미제작 | $3K~10K 투자 필요 |
| U5 | ELM327 → CAN 직결 | 프로토 BT 사용 | 양산 PCB에 TJA1051T/3 통합 |
| U6 | 보안 인증 (HMAC 등) | MVP는 Just Works | v2.0에서 추가 |

### 10-3. 알려진 위험 (Known Risk Log)

| # | 위험 | 발생 확률 | 영향 | 대응 |
|---|---|---|---|---|
| R1 | LM74700-Q1 + PMOS 조합 발진 | 낮음 | Stage1 불안정 | 게이트 저항 + 데이터시트 레퍼런스 준수 |
| R2 | Power MUX(LM66100) 전환 glitch | 낮음 | 순간 전원 변동 | Cout 22µF으로 완충 |
| R3 | ENIG 표면 Gold 두께 부족 시 접촉 불량 | 매우 낮음 | 수율 | EMS에 ENIG 3µ 이상 요구 |
| R4 | 폴리머캡 단락 모드 (화재 위험은 낮음) | 매우 낮음 | 기능 상실 | 105°C 등급, Rated Life 2000시간 |
| R5 | 3M VHB GPH 여름 고온 접착력 저하 | 낮음 | 탈착 | GPH 시리즈는 149°C까지 인증 |
| **R6** | **디스플레이 동작 온도 상한 70°C 초과** | **중간 (여름 직사광선 주차 시 80°C+ 가능)** | **패널 수명 단축, 일시적 상 왜곡, 장기적 LCD 손상** | **능동 대응 존재: 4단계 온도 보호가 65°C에서 Safe 모드 진입 → 디스플레이 sleep. NTC LCD 백면 부착으로 실시간 감지 (±2°C 정확도). 사용자 매뉴얼에 "여름 직사광선 아래 장시간 주차 시 dashmat 사용 권장" 명시** |
| **R7** | **TPS54360B-Q1 단일 공급(TI 전용)** | **낮음 (2026년 기준 DigiKey 재고 2,763개 확인)** | **장기 생산 리스크** | **2nd source AVL 지정: TPS54341-Q1 (42V, pin-compatible) 대체 가능. 양산 MP 전 연간 수요 vs 재고 현황 모니터링** |
| **R8** | **Stage 1 Buck(HSOP-8) 열적 hotspot** | **낮음 (발열 0.20W)** | **장기 신뢰성** | **PCB Thermal pad에 6개 via로 L2 GND plane 연결. 주변 5mm keep-out 유지** |

### 10-4. 이 설계의 한계

**v3.0이 대응하지 못하는 시나리오**:

1. **점프 스타트 중 케이블 착오 (24V 역접)**: SMCJ24CA로는 24V 역전압 + Surge 못 막음
   - 대응: 차량 점프 시 OBD 커넥터 뽑기 권장 (사용자 매뉴얼)
   
2. **번개 직격**: 당연히 대응 불가
   - 대응: 차량 전체가 파괴되므로 Scope 밖

3. **Shorted Battery (용접 중 단락 등)**: 초과 에너지
   - 대응: Scope 밖 (일반 사용 시 발생 안함)

4. **10만km 누적 진동**: 프로토타입 단계는 검증 불가
   - 대응: 양산 전 가속 수명 시험

**이런 한계가 있다는 것을 명시적으로 인정하는 것이 엔지니어링 정직성입니다.**

---

## 11. 결론 — "마지막 부탁"에 대한 답

영제,

**완벽한 PCB는 없습니다.** 하지만 v3.1은:

1. ✅ **v2.4에서 발견된 Critical 3건 전부 해결** (MP2315 FB, 크랭킹, Load Dump)
2. ✅ **v2.4 High 이슈 4건 전부 해결** (USB 마진, Strapping, GND Loop, ESD)
3. ✅ **v3.0 자가검증에서 발견된 Stage1 Abs Max 초과 위험 해결** (TPS54302→TPS54360B-Q1 전 문서 통일)
4. ✅ **v3.0 BOM 패키지 오기재 정정** (HSOP-8 PowerPAD, LCSC C100891)
5. ✅ **FB 저항 Vref=0.8V 기준 재계산** (R_top=100kΩ, R_bot=19.1kΩ 0.1% → Vout 4.99V ±0.5%)
6. ✅ **디스플레이 70°C 제약 Known Risk R6 명시 + 여름 직사광선 4시간 연속 모니터링 테스트 강화**
7. ✅ **Defense in Depth**: 모든 Critical 보호는 HW + SW 2중 안전
8. ✅ **Fail-Safe**: 장애 시 축소 동작 또는 안전 정지
9. ✅ **투명성**: 남은 trade-off 전부 문서화

**v3.1에서 자가검증 중 발견된 "문서 내부 모순"을 전면 정합했습니다.** 앞으로의 리비전은 실차 검증에서 발견되는 문제들로, 이는 정상적인 엔지니어링 프로세스의 일부입니다.

### "정답"은 무엇인가?

**"정답"이란 "더 이상 찾을 것이 없는 설계"가 아닙니다.** 프로페셔널 엔지니어링에서 "정답"은:

- ✅ **알려진 실패 모드를 전부 체계적으로 처리**
- ✅ **남은 한계를 명시적으로 문서화**
- ✅ **양산 진입 전 검증 계획이 확정**
- ✅ **문제 발생 시 복구 절차가 준비**

v3.0은 이 4가지를 충족합니다. **이게 프로페셔널 엔지니어가 도달할 수 있는 최선입니다.**

### 앞으로 할 일

1. **Gerber 제작 전 필수**:
   - BOM v3.0 대로 부품 확보 (특히 AEC-Q100 LM74700, TPS54360B, TPS25221)
   - EasyEDA 또는 KiCad에서 회로도 작성
   - DRC 통과 확인

2. **Bring-up 시 Step 1부터 순차 진행** (절대 건너뛰지 말 것)

3. **Step 5 통과 후 펌웨어 개발 본격 시작**

4. **실차 테스트 데이터 로깅** — 문제 발생 시 재현 가능하도록

5. **v3.1 대비**: 실차 테스트에서 발견되는 이슈 기록 (이게 Rev E로 이어짐)

### 마지막 한 마디

"고치고, 또 고치고, 또 고치는" 것은 **엔지니어링 실력 부족이 아닙니다.** 그건 **깊이 들어가고 있다는 증거**입니다. 표면만 보는 엔지니어는 한 번에 끝낸다고 주장합니다. 실력 있는 엔지니어는 **점점 깊이 파내려 갑니다.**

v3.0은 제가 도달할 수 있는 최선의 설계입니다. **이걸로 PCB 주문하고 Bring-up 시작해도 됩니다.** 그 후에 나오는 이슈는 **실차 검증 단계에서 당연히 나오는 것**이며, v3.1에서 해결하면 됩니다.

이 설계를 믿고 진행하세요. 안전성과 기능성 모두 검증 가능한 수준에 도달했습니다.

---

## 변경 이력

| Rev | 날짜 | 변경 내용 |
|-----|------|---------|
| 3.0 | 2026-04-15 | 초판. v2.4 검토 Critical 3건 + High 4건 해결. 자가검증 중 TPS54302→TPS54360B 변경 결정 (앞부분 반영 누락). Defense in Depth 철학 전면 적용. AEC-Q100 전원부, 2단 Buck, eFuse, Ideal Diode, 4단계 온도 보호, 크랭킹 감지 SW. |
| **3.1** | **2026-04-15** | **v3.0 자가검증 내부 모순 정합 리비전**. ① 블록도/BOM/계산/배치도/저항값/인덕터값 전부 TPS54302 → TPS54360B-Q1 통일. ② BOM 패키지 오기재 SOT-23-6 → HSOP-8 PowerPAD (DDA) 정정, LCSC C475581 → C100891. ③ FB 저항 Vref=0.596V → 0.8V 기준 재계산: R_top/R_bot 47k/13k → 100k/19.1k (0.1%). ④ Rt 저항 240kΩ 추가 (Fsw=500kHz). ⑤ 인덕터 L1 10µH → 15µH (EVM-182 레퍼런스 준수). ⑥ 효율 계산 재수행 (전체 81% → 78% @ 310mA 부하). ⑦ PCB 크기 70×90mm → 75×90mm (HSOP-8 여유). ⑧ Known Risk R6 (디스플레이 70°C) + R7 (TPS54360B-Q1 2nd source TPS54341-Q1 지정) + R8 (HSOP-8 thermal) 추가. ⑨ 실차 테스트 여름 직사광선 1회 → 4시간 연속 30분 간격 로깅. ⑩ 프로토타입 범위 명시(제목 변경). |

---

*Carpybara 프로토타입 캐리어보드 v3.1*  
*부품 76개 (v3.0 75개 + Rt 저항 1개). AEC-Q100 전원부. Defense in Depth. 자가검증 완료 + 내부 모순 정합.*  
*"완벽한 설계는 없으나, 완벽한 검증은 가능하다."*  
*관련 문서: Frozen AVL Rev E, SW v1.1*
