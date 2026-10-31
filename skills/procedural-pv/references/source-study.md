# 六份工程与参考源码的研究地图

> 公开版阅读约定：`<…>` 是待配置的路径占位符；文中原项目文件名属于所指参考仓库或本地案例，不表示随本 skill 附送。原研究的抽样与运行记录仅说明当时的验证范围，不代表你的环境已经通过。跨机器使用先读 [环境接入](environment-setup.md)。

2026-10-31 重新阅读两份本地工程、两份新参考与现存证据；2026-11-01 补全此前已经学习的 FrostNova、Misaka 来源入口，并对比更新前后资料。合计两份本地工程与四个外部参考；没有重渲染这些完整成片。下表区分已记录提交、固定克隆与没有 Git 元数据的本地目录。

| 工程 | 基线 | 可复用的重点 |
|---|---|---|
| 本地 world.execute(me) | `<LOCAL_WORLD_PV>`，V14 native 4K | 语义状态、固定身份、连续几何、原生画布、分段编码 |
| 本地 Machine Love | `<LOCAL_MACHINE_LOVE_PV>`，V2 | NumPy/ModernGL/Pillow 分层、多模型声学对齐、复唱与成熟形态解耦 |
| [FrostNovaOrg/world-execute-web](https://github.com/FrostNovaOrg/world-execute-web) | 既有学习快照 `29aefca50e40c14498420e1c6e1f3a1037727e17`；`<REFERENCE_ROOT>/world-execute-web` | 14 项语义/计算机制、连续世界、相机 rig、音频特征、像素排序与后期分层 |
| [MisakaZentai/world-execute-me-dsh-pv](https://github.com/MisakaZentai/world-execute-me-dsh-pv) | `a598092a1b1db91fa43961a81269353d13419f6f`；独立检出与 `<REFERENCE_ROOT>/world-execute-me-dsh-pv` 的1587份受跟踪文件一致 | 元素因果接力、载体/状态/时钟接续、双画面注意力、逐词时序来源、HTML 截图 + PIL 合成 |
| [Nyankomintsu/world-execute-me-lyric-mv](https://github.com/Nyankomintsu/world-execute-me-lyric-mv) | `93ac63775ee58b33ff102167514f2603a48940cc`，package 4.0.0 | 聊天/版画/代码三种画面语言、调色状态、数据化镜头、WebGL 后期与背压输出 |
| [mexicat/pdoom-video](https://github.com/mexicat/pdoom-video) | `fa164b74fd0b26a99375715b619c55b4a0fb3d05` | 机位与空间转场、字形融景、线描图版、GPU 线批次、自适应时间采样 |

两个新仓库快照根：`<REFERENCE_ROOT>`。本地审计和环境证据：同任务目录的 `work/local-project-audit.md`、`evidence/`。公开源码只是参考，不把作者性能数字/成片自述写成本机验证。

## 既有两份参考：保留并继续使用

FrostNova 的转场专项细化见 [frostnova-transitions.md](frostnova-transitions.md)：2026-11-01 核对远端 HEAD 与既有提交相同，读取有效剪辑表、引擎交接和章节两侧实现。新增原源码函数检查与版本分支说明，原有 14 项视觉机制及 Misaka 来源不替换。

[既有参考的保留与使用](legacy-sources.md) 给出 FrostNova / Misaka 的源码入口、方法与本地适配关系。FrostNova 的 14 项完整笔记继续维护在 [visual-techniques.md](visual-techniques.md)，两库来源和本地逐词时序路径保留在 [project-defaults.md](project-defaults.md)。2026-11-01 与更新前备份比较：四份旧参考正文均无删除；本次修正的是总索引遗漏和入口弱化，并补充 Misaka 的独立方法索引。保留核查证据在 `<PRIVATE_STUDY_ARCHIVE>/pv_skill_refresh_20261031/preservation_20261101/evidence/`。

## Misaka：元素动作、状态与因果叙事

2026-11-01 固定上述提交，沿最终出片安装链得到97个正时长镜头、74个Cut与2个FINISH。详细方法见 [元素连续叙事](misaka-motion-method.md)，十二组链条见 [画面/源码拆解](misaka-motion-cases.md)。源码、隔离运行、官方替身角色和抽样画面证据在 `<PRIVATE_STUDY_ARCHIVE>/pv_misaka_continuity_20261101/evidence/`；保留原逐词来源与旧制作记录。

重点入口：V2的 `cuts.body / SHOT_HOOKS / DELAY` 负责临时接管与持久结果；`kit.reveal / radial / inward / sweep` 编排背景换手；各 `s_*.py / scenes_*.py` 给出共享几何与载体落点；DSH的 `batch_*.py / seg_page.py / dsh_patch_mem.py / dsh_patch_f.py / dsh_patch_g.py` 负责跨批次终态、结构剥离、记忆擦除、错误保留与结尾光标。阅读时通过 `dsh_her.install / finish` 确认覆盖，不把原舞者函数直接当最终窗口效果。

迁移不要求沿用monkey patch架构，也不把编排的模型数据称为实际训练。稳定索引、共享位置函数、区域匹配与语义呼应分别表述；原始24fps、上采样4K和插帧60fps的边界仍按既有参考维护。

## Nyankomintsu：把设计约定变为镜头数据

2026-11-01 歌词专项继续固定同一提交。新增 [歌词叙事方法](lyric-dramaturgy.md) 与 [Nyanko 逐案拆解](nyanko-lyric-cases.md)，把字体身份、歌词消息、几何排版、关键词版画、插镜返回、删除、循环执行与复唱状态串起来。隔离副本补入用户本地歌词，129行无缺失/词序差异；标题和缩写标点等适配、原始来源与实际字体版本记录在 `<PRIVATE_STUDY_ARCHIVE>/pv_lyric_dramaturgy_20261101/evidence/`。抽样渲染属于静音源码研究，不是完整成片或逐句听校。

- `src/scenes/shot.js:sequence`：镜头有 at、lines、moment、layout、palette、camera、enter；计算 cut 两侧的 overlap，限制最多两内容层重叠，防止短镜头被前一转场吞掉。可迁移为自动分镜表/覆盖检查的统一输入。
- `src/engine/engine.js:createEngine / renderFrame`：同一入口预览/导出；等待字体/角色资源；内容 A/B 与 overlay 分层，场景时钟与发布前滚分离。过渡中独立计算两场 fx 再混合，避免新场后期提前影响旧场。
- `docs/v4/PLAN.md`、`src/scenes/kit.js`：界面动作推进叙事，版画提供内心/关键词表现，代码段有独立任务。调色表示上线、热度、故障、离线等状态；可学“颜色随事件改变”，不复制其角色颜色到所有项目。
- `src/engine/art.js`、`scenes/climax.js`：按墨版/剪影做遮罩、局部错版与内容填充；场景通过有限亮度阶梯推进高潮。把故障绑定可见主体和语义事件，避免全屏随机噪声。
- `tools/storyboard.mjs`、`base_sheets.mjs`、`regress_sheets.mjs`、`compare_sheets.py`：镜头表和关键点缩图由数据生成；共享模块修改前后查未计划改变的片段。
- `export/export.mjs`、`wsframes.mjs`：raw RGBA 经 WebSocket 输给 FFmpeg，等待写入确认形成背压；目标分辨率实跑 NVENC 探针后再回退。前滚以音频样本延时，文件记录 mv_start 供验证解释时间。
- `tools/check_sync.py` 将 mux 偏移、画面 cue、歌词与声学事件分开检查；`check_flash.py` 为低分辨率亮度/红闪/单帧突跳近似筛查，不能称为完整安全认证。
- `fill_lyrics.mjs` 将用户词填入无歌词 timing 表，并校验缺行；该库沿用自己的歌词时序，不适合不经重对齐直接换歌。

边界：没有原歌曲/原词时，仓库默认导出前置检查会停止；2026-10-31初次研究未补入素材或完整执行，2026-11-01歌词专项仅在隔离副本补词并调用原渲染API抽样。角色剪影在 CREDITS 中为 CC BY-NC-SA 4.0，代码/自有文档 MIT；不把代码许可扩展到角色、字体和歌曲。

## pdoom：以当前源码解释设计

2026-11-01 歌词专项另见 [pdoom 空间与材料文字案例](pdoom-lyric-cases.md)。重点是歌词成为曲线、提示词、室内标牌、纸面字段、燃烧路径和堆叠层，以及屏幕唱词与世界文字的分工；没有将所有文字都归为3D。实际浏览器抽帧与连续窗口、字形布局/音节进度检查记录在 `<PRIVATE_STUDY_ARCHIVE>/pv_lyric_dramaturgy_20261101/evidence/`。

2026-11-01 深化研究保持同一提交，并核对本地检出与远端 HEAD。新增 [镜头制作细则](pdoom-camera-method.md) 和 [逐场镜头拆解](pdoom-shot-cases.md)，覆盖 loss、room、shoggoth、leftturn、spacetime、stack、ascent、loom/ilya 及 hook/outro。区分导演意图、实际投影模型、有限函数检查与未做的整片视觉验收；证据在 `<PRIVATE_STUDY_ARCHIVE>/pv_pdoom_cinematography_20261101/evidence/`。

优先读 [pdoom-design.md](pdoom-design.md)，那里把相机、机位和跨场空间匹配分开说明。源码入口：

| 主题 | 文件/符号 |
|---|---|
| 设计体系 | `docs/TREATMENT.md`、`ENGINE.md`，与实际 scene 对照 |
| 歌词锚点、拍点切换 | `app/src/timeline.ts:makeTimeline`、engine/audio.ts、lyrics.ts |
| 平面进入地形、多机位 | scenes/loss.ts、room.ts |
| 视角与运动的反差 | scenes/spacetime.ts、stack.ts |
| 定制出口/共享投影 | scenes/hook.ts 的 drawNumberPhase、drawFinalPD、drawLids |
| 递归到下一世界 | scenes/loom.ts、ilya-room.ts、ilya.ts |
| 首尾闭环 | scenes/outro.ts、open.ts、app/plates.json |
| 精密线条和图版纹理 | engine/lines.ts、glsl/common.ts、scenes/shoggoth-glsl.ts |
| 字形/单线字体 | engine/type.ts、stroke.ts |
| HDR、采样、物理分辨率 | engine/engine.ts、post.ts、gl.ts、scale.ts |
| 流式输出、并发、spool | scripts/render.ts |
| 多模型对齐 | analysis/align.py、ctcalign.py、ctc_emissions.py |

这份快照使用 Bun、Vite、TypeScript、Three.js r186 和 playwright-core；原脚本指定 Chrome + ANGLE Metal，声学辅助包含 MLX/平台特定流程。Windows 迁移要改浏览器后端、字体 fallback 和分析工具，再做短片验证；2026-10-31初次研究未安装/验收这套完整依赖。2026-11-01歌词专项在隔离副本使用Node/Vite与Chromium/D3D11复现选定窗口，实际解析Three0.186.1、Vite8.3.5；没有验收原Bun整片导出或MLX声学工具链。

技术学习见 [rendering-techniques.md](rendering-techniques.md)。其每帧自适应采样、分 tile 早退不等于 Multiplane 已有同一实现；不可合并描述。作者的 stateful 固定采样支持也不是通用乱序可重现保证。

## 本地两项目的事实边界

world 的 720p V14 局部复用旧视频，与 4K 全帧原生重算不同；新任务不能混写。其音频缓存有哈希检查，4K chunk signature 却仅覆盖根目录 Python，扩展全资源签名是未来改进。

Machine Love 用 Manim 的 Python 环境但由 ModernGL 渲染；本次当前 7 个 src 哈希匹配 V2 encoded manifest。它的 runtime NPZ 未主动检查 metadata，字幕调度仍固定 24 fps。详见 [machine-love.md](machine-love.md)。

学习与修改保持区别：本次更新技能，没有把 pdoom shader、运动模糊或 Blender 镜头追写成旧成片已使用的技术。
