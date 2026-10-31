# 渲染、时间采样与排版技术

> 公开版阅读约定：`<…>` 是待配置的路径占位符；文中原项目文件名属于所指参考仓库或本地案例，不表示随本 skill 附送。原研究的抽样与运行记录仅说明当时的验证范围，不代表你的环境已经通过。跨机器使用先读 [环境接入](environment-setup.md)。

来源和版本见 [source-study.md](source-study.md)。以下是可迁移方法，不表示旧两部 PV 已实现所有能力。

## 纯时间状态与时间层

画面状态由绝对时间、固定资源、种子/对象 ID 决定；共享函数供预览、抽帧和导出。分别管理歌曲时钟、发布偏移、镜头进度、音乐 beat phase、歌词事件与相机路径。短动作完成后保持终态，长载体运动可继续。

粒子从 birthTime/id 推导位置、速度和消亡，不按每次 render 调用出生。发射速率变化应取出生时刻的 rate(tb)，按固定最大时钟与 hash 筛选；取当前 rate(t) 重算所有出生时间会让旧粒子跳动（pdoom _motifs.ts）。

有状态反馈/物理模拟先固定步长预烘焙。pdoom 的 stateful reset/preroll + 固定 samples 是其专用路径，不证明任意 seek 与多 worker 完全一致；它显式拒绝 stateful 的自适应采样。Multiplane 的 render(t) 不接受这种调用历史依赖。

## 真正的时间采样

围绕主帧 t 在 shutter/FPS 内取子帧并累加，得到运动曝光；低透明度复制、后期方向模糊和空间超采样与它不同。路径先正确再加模糊。

pdoom Engine 的嵌套样本为 4×3^k（4/12/36/108/324），比较增量平均与已有平均的显示误差，按 32×32 逻辑像素 tile 收敛，再用深度掩码避免重复着色已收敛区域。真实 depth-tested 3D 会退回未掩码绘制；跨屏扭曲/宽模糊的中间缓冲不能直接拿屏幕 tile 掩码跳过。学其收敛与分级预算思路，不无条件照抄 GPU 状态劫持。

硬切要独立定义，采样窗口不跨切点；pdoom 当前通用采样只钳制负时间，不提供 Multiplane 的完整 cuts 契约，移植时补上。每帧随机故障保持同一主帧种子，连续几何用子帧 t；参考 frameIdx 的 60fps 假设应改成项目 FPS，最好显式传入主帧编号。颗粒/dither 和稳定 HUD 每帧合成一次，避免子帧平均把质感抹平或增加无效采样。

## GPU 几何和颜色

Machine Love：NumPy 状态 → 动态 VBO/VAO → 实例化点线 → 浮点舞台 → RGB readback → Pillow 字幕。它的 additive/no-depth 设计适合发光结构，不用于不透明实体遮挡。

pdoom LineBatch 把线段展开为 capsule quad，以 SDF 抗锯齿并稳定细线覆盖，使用屏幕/世界宽度两种口径；max blend 避免连接处叠亮，add blend 留给发光信号。参考源码对相机后端点有简化剔除，不复制为完善近面裁剪；跨近面的线要在透视除法前裁切。

HDR 线性色合成、sRGB 文字贴图转换、tone shoulder、bloom/halation 与最后输出转换各自明确。字体主笔画保持清晰，辉光服务信号核心；低分辨率 bloom pyramid 可以保持半径/风格，不代表主体可以低分辨率放大。透明素材另核对 straight/premultiplied alpha，避免黑边/双重乘 alpha。

## 原生高分辨率

逻辑布局坐标稳定，物理 FBO、Canvas backing store、字形 atlas、线条边缘和输出缓冲按目标尺寸生成。WebGL 的 gl_FragCoord/fwidth 是物理单位；算法混用逻辑尺寸时显式转换。4K 不能只是 canvas CSS 拉大、低分辨率文字贴图放大或旧 MP4 上采样。

原生证明需要代码路径、资源/源码签名和编码器输入尺寸；视频元数据只证明编码尺寸。检查 1:1 字形、细线和快速运动，4K 缩略图不够。world highres_canvas 的逻辑代理可复用，但新的类型/绘制操作需覆盖 crop/paste/alpha/字体度量路径，避免重复缩放。

## 字形与歌词

设计先读 [歌词分镜与叙事](lyric-dramaturgy.md)；本节解决实现精度。主唱词、强调词、对话、系统提示、图注和背景字符分别建模，不能将词形纹理或硬编码代码计入完整歌词覆盖。

先做整行布局，得到每字的 x、advance、kerning、基线；高亮/裁切前缀复用这个布局。独立 measure(prefix) 会丢失跨切口 kerning，导致已唱/未唱部分错位。CJK fallback 需明确覆盖字形，准备时等待实际字体；测量和绘制用同一字体/字号。

单线字沿路径累计长度逐段 reveal，笔头位置来自同一长度；沿曲线的文字以弧长和切向布局，避免每字任意旋转。字内 wipe 跟随词 start/end，帧槽容量与完整持留同时满足。图内文字可被合理遮挡但不能在关键唱词持续不可读；稳定字幕保持独立清晰层。标签真实 alpha 合成，不用 RGB 乘小数冒充透明。

时间策略分别记录：pdoom 的 `Lyrics.wordProgress` 可按多音节分段插值，音节间隙保持已完成比例；Nyanko 的行打字用 `min(len / charsPerSec, lineDuration × maxFraction)` 提前完成，重点版画另读 cue。后者不是全片逐词声学对齐。字形在世界中依附也要分级：屏幕固定、世界锚点投影、逐字仿射、曲线切线或真3D字板；关闭深度测试的字板没有自动实体遮挡。代码边界见两份歌词案例。

## 高吞吐导出

Canvas/WebGL 原始像素经 WebSocket 发往 FFmpeg，明确 RGB/RGBA、行方向、像素对齐和帧编号。背压确认后才复用缓冲；并行 worker 完成顺序不同，按编号重排后送编码器。pdoom 使用 RAM 上限 + 磁盘 spool；这不是断点恢复机制，真正 resume 另需输入签名和完整块记录。

关闭 HMR，冻结源码/字体/资源；检查控制台和场景错误占位，不能只等编码器 exit 0。硬件编码先实跑短样片，列出 NVENC 不表示驱动/API可用。再按颜色矩阵、范围与音轨统一拼接，避免不必要的 RGB/YUV 往返。

高级采样、异步像素读取和多 worker 属于按瓶颈采用的工具，不要求每首新歌全部实现。
