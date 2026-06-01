# Videos / GIFs

このディレクトリには走行デモの GIF/動画を配置します。

## 必要な素材一覧

各 GIF は README.md と docs/06_results.md から参照されています。
ファイル名は **そのまま** で配置すれば自動でリンクが繋がります。

| ファイル名 | 内容 | 推奨長さ |
|-----------|------|---------|
| `_PLACEHOLDER_autonomous_drive.gif` | 屋内自律走行 (静的障害物のみ) | 10-15 秒 |
| `_PLACEHOLDER_pedestrian_avoidance.gif` | 歩行者を待機して回避 | 10-20 秒 |
| `_PLACEHOLDER_pedestrian_detour.gif` | 歩行者を迂回 | 10-20 秒 |
| `_PLACEHOLDER_sim_vs_real.gif` | Gazebo と実機の並列比較 | 15-20 秒 |

→ 用意ができたら `_PLACEHOLDER_` プレフィックスを **削除** してファイル名を上書き保存してください。
（README/docs 内のリンクも `_PLACEHOLDER_` を外したものに合わせて更新が必要)

## 撮影 → GIF 変換手順

### 1. 画面録画

```bash
# ROS バッグ再生 + RViz 録画 (例: GNOME Screen Recorder, OBS)
# あるいは実機走行を別カメラで撮影
```

### 2. 動画 → GIF 変換 (ffmpeg)

```bash
# 高品質 GIF (パレット 2 段階生成)
ffmpeg -i input.mp4 -vf "fps=12,scale=720:-1:flags=lanczos,palettegen" palette.png
ffmpeg -i input.mp4 -i palette.png -filter_complex "fps=12,scale=720:-1:flags=lanczos[x];[x][1:v]paletteuse" output.gif
```

ファイルサイズの目安: 1 GIF < 10 MB に抑えると GitHub での読み込みが速い。

### 3. 配置

```bash
cp output.gif /path/to/Scout_autonomous/assets/videos/autonomous_drive.gif
```

### 4. README / docs 内のリンク更新

`_PLACEHOLDER_` を外したファイル名に合わせて、README.md と docs/06_results.md のリンクを書き換え。

## 補助スクリーンショット

走行中の RViz / Gazebo 静止画も `../images/` に置くと、各章の説明強化に使えます。

| 推奨ファイル | 内容 |
|------------|------|
| `rviz_self_filter_before.png` | 自己フィルタなしで自分の周りに点群もや |
| `rviz_self_filter_after.png` | 自己フィルタありでクリーン |
| `rviz_pedestrian_marker.png` | 歩行者検出マーカー |
| `rviz_planned_path.png` | 計画経路 (緑線) |
| `gazebo_world.png` | Gazebo の検証用 world |
| `robot_photo.jpg` | 実機写真 |
