# 既有参考：FrostNova 与 Misaka

> 公开版阅读约定：`<…>` 是待配置的路径占位符；文中原项目文件名属于所指参考仓库或本地案例，不表示随本 skill 附送。原研究的抽样与运行记录仅说明当时的验证范围，不代表你的环境已经通过。跨机器使用先读 [环境接入](environment-setup.md)。

这两个仓库是此前 PV 技能的参考，不能因加入 pdoom / Nyankomintsu 而从研究地图消失。2026-11-01 对比更新前备份，旧 project-defaults、visual-techniques、onetake-continuity 和迁入 world-execute-workflow 的原生产流程正文均保留；本页补足显式路由和 Misaka 方法索引，不取代旧记录。

## FrostNovaOrg/world-execute-web

来源：[仓库](https://github.com/FrostNovaOrg/world-execute-web)，既有学习快照 `29aefca50e40c14498420e1c6e1f3a1037727e17`；本地 `<REFERENCE_ROOT>/world-execute-web`。14 项方法、源码符号和独立适配关系完整保存在 [visual-techniques.md](visual-techniques.md) 及 `<LOCAL_WORLD_PV>/reference_study_v7.json`。不把在线默认分支和已学习快照混为一谈。

适用内容：对象稳定身份；同一世界的多机位；连续运动与有界音乐响应；稀疏建筑/通道；历史轨迹变为新结构；toy Transformer 的实际计算；Chladni 振型；莫尔与匹配入口；内存迁移；故障前沿；执行树与剪辑密度；真实像素排序/后期分层；等弧长边界；关机回收开机对象。具体机制及本地实现边界沿用旧页，避免在两份文档分别改写。

2026-11-01 深化：[转场前后的状态与画面接续](frostnova-transitions.md) 说明 orig/main 有效路径、join 时间窗、共享 rig 与内容/文字分别换手，并详解轨迹→网络、隧道→振板、输入点→内存、心体→尾声及剪辑对照。区分投影匹配、身份接续和 C0/C1；不把注释中的连续意图当作全片像素验收。原 14 项笔记保留。

其相机方法尤其值得与 pdoom 并用：`src/engine/rig.js:poseOf / blendPoses` 将观察点、朝向、观察点处取景尺度与透视强度分开。朝向用四元数 slerp，正尺度在对数空间插值，再反求相机位置；混合正交/透视时围绕目标改变取景，减少直接线性插值相机位置造成的穿角。pdoom 提供多机位编排与定制空间转场，本库提供可复用的观察姿态插值。两者都需要实际画面验证。

本地 world V7/V14 是 NumPy/Pillow 的独立适配：CPU 透视线框、toy 数值、局部固定 ID 与有限相机混合。不能因引用 FrostNova 就宣称本地已经移植其 WebGL2/SDF、四元数 rig、HDR/AgX 或全部 GPU 后期。主代码/画面与第三方素材按仓库 LICENSE、NOTICE 分别确认；原歌曲不随源码分发。已有研究不包含获取或解密站点音轨。

## MisakaZentai/world-execute-me-dsh-pv

来源：[仓库](https://github.com/MisakaZentai/world-execute-me-dsh-pv)，本地 `<REFERENCE_ROOT>/world-execute-me-dsh-pv`。初次整理仅核查本地文件与制作文档；2026-11-01 深化时已独立检出提交 `a598092a1b1db91fa43961a81269353d13419f6f`，与该本地目录的1587份受跟踪文件一致。随后在隔离 runtime 生成官方替身、网页截图并抽样原生合成，不等于原公开视频舞者或全片验收。

深化内容见 [元素连续叙事方法](misaka-motion-method.md) 与 [画面/源码逐链案例](misaka-motion-cases.md)：补充载体临时接管、持久落地状态、背景传播场、几何/身份/语义的区别、双画面因果和终态继承。以下既有时序依赖、网页合成和导出边界继续保留。

### 双画面不是固定左右拼图

`film/pv_dsh_frontend_20260927/dsh_her.py` 的 `COVER` 决定窗口接管范围，`LEAD / levels` 决定当前叙事主体；左主导时右区降到0.42，右主导时左窗alpha为0.7，通常用0.25秒渐变，最终FINISH还可局部回亮或接管黑场。窗口沿用原主体窗格的位置/缩放/滑出动作；`install()` 及 `dsh_patch_*.py` 把它接入既有场景。源码对被窗口替换的区域还关闭原舞者扫描转场、重定位到头像锚点，避免装饰遮住聊天内容。

迁移方法：给同屏多个区域编排主次与交接，动作从正确锚点出发；焦点、亮度、空间占比和文字阅读时段共同变化。0.42 是该片参数，内存替换函数是已有工程组织方式，不要求新项目照搬 monkey patch。这里的 dsh 是离线仿制画面，不需要启动真实聊天服务或调用模型。

### 转场由承接对象推动

`film/tui_pv_world_execute_20260926/continuity_full_v2/cuts.py` 的 `Cut` 在切点两侧拥有时间窗：旧场可继续动作，新场按定义的时钟开始；`Frame.over / under` 定义承接对象与主体的层级，`SHOT_HOOKS / DELAY` 控制前镜留下的状态以及等待载体到达的局部时钟。

实例：`C08` 将 RUN 字形拆为沿曲线飞向词流的词块，同时把预算行搬成计数器；`C09` 将词流收束到损失曲线原点，让计数器接成图表标签。变化有明确来源、去向与落地时刻。可学习“承接物单独绘制，接收场保留它留下的状态”，并用固定种子控制字符替换/揭示；不能只复制一个全屏擦除效果。

`v2.py:active_cut / _frame` 负责选择专用转场、主体滑移、覆盖层和外框。各时段有自己的实现与后续补丁，不要只凭早期 cuts.py 的样片说明判断整片；接续后检查持久状态、遮挡、字幕和屏幕坐标。对已有顺序渲染、CUR 与预帧调用按真实入口检查，不能凭“纯函数”文案就认定整个混合链已通过乱序验收。

### 网页逐帧素材与图像引擎组合

`film/pv_dsh_frontend_20260927/seg_shot.mjs` 在 354×537 逻辑窗格生成截图，等待字体、尝试解码图片；CSS animations 暂停后把 currentTime 绑定到歌曲时间，截图命名使用帧索引。需要时测量光标的实际 DOM 边界并保存，供图像合成侧准确接续。

迁移方法：网页负责复杂排版，PIL/其他舞台负责程序几何；共同约定帧号、时间原点、窗格范围和资源就绪。图片 decode 捕获错误不代表资源一定完整，预览仍需检查。新网页制作按本机 Multiplane 约定接入；继续本仓库则保持其已有专用链路，不能把 .mjs 截图脚本直接当作 Multiplane 项目。

### 歌词来源与两种时钟职责

旧技能在 [project-defaults.md](project-defaults.md) 保留的逐词来源路径仍存在：`film/world_execute_word_timing_20260927/word_timeline.json`。本地 world 的 `render_song.py:ALIGNMENT` 仍指向它；这是实际来源依赖，不只是一条灵感链接。

`tools/lyrics.py` 用 `data/timing/*_notext.json` 与歌词文本重建 LRC/逐词文件，并按行哈希核对。`continuity_full_v2/words.py` 保持 LRC 对分镜与编舞的作用，字幕则按人声逐词时序出现，分别处理短词、延音、器乐间隔的持留与淡出。该模块的 0.25 秒等参数属于本片，不能替代用户已有每帧字数规则；换歌曲或音轨必须重做对齐。

### 导出与素材边界

`dsh_her.py:_chunk / render` 可分块输出无损 RGB 母版，再由 `build.py` 合成音轨和发布编码。原生画布是 1280×720 / 24 fps；`--4k` 是 nearest-neighbour 放大三倍。`docs/60FPS_EXPORT.md` 的 60 fps 是另行运动补偿插帧。这两者分别是空间上采样和时间插值，不是原生 4K / 60 fps 重算，也不同于本地 world V14 的原生 4K 路线。

源仓库的替身角色帧与原作者公开视频舞者不同；代码、角色美术、字体、音乐分别按 LICENSE / NOTICE 确认。使用其 TUI/转场方法不意味着要采用角色、聊天窗、舞者或原歌曲。当前 world 已确认的画面选择以自己的项目默认值为准。

## 维护时保留的关系

- 总地图同时列出两份本地工程与四个外部参考，按各自长处选择。
- 新参考可以更新方法，不能抹去旧来源、时序依赖、源码符号、许可边界和已经验收的项目事实。
- 文档迁移明确新旧路径；保留正文或有可追踪的等价迁移，不把“文件仍存在”当作内容完整的唯一证据。
