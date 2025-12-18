# せいせいAIにたすけてもらいながらさいきょうの Stereo Compressor / Preampをつくる

> 本書は、生成AIを使用して書いたV2164（SSM2164）を用いたステレオ・コンプレッサ／プリアンプの**詳細仕様書**である。
> 今のところ日本語のみで進めています。
> プロフェッショナル機材相当の機能と音質を実現する。

---

## 📋 目次

1. [設計コンセプト](#1-設計コンセプト改良版)
2. [システム全体ブロック図](#2-システム全体ブロック図改良版)
3. [電気的仕様](#3-電気的仕様)
4. [入力段](#4-入力段)
5. [V2164 VCA 段](#5-v2164-vca段)
6. [I/V 変換段](#6-iv変換段可変特性対応)
7. [サイドチェイン処理系](#7-サイドチェイン処理系)
8. [CV 生成・制御系](#8-cv生成制御系)
9. [Makeup Gain 段](#9-makeup-gain段)
10. [Parallel Compression](#10-parallel-compression)
11. [出力段](#11-出力段)
12. [電源設計](#12-電源設計)
13. [完全部品リスト（BOM）](#13-完全部品リストbom)
14. [基板レイアウト指針](#14-基板レイアウト指針)
15. [キャリブレーション手順](#15-キャリブレーション手順)
16. [トラブルシューティング](#16-トラブルシューティング)

---

## 1. 設計コンセプト（改良版）

### 1.1 コアコンセプト

- **音のキャラクター生成**：V2164 の入力電流制御による倍音付加と圧縮
- **音質保持**：OPA2134 系による超低歪み信号処理（THD+N < 0.005%）
- **プロフェッショナル機能**：
  - Threshold / Ratio / Attack / Release / Makeup
  - SC-HPF（可変）/ External Sidechain
  - Parallel Compression（Dry/Wet Mix）
  - Stereo Link / Dual Mono 切替
  - GR メーター / VU メーター
- **用途**：
  - スタジオ・マスタリング
  - ライブ PA（シンセ／ドラム → 卓）
  - バス・コンプレッション

### 1.2 ターゲットスペック

| 項目                 | 仕様                                |
| -------------------- | ----------------------------------- |
| 入力インピーダンス   | 10kΩ 以上（バランス）               |
| 出力インピーダンス   | 600Ω（バランス）                    |
| 最大入力レベル       | +22dBu                              |
| 最大出力レベル       | +24dBu                              |
| ダイナミックレンジ   | 110dB 以上                          |
| THD+N（1kHz, +4dBu） | 0.005%以下（CLEAN）/ 0.5〜3%（SAT） |
| 周波数特性           | 20Hz-20kHz（±0.5dB）                |
| Ratio 範囲           | 1.5:1 〜 20:1 + ∞:1（Limiter）      |
| Attack 時間          | 0.1ms 〜 100ms                      |
| Release 時間         | 50ms 〜 2s + Auto                   |

---

## 2. システム全体ブロック図（改良版）

```
┌─────────────────────────────────────────────────────────────┐
│                        INPUT SECTION                        │
└─────────────────────────────────────────────────────────────┘
  [XLR IN] → [Balanced Receiver] → [LEVEL Att] → [Input Buffer]
                                                        ↓
┌─────────────────────────────────────────────────────────────┐
│                         VCA SECTION                         │
└─────────────────────────────────────────────────────────────┘
  [Trim Gain] → [Rin Select: CLEAN/SAT] → [V2164 VCA] → [I/V]
                                              ↑
                                             CV
┌─────────────────────────────────────────────────────────────┐
│                    SIDECHAIN & CV SECTION                   │
└─────────────────────────────────────────────────────────────┘
  [L+R Sum] ←→ [EXT SC IN]
      ↓
  [SC-HPF: Bypass/80Hz/160Hz] → [Rectifier] → [A/R Generator]
      ↓
  [Threshold Comp] → [Ratio Scale] → [CV Buffer] → V2164
      ↓
  [GR Meter Driver]

┌─────────────────────────────────────────────────────────────┐
│                      OUTPUT SECTION                         │
└─────────────────────────────────────────────────────────────┘
  [Makeup Gain] → [Dry/Wet Mix] → [DC Block] → [BAL OUT]
                                                   ↓
                                        ┌──────────┴──────────┐
                                        ↓                     ↓
                                   [VU Meter]            [HP Amp]

┌─────────────────────────────────────────────────────────────┐
│                    CONTROL & MONITORING                     │
└─────────────────────────────────────────────────────────────┘
  ● Threshold / Ratio / Attack / Release / Makeup
  ● SC-HPF / EXT SC / Stereo Link / SAT Mode
  ● GR Meter / VU Meter / Power LED
```

---

## 3. 電気的仕様

### 3.1 入出力仕様

```
入力：
  - 1/4" TS（Tip=Signal, Sleeve=GND）
  - 入力Z：約47kΩ（LEVEL VR + バッファ）
  - 最大入力：+15dBu（シンセ/ドラムマシン想定）
  - 対応：-10dBV 〜 +4dBu機器

出力：
  - XLRバランス（THAT1646駆動）
  - 出力Z：600Ω（選択可能：10kΩ High-Z）
  - 最大出力：+24dBu
  - 負荷：600Ω 〜 100kΩ
```

### 3.2 ダイナミクス仕様

```
Threshold：-20dBu 〜 +10dBu（連続可変）
Ratio：1.5:1, 2:1, 3:1, 4:1, 6:1, 10:1, 20:1, ∞:1（ステップ＋微調整VR）
Attack：0.1ms, 0.3ms, 1ms, 3ms, 10ms, 30ms, 100ms（ステップ＋微調整VR）
Release：50ms, 100ms, 200ms, 500ms, 1s, 2s, Auto（ステップ＋微調整VR）
Knee：Hard / Soft（切替）
Makeup Gain：0dB 〜 +20dB
```

### 3.3 サイドチェイン仕様

```
SC-HPF：Bypass / 80Hz / 160Hz（ロータリーSW）
External SC Input：
  - 1/4" TRS（Tip=Hot, Ring=Cold, Sleeve=GND）
  - 入力Z：10kΩ
  - 感度：-10dBu = Unity
```

---

## 4. 入力段

### 4.1 TS 入力ジャック（1ch）

**コネクタ：**

- 1/4" TS フォーンジャック（Neutrik NMJ6HCD2 推奨）
- Tip：Signal
- Sleeve：GND

**入力保護：**

```
TS Tip ──[ R1: 1kΩ ]──┬──→ LEVEL Attenuator
                      │
                    [ D1: BAT54S ]
                      │      （双方向ショットキークランプ）
                      GND

過大入力時の保護：
  - R1で電流制限
  - D1（BAT54S）で約 ±0.25〜0.3V にクランプ
  - 過大入力から OPA2134 / V2164 を確実に保護
  - 通常動作（ラインレベル）では完全に非導通で音質影響なし
```

**部品：**

- TS Jack: Neutrik NMJ6HCD2（スイッチング接点付）
- R1: 1kΩ 1/4W（保護抵抗）
- D1: BAT54S

---

### 4.2 LEVEL アッテネータ（1ch）

```
Input ──┬──10kΩ──→ Buffer
        │
      50kB
      [LEVEL]
        │
       GND
```

- 可変範囲：0dB 〜 -∞（カット）
- 実用範囲：0dB 〜 -20dB
- VR：ALPS RK27 50kB（推奨）
- 入力インピーダンス：約 47kΩ（VR 中央時）

**動作：**

- VR 最小（GND 側）：信号完全カット
- VR 中央：約-6dB
- VR 最大（Input 側）：0dB（素通り）

---

### 4.3 入力バッファ＋ Trim Gain

**IC**：OPA2134（U1A）

```
         R1              R2
LEVEL ───10kΩ───┬───→ U1A(+)
                │         |\
                │         | \
              C1 100pF    |  }──→ OUT
                │         | /
               GND        |/
                          │
                    R3 ──┤
                   4.7kΩ  │
                          │
                         GND

OUT ───┬─────────────→ U1A(-)
       │
       R4: 0Ω（CLEAN）
       └─ 4.7kΩ VR（TRIM）
              │
             GND

Gain = 1 + (R4 / R3)
     = 1.0 〜 2.0（0dB 〜 +6dB）
```

**部品（1ch）：**

- OPA2134 ×1（デュアルなので 2ch 共用）
- R1: 10kΩ 1% MF
- R2: Open（直結）
- R3: 4.7kΩ 1% MF
- R4: 5kB VR（Trim Gain）
- C1: 100pF C0G（RF 除去）

---

## 5. V2164 VCA 段

### 5.1 入力抵抗切替（SAT 制御）

**目的**：V2164 への入力電流を変化させ、倍音特性を切り替える。

```
Input Buffer OUT
   │
   ├────[ R5: 33kΩ ]────┐
   │                    ├─→ V2164 Pin1（Vin）
   └────[ R6: 12kΩ ]────┘
            ↑
         DPDT SW
        [CLEAN/SAT]
```

**動作：**
| モード | 入力 R | 入力電流 | 倍音特性 |
|--------|-------|----------|----------|
| CLEAN | 33kΩ | 低 | 超低歪み |
| SAT | 12kΩ | 高 | 温かみ・倍音 |

**Makeup 補正（後述 9.1 節と連動）：**

- CLEAN モード時：Makeup Rf = 10kΩ
- SAT モード時：Makeup Rf = 3.9kΩ
- これにより出力レベルを一定に保つ

**部品（1ch）：**

- R5: 33kΩ 1% MF
- R6: 12kΩ 1% MF
- DPDT Toggle SW（ON-ON）

---

### 5.2 V2164 VCA セル

**ピン配置（DIP16）：**

```
        V2164
     ┌────────┐
Vin ─┤1     16├─ V+（+15V）
Iout─┤2     15├─ CV（制御電圧）
 NC ─┤3     14├─ NC
GND ─┤4     13├─ GND
（以下同様、4セル内蔵）
```

**本設計での使用：**

- セル 1：L ch
- セル 2：R ch
- セル 3-4：未使用（GND 接続）

**CV 入力特性：**

- 感度：-33mV/dB
- 範囲：0V（Unity Gain）〜 -2V（-60dB）
- 入力電流：<10µA（高インピーダンス）

**電源デカップリング（必須）：**

```
Pin16 ─┬─ 0.1µF ─┬─ GND
       │         │
       └─ 10µF ──┘

※V2164直近（5mm以内）に配置
```

---

### 5.3 V2164 保護回路

**CV 入力保護（ショットキーダイオードクランプ）：**

```
CV Source ──[ R7: 1kΩ ]──┬──→ V2164 Pin15
                         │
                       [ BAT54 ]
                         │
                         GND
```

- 正電圧を+0.3V でクランプ
- V2164 のラッチアップを防止
- 電源 ON 時のサージ保護

**部品（1ch）：**

- R7: 1kΩ 1% MF
- D1: BAT54 ショットキーダイオード

---

## 6. I/V 変換段（可変特性対応）

### 6.1 基本回路（1ch）

**IC**：OPA2134（U2A）

```
V2164 Pin2 ──────→ U2A(-)
  (Iout)              |\
                      | \
                      |  }──→ I/V OUT
                      | /
                      |/
                     (+)
                      │
                     GND

Feedback Network:
                 ┌─ Rf: 33kΩ 1% ─┐
I/V OUT ─────────┤               ├──→ U2A(-)
                 └─ Cfb: 選択式 ─┘
```

### 6.2 Cfb 選択（音色調整）

**ジャンパー選択式：**

```
         JP1        JP2        JP3
I/V OUT ─┬─ 22pF ─┬─ 33pF ─┬─ 47pF ─┐
         │        │        │        ├─→ U2A(-)
         └────────┴────────┴────────┘
              ↑選択（1つのみショート）
```

| Cfb 値 | 特性       | 用途                               |
| ------ | ---------- | ---------------------------------- |
| 22pF   | 超高域保持 | トランジェント重視・パーカッション |
| 33pF   | バランス型 | 汎用・標準設定                     |
| 47pF   | 丸く太い   | ビンテージ・バス圧縮               |

**部品（1ch）：**

- Rf: 33kΩ 1% MF
- C2, C3, C4: 22pF, 33pF, 47pF（C0G/NP0）
- JP1-3: ピンヘッダー＋ジャンパーキャップ

---

### 6.3 高域リンギング対策（オプション）

超高域での発振防止：

```
Rf 33kΩ ─┬─ Cfb（選択）
         │
         └─ 100kΩ + 10pF（並列）
```

- 40kHz 以上での位相余裕確保
- 発振傾向がある場合のみ追加

---

## 7. サイドチェイン処理系

### 7.1 内部 SC 信号生成（L+R Sum）

```
L ch（I/V OUT）──[ R8: 10kΩ ]──┐
                               ├──→ SC SUM
R ch（I/V OUT）──[ R9: 10kΩ ]──┘
```

### 7.2 外部 SC 入力（1/4" TS）

**TS 入力ジャック：**

```
TS Tip ──[ R10: 1kΩ ]──┬──→ Buffer U3C(+)
                        │      |\
                     [ C7 ]    | \
                      100pF    |  }──→ EXT SC Signal
                        │      | /
                       GND     |/
                               (-)
                                │
                           [ R11: 10kΩ ]
                                │
                      EXT SC OUT ──┘

Unity Gain Buffer（OPA2134）
```

**仕様：**

- 入力 Z：10kΩ
- 感度：-10dBu = Unity（内部 SC と同レベル）
- 用途：キックでベース圧縮、ダッキング等

**部品：**

- TS Jack: Neutrik NMJ6HCD2
- R10: 1kΩ（保護）
- R11: 10kΩ 1% MF
- C7: 100pF C0G（RF 除去）

### 7.3 SC 信号セレクター

```
Internal SC ──┬──→ DPDT SW ──→ SC Processing
              │    [INT/EXT]
External SC ──┘
```

**スイッチ：**

- 2 回路 2 接点（DPDT）
- センター OFF 不要（必ずどちらか選択）

---

### 7.4 SC-HPF（3 段切替）

**目的**：低域をサイドチェインから除外し、キックやベースでの過圧縮を防ぐ。

**回路（ロータリー SW 使用）：**

```
         Position 1    Position 2    Position 3
SC IN ───┬─ Direct ─┬─ 100nF + 10kΩ ─┬─ 100nF + 20kΩ ─→ Rectifier
         │          │   (160Hz)      │   (80Hz)
         └──────────┴────────────────┘
                     │               │
                    GND             GND

Rotary SW: 1P3T
  Position 1: Bypass（直結）
  Position 2: 160Hz HPF
  Position 3: 80Hz HPF
```

**部品：**

- C5, C6: 100nF Film
- R11: 10kΩ 1% MF
- R12: 20kΩ 1% MF（10kΩ×2 直列でも可）
- SW1: ロータリー SW 1P3T

---

### 7.5 全波整流器（精密検波）

**IC**：OPA2134（U3A, U3B）

**回路：**

```
Stage 1: Half-Wave Rectifier
SC Signal ──[ R13: 10kΩ ]──┬──→ U3A(-)
                            │      |\
                            │      | \
                         [ D2 ]    |  }──→ Half-Wave
                            │      | /
                            │      |/
                            │      (+)
                         [ R14 ]   │
                          10kΩ    GND
                            │
                           GND

Stage 2: Inverter + Summer
Half-Wave ──[ R15: 10kΩ ]──┬──→ U3B(-)
                           │      |\
SC Signal ──[ R16: 10kΩ ]──┤      | \
                           │      |  }──→ Full-Wave OUT
                         [ R17 ]  | /
                          10kΩ    |/
                           │      (+)
                 U3B OUT ──┘      │
                                  GND
```

**部品：**

- D2, D3: 1N4148 または BAT54
- R13-R17: 10kΩ 1% MF（精密マッチング推奨）
- C7: 100pF（ノイズ除去）

---

### 7.6 Peak Hold 回路

```
Full-Wave OUT ──[ R18: 10kΩ ]──┬──→ A/R Generator
                               │
                             [ D4 ]
                               │
                             [ C8 ]
                              1µF
                               │
                               GND
```

- C8 が放電されるのを A/R 回路で制御
- Attack/Release 特性の基礎

---

## 8. CV 生成・制御系

### 8.1 Threshold 比較回路

**目的**：入力レベルが Threshold を超えた分だけ CV 生成

**IC**：OPA2134（U4A）

```
Peak Hold ──[ R19: 10kΩ ]──┬──→ U4A(+)
                           │       |\
                         [ C9 ]    | \
                          100pF    |  }──→ Threshold OUT
                           │       | /
                           GND     |/
                                   (-)
                                   │
                    [ THR VR ] ────┤
                      100kB        │
                       │          GND
                   +10V Ref
                    (Zener)

Threshold Control:
  - VR最小：+10V（Threshold = +10dBu）
  - VR中央：+1V（Threshold = 0dBu）
  - VR最大：+0.1V（Threshold = -20dBu）
```

**部品：**

- R19: 10kΩ 1% MF
- C9: 100pF C0G
- VR1: 100kB（ALPS RK27 推奨）
- Zener: BZX85C10（10V、1W）+ 抵抗分圧

---

### 8.2 Attack/Release Generator

**Attack 回路（充電速度制御）：**

```
Threshold OUT ──[ Attack VR ]──┬──→ A/R Cap
                 100kΩ Log     │
                               ├─[ D5: 1N4148 ]
                               │
                              [ C10 ]
                               10µF
                               │
                               GND
```

**Release 回路（放電速度制御）：**

```
A/R Cap ──[ Release VR ]──[ D6: 1N4148 ]── GND
            1MΩ Log
```

**7 段ステップ選択（推奨）：**

```
         Attack Cap Select (Rotary SW)
              ┌─ 100pF  (0.1ms)
              ├─ 330pF  (0.3ms)
Threshold ────┼─ 1nF    (1ms)
  OUT         ├─ 3.3nF  (3ms)
              ├─ 10nF   (10ms)
              ├─ 33nF   (30ms)
              └─ 100nF  (100ms)
                   ↓
               [ Attack VR: 微調整 ]
                   ↓
               A/R Common Cap
```

同様に Release 側も 7 段＋ VR

**Auto Release 機能（オプション）：**

```
A/R Cap Voltage ──[ Comparator ]──[ FET Switch ]── Release VR Bypass
                        ↓
                  音楽的追従回路
```

- レベルが急激に上がった際、自動的に Release を早める
- FET: 2N7000 等

---

### 8.3 Ratio 制御（CV スケーリング）

**目的**：検波信号の何%を CV 生成に使うか制御

**回路：**

```
A/R OUT ──[ R20: 10kΩ ]───┬──→ U4B(+)
                          │      |\
                       [ C11 ]   | \
                        100pF    |  }──→ CV_scaled
                          │      | /
                         GND     |/
                                 (-)
                                 │
                         [ R21 ] │
                         ────────┤
                    ↑            │
              Ratio Select      GND

R21 = Rf（帰還抵抗）
Gain = 1 + (Rf / Rg)

Ratio対応表（Rg = 10kΩ固定）:
  Rf = 5kΩ   → Gain = 1.5  → Ratio = 1.5:1
  Rf = 10kΩ  → Gain = 2    → Ratio = 2:1
  Rf = 20kΩ  → Gain = 3    → Ratio = 3:1
  Rf = 30kΩ  → Gain = 4    → Ratio = 4:1
  Rf = 50kΩ  → Gain = 6    → Ratio = 6:1
  Rf = 90kΩ  → Gain = 10   → Ratio = 10:1
  Rf = 190kΩ → Gain = 20   → Ratio = 20:1
  Rf = ∞     → Gain = ∞    → Limiter
```

**ロータリー SW 実装：**

```
                    1P8T Rotary SW
U4B OUT ─┬─ 5.1kΩ  ─┐
         ├─ 10kΩ    │
         ├─ 20kΩ    ├──→ U4B(-)
         ├─ 30kΩ    │
         ├─ 51kΩ    │
         ├─ 91kΩ    │
         ├─ 200kΩ   │
         └─ Open   ─┘（Limiter）

微調整VR（5kB）をR21と直列に追加可能
```

**部品：**

- R20: 10kΩ 1% MF（Rg）
- R21a-g: 5.1k, 10k, 20k, 30k, 51k, 91k, 200kΩ（1% MF）
- SW2: ロータリー SW 1P8T
- VR2: 5kB（微調整、オプション）

---

### 8.4 Soft Knee 回路（オプション）

**目的**：Threshold 付近で緩やかに圧縮開始

```
CV_scaled ──┬─ Direct ────────┐
            │                 ├─→ DPDT SW ─→ CV_final
            └─[ 2×1N4148 ]    │   [Hard/Soft]
                 (並列反転)    │
                     ↓        │
                [ 10kΩ + 1µF ]│
                     ↓        │
                   Summer ────┘
```

- Soft 時：低レベルはダイオードで非線形圧縮
- Hard 時：直線的

---

### 8.5 CV Buffer & Offset 調整

**Unity Gain 調整機能付き CV バッファ：**

```
CV_final ──[ R22: 10kΩ ]───┬──→ U5A(+)
                           │       |\
                        [ C12 ]    | \
                         100pF     |  }──→ CV to V2164
                           │       | /
                          GND      |/
                                  (-)
                                   │
                          [ Unity Trim ]
                           10-turn 100kΩ
                                   │
                          ±10mV Ref
                           (Zener)
```

**±10mV 基準電圧生成：**

```
+15V ──[ 1kΩ ]──┬──[ BZX55C10 ]── GND  →  +10V
                │
              [ 2.2kΩ ]
                │
                ├──→ +10mV（分圧後）
                │
              [ 2.2MΩ ]
                │
               GND

同様に-10mVも生成（対称回路）
```

**Unity Gain 調整手順：**

1. Compressor OFF（CV = 0V 固定）
2. 1kHz, -10dBFS 入力（L/R 同時）
3. 出力を DAW で測定
4. トリマで L=R（±0.05dB 以内）に調整

**部品（per ch）：**

- R22: 10kΩ 1% MF
- C12: 100pF C0G
- Unity Trim: Bourns 3296W-1-104LF（100kΩ 10 回転）
- 基準電圧回路：Zener + 高精度抵抗

---

### 8.6 Stereo Link / Dual Mono 切替

```
L ch CV_final ──┬────────────┐
                │            ├─→ DPDT SW ─→ L ch V2164
                ├─[ Max選択 ]│   [Link/Dual]
                │  (Diode OR)│
R ch CV_final ──┴────────────┴─→ R ch V2164
```

**Max 選択回路：**

```
L CV ──[ D7: BAT54 ]──┐
                      ├──→ Common CV
R CV ──[ D8: BAT54 ]──┘
```

- より大きな信号側の CV が両 ch に適用される
- センター定位保持

**スイッチ：**

- Link 時：Common CV を両 ch に供給
- Dual 時：各 ch 独立 CV

---

## 9. Makeup Gain 段

### 9.1 基本回路（反転増幅＋ SAT 補正）

**IC**：OPA2134（U6A）

```
I/V OUT ──[ Rg: SAT連動 ]───────┬──→ U6A(-)
　                        　　　│ 　　 |\
                            [ C13 ]   | \
                             100pF    |  }──→ Makeup OUT
                               │      | /
                              GND     |/
                                      (+)
                                      │
                                     GND

Makeup OUT ──[ Rf: 可変 ]──→ U6A(-)

SAT Mode連動抵抗切替（DPDT連動）:
  CLEAN時: Rg = 10kΩ
  SAT時:   Rg = 3.9kΩ

Makeup Gain制御:
  Rf = 10kΩ + (0〜10kΩ VR)
  Gain = -(Rf / Rg)

  CLEAN時: Gain = -1 〜 -2 (0dB 〜 +6dB)
  SAT時:   Gain = -2.6 〜 -5.1 (+8dB 〜 +14dB)
           ※入力減衰分を自動補正
```

**Makeup Gain 範囲拡張版：**

```
Rf構成（7段ステップ + VR微調整）:

         Rotary SW (1P7T)
            ┌─ 10kΩ   (0dB)
            ├─ 12kΩ   (+2dB)
Makeup  ────┼─ 15kΩ   (+4dB)
  OUT       ├─ 20kΩ   (+6dB)
            ├─ 27kΩ   (+9dB)
            ├─ 39kΩ   (+12dB)
            └─ 56kΩ   (+15dB)
                │
              + VR3: 10kB (微調整 ±3dB)
                │
            → U6A(-)
```

**部品（1ch）：**

- Rg: 10kΩ / 3.9kΩ（DPDT SW 連動）
- Rf: 抵抗セット + VR3（10kB）
- C13: 100pF C0G
- SW3: ロータリー SW 1P7T

---

### 9.2 位相確認

- I/V 段：反転（-）
- Makeup 段：反転（-）
- **合計：正相（+）** ← Dry 信号と同相

---

## 10. Parallel Compression

### 10.1 Dry/Wet Mixer

**IC**：OPA2134（U7A）

```
Dry Signal ─────[ R23: 10kΩ ]────┬──→ U7A(-)
 (Input Buffer)                  │      |\
                                 │      | \
Wet Signal ─────[ R24: 10kΩ ]────┤      |  }──→ Mix OUT
 (Makeup OUT)                    │      | /
                              [ R25 ]   |/
                               10kΩ     (+)
                                 │       │
                      Mix OUT ───┘      GND

Standard Mix (固定):
  R23 = R24 = 10kΩ → Dry:Wet = 1:1

Variable Mix (可変):
  R23 = 10kΩ固定
  R24 = 0〜20kΩ VR → Dry:Wet = 100:0 〜 50:50
```

**Dry 信号遅延補償（オプション・高精度版）：**

Wet 信号は V2164 → I/V → Makeup と 3 段通過するため、Dry 信号との群遅延差が発生。

```
Input Buffer → [ All-Pass Filter ] → Dry Signal

APF (1次):
           R26
IN ──┬─── 10kΩ ────┬──→ U7B(+)
     │             │      |\
   [ C14 ]      [ C15 ]   | \
    10pF         10pF     |  }──→ Delayed Dry
     │             │      | /
    GND           GND     |/
                          (-)
                           │
                      [ R27: 10kΩ ]
                           │
                  Delayed Dry OUT ──┘
```

- 遅延量：約 5µs（Wet 経路と一致）
- 実装は任意（A/B テスト推奨）

**部品：**

- R23-R25: 10kΩ 1% MF
- R26-R27: 10kΩ 1% MF（APF 使用時）
- C14-C15: 10pF C0G（APF 使用時）
- VR4: 20kB（可変 Mix 時）

---

## 11. 出力段

### 11.1 DC Block（クリックノイズ対策強化版）

```
Mix OUT ──[ C16: 10µF NP ]──┬──→ Balanced Driver
                             │
                          [ R28 ]
                           100kΩ
                             │
                            GND
```

**コンデンサ選定（音質優先順）：**

1. WIMA MKP（フィルム、最高音質）
2. ELNA SILMIC II（高音質電解、NP）
3. Panasonic ECW-F（フィルム、廉価）

**部品：**

- C16: 10µF 50V（NP 推奨）
- R28: 100kΩ 1% MF

---

### 11.2 バランス出力ドライバー

**IC**：THAT1646（または SSM2142）

```
DC Block OUT ──[ R29: 604Ω ]──→ THAT1646 Pin3 (IN)

THAT1646:
  Pin6 → XLR Pin2 (Hot)
  Pin7 → XLR Pin3 (Cold)
  Pin4 → XLR Pin1 (GND)

  Pin8  → +15V (0.1µF + 10µF)
  Pin5  → -15V (0.1µF + 10µF)
  Pin1  → GND

Output Impedance Select:
  Pin2-GND間:
    - Open     → 600Ω出力
    - 10kΩ抵抗 → 10kΩ出力（Modern）
```

**部品（1ch）：**

- THAT1646 ×1（または SparkFun BOB-14003）
- R29: 604Ω 1%（THAT1646 推奨値）
- 0.1µF MLCC ×2
- 10µF タンタル ×2
- XLR 3pin オス ×1

---

### 11.3 VU メーター回路（Meter Distortion 防止）

**目的**：メーターコイルをオーディオ経路から完全分離

```
Balanced OUT ──[ R30: 10kΩ ]──→ U8A(+) (Buffer)
                                   |\
                                   | \
                                   |  }──→ Buffered
                                   | /
                                   |/
                                   (-)
                                    │
                            [ R31: 10kΩ ]
                                    │
                           Buffered OUT ──┘

Buffered OUT ──[ Full-Wave Rectifier ]──[ RC Filter ]── VU Meter
                                          (τ=300ms)
```

**VU メーター仕様：**

- 200µA FS（フルスケール）
- -20dB 〜 +3dB 表示
- 0VU = +4dBu
- キャリブレーション：1kHz 正弦波で調整

**部品：**

- R30-R31: 10kΩ 1% MF
- Rectifier: 1N4148 ×4（ブリッジ）
- RC Filter: 100kΩ + 3.3µF
- VU Meter: 200µA（SIFAM AL29WA 等）

---

### 11.4 GR メーター回路

**目的**：CV 電圧を可視化し、圧縮量を表示

```
CV Signal ──[ R32: 10kΩ ]──→ U8B(-) (Inverter)
 (負電圧)                       |\
                                | \
                                |  }──→ Positive CV
                                | /
                                |/
                               (+)
                                │
                               GND

Positive CV OUT ──[ R33: 10kΩ ]──→ U8B(-)

Gain = -1（反転）

Positive CV ──[ Rectifier ]──[ RC Filter ]── GR Meter
                              (τ=100ms)
```

**GR メーター仕様：**

- 200µA FS
- 0dB（圧縮なし）〜 -20dB 表示
- CV -0.66V → -20dB

**Calibration：**

1. CV = 0V → メーター 0dB
2. CV = -0.66V → メーター -20dB
3. トリマで感度調整

**部品：**

- R32-R33: 10kΩ 1% MF
- Rectifier: 1N4148 ×2
- RC Filter: 47kΩ + 2.2µF
- Trim: 10kΩ（感度調整）
- GR Meter: 200µA

---

### 11.5 ヘッドフォンアンプ

**IC**：NJM4556A（低歪み・高出力）

```
Mix OUT ──[ R34: 10kΩ ]──→ U9A(+)
                             |\
                             | \
                             |  }──[ R35: 22Ω ]── HP OUT (L)
                             | /    (出力保護)
                             |/
                            (-)
                             │
                    [ HP Vol: 10kB ]
                             │
                            GND

同様にR ch（U9B）

HP OUT (L/R) → 1/4" TRS Jack
  Tip: L
  Ring: R
  Sleeve: GND
```

**出力仕様：**

- 最大出力：50mW @ 32Ω
- 推奨負荷：32Ω 〜 600Ω
- THD+N: <0.01% @ 10mW

**部品：**

- NJM4556A ×1（ステレオ用）
- R34: 10kΩ ×2
- R35: 22Ω ×2（ワイヤーウンド推奨）
- HP Vol: Dual 10kB（ALPS RK27 推奨）
- HP Jack: 1/4" TRS（Neutrik NJ3FP6C 等）

---

## 12. 電源設計

### 12.1 電源仕様

**入力：**

- AC 12V-0-12V、500mA（トランス）
- または DC ±15V、500mA（スイッチング）

**安定化：**

- LM7815 / LM7915
- 放熱器必須（TO-220）

### 12.2 電源回路図

```
AC 12V ──[ Bridge Rectifier ]──[ C17: 2200µF ]──[ LM7815 ]─┬─→ +15V
                                                             │
                                                          [ C18 ]
                                                           100µF
                                                             │
                                                            GND

AC -12V ──[ Bridge Rectifier ]──[ C19: 2200µF ]──[ LM7915 ]─┬─→ -15V
                                                             │
                                                          [ C20 ]
                                                           100µF
                                                             │
                                                            GND
```

### 12.3 各ブロックへの分配

**スター配線（必須）：**

```
                  +15V Main
                     │
         ┌───────────┼───────────┐
         │           │           │
      [Input]    [V2164]    [Output]
      Section    Section    Section
         │           │           │
         └───────────┴───────────┘
                     │
                   GND Hub
                  (Star Point)
```

**各 IC のデカップリング：**

- すべての IC 電源ピン直近に：
  - 0.1µF MLCC（高周波）
  - 10µF タンタル（低周波）
- V2164 周辺は特に強化（0.1µF + 10µF + 100µF）

**部品（電源全体）：**

- トランス: 12V-0-12V 500mA
- ブリッジ: KBP206（600V 2A）×2
- C17, C19: 2200µF 35V 電解
- LM7815, LM7915: TO-220
- 放熱器: 10K/W 以下
- C18, C20: 100µF 35V タンタル
- 0.1µF MLCC ×20（各 IC 用）
- 10µF タンタル ×20（各 IC 用）

---

## 13. 完全部品リスト（BOM）

### 13.1 IC（ステレオ計）

| 品名               | 型番     | 数量 | 備考             |
| ------------------ | -------- | ---- | ---------------- |
| Precision OpAmp    | OPA2134  | 8    | Dual×8 = 16ch 分 |
| VCA                | V2164    | 2    | L/R 各 1         |
| バランスドライバー | THAT1646 | 2    | Output L/R       |
| HP Amp             | NJM4556A | 1    | Stereo           |
| 基準電圧           | REF3030  | 2    | L/R 独立（推奨） |
| 電源レギュレータ   | LM7815   | 1    | +15V             |
| 電源レギュレータ   | LM7915   | 1    | -15V             |

### 13.2 抵抗（1% Metal Film、ステレオ計）

| 抵抗値 | 数量 | 用途                 |
| ------ | ---- | -------------------- |
| 604Ω   | 2    | THAT1646 入力        |
| 3.9kΩ  | 2    | Makeup Rg（SAT）     |
| 4.7kΩ  | 4    | Input Trim           |
| 5.1kΩ  | 2    | Ratio                |
| 10kΩ   | 50   | 汎用（最多使用）     |
| 12kΩ   | 2    | SAT 入力 R           |
| 20kΩ   | 4    | Ratio, SC-HPF        |
| 22Ω    | 2    | HP 出力保護          |
| 27kΩ   | 2    | Ratio                |
| 30kΩ   | 2    | Ratio                |
| 33kΩ   | 6    | CLEAN 入力 R, I/V Rf |
| 39kΩ   | 2    | Ratio                |
| 47kΩ   | 4    | GR Meter             |
| 51kΩ   | 2    | Ratio                |
| 56kΩ   | 2    | Ratio                |
| 91kΩ   | 2    | Ratio                |
| 100kΩ  | 10   | DC Block, Bias       |
| 200kΩ  | 2    | Ratio                |
| 1kΩ    | 4    | CV 保護              |
| 2.2MΩ  | 4    | Unity 調整基準       |

### 13.3 可変抵抗

| 品名         | 型番         | 数量 | 用途                       |
| ------------ | ------------ | ---- | -------------------------- |
| 50kB         | ALPS RK27    | 2    | LEVEL Input                |
| 5kB          | ALPS RK27    | 2    | Trim Gain                  |
| 10kB         | ALPS RK27    | 4    | Makeup 微調整 ×2, HP Vol×2 |
| 20kB         | ALPS RK27    | 2    | Dry/Wet Mix（オプション）  |
| 100kB        | ALPS RK27    | 2    | Threshold                  |
| 100kΩ Log    | ALPS RK27    | 2    | Attack                     |
| 1MΩ Log      | ALPS RK27    | 2    | Release                    |
| 100kΩ 10turn | Bourns 3296W | 2    | Unity Gain 調整            |
| 10kΩ Trim    | Bourns 3296W | 2    | GR Meter 校正              |

### 13.4 コンデンサ

| 品名           | 型番/仕様 | 数量 | 用途                  |
| -------------- | --------- | ---- | --------------------- |
| 0.1µF MLCC     | X7R 50V   | 30   | デカップリング        |
| 10µF タンタル  | 35V       | 30   | 電源安定化            |
| 100µF タンタル | 35V       | 4    | 電源大容量            |
| 2200µF 電解    | 35V       | 2    | 電源整流後            |
| 10µF NP        | WIMA MKP  | 4    | DC Block（音質重視）  |
| 100nF Film     | WIMA MKS2 | 6    | SC-HPF, Audio         |
| 22pF C0G       | NPO       | 2    | I/V Cfb 選択          |
| 33pF C0G       | NPO       | 2    | I/V Cfb 選択          |
| 47pF C0G       | NPO       | 2    | I/V Cfb 選択          |
| 100pF C0G      | NPO       | 20   | RF 除去、カップリング |
| 1µF Film       | 63V       | 4    | A/R Peak Hold         |
| 10µF Film      | 63V       | 2    | A/R Generator         |
| 3.3µF Film     | 63V       | 2    | VU Meter Filter       |
| 2.2µF Film     | 63V       | 2    | GR Meter Filter       |

### 13.5 ダイオード

| 品名      | 型番     | 数量 | 用途                           |
| --------- | -------- | ---- | ------------------------------ |
| Schottky  | BAT54    | 12   | CV 保護 ×2, Link 選択 ×2, 整流 |
| Signal    | 1N4148   | 20   | A/R, 整流, Meter               |
| Zener 10V | BZX85C10 | 4    | Threshold 基準, Unity 基準     |
| Bridge    | KBP206   | 2    | 電源整流                       |

### 13.6 スイッチ類

| 品名        | 型番           | 数量 | 用途                                |
| ----------- | -------------- | ---- | ----------------------------------- |
| DPDT Toggle | Carling 2-643  | 4    | CLEAN/SAT×2, INT/EXT SC×2           |
| DPDT Toggle | 同上           | 2    | Stereo Link×2                       |
| DPDT Toggle | 同上           | 2    | Hard/Soft Knee（オプション）        |
| Rotary 1P3T | Lorlin CK-1010 | 2    | SC-HPF                              |
| Rotary 1P7T | Lorlin CK-1014 | 2    | Makeup Gain                         |
| Rotary 1P8T | Lorlin CK-1015 | 2    | Ratio                               |
| Rotary 1P7T | 同上           | 4    | Attack/Release 段選択（オプション） |

### 13.7 コネクタ類

| 品名          | 型番              | 数量 | 備考                  |
| ------------- | ----------------- | ---- | --------------------- |
| TS 1/4" Jack  | Neutrik NMJ6HCD2  | 2    | Input L/R             |
| XLR 3pin オス | Neutrik NC3MD-L-1 | 2    | Output L/R            |
| TS 1/4" Jack  | Neutrik NMJ6HCD2  | 2    | Ext SC L/R            |
| TRS 1/4" Jack | Neutrik NJ3FP6C   | 1    | Headphone             |
| DC Jack       | 2.1mm             | 1    | 電源入力（DC 使用時） |

### 13.8 その他

| 品名               | 数量 | 備考                   |
| ------------------ | ---- | ---------------------- |
| VU Meter 200µA     | 2    | Output L/R             |
| GR Meter 200µA     | 2    | L/R                    |
| Power LED          | 2    | 赤（+15V）、青（-15V） |
| トランス 12V-0-12V | 1    | 500mA 以上             |
| 放熱器 TO-220 用   | 2    | LM78/79 用             |
| テストポイント     | 10   | TP1-10                 |
| ジャンパーピン     | 6    | Cfb 選択用             |
| ノブ               | 16   | VR 用（デザイン任意）  |

---

## 14. 基板レイアウト指針

### 14.1 グラウンド設計

**階層構造：**

```
       Signal GND (Audio)
            │
            ├─ Input Section
            ├─ V2164 Section（Island GND）
            ├─ CV Section
            └─ Output Section
            │
       Power GND
            │
       Chassis GND (Safety)
```

**実装ルール：**

1. **スター配線必須**

   - GND ハブ（太い銅箔、直径 20mm）
   - 各セクションから放射状に接続

2. **V2164 周辺はアイランド GND**

   - V2164 の GND ピンを中心に独立 GND 領域
   - 1 点のみでメイン GND に接続

3. **デジタル系（LED/SW）は分離**
   - 最終的に電源 GND で合流

### 14.2 ノイズ対策

**電源ライン：**

- 各 IC に 0.1µF + 10µF を 3mm 以内配置
- V2164 は追加で 100µF 並列

**信号ライン：**

- 高インピーダンス部分は GND ガード
- CV 経路は両側に GND トレース配置

**外部ノイズ：**

- 金属筐体使用（アルミまたはスチール）
- 筐体とシャーシ GND を 1 点接続
- XLR シェルは筐体に接続（ノイズ排出）

---

## 15. キャリブレーション手順

### 15.1 必要機材

- オーディオインターフェース（±0.1dB 精度）
- DAW（波形分析機能）
- デジタルマルチメーター（4 桁以上）
- オシロスコープ（推奨）
- 信号発生器（1kHz 正弦波）
- ダミーロード 600Ω

### 15.2 電源確認（Step 1）

**手順：**

1. 全 IC を未実装の状態で電源投入
2. テストポイントで電圧測定

| 測定点   | 期待値 | 許容範囲 |
| -------- | ------ | -------- |
| +15V     | +15.0V | ±0.2V    |
| -15V     | -15.0V | ±0.2V    |
| GND      | 0V     | ±10mV    |
| リップル | <5mVpp | 電源品質 |

3. 問題なければ電源 OFF

### 15.3 IC 実装・動作確認（Step 2）

**手順：**

1. セクションごとに IC を実装
   - Input Section → V2164 → I/V → Output
2. 各段階で通電確認
3. IC 温度確認（異常発熱なし）

**チェックポイント：**

```
Input Buffer OUT: 無信号時 0V（±10mV）
I/V OUT: 無信号時 0V（±20mV）
Makeup OUT: 無信号時 0V（±50mV）
```

### 15.4 Unity Gain 調整（Step 3）★ 最重要 ★

**目的：**

- L/R 出力レベルを完全一致させる
- 定位ズレを防止

**手順：**

1. Compressor 機能をバイパス
   - CV 入力を 0V に固定（ジャンパー等）
   - または全 VR を「圧縮なし」位置
2. 1kHz, -10dBFS 信号を L/R 同時入力
3. DAW で出力レベルを測定

   ```
   期待値: L = R（±0.05dB以内）
   ```

4. Unity Gain Trimmer（L/R 独立）を調整
   - 右に回す：レベル上昇
   - 左に回す：レベル低下
5. L=R になるまで微調整（10 回転なので精密調整可能）

6. 記録：最終トリマ位置をマーク

**確認：**

- 定位テスト：センター信号が完全に中央に定位
- 周波数依存性：100Hz, 1kHz, 10kHz で再確認

### 15.5 Threshold 校正（Step 4）

**手順：**

1. 1kHz, 0dBu 信号を入力
2. Threshold VR を中央位置
3. GR Meter が動き始めるポイントを確認
4. 期待値：0dBu（±1dB）
5. ズレがある場合：Threshold 回路のバイアス調整

### 15.6 Ratio 検証（Step 5）

**手順：**

1. Threshold = -10dBu
2. Ratio = 4:1
3. 入力レベルを変化：-10dBu → 0dBu（+10dB）

**期待出力変化：**

```
入力: -10dBu → 0dBu（+10dB増加）
出力: -10dBu → -7.5dBu（+2.5dB増加）

計算: 10dB ÷ 4 = 2.5dB ✓
```

4. Ratio 2:1, 10:1 でも同様に確認
5. 誤差>1dB の場合：CV Scale 回路を見直し

### 15.7 Attack/Release 動作確認（Step 6）

**Attack 測定：**

1. バースト信号（1kHz, 10ms ON / 100ms OFF）入力
2. オシロで出力波形観測
3. 圧縮開始までの時間を測定

| Attack 設定 | 期待時間 | 許容範囲 |
| ----------- | -------- | -------- |
| 0.1ms       | 0.1ms    | ±0.05ms  |
| 1ms         | 1ms      | ±0.2ms   |
| 10ms        | 10ms     | ±2ms     |

**Release 測定：**

1. 持続音（1kHz）を 5 秒入力後、停止
2. GR Meter が 0dB に戻るまでの時間測定

| Release 設定 | 期待時間 | 許容範囲   |
| ------------ | -------- | ---------- |
| 100ms        | 100ms    | ±20ms      |
| 1s           | 1000ms   | ±100ms     |
| Auto         | 可変     | 音楽的追従 |

### 15.8 VU/GR メーター校正（Step 7）

**VU メーター：**

1. 1kHz, +4dBu 入力
2. メーターが 0VU を指すよう調整
3. ±3dBu でスケール確認

**GR メーター：**

1. CV = 0V → 0dB（圧縮なし）
2. CV = -0.66V → -20dB
3. Trim 調整で感度合わせ

### 15.9 THD+N 測定（Step 8）

**CLEAN Mode：**

```
入力: 1kHz, +4dBu
期待THD+N: <0.005%（-86dB）
```

**SAT Mode：**

```
入力: 1kHz, +4dBu
期待THD+N: 0.5〜3%（音楽的倍音）
```

**測定機材：**

- Audio Precision APx525（理想）
- または REW + Focusrite Scarlett（実用）

### 15.10 最終チェックリスト

- [ ] 電源電圧：±15V ±0.2V
- [ ] Unity Gain：L=R ±0.05dB
- [ ] Threshold 精度：±1dB
- [ ] Ratio 精度：±1dB
- [ ] Attack/Release 動作
- [ ] VU/GR Meter 校正
- [ ] THD+N 測定（CLEAN/SAT）
- [ ] ノイズフロア：<-90dBu
- [ ] クロストーク：<-60dB @ 1kHz
- [ ] 筐体アース確認
- [ ] 全スイッチ動作確認
- [ ] ヘッドフォン出力確認

---

**END OF DOCUMENT**
