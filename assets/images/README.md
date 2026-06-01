# Images

SLAM 評価グラフを `slam_*.png` として配置しています。
追加のスクリーンショット (RViz / Gazebo) や系統図はここに追加してください。

## 既存ファイル

| ファイル | 内容 | 参照元 |
|---------|------|--------|
| `slam_kiss_yaw_vs_time.png` | KISS-ICP の yaw vs 時間 (校正前/後比較) | docs/06_results.md |
| `slam_wheel_yaw_vs_time.png` | wheel odometry の yaw vs 時間 (校正前/後比較) | docs/06_results.md |
| `slam_kiss_xy_trajectory.png` | KISS-ICP の XY 軌跡 (校正前/後比較) | docs/06_results.md |
| `slam_wheel_xy_trajectory.png` | wheel odometry の XY 軌跡 (校正前/後比較) | docs/06_results.md |
| `slam_repeatability_yaw.png` | KISS-ICP の 3 ラン繰り返し yaw | docs/06_results.md |
| `slam_repeatability_xy.png` | KISS-ICP の 3 ラン繰り返し XY 軌跡 | docs/06_results.md |
| `slam_three_sources_yaw.png` | wheel / KISS / EKF の 3 ソース比較 yaw | docs/03_slam.md, docs/06_results.md |
| `slam_three_sources_xy.png` | wheel / KISS / EKF の 3 ソース比較 XY | docs/03_slam.md, docs/06_results.md |

## 推奨追加

| ファイル名案 | 内容 | 目的 |
|------------|------|------|
| `robot_photo_front.jpg` | 実機正面 | README ヒーロー |
| `robot_photo_side.jpg` | 実機側面 (LiDAR 取付位置が見える) | docs/02_system.md |
| `system_architecture.png` | ROS 2 ノードグラフ図 | docs/02_system.md |
| `tf_tree.png` | TF ツリー可視化 (rqt_tf_tree) | docs/02_system.md |
| `stp4_concept.png` | (x, y, t) 探索の概念図 | docs/04_planning.md |
| `pedestrian_pipeline.png` | 検出パイプライン図 | docs/05_pedestrian.md |
| `self_filter_diagram.png` | 自己フィルタ 3D ボックス図解 | docs/07_lessons.md |
| `rviz_before_self_filter.png` | 自己フィルタ前 RViz | docs/07_lessons.md |
| `rviz_after_self_filter.png` | 自己フィルタ後 RViz | docs/07_lessons.md |
