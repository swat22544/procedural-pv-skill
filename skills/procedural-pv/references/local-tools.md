# PV 工具路线与历史接入记录

> 公开版阅读约定：`<…>` 是待配置的路径占位符；文中原项目文件名属于所指参考仓库或本地案例，不表示随本 skill 附送。原研究的抽样与运行记录仅说明当时的验证范围，不代表你的环境已经通过。跨机器使用先读 [环境接入](environment-setup.md)。

## 新环境先按任务选择

以下是按镜头选择的路线，不是必须全部安装的软件清单。程序入口由当前项目记录；公开版不继承作者机器的路径、连接状态和授权。

| 工具 | 适合的工作 | 前提与边界 |
|---|---|---|
| [FFmpeg / FFprobe](https://ffmpeg.org/) | 拼接、编码、混音、换音轨、媒体检查 | 核对所需 encoder、滤镜和实际解码；NVENC 要做短编码试验 |
| [Multiplane](https://github.com/LBEILC/Multiplane) | 网页、动态文字、2.5D、适配后的 Three.js 确定性导出 | CLI 路线需要 Node.js 24+、匹配的 Playwright Chromium 和独立 FFmpeg；桌面发行包可自带 Node，以所装版本文档为准 |
| [NumPy](https://numpy.org/) + [Pillow](https://pillow.readthedocs.io/) | 数学状态、几何、字形、离线像素与合成 | Python 与字体；CPU 渲染也是有效选择 |
| [ModernGL](https://moderngl.readthedocs.io/) | 批量点线/三角形、浮点舞台、GPU readback | 兼容 OpenGL 上下文；案例使用 3.3，自有 shader/平台另验 |
| [Three.js](https://threejs.org/) / WebGL | 网页三维、实例化几何、着色器和后期 | 固定同版本 core/addons；Three.js 本身不是成片导出器 |
| [Blender](https://www.blender.org/) + bpy | 实体遮挡、材质、灯光、复杂相机、程序建模 | EEVEE/Cycles 渲染，先核对设备、颜色管理和模拟缓存 |
| [MCP for Blender](https://github.com/ahujasid/mcp-for-blender) | agent 查询、编辑、查看实时 Blender 工程 | 可选控制接口，需匹配 server/addon 与运行中的 Blender；不是独立渲染引擎 |

数学公式按需加 Manim/MathTypst，声学分析按需加 stable-ts、faster-whisper、MMS_FA、Demucs；已有 AE 工程用 JSX/aerender。VOCALOID、图像生成、Cinema 4D、Premiere 都不是默认必需；本技能不依赖 DeepSeek/GLM helper。

先检查单帧和短窗，再冻结通过的版本与参数。Multiplane 的详细操作归其官方文档与配套技能管理；不能把下列 Windows 历史兼容设置套到所有系统。已有专用导出链保留，工具缺项按 [环境接入](environment-setup.md) 处理。

## 以下为作者机器的历史记录

下文保留研究依据和能力边界，**不是当前读者环境的检测结果或最低版本要求**。若需要重做这些探针，应在自己的项目记录实际结果。

原入口索引：`<LOCAL_TOOL_INVENTORY>`。若当前用户有自己的索引，先按相关 id 读取，再检查路径、依赖与当前会话工具；没有时直接发现可用程序，不为此创建虚构索引。不要自动升级已固定工具。

2026-10-31 的环境复查记录在 `<PRIVATE_STUDY_ARCHIVE>/pv_skill_refresh_20261031/evidence/`。旧索引大部分时间戳为 2026-10-18，Blender MCP 的安装验收另在 2026-10-30；不同日期的结果不混成一次测试。

## 主路线

| 工具 | 本机入口 | 已知状态与定位 |
|---|---|---|
| Multiplane 0.3.11 | `<MULTIPLANE_BIN>` | 本次 doctor 通过；Node 24.22.1、Chromium 153 headless shell、RTX 4060 D3D11 硬件光栅、FFmpeg 可用 |
| ModernGL 5.13.0 | `<PV_PYTHON_BIN>` | 本次实际创建 OpenGL 3.3 context，RTX 4060；NumPy 2.5.3、Pillow 12.3.0、Manim 0.21.0 导入通过 |
| Three.js r186 | `<THREE_MODULE_ROOT>` | 本次入口/后期模块路径确认；旧运行证据在 three_runtime_check/runtime_report.json |
| Multiplane 内置三维依赖 | `tools/multiplane/node_modules/three` | 独立固定 0.180.0；与 r186 不能混用 imports/addons |
| Blender 5.2.2 LTS | `<BLENDER_BIN>` | 本次 --version 通过；MCP 已加载但未连接运行中的 Blender |
| FFmpeg / FFprobe 9.0.2 | `<FFMPEG_DIR>/` | 编码、拼接、音轨、检查；NVENC 需目标配置短编码探针 |
| Manim / MathTypst | `<PV_PYTHON_BIN>` | 数学推导/公式独立镜头；旧 Cairo/Text/MathTypst 样片不证明整片性能 |
| 歌词对齐 | `<ALIGN_PYTHON_BIN>` | stable-ts/faster-whisper、MMS_FA、HDEMUCS 等按本地脚本与缓存；本次路径确认，未重跑模型 |

本次另做中性几何探针：Multiplane 单 worker、320×180、12 fps、0.5 秒/6 帧，lint、stills、乱序/重载 verify、render/output check、独立 FFprobe 计帧和 FFmpeg 完整解码均通过；证据为 `evidence/multiplane-*.jsonl`、`engine-smoke-ffprobe.json` 与 `engine-smoke.mp4`。这不是 pdoom 风格样片，也不证明多 worker、长片或 4K 工程已重渲染通过。

GPU 实际上下文不等于完整项目渲染通过。RTX 4060 Laptop 显存约 8GB 的历史记录应在重负载前重新查询；并发浏览器、Blender、音频模型按实测内存预算。

## 原生与可选路线

- AE：索引 id `ae/aerender` 定位 AfterFX.exe / aerender.exe；当前会话可发现 after_effects MCP，优先使用实际 schema。历史 2026-10-18 已有 720p30、6秒原生 AEP/渲染验证，本次仅路径核对，未重跑。
- Cinema 4D 2026：`<CINEMA4D_BIN>`；入口存在，授权/自动化/渲染没有本次验收，按用户指定工程再核查。
- Premiere 的旧盘点停在空工程保存，导出未通过；不要把“安装”写成“已能自动剪辑导出”。
- 图像素材、VOCALOID/BVWA 等按任务和各自技能调用，不是每部代码 PV 的必需依赖。

这些工具可用范围不代表它们参与了 world 或 Machine Love 的正式制作。不要为更新技能自动安装新模型、替换全机版本或启动付费服务。

## Multiplane 特有本机事项

完整规范归 `<INSTALLED_SKILLS>/multiplane/SKILL.md` 与 `references/windows-local.md` 管理，本技能不复制上游手册。

- 保留工程 cwd 或传 --root；资源限制在根内，本地字体与图片在 setup/ready 完成加载。
- 当前匹配 headless shell 通过 MULTIPLANE_WINDOWS_HEADLESS_SHELL=1 绕过完整 Chromium 的 SxS 问题；仍是 NVIDIA 硬件渲染，不叫软件光栅。
- 启动器支持 MULTIPLANE_NODE；捆绑 runtime 路径失效时用 load_workspace_dependencies 重新定位，不猜新缓存路径。
- 默认先 1 worker；高分辨率/长任务先实测，再提高并发；中断按原 job 条件 resume。
- 学习 pdoom 的相机/材质不需要换掉 Multiplane；学习其专用采样器也不代表必须嵌套两套 renderer。

## Blender MCP

工具发现、连接、文件保存、原生渲染见 [blender-workflow.md](blender-workflow.md)。本次没有打开 Blender 来改变当前场景；连接失败如实记录，后续真正制作时再启动并检查。
