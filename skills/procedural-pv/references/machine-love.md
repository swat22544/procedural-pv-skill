# Machine Love：工程与可复用方法

> 公开版阅读约定：`<…>` 是待配置的路径占位符；文中原项目文件名属于所指参考仓库或本地案例，不表示随本 skill 附送。原研究的抽样与运行记录仅说明当时的验证范围，不代表你的环境已经通过。跨机器使用先读 [环境接入](environment-setup.md)。

2026-10-31 复读 `<LOCAL_MACHINE_LOVE_PV>`。有效交付为 V2，顶层旧说明仍写 v1；以 V2 encoded manifest 与 delivery report 为准。

## 有效入口与证据

- `src/render_pv.py`：状态、GPU 舞台、Pillow 字幕、rawvideo → FFmpeg。
- `src/gpu_stage.py`：持久 OpenGL 3.3 context、动态缓冲、实例化点线/三角形、RGBA16F 舞台和 RGB8 readback。
- `src/scenes.py`、`semantic_details.py`、`semantic_world.py`、`model_boot_v2.py`、`semantic_expansion_v2.py`：语义几何、toy 数值、片头与后段。
- `analysis/alignment_finalize.py`、`audio_extract_features.py`：冻结演唱事件、100 Hz 特征。
- `analysis/verify_export.py`、`verify_previews.py`、`v2_motion_audit.py`、`deliver_v2.py`：检查/派生交付，运行前核对目标。
- `out_v2/Machine_Love_PV_v2_4K.manifest.json`、`delivery_report.json`、`verification/summary.json`：现有媒体记录。

既有主片 3840×2160、24 fps、6877 帧、286.541667 秒；报告记录 22 项通过。本次只读复查当前 7 个 src 文件哈希全部匹配 manifest，未重跑整片或逐句听校。时序仍为自动多模型对齐，日文短语内部有字符插值。

## 值得迁移的架构

state_for(t,...) 提供 points、lines、triangles、颜色/尺寸、camera、viewport、labels；Stage 不推进模拟、不生成新 ID、不改歌词。固定核心对象在相邻形态间延续，附属点线随所属结构淡出。语义进度完成后保持，载体运动继续；复唱更新文字事件而不重置成熟形态。

开场分 checkpoint、decoder、KV 与 session ready，是语义编舞而非真实加载测量。低秩更新/矩阵乘法使用 seeded toy 数值时，动作跟随实际计算且说明规模。

## 渲染边界与迁移陷阱

- 使用 `tools/manim/.venv/Scripts/python.exe` 中的 ModernGL，不是 Manim Scene。
- 线段先在 view-space 近面裁剪再透视除法；点宽有界，文字和线宽原生缩放。
- additive blend、关闭 depth test，适合发光点线；实体遮挡/PBR 要改设计或选 Three.js/Blender。
- 局部 halo 与 finish shoulder 不等于完整 bloom、AgX/ACES 或时域抗锯齿。
- char_schedule 硬编码 24 fps；迁移其他 FPS 要贯穿字幕、光标、导出和验证。
- runtime 直接读取 NPZ，不自动核对 metadata；换轨显式检查哈希/重建。
- 音轨、字体、FFmpeg 仍有绝对路径依赖，入口快照不是可搬迁包。

双向帧槽调度保留完整字串；极短句按容量明确允许多字。重叠呼应保留主句，光标和排版共用字体 advance。舞台标签与几何共用投影，独立 RGBA 合成，防止 RGB 画布忽略 alpha 使旧字覆盖新字。

## 对齐

HDEMUCS 分离只用于分析；stable-ts/faster-whisper 候选与 MMS_FA CTC 交叉核对，最终仍用原 WAV。保留模型分歧、口白、重叠、低置信和插值标记；event_id 与 source_line_index 分开。沿用方法，不继承本歌的 66 行、72 事件和固定时间窗。
