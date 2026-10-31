# FrostNova：转场前后的状态与画面接续

> 公开版阅读约定：`<…>` 是待配置的路径占位符；文中原项目文件名属于所指参考仓库或本地案例，不表示随本 skill 附送。原研究的抽样与运行记录仅说明当时的验证范围，不代表你的环境已经通过。跨机器使用先读 [环境接入](environment-setup.md)。

当任务涉及跨镜头、跨章节、形态接续、接点跳变或“转场后像换了一套东西”时读取本页。与 [pdoom-design.md](pdoom-design.md) 的镜头编排、[onetake-continuity.md](onetake-continuity.md) 的屏幕运动检查配合；FrostNova 原有 14 项机制仍在 [visual-techniques.md](visual-techniques.md)。本页细化交接实现，不要求全片一镜到底。

检索入口：**剪辑版本 → 连续类型 → 时间窗 → 相机/画面/文字换手 → 五个源码案例 → 粒子对应 → 交接单 → 验收**。

## 1. 先确认研究的是哪个剪辑版本

来源：[FrostNovaOrg/world-execute-web](https://github.com/FrostNovaOrg/world-execute-web)，固定提交 `29aefca50e40c14498420e1c6e1f3a1037727e17`；2026-11-01 查询远端 HEAD 仍为此提交。本地 `<REFERENCE_ROOT>/world-execute-web`。本次读取核心引擎、剪辑表和相关章节，并对照固定提交文件；不要把文件中的所有备用镜头都当作当前播放路径。

- `src/engine/engine.js:74` 默认 `edit="main"`；`src/edit/table.js:559–565` 将 main/remake 等映射到 REMAKE。
- `orig` 不使用编辑表，沿章节自己的时间与 transitionIn；`src/lib/published.js` 的 `remade(ctx)` 区分它与重制分支。这里的 published 是代码分支名，不是本次对线上视频版本的鉴定。
- 编辑表能更改入点、join、构图 shift、关闭镜头；章节自己的绘制函数还可能按 remade 切换。因此先解析有效镜头，再研究相邻关系。
- `cuts = hardCuts(REMAKE, overrides)` 提供比较版，只把 relay 改硬切，保留 points/fade 等其他 join；其中少量入点/构图另有覆盖，不能称为“所有转场关闭且其他参数绝对不变”。

研究或制作时记录 `{source_commit, edit_name, shot_a, shot_b, resolved_join}`。现有 source index、许可及本地独立适配边界见 [legacy-sources.md](legacy-sources.md)。

## 2. 先选承接关系，再决定效果

| 承接类型 | 必须保留的依据 | 不应误称为 |
|---|---|---|
| 同一世界换机位 | 相同全局时间、状态函数、主体身份；构图有明确落点 | 每次换镜头都重开模拟 |
| 两场相机接力 relay | 同时刻 A/B 相机姿态可查询；交接期间用共同姿态绘制 | 只叠两张不同视角的图就形成连续空间 |
| 投影/形状匹配 | 主导轮廓的屏幕位置、占幅、方向和退出/进入条件 | 所有粒子、速度、后期都完全相同 |
| 屏幕锚点匹配剪辑 | 观众视线跟踪的落点、大小/亮度层级、语义因果 | 两个场景共享真实三维坐标或数据身份 |
| 对象/状态接续 | 索引或显式父子映射、形态参数、时钟、事件阶段 | 仅同色、同名或同样使用粒子 |
| 有意硬切 | 拍点/词头/语义落点，切后主体可立即定位 | 连续性的失败，或必须补溶解 |

对声明连续的量，再区分 **C0：边界值相接**，**C1：边界值与一阶变化率相接**。这是逐项性质：相机 C0 不代表粒子 C0，主体位置 C1 不代表曝光 C1。动作故意突然停住时记录停顿意图；不要把所有跳变抹平。

把每个接点分为四段编排：**出镜准备**（主体移到承接位置/铺出下一结构）→ **交接**（保持约定量、转移绘制责任）→ **接收落稳**（新场延续旧状态）→ **新动作**（落稳后才启动新增结构/标签/模拟）。四段可以重叠，也可以压缩成匹配硬切；关键是知道各自何时开始，而不是统一套一个 fade。

## 3. 转场时间属于相邻两镜

源码：[edit.js](https://github.com/FrostNovaOrg/world-execute-web/blob/29aefca50e40c14498420e1c6e1f3a1037727e17/src/engine/edit.js)、`timeline.js:43–75,97–98,129–148`。

入镜点为 `tb`，转场时长 `D`，bias 为 `b`：

```text
joinStart = tb - D*b
joinEnd   = tb + D*(1-b)
p         = (t-joinStart)/D
```

- **whip**：默认 6 个工程帧，bias=1，运动在切点前完成；本库工程为 60 fps，6 帧是 0.1 秒。不是换成其他 FPS 后仍照搬 6 帧，也不是从拍点后才开始甩镜。
- **glide**：时长按 beatLen，bias=.5，跨切点居中；需要为双方留出可求值范围。
- **xfade**：bias=0，从切点开始保留旧画面尾音般退出；选择理由是声音/内容延续，不能当所有转场的默认。
- HIT/VOX/SYL 可对齐实测起音，并提前一个工程帧。HIT/SYL 找不到容差内起音会抛错；VOX 有词头回退。学习的是有明确容差与回退的事件选择，不强制每个作品提前一帧。

引擎对 A、B 都传入**同一个歌曲时间 t**，只令 `lt=t-shot.start` 不同。入场前 B 的 lt 可为负，出场后 A 的 lt 可大于时长；`p` 被钳到 [0,1] 不等于 lt 也被钳住。每个状态/相机明确选择继续、保持端态或预演，不能在转场中途突然重置时间。重要事件使用解析后的入点 `startOf(id)`，避免改剪辑表后关键词动作仍按旧时序触发。

Timeline 能报告相邻 join 时间窗重叠，但报告是 warning，不是自动拒绝渲染。制作时根据节奏缩短/移位/合并镜头，先消除非预期三场争用；不能把“日志里有检查”写成“已拦截错误”。

## 4. 相机、场景内容与文字各有交接时刻

源码：[engine.js:421–510](https://github.com/FrostNovaOrg/world-execute-web/blob/29aefca50e40c14498420e1c6e1f3a1037727e17/src/engine/engine.js#L421)、[rig.js](https://github.com/FrostNovaOrg/world-execute-web/blob/29aefca50e40c14498420e1c6e1f3a1037727e17/src/engine/rig.js)。

### 相机接力：先统一观察，再换内容

`_probe(A,t)`、`_probe(B,t)` 取得该时刻双方的主相机。`blendPoses` 得到共同 pose；`rig.drive` 用此 pose 分别渲染 A/B，然后混合场景。这里的端点随 t 继续运动，不是截取切点两张静止姿态。

`blendPoses` 分开处理：观察点线性插值；四元数 slerp 朝向；观察点平面的正取景半高按对数插值；透视半视角的 tan 值插值；再由观察点、朝向和距离反求相机位置。这样可以围绕主体转动，而非让相机位置直线穿过主体。

正交/透视混合以有限的虚拟距离 600 近似正交端，并调整 near/far。**焦平面尺度可对齐，离开该平面仍有透视差**，不能承诺全部三维空间零跳变。迁移前检查主体厚度、前后景视差与裁剪；虚拟距离和 near/far 是该工程实现选择，不是通用常数。

画面交接进度 `m` 另算：glide 默认在 p=.35→.65 混内容，whip 在 .45→.85；相机运动分别用 inOutSine 与 `p^2.4`。这不是全程等比例叠图。运动曲线与换手窗应随遮挡、空间位置及音乐编排，不固定这些数字。

注意 whip 的 `p^2.4` 末端斜率不为零，它表现快速到拍的甩镜，不保证 C1 刹停。即使 glide 缓动的端点导数为零，移动端点、构图 shift 和接点新动作仍需一起检查。

### 主相机、插入画面与缺失相机

`rig.cam` 跳过 inset，并处理 main/used；probe 到主相机后提前退出。把主相机求值放在有副作用的绘制之前，避免探测时先推进模拟。嵌套回忆画面 `renderShotAt` 用 `rig.isolate` 保留其自己的相机，并抑制它的文字层；否则宿主的 relay 会把回忆视角也拖走。

若任一场没有可探测相机，源码记录 note 并退成硬切。日志里有 relay 配置不代表画面实际执行了 relay；验收时检查 effective join 和 notes。可检查纯屏幕 shader 的转场，但不能虚构不存在的主相机。

### 文字、视线与后期

- `textLead` 对 relay 只选择一侧文字：glide 在内容换手中点切换，whip 从第一帧就由入场文字负责，使绑定新主体的标签随相机到位，避免六帧中间突然跳标签。该规则只适用于 relay；普通 fade 的文字 alpha 规则不同。
- `_dress` 的 `shift:[dx,dy,settle]` 在投影矩阵上偏移构图，使新镜头主体先出现在上一镜留下的视线位置，再回到自己的构图。dx/dy 是设计像素，转换成 NDC；不能直接当世界坐标偏移。probe 返回的原始 pose 未包含最终 shift/punch，速度验收应取实际最终投影。
- 两场先各自计算后期参数，再按交接进度 `lerpPost`，减少新场 grade 突然接管。颜色/曝光仍需看画面：参数插值不等于视觉亮度恒定。
- 时间采样只让中间子帧绘制文字；scene 文字在时间累计之后、Bloom 之前，overlay 在最终输出合成。世界里真实的字形粒子仍属于场景几何，不能误称“所有文字都不模糊”。

## 5. 五个具体案例：前镜交什么，后镜接什么

### A. 双体轨迹展开成网络：同一计算规则，比复制外形更强

`src/ch/pre1/duo.js:15–185` 的 `duoKeys/buildTraj/duoState` 生成同一轨道时间表和历史轨迹；`04_pre1.js` 与 `05_c1.js` 分别初始化，但复用这些函数。粒子 shader 以同一个 i 计算历史位置 p1 和网络位置 p2，`uUnwind` 按轨迹位置错峰变换，弧线项 `sin(πk)` 在两端归零。me 从上方展开，you 从底部汇成输入脉冲，再由 uInject 注入网络；`05_c1.js:283–324` 同步撤去横档、增加连边和热图。

可迁移：前镜的运动历史本身成为后镜结构；双方调用同一个 state(t)/生成器，形变带方向和因果。不要先删除螺旋、再新建随机网络并称其连续。

**版本差异**：orig 的 `pre1/spin2` 和 `c1/unwind` 使用相同 `pairCam(t,K)`；后者先继承，再逐渐偏离。main 在 `table.js:319–320` 关闭 spin/spin2，前镜变为带旋转/压暗的 helixTop，因此不能把 orig 的相机连续直接归给 main。共享几何机制仍可单独学习，最终剪辑关系要另算。

### B. 隧道口变振板：在换坐标系前对齐屏幕占幅

`src/ch/07_pre2.js:tunnelR/plateOpen/plateAt`（1098–1282）在 main 的一镜内完成：收束滚转 → 圆截面 n=2 变为近方 n=40 → 振板出现在隧道口 → 墙体退出 → 换成俯视振板 → 转到下一章入镜角度。table 关闭 tunnel2/squareEnd/plate 的独立镜头，因为它们已成为这一镜的内部阶段；不是丢失这些动作。

滚转先在截面仍圆、方向变化较不显眼时收束，再让方口定向；其他纹理或主体仍可能显示滚转，因此这是感知遮蔽策略，不是严格旋转不变证明。退场双体、隧道墙、板上支架分别安排窗口，使观众一次只需跟踪主要变化。

隧道端是 FOV 62°、板 scale=2、距离 `EXIT_END=3.6`；俯视阶段是 FOV 36°、scale=1。源码用：

```text
R0 = EXIT_END * tan(31°) / (2*tan(18°))
```

以物体尺寸、距离及 `tan(FOV/2)` 的关系保持投影占幅。之后 r/az/el 走向 3.5/.35/1.2，fov=36，与 `08_c2.js:726–742` 的 plate 首姿态相接；地面、杆与圈也逐步进入后镜的摆放。

Sprinkle 与 Sand 各自构造，但使用相同 1024² 格点及 cell hash，基础点索引相容。不能据此写成整帧完全相等：新场仍有微小 jitter/buzz、HUD/场变化，az 速度也由前段末端的零变成 .06 rad/s。此例支持投影/姿态 C0 与基础布局匹配，不支持无条件 C1。Moire 到 tunnel 另有剪辑关系，也不是同一网格连续变形。

### C. 输入框收为中心，再展开内存：匹配视线的切镜

`08_c2.js:promptR/isolationR`（1987–2025、1527–1562）先保持输入框的终态：相机 pos=[0,.9,6.5]、look=[0,.55,-6]、FOV36，框尺寸、填充及提示不突变。退镜时框按 `.1^pull` 缩小，光标朝设计坐标 [960,540] 移动，旧世界逐步退去。

`09_c2x.js:memory`（697–750）以顶部正交视图把亮点放在同一屏幕中心，再让 `heapState.reveal` 于 .75 秒内展开内存范围；`drawHeap` 初始半径 .8，故不是绝对“首帧只有一点”。

可迁移：前镜交出稳定视线落点，后镜围绕它增加信息。heap 用独立 `makeHeap(324)` 构造，没有把聊天气泡 ID 转成内存块 ID；这是语义/屏幕锚点匹配，不是相机物理穿入或完整状态迁移。回忆缩图也只复用视觉内容。

### D. 心体进入尾声：继承运动，再开新动作

源码：[14_love.js:2708](https://github.com/FrostNovaOrg/world-execute-web/blob/29aefca50e40c14498420e1c6e1f3a1037727e17/src/ch/14_love.js#L2708)、[outro/heart.js](https://github.com/FrostNovaOrg/world-execute-web/blob/29aefca50e40c14498420e1c6e1f3a1037727e17/src/ch/outro/heart.js)、`15_outro.js:812–846`。

前镜 finale 的相机时间窗结束于章界 **之后 .5 秒**。outro 的 `finaleCamera` 继续同一个 k、az/r/el/target 公式，再渐进添加 pull/recentre/rise；它们在章界为零。章节分段不是动画归零点。

尾声将主体、心体及相机/目标一起 y+1.45，使新舞台坐标适合后续动作而相对视图保持。判断连续应比较最终投影或完整相对变换，不能看到世界坐标改变就判跳变。

两章用相同生成器、数量、seed（心体 surface21、主体 ball31）和挤压位置规则重新生成兼容几何；它们不是同一个 JS 对象引用。heart dissolve 在前镜为 x²，后镜以边界值 d0、导数 s0 接指数尾段：

```text
A = limit-d0
d(t) = d0 + A*(1-exp(-(t-tb)*s0/A))
```

在本片有效参数下，d(tb)=d0、d'(tb)=s0，之后趋向 .42。迁移公式时先确认 A>0、时间区间有效；若目标不在运动方向上，另设计减速曲线，不能直接除以零或套发散分支。这是已实跑的标量 C1 接续例，不代表全部渲染通道已验 C1。

clear 在边界后 .05 秒启动，文字粒子 .08 秒，release .15 秒，漂移 .25 秒；原挤压状态先被下一镜接住，新动作再逐项展开。末字也继承原位置、字号、tracking、起点，在实际 sungEnd 后退出。学习这套因果顺序，不把具体延迟固定给所有歌曲。

### E. 相同内容，多种剪接：用对照判断运镜是否有帮助

`table.js` 中 gauge→gaugeTilt、wall→wallOblique 使用短 whip；nodeMacro、waiting2 等使用 glide；关键词镜头有 cut/punch。通过 `hardCuts` 对照同一组镜头，可以判断 relay 是否帮助看懂结构，或只是延长观看负担。

做自己的对照时锁定歌曲时间、主体状态和内容，只改变待评估的连接；另记录必要构图修正。评估观众能否持续定位主体、接收新信息、读完关键词，而不是转场数量或运动量。

## 6. 两种“粒子转场”不要混为一谈

`src/engine/swarm.js:shapeTexture/VERT` 用同一纹理坐标/粒子 i 在 A/B 取形状；延迟可以按 hash 或离波源距离，弧度项在端点归零。要保持语义身份，还要给形状采样规定同一顺序/显式映射；只固定随机 seed 和 count 不一定保证邻接或语义对应。Swarm 的 shape cache 以 key@count 为键，改变生成规则时需新 key 或失效旧缓存。

`src/engine/pointcut.js` 则对两幅画面的相同 UV 网格采色，让图块收成球再散开到新画面。原图在开头 0→.14 交给点，球心期间换色，末尾 .86→1 交回目标图；中心/边缘错峰和固定 seed 避免全部同时开关。低亮画面中额外固定份额的点亮起，使中间球仍可见。

这是一种**图像载体**，不是原三维节点变成新三维节点。它适合把图像收束再展开的语义，不能替代真正需要身份保持的形变。PointCut 的网格数随输出尺寸变化，固定 seed 只保证同一规格下索引可复现；不能宣称任意分辨率都有逐点对应。

## 7. 将方法写进自己的交接单

只为需要设计/检查的镜头边界记录下列字段，可放在现有 shot 数据，不强制新增文件格式：

```yaml
id: shotA_to_shotB
edit: chosen_version
kind: state_carry       # relay / projection_match / anchor_cut / hard_cut
event: resolved_music_event
window: {start: ..., cut: ..., end: ...}
carrier: {from: object_id, to: object_id_or_mapping, relation: ...}
clock: {basis: song_time, before_start: hold_or_continue, after_end: ...}
inherit: [shape, seed_and_index_order, phase, camera_target, velocity]
screen: {anchor: ..., extent: ..., orientation: ..., travel_direction: ...}
camera: {source: ..., target: ..., projection: ..., shift: ..., clip: ...}
handover: {picture: ..., scene_labels: ..., overlay: ..., grade: ...}
next_action: {start: ..., condition: carrier_landed}
expect: {position: C0, velocity: C1_or_intentional_stop, cut_reason: ...}
check: {boundary_samples: ..., clip_range: ..., coverage: ...}
```

值填实际事件/数据；删除不适用字段，不为简单硬切伪造对象映射。前后场尽量共享 stateAt(t)、cameraAt(t)、projectAnchor(t) 等可查询函数；若分别重建，记录相同生成器版本、count、seed、排序、父变换和缓存键。只有确实要求“同一个对象”时才要求这种对应。

跨渲染器（例如 Blender → Three.js）交接记录世界单位/轴向、完整变换、FOV/正交高度、主点、aspect、实际 viewport 和颜色空间。用最终输出投影校准承接主体；不同工具的相同 XYZ 不等于同一构图。

## 8. 验收前后关系，而不只看中间一张图

1. **有效路径**：解析所选 edit，列出实际 A/B 和 join。检查 off、覆盖入点、缺相机退切、窗口重叠、异常占位；不能只看声明。
2. **同一边界状态**：双方在同一个 tb 求出交接量，比较几何/身份/阶段/目标投影。C0 不是要求运动中相邻两帧坐标完全相等；相邻帧本来就应移动。
3. **速度与落点**：用各自 tb 左右的小时间步估算单侧速度，单位为输出 px/s（尺度可用 d log(size)/dt）。需要 C1 才比导数；声明的硬切/急停另验理由。包含父变换、相机、projection shift、post zoom 和 viewport，不能只比 raw pose。
4. **完整窗口**：至少覆盖 joinStart 前、start/mid/end、tb 两侧及新场落稳后；从时间量化后的真实帧再抽样，必要时增加子帧。看连续片段，检查遮挡、裁剪、主体消失、假重复、闪白与阅读时间；不以单一整帧差分阈值评分所有转场。
5. **分层换手**：确认旧物退出后不重现，新物没有在落地前自行运行；稳定字幕无重影，世界标签属于正确主体，后期和背景不突然改亮度。独立测试关掉模糊/光效后接点仍成立。
6. **时间确定性**：顺序、乱序、重复、分段起渲都查询同一 t。源码 `_advance` 按工程整数帧重放模拟，不是任意子帧积分；有模拟时检查子帧查询能力，必要时烘焙并插值。
7. **采样切点**：FrostNova `renderFrame` 对中心附近多时刻累计，但没有在这一循环中按硬切裁剪快门。移植/改写时沿我们现有流程声明硬切并限制跨切采样；不能把该实现说成已具备通用切镜保护。内外渲染器只选一层时间采样。
8. **覆盖与证据**：每个声明连续的接点写明已查/未查/失败及原因；代码数学、真实画面、编码后片段分别报告。没有主体数据、零个有效接点、缺帧都不能算通过。可在预览加锚点轨迹与 join 进度，仅作为调试层。

本次证据：`<PRIVATE_STUDY_ARCHIVE>/pv_frostnova_continuity_20261101/evidence/` 保存来源比对、17 项原源码函数检查、文档保留/链接与技能验证。检查直接导入未修改的 Timeline/edit/rig/outro-heart，使用合成时间轴/相机样例；覆盖时间窗、文本责任、投影尺度、近似正交边界、相机隔离与 dissolve 的值/导数。这不等于原片全章视觉验收，也没有把这些函数自动移植到已交付 PV。
