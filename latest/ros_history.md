# ROS の歴史と技術変遷

> ロボティクスに新規参画したエンジニアが、ROS エコシステムの全体像を素早く理解するためのガイド。

## 目次

- [30秒で理解する ROS の歩み](#30秒で理解する-ros-の歩み)
- [ROS1 の時代 (2007–2025)](#ros1-の時代-20072025)
- [ROS2 への移行 (2014–)](#ros2-への移行-2014)
- [通信方式の進化: rosmaster から DDS へ](#通信方式の進化-rosmaster-から-dds-へ)
- [録画フォーマットの進化: .bag → .db3 → .mcap](#録画フォーマットの進化-bag--db3--mcap)
- [DDS 実装の選択](#dds-実装の選択)
- [ガバナンスの変遷](#ガバナンスの変遷)

---

## 30秒で理解する ROS の歩み

```
2007  ROS 誕生 (Willow Garage)
  │   通信: rosmaster (中央集権)
  │   録画: .bag
  │
2017  ROS2 初版リリース
  │   通信: DDS (分散 P2P)
  │   録画: .db3 (SQLite3)
  │
2022  Humble (LTS, 本プロジェクト採用)
  │   MCAP フォーマット公開 (Foxglove)
  │
2023  Iron (MCAP がデフォルトに)
  │
2025  ROS1 Noetic EOL ← ROS1 の終焉
  │   Kilted Kaiju リリース
  │
2026  現在 ← 本プロジェクト
```

**結論**: ROS1 は終了した。ROS2 + DDS + MCAP が現在の標準。

---

## ROS1 の時代 (2007–2025)

### 概要

ROS (Robot Operating System) は 2007 年に **Willow Garage** (Scott Hassan が 2006 年に設立) で開発が始まった。OS ではなくロボットソフトウェア開発のための**ミドルウェアフレームワーク**。

### Willow Garage と ROS の誕生

Scott Hassan は Google の初期開発に関わった人物で、2006 年にロボティクス研究所 Willow Garage をシリコンバレーに設立した。外部のベンチャーキャピタルを入れず、Hassan の個人資金 (累計約 8000 万ドル / 約 120 億円) で運営するという異例のモデルだった。

Willow Garage は ROS の開発に加え、PR2 (研究用ヒューマノイドロボット) や TurtleBot (教育用ロボット) を生み出し、ロボットソフトウェアのオープンソース文化を確立した。しかし年間経費が約 2000 万ドルに達し、2013 年に商業化への転換を試みたものの十分な収益を上げられず、Hassan が自身のもう一つの事業 (Suitable Technologies) に注力する判断をしたことで **2014 年初頭に閉鎖**。従業員の大半は Suitable Technologies に移籍し、ROS と Gazebo の開発は **OSRF** (Open Source Robotics Foundation) に移管された。

個人の資金力で始まったプロジェクトが、閉鎖後もオープンソースコミュニティとして発展し続けているのは、Willow Garage が最初から OSS として設計した成果といえる。

### 命名規則

ディストリビューション名はアルファベット順に進み、全て**亀の種類**が使われている。亀は ROS の文化的シンボルで、Logo 言語のタートルグラフィックスに由来する学習用シミュレータ「turtlesim」や、Willow Garage で開発された教育用ロボット「TurtleBot」がその起源。

- **ROS1**: 亀の種類をそのまま使用 (**B**ox Turtle → **C** Turtle → ... → **N**oetic Ninjemys)
- **ROS2**: 「形容詞 + 亀の種類」のパターンに変更 (**A**rdent Apalone → **H**umble Hawksbill → **K**ilted Kaiju)

### 主要ディストリビューション

| 年 | ディストリビューション | 備考 |
|----|----------------------|------|
| 2010 | Box Turtle | 初のディストリビューション |
| 2014 | Indigo Igloo | Willow Garage 閉鎖後、OSRF に移管 |
| 2018 | Melodic Morenia | Python 2 最後の LTS |
| 2020 | **Noetic Ninjemys** | **最後の ROS1 リリース** (Python 3 対応) |
| **2025/5** | - | **Noetic EOL = ROS1 の終焉** |

### ROS1 のアーキテクチャ

```
[rosmaster]  ←── 全ノードがここに登録 (単一障害点)
     │
     ├── [Node A] ──TCPROS──▶ [Node B]
     └── [Node C] ──UDPROS──▶ [Node D]
```

**限界:**
- rosmaster がクラッシュするとシステム全体が停止
- リアルタイム制御に不向き (タイミング保証なし)
- Linux のみサポート
- セキュリティ機構なし

---

## OSRF の設立と ROS2 の始動

Willow Garage は閉鎖前の **2012年** に、ROS の長期的な存続を見据えて非営利団体 **OSRF (Open Source Robotics Foundation)** を設立し、ROS と Gazebo (ロボットシミュレータ) の知的財産を移管した。CEO には Willow Garage 出身の **Brian Gerkey** (Player/Stage プロジェクト出身) が就任。

OSRF は ROS1 のメンテナンスを引き継ぐと同時に、ROS1 の根本的な限界を解消するために **ROS2 の設計を開始**。2014年の ROSCon で ROS2 を発表し、2017年に初版 (Ardent) をリリースした。2017年には組織名を **Open Robotics** に改称し、研究コミュニティだけでなく産業界への普及にも注力するようになった。

Willow Garage が「閉鎖前に後継組織を用意し、OSS として存続できる体制を整えた」ことが、ROS が一企業の消滅に影響されず発展し続けている理由である。

---

## ROS2 への移行 (2014–)

### なぜ ROS2 が必要だったのか

| ROS1 の課題 | ROS2 の解決策 |
|------------|--------------|
| rosmaster が単一障害点 | DDS による分散ディスカバリ |
| リアルタイム性なし | DDS の QoS による配信保証 |
| Linux 限定 | Windows / macOS / 組み込み対応 |
| セキュリティなし | DDS-Security 統合 |
| 単一マシン前提 | マルチロボット / WAN 対応 |

### ROS2 ディストリビューション

| 年 | ディストリビューション | サポート | デフォルト DDS | デフォルト録画形式 |
|----|----------------------|---------|--------------|-----------------|
| 2020/5 | Foxy Fitzroy | 3年 LTS | Fast DDS | SQLite3 (.db3) |
| 2021/5 | Galactic | 1.5年 | CycloneDDS | SQLite3 (.db3) |
| **2022/5** | **Humble Hawksbill** | **5年 LTS** | **CycloneDDS** | **SQLite3 (.db3)** |
| 2023/5 | Iron Irwini | 1.5年 | Fast DDS | **MCAP (.mcap)** |
| 2024/5 | Jazzy Jalisco | 5年 LTS | Fast DDS | MCAP (.mcap) |
| 2025/5 | Kilted Kaiju | 1.5年 | Fast DDS | MCAP (.mcap) |

> **本プロジェクトは Humble を採用。** Humble のデフォルト録画形式は SQLite3 だが、`-s mcap` オプションで明示的に MCAP を指定している。
>
> デフォルト DDS は Galactic〜Humble で CycloneDDS に切り替わり、Iron 以降は Fast DDS に戻された。ただし DDS は環境変数 `RMW_IMPLEMENTATION` で自由に切り替え可能。

---

## 通信方式の進化: rosmaster から DDS へ

### ROS1: 中央集権型

```
[rosmaster] が全ノードの名前解決を担当
     ↓
Node A → rosmaster に「/topic を publish する」と登録
Node B → rosmaster に「/topic を subscribe したい」と問い合わせ
     ↓
rosmaster が Node A の IP:Port を Node B に教える
     ↓
Node B → Node A に TCPROS で直接接続
```

### ROS2: DDS による分散型

```
rosmaster 不要

Node A → DDS Discovery (UDP マルチキャスト) で自動的に相手を発見
Node B → 同じ DDS ドメイン内なら自動接続
     ↓
RTPS プロトコルで直接 P2P 通信
```

**RTPS (Real-Time Publish-Subscribe)**: DDS の通信プロトコル標準。OMG (Object Management Group) が規格化。異なる DDS 実装 (CycloneDDS, Fast DDS 等) 間でもこのプロトコルで相互通信できる。

### ROS_DOMAIN_ID

DDS ドメインを分離するための仕組み。同じ `ROS_DOMAIN_ID` を持つノード同士だけが通信する。ネットワーク上に複数のロボットシステムがある場合に干渉を防ぐ。

---

## コラム: DDS の歴史 — ROS とは独立した 20 年の歩み

DDS は ROS2 のために作られた技術ではない。ROS とは全く独立した文脈で、**軍事・航空宇宙の分散リアルタイム通信**のために発展してきた規格である。

### タイムライン

| 年 | イベント |
|----|--------|
| 1991 | **RTI (Real-Time Innovations)** 設立 (Stan Schneider、シリコンバレー)。後に DDS 最大手となる |
| 2003 | RTI が OMG (Object Management Group) で DDS 標準化を主導 |
| **2004** | **OMG が DDS v1.0 を公開**。リアルタイム分散システムの Pub/Sub 標準として策定 |
| 2004 | **eProsima** 設立 (スペイン)。後に Fast DDS (旧 Fast RTPS) を開発 |
| 2015 | DDS v1.4 公開 |
| 2016 | ADLINK が **CycloneDDS** を開発開始 (後に Eclipse Foundation に寄贈) |
| **2017** | **ROS2 が DDS を採用して初版リリース** — ここで初めて ROS と DDS の歴史が合流 |

### ROS2 が DDS を選んだ理由

ROS2 チームは独自プロトコルを一から作る代わりに、20 年の実績がある DDS を採用した。[設計文書 (ROS on DDS)](https://design.ros2.org/articles/ros_on_dds.html) では以下が選定理由として挙げられている:

- OMG 標準として仕様が明文化・監査済み
- 軍艦、航空管制、自動運転車など**人命に関わるシステム**で実績がある
- 複数の実装から用途に応じて選択可能 (ベンダーロックインの回避)
- Pub/Sub + QoS + 分散ディスカバリが ROS2 の要件にそのまま合致

### DDS の採用分野

DDS は ROS2 の通信層としてだけでなく、以下のような分野で独立して使われている:

| 分野 | 用途例 |
|------|--------|
| 防衛・軍事 | 艦艇の戦闘管理システム、無人航空機 (MQ-9) |
| 航空管制 | Airbus のリアルタイムレーダー統合 |
| 自動運転 | 車載 ECU 間通信 (XPENG 等、200万台以上) |
| 医療機器 | 手術ロボット、患者モニタリング |
| 金融 | 株式・オプションの高頻度取引 |

> ロボット開発者にとっての DDS は「ROS2 の通信層」だが、DDS 自体は ROS とは無関係に発展した産業用ミドルウェアである。ROS2 はその成熟した技術を取り込むことで、研究用途から産業用途への飛躍を実現した。

---

## 録画フォーマットの進化: .bag → .db3 → .mcap

### タイムライン

| 時期 | 形式 | 特徴 |
|------|------|------|
| ROS1 (2010–) | **.bag** | ROS 独自バイナリ形式。シンプルだが ROS 専用 |
| ROS2 Foxy–Humble (2020–2023) | **.db3** (SQLite3) | 汎用 DB。クエリ可能だがストリーミング書込に弱い |
| ROS2 Iron– (2023–) | **.mcap** | Foxglove 開発。クラッシュ耐性、高速書込、自己完結型 |

### なぜ MCAP が採用されたか

| 特性 | SQLite3 (.db3) | MCAP (.mcap) |
|------|:-:|:-:|
| クラッシュ時のデータ損失 | 大 (WAL 未コミット分) | 最小 (数メッセージ) |
| 書込スループット | 中 | 高 |
| CRC 破損検知 | なし | あり |
| スキーマ自己完結 | なし (外部依存) | あり (ファイル内に埋込) |
| 圧縮 | なし | zstd / lz4 |
| Foxglove Studio 対応 | 変換必要 | ネイティブ対応 |

### MCAP の誕生と Foxglove

**Foxglove** は 2021年にサンフランシスコで設立されたロボティクス開発ツールの会社。創業者の Adrian Macneil と Roman Shtylman は自動運転の **Cruise** (GM 傘下) 出身で、ロボットデータの可視化・デバッグツールが業界に不足していることに着目して起業した。

Foxglove は可視化ツール (Foxglove Studio) の開発と並行して、既存の録画フォーマット (ROS1 の .bag、ROS2 の SQLite3) の課題を解決する新しいコンテナフォーマット **MCAP** を設計した。

- **2021年**: Foxglove 社設立
- **2022年2月**: MCAP v1.0.0 をオープンソース公開
- **2023年5月**: ROS2 Iron でデフォルト rosbag フォーマットに採用
- **2025年11月**: Series B で 4000万ドルを調達。自動車、航空宇宙、防衛など数百社が利用

MCAP はシリアライゼーション非依存の**コンテナフォーマット**。ROS2 の CDR だけでなく、Protobuf や JSON もサポートする汎用設計。Web 開発に例えると、Foxglove は「ロボット版の Datadog + Chrome DevTools」のような位置づけで、本プロジェクトが MCAP で録画したデータは Foxglove Studio で直接可視化できる。

---

## DDS 実装の選択

### 主要な DDS 実装

| 実装 | 開発元 | ライセンス | 特徴 |
|------|--------|----------|------|
| **CycloneDDS** | ZettaScale (旧 ADLINK) | EPL 2.0 | 軽量、低レイテンシ。Humble のデフォルト |
| **Fast DDS** | eProsima | Apache 2.0 | 機能豊富。Iron 以降のデフォルト |
| **Connext DDS** | RTI | 商用 | 高信頼性。産業用途向け |

### 本プロジェクトの DDS

Humble のデフォルトである **CycloneDDS** を使用。ロボット側が Fast DDS 等の異なる実装であっても、RTPS 標準プロトコルにより相互通信可能 (詳細は [ARCHITECTURE.md](../ARCHITECTURE.md) の DDS 通信セクションを参照)。

---

## ガバナンスの変遷

ROS の管理組織は幾度かの変遷を経ている:

| 年 | イベント |
|----|--------|
| 2006 | Willow Garage 設立 (ROS の開発を開始) |
| 2012 | **OSRF** (Open Source Robotics Foundation) 設立。ROS と Gazebo の知的財産を Willow Garage から移管 |
| 2014 | Willow Garage 閉鎖 |
| 2017 | OSRF が **Open Robotics** に改称。営利部門 **OSRC** (Open Source Robotics Corporation) を通じて商業サポートを展開 |
| 2022 | **Intrinsic** (Alphabet/Google 傘下) が OSRC を買収。OSRF (非営利) は独立を維持し、ROS の知的財産は OSRF に残留。OSRC のエンジニアが Intrinsic に移籍 |
| 2024 | **OSRA** (Open Source Robotics Alliance) 設立。NVIDIA、Intrinsic、Qualcomm 等が参加し、ROS の持続的な開発体制を確立 |
| 2025/10 | Intrinsic が **Foxconn** と合弁事業を発表 (工場自動化向け汎用知能ロボット) |
| **2026/2** | **Intrinsic が Google に統合**。Google DeepMind と連携し、Gemini AI をロボティクスに活用する方針を発表 |

### 現在の構造

```
[OSRF] (非営利、独立)
  │  ROS / Gazebo の知的財産を保有
  │
  ├── [OSRA] (開発の実務)
  │     NVIDIA, Intrinsic, Qualcomm 等が参加
  │
  └── ROS はオープンソースのまま維持

[Google]
  └── [Google DeepMind]
        └── [Intrinsic] (旧 OSRC のエンジニアリングリソース)
              Gemini AI × ロボティクス
              Foxconn との工場自動化合弁
```

ROS の知的財産は OSRF が保有しており、Google の所有物にはなっていない。しかし開発の実働部隊の多くが Google 傘下にいるため、Google はロボティクス OSS エコシステムにおいて最大の影響力を持つ存在となっている。

---

## コラム: Google DeepMind と Physical AI

### Google DeepMind とは

2023年に **Google Brain** と **DeepMind** が統合されて誕生した Google の AI 研究部門。CEO は DeepMind 創設者の **Demis Hassabis** (2024年ノーベル化学賞受賞)。

| 前身 | 設立 | 代表的成果 |
|------|------|-----------|
| **DeepMind** | 2010年 (ロンドン)、2014年に Google が約5億ドルで買収 | AlphaGo (囲碁AI)、AlphaFold (タンパク質構造予測) |
| **Google Brain** | 2011年 (Google 内部) | Transformer アーキテクチャ (現在の LLM の基盤)、TensorFlow |

2023年の統合後、Gemini モデルの開発を主導。世界最大規模の AI 研究組織の一つ。

### なぜロボティクスと関係するのか

2026年2月に Intrinsic (旧 OSRC のエンジニアを含む) が Google DeepMind に統合された。これは Google が **Physical AI** (AI を物理世界のロボットで動かす) に本格的に投資していることを意味する。

```
Google DeepMind
  ├── Gemini (大規模言語モデル)
  ├── AlphaFold (科学研究)
  └── Intrinsic (ロボティクス) ← ROS/Gazebo の開発者が多数在籍
        └── Foxconn との合弁 (工場自動化)
```

AI が言語や画像だけでなく、ロボットを通じて物理世界に作用する時代に向かっている。ROS エコシステムで培われた技術と人材が、その中核を担う位置にいる。

---

## コラム: HuggingFace と LeRobot

### HuggingFace とは

2016年にフランスで設立された AI プラットフォーム企業。当初はチャットボットの会社だったが、AI モデルとデータセットの共有プラットフォームに転身し、**「AI の GitHub」**と呼ばれる存在になった。2023年にシリーズ D で 2.35億ドルを調達 (評価額 45億ドル)。

NLP (自然言語処理) や画像認識の世界で HuggingFace がやったこと — モデルとデータセットの標準化・共有・再利用 — を、ロボティクスでも実現しようとしているのが **LeRobot** プロジェクト。

### ロボット学習データフォーマットの変遷

LeRobot が登場する前は、統一的なフォーマットがなく、研究グループごとにバラバラだった:

| 時期 | フォーマット | 技術基盤 | 主な用途 |
|------|------------|---------|---------|
| 〜2021年 | **HDF5** | HDF5 (.h5) | ALOHA 初期、ACT 論文 |
| 2021年〜 | **RLDS** | TensorFlow Datasets (TFRecord) | Google RT-1/RT-2、Open X-Embodiment |
| 2024年〜 | **LeRobot v3.0** | Parquet + MP4 | HuggingFace エコシステム |

**RLDS (Reinforcement Learning Datasets)** は Google が 2021年に公開したフォーマットで、TensorFlow Datasets の上に構築されていた。Google の RT-1/RT-2 や、22種のロボットから 100万エピソード以上を集めた Open X-Embodiment プロジェクトで使用され、一時はデファクト標準だった。

しかしロボット学習の研究コミュニティが **TensorFlow/JAX から PyTorch に移行**する流れの中で、TensorFlow に依存する RLDS は使いづらくなった。この空白を埋める形で 2024年に HuggingFace が **LeRobot** をオープンソース公開した。

### LeRobot の位置づけ

ROS がロボットの**通信**を標準化したのに対し、LeRobot はロボットの**学習データとモデル**を標準化する位置づけ。

```
従来のロボティクス                    AI/ML プラットフォーム
(ROS, DDS, ハードウェア制御)         (モデル共有, データセット管理)
         │                                    │
         │          2024年に合流               │
         └──────────────┬─────────────────────┘
                        ▼
              LeRobot (HuggingFace)
              ├── データフォーマット標準化 (LeRobot v3.0)
              ├── モデル・データセットの Hub 共有
              └── 学習・デプロイの統合フレームワーク
```

RLDS から LeRobot への移行も進んでおり、Open X-Embodiment データセットも LeRobot 形式で再公開されている。

### ロボティクスへの影響

| 分野 | HuggingFace がもたらした変化 |
|------|----------------------------|
| **データ共有** | Hub 上でロボットデータセットを公開・再利用可能に (Open X-Embodiment 等) |
| **フォーマット標準化** | LeRobot v3.0 が事実上のデータセット標準になりつつある |
| **再現性** | 学習済みモデルとデータセットをセットで共有でき、研究の再現が容易に |
| **コミュニティ** | ALOHA, Koch v1.1 等の低コストロボットとの統合が進み、個人レベルでの模倣学習が可能に |

ROS (通信) → MCAP (録画) → LeRobot (学習) という流れで、本プロジェクトはこのパイプラインの中間に位置する。

---

## 参考リンク

- [ROS 公式ドキュメント](https://docs.ros.org/)
- [ROS on DDS (設計文書)](https://design.ros2.org/articles/ros_on_dds.html)
- [REP 2000 - ROS 2 Releases](https://www.ros.org/reps/rep-2000.html)
- [MCAP Specification](https://mcap.dev/spec)
- [Foxglove - Introducing MCAP](https://foxglove.dev/blog/introducing-the-mcap-file-format)
- [OSRA 設立発表](https://www.openrobotics.org/blog/2024/3/18/announcing-the-open-source-robotics-alliance-osra)
