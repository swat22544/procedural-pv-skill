# 参考作品、固定版本与研究范围

本 skill 从四份代码 PV 中提取不同层次的制作方法，并结合两份本地工程的实践。方法归纳是我们的理解，原作品、设计文档、代码与素材分别归原作者。需要判断某个实现时，沿固定提交和笔记中的函数路径回到源码，不以本表代替源码。

## 四份主要 PV 参考

| 项目与作者 | 研究使用的固定提交 | 本 skill 对应研究 |
|---|---|---|
| [pdoom-video — mexicat / Giacomo Magnanini](https://github.com/mexicat/pdoom-video) | [`fa164b74fd0b26a99375715b619c55b4a0fb3d05`](https://github.com/mexicat/pdoom-video/tree/fa164b74fd0b26a99375715b619c55b4a0fb3d05) | [整体设计](skills/procedural-pv/references/pdoom-design.md)、[机位方法](skills/procedural-pv/references/pdoom-camera-method.md)、[镜头案例](skills/procedural-pv/references/pdoom-shot-cases.md)、[文字案例](skills/procedural-pv/references/pdoom-lyric-cases.md) |
| [world-execute-web — FrostNovaOrg / FrostNova](https://github.com/FrostNovaOrg/world-execute-web) | [`29aefca50e40c14498420e1c6e1f3a1037727e17`](https://github.com/FrostNovaOrg/world-execute-web/tree/29aefca50e40c14498420e1c6e1f3a1037727e17) | [转场与状态接续](skills/procedural-pv/references/frostnova-transitions.md)、[视觉机制库](skills/procedural-pv/references/visual-techniques.md)、[既有来源](skills/procedural-pv/references/legacy-sources.md) |
| [world-execute-me-dsh-pv — MisakaZentai](https://github.com/MisakaZentai/world-execute-me-dsh-pv) | [`a598092a1b1db91fa43961a81269353d13419f6f`](https://github.com/MisakaZentai/world-execute-me-dsh-pv/tree/a598092a1b1db91fa43961a81269353d13419f6f) | [元素叙事方法](skills/procedural-pv/references/misaka-motion-method.md)、[逐链案例](skills/procedural-pv/references/misaka-motion-cases.md)、[逐词来源与早期研究](skills/procedural-pv/references/legacy-sources.md) |
| [world-execute-me-lyric-mv — Nyankomintsu / Nyankomint](https://github.com/Nyankomintsu/world-execute-me-lyric-mv) | [`93ac63775ee58b33ff102167514f2603a48940cc`](https://github.com/Nyankomintsu/world-execute-me-lyric-mv/tree/93ac63775ee58b33ff102167514f2603a48940cc) | [歌词叙事方法](skills/procedural-pv/references/lyric-dramaturgy.md)、[字体身份与歌词案例](skills/procedural-pv/references/nyanko-lyric-cases.md) |

### pdoom：观察任务决定镜头

重点研究相机位置、目标、焦距与滚转如何各司其职，为什么在某个时刻跟随主体，又为什么停追或拉远。画面空间、前景参照、物体运动与文字阅读共同决定机位。空间与材料文字的研究包括曲线、标牌、纸面和堆叠层；屏幕文字与世界文字分别安排。

源码研究涉及场景、相机、字形、GPU 线批次、HDR/时间采样和输出队列。原工程的 Bun/Vite/TypeScript、浏览器后端和平台相关声学流程，与我们本地网页渲染入口并不完全相同。移植方法不意味着整套工具链原封不动可运行。

先读过设计文档、实际实现与仓库图版，随后对选定镜头窗口做隔离复现和有限函数检查；没有完整重渲染作者整片，也没有把某个抽样窗口的通过推广为所有镜头通过。

### FrostNova：镜头边界两侧共享什么

重点研究有效剪辑表、共同歌曲钟、场景交接、投影锚点、相机接力和文字换手。位置连续与速度连续按通道讨论；同一时刻、同一状态与实际屏幕投影，比“用了某个转场名”更重要。旧有 14 项语义/计算机制研究继续保留。

本地 world 的 NumPy/Pillow 实现属于独立适配。它不因此获得原项目的全部 WebGL2/SDF、四元数 rig、HDR 或 GPU 后期能力。研究没有获取或解密参考网站的录音。

### Misaka：动作留下下一镜需要的结果

重点沿最终生效的安装/补丁链阅读，检查来源、载体、映射、落点、绘制责任和下游状态。双画面关注同时观看是否有比较或因果价值；元素到达之后必须看接收场怎样保留结果。

该研究快照按最终链计得 97 个正时长镜头和 74 个 Cut 实例；这与上游 README 的其他计数口径需要区分。研究使用隔离运行与仓库提供的替身角色抽样，不能将它描述为原公开视频舞者的完全复现。早期本地 world 使用的逐词时序来源也继续保留。

### Nyanko：文字是一种行为

重点研究说话者的字体身份，歌词如何成为消息、主观画面、执行或删除，关键词独镜怎样返回原时间线，以及复唱如何改变既有状态。文字不仅与背景同色，还与场景共享材料、结构、动作规则和叙事后果。

研究固定在上述快照（package 4.0.0）。原仓库不含歌曲与歌词，原研究在隔离副本补入本地合法输入后进行选定窗口抽样；这些输入不随本 skill 发布。抽帧和布局检查不能代替逐句听校或整片视觉验收。

## 两份本地工程与应用样片

- **world.execute(me) / V14 native 4K**：状态与语义、双主体身份、连续几何、字幕布局、片头偏移、分段编码的实践来源。
- **Machine Love / V2**：NumPy 状态、ModernGL 舞台、Pillow 字幕、分析分轨与多模型对齐、复唱和视觉成熟度分离的实践来源。
- **《把回声点亮 / ECHO, RETURNED》**：更新后方法的 40 秒网页应用样片；不计入最初六份研究工程。

它们不是这里新声明的公开源码仓库，详见 [本地实践说明](docs/LOCAL_PROJECTS.md)。公开版保留案例记录与方法，没有上传完整工程、原曲和私有证据目录。

## 其他方法与工具来源

- [feitangyuan/onetake](https://github.com/feitangyuan/onetake)，研究快照 [`cf09bde3e392c9aa32c4157f80cdbe1fa556685e`](https://github.com/feitangyuan/onetake/tree/cf09bde3e392c9aa32c4157f80cdbe1fa556685e)：动作节奏、跨段承接与连续性检查。原研究运行过动作库的有限测试，没有取得未公开的完整案例实现，也没有分发上游库。
- [LBEILC/Multiplane](https://github.com/LBEILC/Multiplane)：网页镜头的实际导出工具。安装、CLI 协议、时间与硬切契约以其官方文档为准。
- [Blender](https://www.blender.org/) 与 [MCP for Blender](https://github.com/ahujasid/mcp-for-blender)：原生三维制作与可选实时控制接口，二者职责分开。
- 其余引擎、字体、分析与输出工具入口见 [工具清单](docs/TOOLS.md)。

版权与许可边界见 [NOTICE.md](NOTICE.md)。感谢所有原作者公开制作过程；欢迎直接阅读他们的源码和设计文档，形成自己的理解。
