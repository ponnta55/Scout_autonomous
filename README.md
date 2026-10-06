# Scout Autonomous — 実環境における時空間ナビゲーション

> 動的環境 (歩行者) に対応する時空間ナビゲーションを、4輪台車 + 3D LiDAR でゼロから組み上げた研究プロジェクトの **成果まとめ** リポジトリです。
> 時空間アルゴリズムは **シミュレーション (Gazebo Harmonic) で歩行者回避動作を検証** しました。実機では **3D LiDAR-IMU SLAM (GLIM) と自作の事前地図ローカライゼーションの上で自律走行** まで到達しています (2026-10)。動的環境での実機の定量検証は今後の課題です。
>
> 本リポジトリにはソースコードは含まれていません。**設計判断・検証・結果** を中心に構成しています。

---

## English Summary (90 seconds)

I built an autonomous navigation stack on an AgileX SCOUT MINI equipped with a Hesai QT128 3D LiDAR, running on a Jetson AGX Orin. The system performs spatio-temporal path planning that treats pedestrians as **future** obstacles in `(x, y, t)`, onboard pedestrian detection / tracking / trajectory prediction, and LiDAR-IMU SLAM. Localisation started as LiDAR-only KISS-ICP and has since moved to GLIM (3D LiDAR-IMU SLAM) with a prior-map localisation module I wrote so that waypoints stay valid across sessions. Detection now uses a GPU CenterPoint backend; on 389 hand-labelled pedestrians it scores F1 0.941 against 0.456 for the original classical detector, which overturned my earlier eyeball-based conclusion. The contribution of this work is not the individual algorithms — it is the systems-level work of taking a stack that worked in Gazebo and making it work in the real world: diagnosing a wheel-odometry calibration bias, fixing self-point contamination of the LiDAR, tracing a "slow SLAM" symptom to DDS dropping 2.8 MB point-cloud frames, and finding a unit bug in ego-motion compensation that made pedestrians behind the robot appear to overtake it. Pedestrian avoidance was verified in Gazebo simulation; on the real robot, autonomous waypoint driving on a prior map works (October 2026), while quantitative validation of pedestrian avoidance on hardware is ongoing work.

---

## ハイライト

| 項目 | 内容 |
|------|------|
| **プラットフォーム** | AgileX SCOUT MINI (4輪 skid-steer) + Hesai QT128 (128ch 3D LiDAR) |
| **SLAM** | GLIM (3D LiDAR-IMU, GPU) + 自作の事前地図ローカライゼーション / 従来構成 KISS-ICP + EKF も切替で温存 |
| **経路計画** | 時空間プランナ STP4 — `(x, y, t)` 探索で「待機 vs 迂回」を自動選択 / 後継の時空間 Hybrid A* — `(x, y, θ, t)` で旋回・加減速の時間も数える (切替式, 2026-10) |
| **歩行者検出** | CenterPoint (GPU DNN, 既定) / Classical (DBSCAN → 特徴量 → RF) を切替 → ByteTrack 追跡 → ONNX 軌道予測 |
| **検出精度** | 人手 GT 389 人で CenterPoint **F1 0.941** (Classical 0.456) |
| **計算** | Jetson AGX Orin。Classical は CPU のみで動作、CenterPoint は GPU で検出〜予測 約 65 ms |
| **実証範囲** | Sim: 動的歩行者環境での回避動作 / 実機: GLIM ローカライゼーション上の自律走行・一時停止点での停止と再開 |
| **今後の課題** | 実機での歩行者環境の定量検証、GNSS 融合の屋外検証、差動駆動の旋回時間を計画に反映 |

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

12 章構成。**興味のある章から読めます**が、まず [01_overview](docs/01_overview.md) を読むと全体像が掴めます。

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
| 09 | [DNN 検出器の導入と定量評価](docs/09_dnn_detection.md) | CenterPoint 導入、人手 GT による比較で評価が逆転、誤検出の正体は追跡 |
| 10 | [IMU・GLIM・ローカライゼーション](docs/10_glim_localization.md) | IMU 統合、GLIM 実機稼働、DDS フレーム落ちの真因、事前地図ローカライゼーション |
| 11 | [実機自律走行の運用化](docs/11_field_navigation.md) | 走行管理・障害物余白・ポーズポイント、歩行者予測の単位バグ修正 |
| 12 | [時空間 Hybrid A*](docs/12_st_hybrid_astar.md) | 向きと時間を同時に扱う局所プランナ。待機を出発遅延で表し 42 ms 以下、STP4 との比較動画 |

---

## 私が担当した範囲 (Contribution Statement)

このプロジェクトでは、以下を **個人で** 担当しました。
- **動的歩行者環境の経路計画** — 既存の歩行者検出器を用いてシミュレーション上で動的環境での経路計画を実施
- **要件整理と全体設計** — シミュレーション版アルゴリズムを実機に展開するためのアーキテクチャ設計
- **ROS 2 ノード実装の統合** — 検出 / 追跡 / 予測 / 計画パイプラインを単一ノードに統合
- **SLAM スタックの構築** — KISS-ICP の導入、TF 統一、EKF 構成の検証
- **計測実験と分析** — 純回転試験による SLAM 精度評価、ホイール校正バイアスの発見と原因究明
- **実機トラブルシューティング** — 自己点群問題、計画継続性、CAN/Ethernet 設定など
- **DNN 検出器の導入と定量評価** — GPU 検出器の統合、GT 作成と評価手法の設計
- **3D SLAM とローカライゼーション** — IMU 統合、GLIM の実機移植、事前地図ローカライゼーションの実装
- **ドキュメンテーション** — 設計判断と検証結果の体系的な記録

歩行者検出器の **原型** は研究室の先行研究から引き継いだものを、実機向けにチューニング・統合しました。

---

## 主要な技術判断 (TL;DR)

| # | 判断 | 根拠 | 詳細 |
|---|------|------|------|
| 1 | **EKF の wheel 入力を切る** | 純回転試験で wheel バイアスを発見、EKF は実質 KISS のスムージング層になっていた | [03_slam](docs/03_slam.md), [07_lessons](docs/07_lessons.md) |
| 2 | **LiDAR-only SLAM (KISS-ICP)** | 軽量・CPU 動作・チューニング項目が少ない (初期構成。後に #8 で GLIM へ) | [03_slam](docs/03_slam.md) |
| 3 | **時空間プランナ (STP4)** | 静的 2D 計画では待機戦略が表現できない | [04_planning](docs/04_planning.md) |
| 4 | **3D ボックス自己フィルタ** | sim では発覚しなかった「ロボット自身を障害物と誤認」を解決 | [07_lessons](docs/07_lessons.md) |
| 5 | **TF 全フレーム base_link 統一** | LiDAR フレームでの推定値を base_link に正しく投影、SLAM と planner の座標系を一致 | [02_system](docs/02_system.md) |
| 6 | **出力ビット一致を保つ決定的並列化** | 高速化 (Jetson 3.0x) しても回帰テストで等価性を証明できる形に制約 | [08_jetson](docs/08_jetson.md) |
| 7 | **既定の検出器を CenterPoint に** | 人手 GT で F1 0.941 vs 0.456。目視評価の結論を撤回 | [09_dnn_detection](docs/09_dnn_detection.md) |
| 8 | **SLAM を GLIM へ、IMU はヨー角速度のみ融合** | 加速度の融合は静止時に位置が流れた。地図座標を固定するためローカライゼーションを自作 | [10_glim_localization](docs/10_glim_localization.md) |
| 9 | **点群の DDS 配送設定を全ノードに適用** | 2.8 MB の点群が共有メモリに入らずフレーム落ち (39% → 100%) | [10_glim_localization](docs/10_glim_localization.md) |
| 10 | **待機を「出発の遅延」で表す時空間 Hybrid A*** | 待機ノードは状態爆発 (200 ms 超)。SIPP 型の支配と出発遅延で 42 ms 以下に。加速して抜ける動きは入れない | [12_st_hybrid_astar](docs/12_st_hybrid_astar.md) |

---

## License
All Rights Reserved. 本リポジトリは未発表研究を含むため、無断複製・転載・再配布を禁止します。
