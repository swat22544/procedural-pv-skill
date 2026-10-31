# 工具准备与渲染路线

本 skill 不捆绑渲染器。只讨论故事、分镜或审片时无需先装齐软件；需要实际输出媒体时，再按下表选择。默认网页路线和两个历史 Python 案例是不同入口，不要混用它们的发布脚本。

## 先选一条主路线

| 要做的镜头 | 最小组合 | 可选补充 |
|---|---|---|
| 动态文字、图形、卡片、网页、轻量 2.5D | Multiplane + Chromium + FFmpeg/FFprobe + 本地字体/素材 | Three.js、SVG/Canvas、自定义 shader |
| 数学几何、发光点线、离线像素 | Python + NumPy + Pillow + FFmpeg/FFprobe | ModernGL 与兼容 OpenGL 上下文 |
| 实体空间、遮挡、材质、照明、复杂相机 | Blender + bpy + EEVEE/Cycles + FFmpeg/FFprobe | MCP for Blender、Geometry Nodes、模拟缓存 |
| 数学公式或推导 | Manim + 相应公式依赖 + FFmpeg/FFprobe | MathTypst/Typst，或需要时配置 LaTeX |
| 已有 AE 项目 | 有效的 After Effects 环境 + JSX/aerender | 对应 MCP 控制接口 |
| 只换音乐、去人声轨、剪接或转码 | FFmpeg/FFprobe + 正确输入轨道 | 只在确有需要时使用源工程 |

“去人声”需区分是否已有独立器乐轨。如果只有混合录音，不能把禁用音轨或简单调音量描述为成功分离人声；有原始分轨时直接使用正确轨道更容易保留质量。

## 1. 网页路线：Multiplane

上游：[LBEILC/Multiplane](https://github.com/LBEILC/Multiplane)。命令行方式需要 **Node.js 24+**、与安装包匹配的 **Playwright Chromium** 和独立 **FFmpeg**。官方桌面发行包可包含 Node 运行时，浏览器与 FFmpeg 的准备按对应版本文档进行。

在自己的新工程里，按上游安装说明执行：

```sh
npm install multiplane
npx playwright install chromium
npx multiplane doctor
```

安装后保留 lockfile，记录实际 Multiplane、Chromium、Node 与 FFmpeg 版本，再制作短探针。原本地研究记录的 **0.3.11 是那份本地安装包的版本**；公开发布时查询 npm 的该版本得到不存在，不能据此给新用户写死 `multiplane@0.3.11`。应使用公开渠道实际提供的版本，按其文档重新验证，不把本机测试自动转移到另一版本。

需要配套技能时，按上游提供的入口安装并核对宿主：

```sh
npx multiplane skill install --host all
```

当前工程已经有 `shot.html`、对应时间契约和资源后，基本工作顺序是：

```sh
npx multiplane lint shot.html --json
npx multiplane stills shot.html --at '0,1' --json
npx multiplane verify shot.html --workers 1 --json
npx multiplane render shot.html --workers 1 --out out/preview.mp4 --json
```

时间点必须处于镜头范围；这些命令是调用形态，仓库本身不附带 `shot.html`。首次运行以所装版本的 `--help` 和协议为准。PowerShell 的逗号时间列表需要整体加引号。

资源在初始化/ready 阶段加载完成，`render(t)` 只根据绝对时间作画；固定随机种子，显式声明硬切，避免运动模糊跨越切点。先测试 1 worker，按最复杂窗口的显存、内存和速度增加并发。断点续渲必须确认输入、工具版本与参数仍满足原作业条件。

固定的 `render(t)` 接口便于抽帧和复现，但不同 GPU/浏览器/字体仍可能造成像素差异。`lint`、抽样 `verify` 和完整视觉检查是不同层次的证据。

## 2. FFmpeg 与 FFprobe

取得渠道：[FFmpeg 官方下载页](https://ffmpeg.org/download.html)。选择支持目标编码器的构建；H.264 路线通常需要 libx264。先核对入口：

```sh
ffmpeg -version
ffprobe -version
```

Multiplane 可以通过 `--ffmpeg`、`MULTIPLANE_FFMPEG` 或其配置识别程序，具体优先级以所装版本说明为准。不需要为了单个项目修改全机 PATH。

成片后至少确认实际视频/音频流、尺寸、FPS、帧数、时长、时间偏移与完整解码，再查看编码后的静帧和连续片段。媒体检查命令示例：

```sh
ffprobe -v error -count_frames -show_streams -show_format -of json output.mp4
ffmpeg -v error -i output.mp4 -f null -
```

这些只检查媒体结构和解码，不证明字已读清、故事已成立或歌曲逐句对齐。

## 3. Python / ModernGL / Three.js

- [NumPy](https://numpy.org/) 处理数值状态；[Pillow](https://pillow.readthedocs.io/) 处理字形、图像与合成。使用项目独立环境并固定依赖。
- [ModernGL](https://moderngl.readthedocs.io/) 提供 OpenGL 绘制。Machine Love 的案例使用 OpenGL 3.3、浮点舞台与读回，但关闭深度测试的发光点线并不具备实体遮挡/PBR 的效果。
- [Three.js](https://threejs.org/) 处理网页三维场景。core、addons 与后期模块固定为兼容版本；Multiplane 自带依赖和其他本地 Three.js 不能混用 import。WebGL 与 WebGPU 的后期链另行设计。

先分开 `state_at(t)` 与绘制，不让导出器推进隐藏状态。逻辑坐标与物理像素分开；正式输出在目标尺寸生成几何与文字，不能把放大旧视频叫作原生重渲染。

## 4. Blender 与 MCP

Blender：[官方下载](https://www.blender.org/download/)；原生脚本使用 Blender 自带 Python 的 `bpy`。选择 EEVEE 或 Cycles 后核对设备和颜色管理，GPU 存在不代表 Cycles GPU 已配置。

本地研究使用的是 [ahujasid/mcp-for-blender](https://github.com/ahujasid/mcp-for-blender)，其 Python 包为 `mcp-for-blender`（历史安装记录 2.2.6）。以所装版本的 server/addon 和实际工具 schema 为准。它的职责是：

| 接口类别 | 用途 |
|---|---|
| 状态与场景查询 | 检查 addon、Blender 版本、当前工程、对象与资源开关 |
| 代码编辑 | 在实时 Blender 中通过 bpy/bmesh/mathutils 建模、调整相机、材质、动画与保存 |
| 图像观察 | 查看视口、相机、多角度、帧条和实际渲染结果 |
| 可选资源操作 | 按任务与许可搜索、导入或生成素材；不因安装接口而自动启用付费服务 |

**MCP 是控制接口，EEVEE/Cycles 才是渲染引擎。** 不装 MCP 也能通过后台脚本制作；使用 MCP 时先确认连接和现有场景，避免覆盖未保存工作。

正式长片更适合保存 `.blend` 与必要缓存，再后台输出 PNG/EXR 序列，由 FFmpeg 合成音乐并检查。模拟应烘焙或使用明确的固定步长；输出区间、子帧、运动模糊、随机种子和颜色设置保持一致。完整工作约定见 [Blender 与 MCP](../skills/procedural-pv/references/blender-workflow.md)。

## 5. 数学、声学与原生合成辅助

| 工具 | 作用 | 什么时候不用装 |
|---|---|---|
| [Manim](https://www.manim.community/) / [MathTypst](https://docs.manim.community/en/stable/reference/manim.mobject.text.typst_mobject.MathTypst.html) | 数学公式、推导与几何动画 | 普通动态文字与已有 Canvas/Python 场景不需要为此迁移 |
| [stable-ts](https://github.com/jianfch/stable-ts) / [faster-whisper](https://github.com/SYSTRAN/faster-whisper) | 自动时序候选、语音分析 | 已有可靠人工时序或只做无歌词样片时 |
| [Torchaudio MMS_FA](https://docs.pytorch.org/audio/stable/generated/torchaudio.pipelines.MMS_FA.html) | 文本约束的强制对齐候选 | 当前语言、版本或模型不适配时不强求；依赖与权重另核对 |
| [Demucs / HDEMUCS](https://github.com/facebookresearch/demucs) | 为分析提供人声分离轨 | 有原始分轨或无需分析时；分离轨不默认替代最终混音 |
| [After Effects](https://www.adobe.com/products/aftereffects.html) / JSX / aerender | 原生工程与后期合成 | 网页/Blender/Python 路线已经能满足需求时 |

对齐结果保留口白、重叠、低置信、插值与模型分歧；换音轨重建相关缓存。声学模型不是必装套餐，更不是自动逐字正确的保证。

## 版本与验证记录

本仓库主要来自本地 Windows 制作。历史文档中列出的 GPU、工具版本、测试通过与未连接状态有各自日期，不是当前机器的检测结果，也不是推荐最低配置。新环境按自己的任务重做代表性静帧、短片和媒体检查；能导入依赖不等于能完成整片。

把实际程序路径、版本、字体、音轨哈希、目标分辨率/FPS、颜色、输出目录和验证结果写进自己的项目记录。路径占位与私有记录的解释见 [环境接入](../skills/procedural-pv/references/environment-setup.md)。
