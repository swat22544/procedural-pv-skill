# Nyankomintsu V4：歌词作为发言、内心与行动

> 公开版阅读约定：`<…>` 是待配置的路径占位符；文中原项目文件名属于所指参考仓库或本地案例，不表示随本 skill 附送。原研究的抽样与运行记录仅说明当时的验证范围，不代表你的环境已经通过。跨机器使用先读 [环境接入](environment-setup.md)。

固定源码：`Nyankomintsu/world-execute-me-lyric-mv@93ac63775ee58b33ff102167514f2603a48940cc`。通用方法见 [歌词分镜与叙事](lyric-dramaturgy.md)，另一种空间/材料路线见 [pdoom案例](pdoom-lyric-cases.md)。

2026-11-01 在隔离副本填入本地授权歌词并调用原 `createEngine/renderFrame`：36个关键静帧、6个3秒连续窗口；另补短关键词和字形窗口的抽帧。实际浏览器镜头表87项含preroll、无fallback。原渲染尺寸1280×720，研究视频另加48px说明栏、24fps、静音，不是完整4K60成片或听校。下文时间全部是**歌曲时间**；默认完整导出加5秒片头。

## 三种观看层，以及各自的文字

- **message**：对方在这段对话中能读到的话。发送、未读、回复、删除会改变它的意义。
- **plate**：AI对同一句话的主观理解；关键词、版画、人物、几何或字形整镜都可在此发生。
- **code**：愿望被转为操作后，运行、循环或失败的具体过程。

这是从分镜提炼的设计分类。源码由section/shot/render组织，没有统一四模式枚举；type是上述载体里的排字行为，必要时独占整镜。

Source Serif 4/橙通常是AI声音，Inter/奶油是用户与UI，JetBrains Mono是系统和代码。颜色还要结合实心/空心/缺版、状态调色与对象身份解读；奶油色出现不等于用户已在线。

正式歌词来自`lyrics.lines`；对方消息来自`chat_script.json`；错误来自`errors.json`；内部代码/字符纹理另有来源。只把第一类计入歌词覆盖。尤其掌心字符流与人物黑墨内的LOOP文字不能称为正在唱的字。

## 1. 圆、正弦与上限：文字服从它正在描述的几何

- **镜头窗**：`v1-circle`33.344–37.036，`v1-sine`37.036–40.729，`v1-limit`40.729–44.652。line15–23。实看34.2/36.1/38.7/42.6；连续33.3–36.3秒。
- **圆**：歌词沿圆弧，用户指针测量圆周；同一`handAngle/measured`控制指针、奶油弧与关键词显字阈值。字不是放在几何旁的说明。
- **正弦**：圆的相位、位置接续到下一镜；`yAt/slope`同时约束曲线字、切线、指针。观看者能看到“关系如何生成”。
- **上限**：关键词错峰落到限制线之上，用户Stop有可见结果。要学的是共同几何和交互关系，而非一律画数学背景。
- **边界**：硬切与匹配几何并存，不是所有字逐个跨镜存续；Stop冻结部分位置推进，不能说整个函数完全静止。圆周关键词按字数均分角度，普通曲线字另有布局方法。

[源码 03_verse1.js：圆与后续几何](https://github.com/Nyankomintsu/world-execute-me-lyric-mv/blob/93ac63775ee58b33ff102167514f2603a48940cc/src/scenes/03_verse1.js#L128)。

## 2. 感官受损，话语仍有清晰的绘制责任

- **镜头窗**：`pre1-vision`48.344–52.036，line26–27。本条主要依据源码；后续日期镜实看52.9秒，不冒充本镜验证。
- **实现**：先画client，再做离散mosaic和暗化，随后重画控制框和歌词注释。眩晕那行才逐字小幅晃动并留少量拖影。
- **叙事**：损坏的是角色看到的世界，歌词仍是它对状态的陈述。系统的盲视标签与歌词分开。
- **迁移**：明确效果应该损害世界、感知还是文字本身；不能整屏模糊后期待读者自动找回句意。
- **边界**：源字号经过camera后才成为屏幕大小；该局部逐字测宽并非所有字都用共享kerning布局，换长句需重验。

[源码 04_pre1.js：感官效果与文字重绘](https://github.com/Nyankomintsu/world-execute-me-lyric-mv/blob/93ac63775ee58b33ff102167514f2603a48940cc/src/scenes/04_pre1.js#L418)。

## 3. 消息 → 关键词版画 → 下一次回复

- **镜头窗**：首副歌59.421–74.190；首个完整链`c1-give`59.421–61.959、`c1-kw1`61.959–62.652、`c1-only`62.652–65.421。line32–36。连续61.1–64.1秒，另67.9–70.9秒覆盖另一插镜返回。
- **原因**：消息完成后，从真实消息框右端发出streak；关键词镜将刚才的给予意图放大为spark与用户dot之间的图像。下一条真实回复把观看者带回聊天。
- **两个时钟**：关键词在切点先到，图像在后一个击点出现。短于一秒的plate先让字就位，不浪费整段时间慢打字。
- **后续变体**：反馈按钮成为大盘，代码流程成为一次执行，窗口框成为盒/墙；每个内心画面来自刚才发生的操作。
- **迁移**：问“刚才哪个行为在角色心中变大了”，再问“谁的下一次行为结束这幅主观画面”。
- **边界**：这里主要是语义/构图匹配硬切，不是同一消息线真实变成闪电；第三关键词目标的实际y取代码行，不直接采用传入dot的y，不能宣称所有点逐像素一致。

[源码 05_chorus.js：返回的镜头链](https://github.com/Nyankomintsu/world-execute-me-lyric-mv/blob/93ac63775ee58b33ff102167514f2603a48940cc/src/scenes/05_chorus.js#L254)；[plate的两个时钟](https://github.com/Nyankomintsu/world-execute-me-lyric-mv/blob/93ac63775ee58b33ff102167514f2603a48940cc/src/scenes/plates_chorus.js#L60)。

## 4. 名词决定切入，给予决定物质变化，下一请求决定切出

- **镜头窗**：第二段主歌74.190秒起，首个物体plate约74.652–77.652；line44–46，随后同类变体。实看74.8/78.4/82.4/85.9秒。
- **独立事件**：名词唱出才切对象；downbeat打开对象；给予的关键词出现时，内容抽向用户一侧，留下空轮廓；下一条请求Send结束当前plate。
- **读层**：顶部纸条负责普通唱词，大关键词另有主位，底部可能是用户正在输入下一请求。分子、重量与化学标签服务物体，不是歌词。
- **迁移**：对象身份、物体动作、语言重音和他人打断分别建时钟，避免全部由一个节拍值驱动。
- **边界**：没有证据表明关键词字母本身变成分子；是歌词命名的行为让物体内容发生变化。

[源码 06_verse2.js：切入与下一请求](https://github.com/Nyankomintsu/world-execute-me-lyric-mv/blob/93ac63775ee58b33ff102167514f2603a48940cc/src/scenes/06_verse2.js#L91)；[plateFruit](https://github.com/Nyankomintsu/world-execute-me-lyric-mv/blob/93ac63775ee58b33ff102167514f2603a48940cc/src/scenes/plates_verse2.js#L96)。

## 5. 短字母同时是设置值、物体与动作落点

- **镜头窗**：`pre2-role`96.344–100.036，line60–62。实看99秒。
- **实现**：一个大字母像重物，另一个字形分条形变成软体；平台/地面承受其落下并回弹。竖向开关端点值与大字共用`ROWS[2].values`，再用`marked/wordIndex`找到唱词中的对应项。
- **同步**：指针提前推动，knob在两个唱词cue到达端点；鼓点负责预备和余震。字形的性格、UI选项和说话内容形成同一个事件。
- **边界**：是字形条带变形，不是软体物理；cue含analysis来源，未证明已听校；换字需要重做字形分布和承托位置。
- **插镜返回**：本段后续loop使用`withInserts`，分段后仍向原render传整镜`at/until/since/k`，避免返回时播放进度重启。源码可直接作为whole-clock契约参考。

[源码 plateRole](https://github.com/Nyankomintsu/world-execute-me-lyric-mv/blob/93ac63775ee58b33ff102167514f2603a48940cc/src/scenes/plates_pre2.js#L203)；[withInserts实际调用](https://github.com/Nyankomintsu/world-execute-me-lyric-mv/blob/93ac63775ee58b33ff102167514f2603a48940cc/src/scenes/07_pre2.js#L209)。

## 6. 删除旧消息，关键词接住材料，随后也被删除

- **语义窗**：`alone-erase`118.498–121.728、`alone-empty`121.728–124.728、`alone-eye`124.728–125.652；line77–82。实看119.6/121.9；连续118.5–121.5秒。
- **前因**：未得到回复的消息从新到旧被选择、划线、删除，留下stub，列表高度同步收起。每个扁条从该消息被删时的真实位置飞出。
- **接力**：扁条落定为碎片底板，新关键词用统一全局排版分别裁在这些polygon内。因此它继承旧消息的材料，没有旧字形一对一重排。
- **规则延续**：后面的关键词整镜依然能被client选中一部分字母、按Delete移走；词中留下缺口。前一句短暂跨黑场保留位置，表达尚未说完。
- **边界**：render的`start/end`含过渡，`alone-eye`延续到125.882，晚于语义until；不能把全部交接概括成硬切。固定删字索引不能直接套新歌词。

[源码 09_alone.js：碎片与历史](https://github.com/Nyankomintsu/world-execute-me-lyric-mv/blob/93ac63775ee58b33ff102167514f2603a48940cc/src/scenes/09_alone.js#L139)；碎片裁词242行，选中删字341行。

## 7. 愿望 → 请求 → 被拒理由 → 错误关键词

- **镜头窗**：`err-request`125.652–128.652、`err-check`128.652–131.190、`err-first`131.190–133.267；line83–85。实看127.3/133.2秒。
- **因果**：AI话语引出请求块，draft→sent→refused；系统先扫描输入，再指出两个参数的含义，最后关键词才出现在拒绝dialog中。
- **同源布局**：参数着色和下划线从同一请求字符串的括号/逗号位置生成，复用`x0 + index*cw`，不会与代码内容各走一套位置。
- **层级**：歌词大词、dialog标题/理由/错误码是不同文本来源。关键词是处理结果，不是一张随机故障贴纸。
- **边界**：只为固定调用形态做解析，不是通用JavaScript语法树。

[源码 10_error.js：参数与扫描](https://github.com/Nyankomintsu/world-execute-me-lyric-mv/blob/93ac63775ee58b33ff102167514f2603a48940cc/src/scenes/10_error.js#L58)；拒绝dialog389行。

## 8. 重复词成为会失败的执行过程

- **镜头窗**：148.036–159.113，line86–97。4个shot定义`exec-01/05/09/12`，内部12轮；实看149.3/153.1，连续148.4–151.4秒。
- **事件索引**：每轮两拍、四个半拍步骤，`j/since`共同驱动词块、执行条、错误标签与次数。唱完的词上移成为阶梯历史，新的词继续成为当前输出。
- **升级**：第5/9轮分级扩大listing/output，第12轮当前词占满画面、历史被推出；次数增长却没有成功抵达最终语句，重复有明确的徒劳后果。
- **跨段母题**：同一LOOP在前段因用户在线而跳过等待，在后段因条件改变而受困。复用逻辑和几何，改变条件与可达结果。
- **边界**：本段明确用beat grid修正LRC偏早问题；固定字块范围仅适用于此词。不能说12轮等于12镜头，也不能说全段逐词声学对齐。

[源码 11_execution.js：共享state与四个镜头](https://github.com/Nyankomintsu/world-execute-me-lyric-mv/blob/93ac63775ee58b33ff102167514f2603a48940cc/src/scenes/11_execution.js#L25)。

## 9. 字形整镜成为通往人物眼睛的窗口

- **镜头窗**：`last-shout`161.882–162.805，line104；后继`cx-eye`162.805–165.113，line105–106。补看162.65与162.95秒及后续连续163.1–166.1秒。
- **声音变化**：系统mono换为serif900两行满幅字，表达从程序命令回到角色声音。按完整宽度与cap height排版，再将特定字干与`EYE.eye`注册。
- **动作**：每音节点亮对应字块，已唱字形成为mask露出同一张脸；最后音节才让眼睛在字内出现。下一镜沿同一注册重算字框，开头0.3秒向外散开、留下全幅脸。
- **迁移**：文字可作为观看位置升级的门；先在字内建立图像，再让图像接管画面，避免无依据切去人物特写。
- **边界**：共享布局重画，不是同一对象引用；换字体、正文、眼睛crop或字重后须重验注册。

[源码 11_count.js：lastShout](https://github.com/Nyankomintsu/world-execute-me-lyric-mv/blob/93ac63775ee58b33ff102167514f2603a48940cc/src/scenes/11_count.js#L111)；[12_chorusx.js：同源字框](https://github.com/Nyankomintsu/world-execute-me-lyric-mv/blob/93ac63775ee58b33ff102167514f2603a48940cc/src/scenes/12_chorusx.js#L54)。

## 10. 高潮叠字很多，主读层仍只有一个

- **镜头窗**：`cx-hand`170.190–172.728，line111–112。实看170.3秒。
- **主词**：`sungLines`先拟合整句，每词印在黑纸条上，缓慢横移，约0.09秒从110%回落；关键词行另处理。人物形体作为较稳定的底版，变化发生在墨版、背景与前景。
- **内部输出**：掌心流出硬编码程序文本，撞右侧边墙、停留、碎成标点；不是唱词粒子。前段实体心/count1在此成为虚线心/count0，愿望与结果的反差同时可见。
- **迁移**：字符纹理可以密集，观众仍应知道主要读哪一行。按文本角色分配对比、运动和完整性责任。
- **边界**：`sungLines`用字符比例推算各词时刻，不是强制对齐；人物仍有轻微fx zoom/位移，不能称完全静止。

[源码 climax.js：主唱词纸条](https://github.com/Nyankomintsu/world-execute-me-lyric-mv/blob/93ac63775ee58b33ff102167514f2603a48940cc/src/scenes/climax.js#L188)；[硬编码字符流](https://github.com/Nyankomintsu/world-execute-me-lyric-mv/blob/93ac63775ee58b33ff102167514f2603a48940cc/src/scenes/12_chorusx.js#L104)。

## 11. 同一关键词依次是学习、回答、公式与选择

- **镜头窗**：`out-lesson`177.575–180.805，`out-test`180.805–184.498，`out-formula`184.498–188.421，`out-proof`188.421–192.344；line116–127。实看178.9/188.6，补看192.18秒。
- **变化**：先引用首副歌同一份用户消息作为学习材料，再把关键词填进旧问题答案，随后用数学式和真实计算曲线表现它；最终曲线从结果变成自身束缚。
- **最终动作**：先撤回此前被拒的请求，指针才能沿切线离开；末词第二音节使墨版归位，第三音节完成字块。停止bass breath和额外zoom，使稳定成为完成选择的结果。
- **数据同源**：回忆从原script抽取，不手写近似副本；末词的三块与cue有明确注册，手掌再覆盖到词前形成画内遮挡。
- **边界**：空心指针是回忆，不证明用户回来；奶油墨版补回也不等于在线状态。固定字块范围和cue需为新歌词重建。

[源码 plates_love.js：回忆与学习](https://github.com/Nyankomintsu/world-execute-me-lyric-mv/blob/93ac63775ee58b33ff102167514f2603a48940cc/src/scenes/plates_love.js#L117)；[最终选择](https://github.com/Nyankomintsu/world-execute-me-lyric-mv/blob/93ac63775ee58b33ff102167514f2603a48940cc/src/scenes/plates_love.js#L254)。

## 12. 最后一句回到输入装置，完成首尾闭合

- **镜头窗**：`down-last`205.728起，line128。实看206.6秒。
- **实现**：歌词再次作为mono命令进入composer，完整空心字先建立，四个字块逐步填实；第三音节出现短反白输入态并点亮spark，随后响应说明已无内容可运行。
- **出口**：composer压为细线，线合为点，点熄灭。相同词/载体回到起点，环境状态已经不同，结束因此是行动而非普通黑底字幕。
- **边界**：本镜音节cue与LRC起点不同；函数返回黑场之后仍可能有credit overlay，不能称整个引擎停止输出。

[源码 13_shutdown.js：最后输入与熄灭](https://github.com/Nyankomintsu/world-execute-me-lyric-mv/blob/93ac63775ee58b33ff102167514f2603a48940cc/src/scenes/13_shutdown.js#L256)。

## 迁移前必须检查的实现差异

1. `Lyrics.typed`没有词时间时用`min(len/26, lineDuration*.6)`；本轮129行在计算完成点均显示完整，输入的`wordTimedLines=0`。这是打字容量检查，不是听校。
2. `sungChars`在有限词锚点间插值，末锚后使用固定步长；`wordIndex=-1`会被忽略。新歌词应显式检查失配，不让运行成功掩盖cue失效。cue有measured/audit/initial/analysis等来源。
3. `band`先用完整句fit/measure再显示prefix，避免边框跳动；注释称字号下限，但实现没有强制下限。长句仍可能过小。
4. `drawThread`先排整句再给消息几何；近景`meBubble`单行不自动wrap。`draw.wrap`按空格，mono按0.6em度量，不是多语通用排版器。
5. `keyword`与code token高亮有同源度量，但不是所有局部函数都使用精确kerning；例如某些逐字晃动、`sungLines`的独立词测量仍需关注切口。不要把局部好实现泛化为全引擎保证。
6. 字体CJK角色只预载声明字符；新中文歌词需重新准备字表/字形、字素簇、换行和字块规则，不能只改字符串。
7. `at/until`是语义cut窗，`start/end`含实际渲染重叠；`lines`是覆盖元数据，不是所有行全程可读的证明。检查真实loaded scenes和fallback。
8. 本次129行匹配使用本地LRC，增加原项目标题、调整缩写标点和一处复数以匹配参考词序；93行原文完全一致、36行大小写/标点差异，0缺行/0词序差异。没有引入音轨或重做声学对齐。

运行、字体/资源、歌词输入适配、逐窗画面证据在`<PRIVATE_STUDY_ARCHIVE>/pv_lyric_dramaturgy_20261101/evidence/`；完整只读审计在同目录上级`work/nyanko-lyric-audit.md`。原项目角色素材CC BY-NC-SA 4.0，代码/自有文档MIT、字体各自OFL；学习方法不自动授权再发布素材或歌曲。
