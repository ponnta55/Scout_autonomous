# Scout Autonomous — 実環境における時空間ナビゲーション

> 動的環境 (歩行者) に対応する時空間ナビゲーションを、4輪台車 + 3D LiDAR でゼロから組み上げた研究プロジェクトの **成果まとめ** リポジトリです。
> 時空間アルゴリズムは **シミュレーション (Gazebo Harmonic) で歩行者回避動作を検証**、実機では **SLAM スタックの構築と静的環境での自律走行を実証** まで到達しています。動的環境での実機検証は今後の課題です。
>
> 本リポジトリにはソースコードは含まれていません。**設計判断・検証・結果** を中心に構成しています。

---

## English Summary (90 seconds)

I built an autonomous navigation stack on an AgileX SCOUT MINI equipped with a Hesai QT128 3D LiDAR. The system performs LiDAR-only SLAM (KISS-ICP), spatio-temporal path planning that treats pedestrians as **future** obstacles in `(x, y, t)`, and onboard pedestrian detection / tracking / trajectory prediction. The contribution of this work is not the individual algorithms — it is the systems-level work of taking a stack that worked in Gazebo and making it work in the real world: diagnosing a wheel-odometry calibration bias, dropping wheel input from the EKF after data-driven analysis, fixing self-point contamination of the LiDAR (the robot detected itself as an obstacle), and stabilizing the planner under intermittent localisation. The spatio-temporal pedestrian-avoidance behavior was verified in Gazebo simulation; on the real robot, SLAM and waypoint-following in a static environment are working, while validating dynamic-pedestrian avoidance on hardware is ongoing work.

---

## ハイライト

| 項目 | 内容 |
|------|------|
| **プラットフォーム** | AgileX SCOUT MINI (4輪 skid-steer) + Hesai QT128 (128ch 3D LiDAR) |
| **SLAM** | KISS-ICP (LiDAR-only) + EKF (将来 GNSS/IMU 統合用に温存) |
| **経路計画** | 時空間プランナ STP4 — `(x, y, t)` 3D 探索で「待機 vs 迂回」を自動選択 |
| **歩行者検出** | DBSCAN → 32次元特徴量 → ONNX (Random Forest) → ByteTrack 追跡 → ONNX 軌道予測 |
| **計算** | GPU 不要・CPU のみ・全パイプライン 5Hz でリアルタイム |
| **実証範囲** | Sim: 動的歩行者環境での回避動作 / 実機: SLAM + 静的環境での自律走行 |
| **今後の課題** | 実機での歩行者環境検証、屋外運用、GNSS/IMU 統合 |

---

## デモ

<!-- TODO: GIF を assets/videos/ に配置したらここに埋め込み -->

| シーン | 内容 | 環境 |
|--------|------|------|
| ![実機自律走行 GIF プレースホルダ](assets/videos/_PLACEHOLDER_autonomous_drive.gif) | 屋内自律走行 (静的障害物) | 実機 |
| ![Sim 歩行者回避 GIF プレースホルダ](assets/videos/_PLACEHOLDER_sim_pedestrian_avoidance.gif) | 歩行者を「未来の障害物」として扱う待機戦略 | C++ |
| ![Sim 比較 GIF プレースホルダ](assets/videos/_PLACEHOLDER_sim_world.gif) | Gazebo Harmonic 上の検証 world | Gazebo |

> GIF 差し替え手順は [assets/videos/README.md](assets/videos/README.md) を参照。

---

## ドキュメント目次

8 章構成。**興味のある章から読めます**が、まず [01_overview](docs/01_overview.md) を読むと全体像が掴めます。

| # | 章 | 内容 |
|---|----|------|
| 01 | [プロジェクト概要](docs/01_overview.md) | 研究背景・課題設定・私の貢献 |
| 02 | [システム構成](docs/02_system.md) | ハード/ソフト構成、ROS 2 ノードグラフ、TF ツリー |
| 03 | [SLAM (自己位置推定)](docs/03_slam.md) | KISS-ICP の採用判断、EKF 構成の検証と統廃合 |
| 04 | [時空間経路計画](docs/04_planning.md) | STP4 — `(x, y, t)` 探索で動的環境に対応 |
| 05 | [歩行者検出・追跡・予測](docs/05_pedestrian.md) | LiDAR → 検出 → 追跡 → 軌道予測パイプライン |
| 06 | [評価・結果](docs/06_results.md) | SLAM 精度、計画タイミング、実環境走行の結果 |
| 07 | [Sim → Real で学んだこと](docs/07_lessons.md) | 自己点群問題、wheel 校正バイアス、その他のハマりどころ |
| 08 | [Jetson 実機移行と並列高速化](docs/08_jetson.md) | 開発PC→Jetson 移行、クロスマシン互換基盤、検出前段 3 倍高速化 |

---

## 私が担当した範囲 (Contribution Statement)

このプロジェクトでは、以下を **個人で** 担当しました。
- **動的歩行者環境の経路計画** — 既存の歩行者検出器を用いてシミュレーション上で動的環境での経路計画を実施
- **要件整理と全体設計** — シミュレーション版アルゴリズムを実機に展開するためのアーキテクチャ設計
- **ROS 2 ノード実装の統合** — 検出 / 追跡 / 予測 / 計画パイプラインを単一ノードに統合
- **SLAM スタックの構築** — KISS-ICP の導入、TF 統一、EKF 構成の検証
- **計測実験と分析** — 純回転試験による SLAM 精度評価、ホイール校正バイアスの発見と原因究明
- **実機トラブルシューティング** — 自己点群問題、計画継続性、CAN/Ethernet 設定など
- **ドキュメンテーション** — 設計判断と検証結果の体系的な記録

歩行者検出器の **原型** は研究室の先行研究から引き継いだものを、実機向けにチューニング・統合しました。

---

## 主要な技術判断 (TL;DR)

| # | 判断 | 根拠 | 詳細 |
|---|------|------|------|
| 1 | **EKF の wheel 入力を切る** | 純回転試験で wheel バイアスを発見、EKF は実質 KISS のスムージング層になっていた | [03_slam](docs/03_slam.md), [07_lessons](docs/07_lessons.md) |
| 2 | **LiDAR-only SLAM (KISS-ICP)** | 軽量・CPU 動作・チューニング項目が少ない | [03_slam](docs/03_slam.md) |
| 3 | **時空間プランナ (STP4)** | 静的 2D 計画では待機戦略が表現できない | [04_planning](docs/04_planning.md) |
| 4 | **3D ボックス自己フィルタ** | sim では発覚しなかった「ロボット自身を障害物と誤認」を解決 | [07_lessons](docs/07_lessons.md) |
| 5 | **TF 全フレーム base_link 統一** | LiDAR フレームでの推定値を base_link に正しく投影、SLAM と planner の座標系を一致 | [02_system](docs/02_system.md) |
| 6 | **出力ビット一致を保つ決定的並列化** | 高速化 (Jetson 3.0x) しても回帰テストで等価性を証明できる形に制約 | [08_jetson](docs/08_jetson.md) |

---

## License
All Rights Reserved. 本リポジトリは未発表研究を含むため、無断複製・転載・再配布を禁止します。
