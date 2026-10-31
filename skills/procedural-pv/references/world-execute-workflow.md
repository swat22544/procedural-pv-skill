# world.execute(me) 案例：生产与排查

> 公开版阅读约定：`<…>` 是待配置的路径占位符；文中原项目文件名属于所指参考仓库或本地案例，不表示随本 skill 附送。原研究的抽样与运行记录仅说明当时的验证范围，不代表你的环境已经通过。跨机器使用先读 [环境接入](environment-setup.md)。

本页完整保留原生产参考，范围为本地 world.execute(me) 的历史交付与排查。通用新项目入口见 [production-workflow.md](production-workflow.md)，当前环境按 [local-tools.md](local-tools.md) 复查；本页“已验证”均属相应历史报告，不代表新一次测试。

## 当前可用工具

Three.js 本地库：`<THREE_MODULE_ROOT>`，`package.json` 版本0.186.1、REVISION186。2026-10-08实际浏览器测试已通过：WebGL2经ANGLE/D3D11使用RTX4060 Laptop GPU，自定义GLSL、931个粒子、256个实例化几何、EffectComposer/UnrealBloomPass/OutputPass、3840×2160绘制与PNG导出均成功；报告为 `<PRIVATE_STUDY_ARCHIVE>/three_runtime_check/runtime_report.json`，测试页与本地服务源码在同目录。此为功能验证，未测实时4K帧率，也未验证WebGPU或迁移现有MV。

后续Three.js场景可复用本地HTTP服务＋import map：`three`映射到该目录的`build/three.module.js`，`three/addons/`映射到`examples/jsm/`，保留相对依赖`three.core.js`和整个addons结构；已有构建文件不要求先npm install。测试服务入口为 `three_runtime_check/server.py`，仅监听127.0.0.1:8766；服务不是系统安装项，使用前确认进程仍在。4K canvas绘制尺寸与CSS显示尺寸分开，`renderer.setSize(width,height,false)`配合自适应CSS，避免大画布只显示左上角。WebGL的EffectComposer不能直接移给WebGPURenderer；更换后端需另外设计与验证后处理。

Blender 入口以当前环境的 `<BLENDER_BIN>` 为准。原案例在已授权的制作中按镜头需要选择 Blender；这项历史授权不传递给其他用户或工程。继续按当前任务选工具，不因发现 Blender 就自动重做已交付视频。技能的 `agents/openai.yaml` 启用 `allow_implicit_invocation: true` 只负责技能发现，不授予外部操作权限。

2026-10-08实际验证：Blender 5.2.2 LTS，后台Python创建12个对象，EEVEE成功输出640×360 PNG，已目测非空线条心形与轨道。源码、图片与报告为 `<PRIVATE_STUDY_ARCHIVE>/blender_runtime_check/probe.py`、`blender_probe.png`、`runtime_report.json`。此验证只覆盖脚本建场景及单帧EEVEE渲染，未测Cycles／OptiX、4K吞吐或整片迁移；报告中含启动／建场景开销的单帧耗时不可当作连续帧速度。

PowerShell调用形态：`& '<BLENDER_BIN>' --background --factory-startup --python-exit-code 1 --python '<场景脚本绝对路径>'`，由Blender自带Python运行 `bpy`，无需在Codex的普通Python环境另装 `bpy`。本机5.2的EEVEE引擎标识实测为 `BLENDER_EEVEE`，旧版 `BLENDER_EEVEE_NEXT` 在此版本会报enum错误；换版本先核对可用引擎。`--factory-startup`适合独立代码生成场景；编辑已有blend项目时应加载目标文件，不重置其内容。可按绝对帧时间生成原生分辨率图像，字幕与FFmpeg编码继续按项目流程合成。Blender资源并发与GPU可用性需单独测量，不套用下文Pillow工作进程数或FFmpeg NVENC限制。

Manim 本地独立环境：`<PV_PYTHON_ENV>`，启动器为 `<MANIM_BIN>`，Python为 `.venv/Scripts/python.exe`。2026-10-08按用户请求从PyPI安装 `manim[typst]==0.21.0`；依赖锁定清单为 `tools/manim/requirements.lock.txt`。启动器设置进程内UTF-8及既有FFmpeg路径，无需激活环境或修改系统PATH。数学／向量／矩阵等程序动画可按任务选用此环境；它基于Codex自带Python3.12.14创建，基础解释器移位时需重建venv，不向基础环境直接升级依赖。

已验证Cairo渲染器、几何动画、Consolas英文、Microsoft YaHei中文、`MathTypst`公式及H.264输出。样片为 `<PRIVATE_STUDY_ARCHIVE>/manim-verification/media/videos/probe/720p24/manim_ready.mp4`：1280×720、24fps、146帧、6.0833秒，完整解码与时间戳检查通过，实际编码抽帧已目测，`pip check`无依赖冲突；报告和源码在同目录上层 `verification`。随附Typst0.15.0用Python包直接编译SVG，公式使用 `MathTypst`，例如 `frac(e^(z_i), sum_j e^(z_j))`；`Tex`／`MathTex`需要另装LaTeX，当前未安装。ModernGL5.13.0作为依赖已安装，但此次没有测试Manim OpenGL或独立GPU渲染；不把720p安装样片当作4K速度验证。

调用示例：`& '<MANIM_BIN>' -r 1280,720 --fps 24 --media_dir '<输出目录>' '<脚本绝对路径>' <Scene类名>`。Manim0.21使用PyAV编码，已有FFmpeg仍可用于最终音乐／字幕合成与独立检验。官方依据：[本地安装](https://docs.manim.community/en/stable/installation/uv.html)、[MathTypst](https://docs.manim.community/en/stable/reference/manim.mobject.text.typst_mobject.MathTypst.html)。

本机 Python：`<PYTHON_BIN>`，已具备 NumPy、Pillow。

本机 FFmpeg：`<FFMPEG_BIN>`；ffprobe 在同目录。便携工具直接运行，不要求加入 PATH 或执行安装。用户早先给的 `<HISTORICAL_FFMPEG_CANDIDATE>` 是候选目录，实际生产使用上面的已验证路径。

新环境先核实工具与依赖；缺少现成工具时再向用户说明具体缺什么。不要因终端缺少 git 就停止源码学习；当前第二仓库曾通过 codeload ZIP 获取与解压，未运行它的脚本或下载／解密站点音乐。

## V14 原生4K派生流程

`render_release_4k.py`在独立worker中安装 `highres_canvas.install(3)`，重算所有5398帧，输出 `out_song_v14_4k/world_execute_computation_v14_4K.mp4`。不读取旧MP4画面，不做整体resize或帧插值。逻辑1280×720与native3840×2160分开；片头312帧、24fps、13秒音轨偏移与V14语义事件不变。下面的441帧局部重绘／4645帧复用仅属于720p V14快速发布，不适用于这个4K入口。

逐帧向rawvideo编码器写 `im.native.tobytes()`，并断言native尺寸；`np.asarray(Canvas)`只给旧故障特征检测返回逻辑分辨率，不能拿它送编码器或回写清晰主体。Draw按scale映射坐标、默认及显式线宽、半径、文字stroke／spacing，字形用原字体的 `font_variant(size*scale)`重光栅化。原字体对象与布局metrics保持逻辑单位，避免advance／baseline再次乘三。

Image／ImageChops代理让new、crop、paste、resize、copy、channel、point、blend、merge、composite、alpha_composite等在native图层上闭合，坐标只转换一次；不要只代理Draw而让中间图层回到720p，也不要先把源码坐标乘三再由代理重复缩放。`highres_finish`直接使用真实PIL图和物理坐标，必须排除于代理patch列表。sharp stage、实际像素排序、暗角和压缩保持native；只让低频glow seed使用原320×118缓存，低频背景可保留逻辑缓冲，不能将字体／几何的清晰层一起降采样。

分辨率相关背景与图像缓存清理后再调用；state、歌词schedule与模型数据不乘scale。检查片头共享helper／import别名、末帧与MV首帧、中文fallback及光标。查看1:1原生PNG和实际编码抽帧，不能用缩略图或MP4宽高证明native细节。

分段计划默认240帧，本曲23段（最后118帧），工程根目录全部Python源哈希组成signature。只在signature相同、chunk完成标记及字节数匹配时续用分段；参数或源码变化要重新核对已有chunk。各段libx264 medium／CRF14、H.264 High5.1、YUV420p，最终video stream-copy拼接，音乐单独编码AAC320kbps并延时13秒。显式x264 VUI `colorprim=bt709:transfer=bt709:colormatrix=bt709:fullrange=off`，同时核对实际ffprobe颜色字段。颜色格式转换与原生绘制来源分别验证。

本机16GB内存实际限制为2个workers、每个encoder线程3、BLAS／OMP线程1；原生RGB单帧约24.9MB，后处理同时持有多个native浮点数组，字体／背景缓存和encoder还占额外内存。并行数由实测工作集与系统余量确定，不按CPU核心数盲目增加。worker只传递小job元数据，持续编码当前帧，不缓存整片原生帧。当前NVENC虽在encoder列表中，短测报Required API13.1／Found13.0，实际不可调用；回退CPU，不因列表存在就选择GPU，也不自行修改驱动。新机器或驱动变化后重新短测。

制作命令：`render_release_4k.py --workers 2 --chunk-frames 240`。完整文件编码关闭后调用 `revision_4k/verify_4k.py <4K文件> --audio-source <原MP3> --report <报告>`；默认检查3840×2160、5398帧、24fps、224.916667秒、完整解码、实际解码帧数、颜色与原曲延时／双声道静音。缺省不传audio-source时只检查音轨编码／解码，不能据此称音乐对齐通过。独立验证入口不导入render模块；源哈希和render manifest提供原生来源证据，主任务仍应目测实际编码细节。

4K实际成片3840×2160／24fps、5398帧／224.916667秒，文件172225567字节，编码、颜色标记、声明与实际解码帧数、时长及完整音视频解码检查通过。原曲延时13秒，波形相关性0.996122686494，解码长度差0.011375秒；双声道静音片头主体区和最后50ms AAC边界分别检查。原生绘制来源由完整render manifest、源文件哈希与逐帧native缓冲入口佐证；3840×2160元数据本身不能证明没有上采样。正式报告在 `out_song_v14_4k`；歌词时序仍为自动稿，媒体检查不等于逐句人工听校。

## V14 720p已验证发布入口与局部重绘

使用 `render_release_v14.py render` 导出 `out_song_v14_continuity/world_execute_computation_v14_continuity.mp4`，`render_release_v14.py preview` 从实际成片剪出片头与三处修改段，`revision_v14/verify_release.py` 验证实际媒体。`prelude_v14.py`生成13秒／312帧，发布总5398帧／224.916667秒。警告输入2字／帧、退格4字／帧是用户明确要求更快的片头选择；署名与歌内字幕保持1字／帧。所有事件仍使用离散帧表，并分别检查正文字符和结构性行移除。

本轮用户点位是歌内时钟。场景函数保持原时间；发布字幕、字符帧和原曲统一后移13秒／312帧／13000ms。本地 `render_song.py` 输出改为 `out_song_v14_scene_source/world_execute_computation_v14_song_only.mp4` 防止覆盖历史版，该song-only输出尚未生成；直接运行歌内入口不会制作发布版。

发布器解码V11原MV，将歌内首6帧及 `[342,483)`、`[2044,2125)`、`[4431,4644)` 用当前代码重绘，其他4645帧使用V11编码图像。区间来自 `(14.25,20.125)`、`(85.132,88.540)`、`(184.601,193.46)` 的ceil量化，共441帧重新计算；全片输出统一重编码一次。新增代码在PATCH_WINDOWS之外不会自动出现在这种发布版，后续修改必须同步更新精确覆盖窗。不能把重新压缩后的保留帧称逐像素相同。

颜色传输采用V11实际使用的BT.709 limited／YUV420p：保留画面直接读取native YUV420p，不经过YUV→RGB→YUV往返；新片头与重绘RGB画面先转换一次到相同矩阵／范围／采样格式，存入 `redrawn_native_yuv.mkv` 的无损FFV1缓存，再按整数帧索引拼接并统一编码。FFV1无损指保存已转换的YUV帧，不代表RGB→YUV420p的颜色／色度采样转换逐像素无损。

首轮V14导出发现约1–2个RGB级的负偏差，诊断 `revision_v14/media/color_roundtrip_diagnostic.json` 记录默认swscale舍入与额外RGB往返；未发现源／输出BT.709、tv范围元数据不一致。首轮导出已停用，替换版按native-YUV方案重新导出；诊断数值不能作为替换成片的PASS结果。样帧核对在输出RGB转换处使用 `-sws_flags accurate_rnd+full_chroma_int`（位于 `-i` 之后），实际成片与对比基线使用同一明确解码路径；原MAE／SSIM阈值保持。替换MP4编码关闭后重新解码、抽样、看接点，再登记正式媒体结果。

分清局部源码验证与完整媒体状态。局部证据 `revision_v14/prelude`、`world`、`proof`、`love` 检查实际帧速率、保护区域、位置／速度、同一边界以及连续小片段；全片正式结果应取 `out_song_v14_continuity/media_verification.json`、`verification.json`、`showcase_verification.json`。修改前V13源码在 `source_archive/v13_before_pacing_v14`，V13媒体与报告保留。

V14实际成片5398帧／224.916667秒，1280×720／24fps，完整解码、帧时间戳、逐帧扫描、修改帧抽样与接点检查通过。4645个保留MV帧的80×45空间签名比较通过；这是重新编码后的近似画面检查，不能称逐像素相同。原曲延迟13秒，相关性0.999276727961，同采样率解码长度差0.011375秒。前12.95秒双声道静音主体区与最后50msAAC边界分别检查；具体峰值见报告。重点预览902帧／37.583333秒完整解码通过。源码／局部状态与实际编码媒体分别留证；主任务的最终目测记录见 `verification.json`。歌词时点仍为自动稿，本次抽样目测与帧扫描不等于逐帧人工观看或逐句听校。

## V13 历史发布片头入口

历史V13已完成并验证的发布成片为 `out_song_v13_terminal/world_execute_computation_v13_terminal.mp4`。使用 `render_release_v13.py render` 制作发布版，`render_release_v13.py preview` 导出前34秒；`revision_v13/verify_media.py` 检查完整媒体，`revision_v13/review/validation.json` 与 `mv_subtitle_validation.json` 记录片头和歌内字幕审查。本轮局部检查与实际完整媒体检查均通过，事实见 `out_song_v13_terminal/verification.json`。该历史入口制作时歌内入口仍输出V11原MV；当前歌内输出已经改为V14 scene source，不能据历史说明推定当前输出路径。V12的16秒版保留在 `out_song_v12_prelude`，已被新片头替代。

`prelude_v13.frame` 生成624帧／26秒静音片头，复用 `prelude_v12.py` 的端点和共用画面helper。警告正文151字符输入／删除，四行片名署名85字符输入；`WARNING_SCHEDULE`、`DELETE_EVENTS` 和署名末行删除事件从绝对帧索引求状态。完整清空警告后才启用署名。最后片头帧与 `release_head_frame(0)` 相同；前6个MV帧附在lead，V11从第6帧接入。头3帧保持UI常亮，之后恢复既有画面。发布总帧为624+5086=5710，237.916667秒；发布帧=歌内帧+624。音乐从原MP3延迟26000毫秒，外部字幕和字符事件加相同偏移，歌内场景时钟不改。

终端输入／退格的允许速率先按用户要求确定，再检查离散事件表。要求1字／帧时不能采用 `round(length * eased_fraction)`，它会在较快区间跳字；用户明确加快时可按指定每帧容量分配事件，仍需完整输入和删尽。像输入一样给每个删除动作分配明确帧，绘制、测宽与光标都从同一可见字符数读取。分别验证正文变化与行移除，不能因隐藏整行而声称正文已逐字删完。混合中英文使用显式CJK字体fallback与共同基线，测量和绘制共用同样的逐字advance，避免光标错位。

拼接后全片重新编码，画面对比使用精确帧索引与SSIM／误差，不能要求重新压缩后像素相同；不在非关键帧处直接stream-copy截尾。AAC边界可有极微弱预回声，静音检查分别记录主体区和最后50毫秒；整片完整关闭后再验解码。本轮624帧片头、全部5086个歌词状态、5710帧完整解码与帧时间戳、5080个保留MV帧的空间签名、26个高清同歌帧对比均通过；分别保留源码审查与编码媒体证据。

V13历史局部证据在 `revision_v13/review`，正式报告为 `out_song_v13_terminal/verification.json`、`media_verification.json`、`showcase_verification.json`、`revision_summary.json`。主任务已目测16张实际编码片头帧，图为 `encoded_prelude_review_1.jpg` 至 `encoded_prelude_review_4.jpg`。预览34秒／816帧、独立无音轨片头26秒／624帧已完整解码通过；修改前V12源码在 `source_archive/v12_before_warning_typing`。V12旧报告仍在 `out_song_v12_prelude`、`revision_v12`；它的5470帧与16秒音频偏移不适用于V13。43个歌内Python源文件与 `source_archive/v11_before_prelude` 保持同字节。

## 实际代码位置

| 模块 | 职责／时间窗 |
|---|---|
| `render_release_4k.py`、`highres_canvas.py`、`highres_finish.py` | V14原生3840×2160全帧重绘、逻辑坐标代理、native光学后处理、分段编码与流复制拼接。 |
| `prelude_v14.py`、`render_release_v14.py` | 已验证720p版13秒片头、用户加快的警告事件、指定窗441帧重绘与4645帧V11画面复用；`prelude_v13.py`／`render_release_v13.py`保留26秒历史版，`prelude_v12.py`保留共用helper。 |
| `render_sample.py` | 通用绘制、字体、backdrop 与 seeded toy MODEL。 |
| `render_song.py`、`render_song_v11_particle_hearts.py` | 当前入口与 V11 入口快照；资源、时序、dispatch、逐字字幕、全片编码；入口导入时会读本机音频和对齐文件。V9 快照 `render_song_v9_expanded.py` 保留。 |
| `cinematic_world_v7.py` | 0–29.772：开机、计算建筑、主体与词元接续；V14的world_pose／world_identities／world_link对15.319和19.15的状态接续。 |
| `cinematic_signals_v7.py` | 29.772–110.405：沿用精确 V6 几何／电流／时间／角色模块，扩充双体、计算网络、线条物品、莫尔与驻波补全。 |
| `cinematic_late_v7.py` | 110.405–207.1：离开、重建、规则拒绝、递归、删除、学习、心形边界与关机。 |
| `semantic_geometry.py`、`semantic_bridge.py`、`semantic_roles.py`、`semantic_resonance.py` | 歌词动作的具体几何与精确时点；部分从沿用 `_v7.py` 名称的包装模块调用，不能假定是死代码。 |
| `semantic_proof_v14.py` | 85.132–88.540：ONLY候选收缩、固定三维toy线性系统独立观测和唯一解重构；由cinematic_signals的proof调用。 |
| `semantic_agency.py` | 已确认的删除状态、结尾进度公共 helper，以及 207.1 后的英文结束卡。 |
| `semantic_end.py` | 已修复的完整心形曲面，V11 两段心形只复用其轮廓／投影，改由 `particle_heart_v11` 绘制粒子与线体。 |
| `semantic_intro.py` | 开机关机共用的 lattice、tensor 等结构。 |
| `cinematic_math_v7.py` | look-at 透视、旋转、曲线、等弧长取样；含可选对数跨度工具。 |
| `audio_features_v7.py` | 源音频独立频谱与包络控制，哈希验证缓存。 |
| `cinematic_finish_v7.py` | 舞台亮部柔光、压缩、暗角及指定失效段的实际像素排序。 |
| `particle_heart_v11.py` | 两段共享粒子与开放线体；固定ID、重建／失败状态、完整轮廓和后段主体共用投影，避免中心重复和实体填充。 |
| `fault_theme_v10.py` | 147≤t<178 的舞台红色重映射、亮像素锚定故障和有界网格损坏；删除保护窗159.05–162.75禁用空间位移，顶部72像素舞台标题区保护。178秒恢复输入。 |
| `execution_style_v10.py` | 纯代码红底黑字 `Execution` 标签，统一测量 advance／绘制／居中接口、前缀底板、历史 alpha 与舞台尺寸约束；供18处歌词及3处舞台标识共用。 |
| `verify_v11.py`、`verify_v5.py` | V11 实际成片验证与复用的数值／字幕／音频辅助；`verify_v9.py`、`verify_v8.py`、`verify_v7.py` 保留为历史验证。新版本要更新输出路径和事实。 |
| `export_showcase.py` | 从已经完成的全片按帧剪取预览，音画共用区间，并检查解码和时长。 |

当前场景模块仍沿用 `_v7.py` 名称，文件名不表示内容停留在 V7；V14局部更改与发布复用范围须同时查看。`frame(n)` 中优先尝试这三个场景模块，之后才是语义 fallback。V11 仍在所有时间窗抑制旧通用 tunnel／glitch；新增的 `fault_theme_v10.apply` 仅在147≤t<178生效，并在 `cinematic_finish_v7.apply` 之后执行。舞台为 `(0,122,1280,594)`，图像后期只作用于舞台，之后恢复页眉／字幕底板并最后绘制文字。故障阶段保护舞台标题；删除时关闭空间位移和拖影，178秒恢复正常配色。207.1 秒后提前 return 给单句英文卡。改动模块时核实 ownership，不叠加一层会遮住动作的通用效果。

## 时间与音频

先确认用户音轨对应的版本。原歌导入时序前做过波形位置对比；新歌或不同混音不能继承旧对齐。秒数转换为帧采用明确的 round／ceil 规则；两侧事件不能独立漂移。chapter 是五字段 `(start,end,mode,title,mechanism)`，迭代边界用 `for left,right,*_ in CHAPTERS`。

旧粒子 bug：路径时钟为 `t*(0.52+0.34*instantaneous_RMS)`，同一物体随着音量变化出现很大位移；近面透视和模运算 wrap 放大为放射状喷射。改为连续位置时钟，同时处理近面与远面；增强光效不能修复运动时钟。

音乐控制的 FFT 窗中心按每个绝对帧时间独立计算。缓存键包含源音频 SHA256、FPS、帧数和分析版本。混合谱起音不是鼓分轨，频带强度不是声源识别。当前本地节拍相位候选落在搜索边界，因此未接受；文档和缓存 scope 必须准确反映最终应用的相位。

歌词中的电流、看不见、年代、融合、完成、删除等是离散事件；起音增强亮度时不得提前改变这些事件的语义状态。最后 100% 与最后一次命令共用时点，中央和 footer 共用 `ending_progress`。

## 修改前后的有效检查

新歌曲从独立的场景入口和配置开始，不直接运行导入时读取旧音轨／旧对齐的 `render_song.py`。尚无歌词或音乐时，可沿用投影、曲线和 toy 数值方法制作用户要求的静音样片，提供替换事件、音乐特征与字幕的接口；没有实际歌词时不宣称完成逐句语义映射。

以当前变更选择检查，复用已经通过的可靠辅助，避免每次重建整套测试。

- 对新动作看连续片段和开始／中间／终点，不能只看一个漂亮截图。检查镜头边界、同批粒子轨迹和前后状态。
- 透视输出必须有限，背面点应裁掉，连线不能穿出字幕区。后期前后页眉与字幕区域应保持一致。
- 因果注意力上三角为零、行和为一；softmax 输出有限且归一化。`A` 与 `p` 是不同张量。学习反馈修改哪项参数要与画面和说明一致。
- 逐字字幕检查字符完整、每帧最多一个新字符、调度不越出显示区间。量化不足时压缩字符间隔，并保留至少一帧间隔；不能丢字或用连续 float 时间忽略帧约束。 特殊词样式按完整行中的词边界判定，再传入可见前缀；分别检查词前一帧、首字、整词、句中尾随文字及历史行。测量 advance 须包含底板 padding，绘制返回末端须与光标一致；前缀只改变宽度，不使基线或底板高度跳动。普通文字辉光先合成，标签底板后合成；历史使用 alpha，并验证空前缀与零透明度逐像素不改变底图。
- 粒子／线体心形按最新材质需求检查：开放线体不等于错误孔洞，完整性指轮廓与预期连接连续。投影后检查重复中心、粒子叠加亮核、柔光后变实心、表面与碰撞边界分离；固定ID和绝对时间重建保证逐帧导出可重复。
- 心形受阻／离开检查对象与可见心体共用投影，出口取自实际轮廓邻近点，并比较路线转入／转出位置与导数；折线等弧长只近似恒速，不保证切线连续，可用共享切线的Hermite段消除拐角速度突变。
- 心形检查完整网格、轮廓与旋转中的中心／局部缺面；有意失败段另行评估。检测中心像素只能作为线索，仍需动态目测。
- 重建末段也要检查遮挡。降低 RGB 颜色增益会把对象涂黑，并不产生透明淡出；内存方块应在心体后方绘制，或用独立 alpha 图层合成，避免在完整表面留下假孔洞。检查低 gain 的细线、拖尾与粒子光晕：平底色上可从 BG 向目标色插值并跳过近零 gain；覆盖已有内容时仍须 alpha 或正确绘制顺序。不要只检查完全出现的中间帧。
- 同一网络的节点、连线、传输粒子共用 camera/span/center；连续镜头比较同一 ID 在交接时的投影端点、速度、镜头目标与取景跨度，不能只检查数量和ID。移动中的分支交接要由实际 `p(t0)` 和 `v(t0)` 构造下一条路线；仅从当前位置重新ease到目标会突然刹停。检查近面、路径包相位以及下一接点，不能用全屏交叉淡出掩盖几何跳变。
- 短进度动画的 exclusive end 可能落在两帧之间。检查最后实际可见帧已到 100%，需要完成语义时为终点留出几帧保持。
- 学习反馈线路应回到实际更新的张量或控制节点。若 E/Q/K/V/A 固定、仅 logits 更新，反馈应指向 policy/logits，不能用回到 E 的线路暗示嵌入也被更新。
- 删除检查歌词事件时点、剩余数与不可逆性；该时段禁用空间拖影、对象复制及空白处重新点亮，避免残像改变可见数量。标题与字幕须不受故障位移影响。有不同新歌含义时使用新的状态规则。
- 导出结束后 ffprobe 核实视频／音频流、尺寸、FPS、帧数和时长。FFmpeg 整片解码检查不能对正在写入的 MP4 执行。
- 对源音乐和成片解码为同采样率波形，比较零偏移相关性与长度；AAC 是有损转码，不能要求 PCM 哈希一致。V10 延续的相关性阈值为 .995、长度差小于 .10 秒；选择新材料阈值时考虑其变换。
- 从实际编码成片截取关键帧，再看连续修改片段。正常数值、较大帧差或高粒子数不能自动证明视觉美感。
- 主题重映射前后比较同一结构的中间调亮度与黑底像素，再看实际编码线条是否可读；“红色占比高”只能证明色相，不能证明感知亮度保持。避免只把最大通道复制到红通道。
- 稀疏结构故障的候选条带应与当前亮像素相交，位移范围受限；检查效果确实命中结构而不是落在空白，并确认保护区域、退出窗口与任意调用顺序的确定性。相关设计原则见 `visual-techniques.md` 第12项。

- 对历史拖影复用实际几何的过去时间状态，检查当前姿态、历史姿态与退出轮廓均有限；按任意帧顺序调用时应得到同一结果。
- 对供给／防护链检查载荷到达数单调、保护完成早于首次捕获、每个扰动状态只向前推进；避免取模运动把尚未捕获的对象绕回边界内部。对长间奏记录明确阶段与终点，并验证下一场接续对象的投影端点。

## 已验证720p发布工程命令

以下在 `<LOCAL_WORLD_PV>` 中运行。先核对当前发布器的OUT、SOURCE与PATCH_WINDOWS，避免覆盖历史成片。`render`包含prepare；需要重做某个区间时同时调整发布器覆盖窗和验证抽样。局部连续片段通过且用户已请求修改后，导出、检查，再从完成的成片剪重点预览：

```powershell
$env:PYTHONUTF8='1'
& '<PYTHON_BIN>' render_release_v14.py render
& '<PYTHON_BIN>' revision_v14/verify_release.py
& '<PYTHON_BIN>' render_release_v14.py preview
```

完整媒体验证后还需看实际编码片头、三处修改段与patch接点，再形成主任务最终 `verification.json`。单独运行 `render_song.py` 只会制作当前song-only资源，不替代发布流程；原 `verify_v11.py` 只能验证历史V11目标。新版本改名时更新对应验证脚本和预览引用。保存版本入口快照，但仍保留依赖模块；需要可搬迁包时另做显式配置与依赖打包，不宣称入口快照包含全部源代码。

## 证据位置

原生4K局部／adapter／清晰度证据在 `revision_4k`；正式4K成片、chunk与源哈希在 `out_song_v14_4k`，验证以完成后的4K报告为准。技能更新脚本 `revision_4k/update_skill_records.py`只stage草稿，待主任务完成验证后才能apply三个reference并运行skill-creator `quick_validate.py`，现有PyYAML依赖在 `<SKILL_VALIDATION_ENV>`。

V14局部证据在 `revision_v14/prelude`、`world`、`proof`、`love`，正式媒体证据由 `out_song_v14_continuity` 保存；13秒片头、441帧局部重绘／4645帧复用和三处patch接点都需有实际成片证据。参考文件更新草稿脚本 `revision_v14/update_skill_records.py` 默认只stage，经主任务提供完成验证报告后才apply，并执行skill-creator的 `quick_validate.py`；它只修改三个既有reference文件。

V11歌内成片证据为 `out_song_v11_particle_hearts/verification.json`、`showcase_verification.json`、`revision_summary.json`；实际编码抽帧与三张汇总图在同目录。局部语义／几何与连续片段在 `revision_v11/hearts`，独立范围和区间外像素检查在 `revision_v11/review`。入口快照 `render_song_v11_particle_hearts.py` 仍依赖同目录模块；修改前源码在 `source_archive/v10_before_particle_hearts`。

V10 历史完整证据为 `out_song_v10_fault/verification.json`、`out_song_v10_fault/showcase_verification.json`、`out_song_v10_fault/revision_summary.json`；实际编码抽帧为同目录 `encoded_*.png`，三张目测汇总图为 `encoded_review_1.jpg` 至 `encoded_review_3.jpg`。正式报告确认5086帧／211.916667秒全片和911帧／37.958333秒预览完整解码通过，音频相关性0.9992783274210483、同采样率长度差0秒；主任务已目测12张编码关键帧。逐词起点仍为自动稿，未逐句人工听校。V10 入口快照 `render_song_v10_fault.py` 仍依赖同目录模块；修改前 V9 源码归档为 `source_archive/v9_before_fault_theme`。

V9 历史完整证据为 `out_song_v9_expanded/verification.json`、`showcase_verification.json`、`revision_summary.json`，实际编码抽帧为同目录 `encoded_*.png`。局部验证、连续片段与阶段图在 `revision_v9/motion`、`attributes`、`choice`；源函数范围核查为 `revision_v9/source_scope.json`。

V8 历史完整证据位于 `out_song_v8_final/verification.json`、`out_song_v8_final/showcase_verification.json`，编码后抽帧为该目录的 `encoded_*.png`。最后一轮全片审查摘要在 `final_review_v8/review_summary.json`，局部修正与画面对比在 `final_review_v8/opening`、`middle`、`late`，全局检查在 `final_review_v8/global`。

V7 历史完整证据仍保留于 `out_song_v7_cinematic/verification.json`、`showcase_verification.json`、`encoded_review.jpg`；模块证据在 `world_v7_test`、`signals_v7_test`、`late_v7_test`，整合证据在 `integration_v7_audit` 与 `integration_v7_final_late`。修正前源码归档为 `source_archive/v7_before_final_review`。V7 历史通过结果不能代替 V8 或后续版本的新证据。

后续扩充的局部证据在 `revision_v9/motion`、`revision_v9/attributes`、`revision_v9/choice`，包含相应状态检查、连续片段或阶段图；choice 还保存与执行树的交接比较。这些局部通过结果不代表新整片已经导出、解码或通过音画验证，最新交付状态应以完成后的全片报告为准。

V10 局部证据：`revision_v10/badge/validation.json` 与样张验证标签尺寸、基线、前缀和透明度；`revision_v10/integration` 保存18处歌词的集成检查；`revision_v10/fault/validation.json` 与连续预览检查结构命中、删除保护及故障退出。局部结果与上面的正式全片报告分别保留；后续修改不能沿用V10通过状态。Skill 版本记录更新脚本为 `revision_v10/update_skill_records.py`，只从正式报告核对事实并修改项目参考文档。

交付使用绝对路径的视频链接或内嵌视频，概括动作变化、实际验证与仍为自动稿的歌词时序。参考方法保留来源和实现边界；不要宣称实现了未移植的 shader 或真实训练模型。
