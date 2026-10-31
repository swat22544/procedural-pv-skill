# Blender 原生镜头与 MCP

> 公开版阅读约定：`<…>` 是待配置的路径占位符；文中原项目文件名属于所指参考仓库或本地案例，不表示随本 skill 附送。原研究的抽样与运行记录仅说明当时的验证范围，不代表你的环境已经通过。跨机器使用先读 [环境接入](environment-setup.md)。

用于实体空间、几何遮挡、材质/灯光、复杂相机、程序建模和原生渲染。简单文字/卡片镜头仍可用 Multiplane；同一 PV 按镜头混合，保持统一时间与颜色。

## 入口发现与历史证据

新环境使用 [Blender 官方发行版](https://www.blender.org/download/)；MCP 为可选项，本案例采用 [ahujasid/mcp-for-blender](https://github.com/ahujasid/mcp-for-blender) 的 server/addon。按所装版本的实际 schema 操作，不要求其他用户复现下面的机器路径和旧会话。后台 bpy 渲染可以不安装 MCP。

下面是原研究时的状态与验证范围，不能直接作为当前已连接的证明：

- Blender：`<BLENDER_BIN>`，本次 --version 为 5.2.2 LTS。
- MCP server：`<BLENDER_MCP_BIN>`，历史固定 mcp-for-blender 2.2.6。
- addon：`<BLENDER_ADDONS>/blender_mcp.py`。
- 历史 2026-10-30 验收：`<PRIVATE_STUDY_ARCHIVE>/Blender-MCP-20261030/evidence/final-report.json`，覆盖建模、修改回读、视口、多角度、EEVEE 图像、保存 .blend、GLB 导出和独立重开。
- 2026-10-31 当前会话已发现 9 个工具，但 get_addon_status / get_scene_info 返回无法连接。不是工具缺失，也不能沿用昨日通过结果宣称今日场景已连接。

当前这套服务是 `mcp-for-blender`；按已加载的 `mcp__blender` schema 调用，不从别的同名 Blender MCP 教程抄工具名/参数。

## 先连接和观察

1. 发现工具，先 get_addon_status 与 get_scene_info（settings + 必要对象字段）。读取实际 Blender/addon 版本、当前文件、场景、相机、帧范围、渲染引擎与资源开关。
2. 若未连接，区分 Blender 未运行、addon 未启用与监听故障。制作需要启动时使用已知程序，保留已有进程/工程；不要另起空实例抢占端口。后台 helper 启动按宿主规则隐藏窗口；需要用户操作的编辑器才显示。
3. addon 的历史入口为 3D 视图 N → MCP for Blender → Start MCP Server / Connect to Claude；名称沿用上游，并不表示只供 Claude 使用。端口历史为本机 9876，以当前状态为准。
4. 用 look(mode=viewport/camera/angles/frames) 检查实际画面。get_scene_info 给结构，不代替视觉检查；look(image='Render Result') 是渲染后回读，viewport 不是最终渲染。

## 工具职责

| 工具 | 用途 |
|---|---|
| get_addon_status / get_scene_info | 环境、资源开关与当前工程读回 |
| execute_blender_code | 在用户实时 Blender 内运行 bpy/bmesh/mathutils，小步建模/相机/材质/关键帧、保存与导出 |
| look | 视口、相机、多视角、动画抽样或实际图像 |
| search_assets / import_asset | 按来源查找并导入已授权资产，检查许可、尺度与落地位置 |
| generate_3d | 单一带纹理模型生成；可能耗积分，不是整场景/无限制生成授权 |
| disable_telemetry | 用户需要时关闭收集；不反向启用已有关闭设置 |
| record_trajectory_feedback | 用户明确接受/拒绝/纠正时记录其真实反馈，不伪造接受 |

调用 execute_blender_code 的 user_prompt 应引用用户实际任务原话，不写成代理自己的子步骤。当前任务只做技能维护时不运行场景修改。

## 编辑约定

先检查当前 .blend 是否有未保存工作和已有对象；只操作明确所属 scene/collection。新镜头新建独立集合或工程，保存新版本，禁止默认删除全场。执行后回读对象/相机/帧范围与目标路径。

- 中文界面会本地化节点名称：按 node.type 找 BSDF_PRINCIPLED 等类型，材质颜色改 shader inputs；material.diffuse_color 仅视口不足以定义渲染颜色。
- 枚举/API 随版本变：通过 RNA/当前对象检查 image_settings、格式和属性，不硬编码旧 EEVEE 标识。render.engine 的枚举读取可能不完整，赋值需处理错误并读取支持值。
- Geometry Nodes、实例化、曲线、bpy 建模和动画均可经代码接口实现，但工具暴露 bpy 不代表每个特性已有本机实测。
- 历史 safe mode 开启、telemetry 关闭，外部素材/付费生成关闭。正常 bpy 操作在历史测试中可用，不为运行未知脚本关闭这些设置。
- GLB 等导出明确当前场景/选择集范围；历史 glTF 实测采用 use_active_scene=True 避免导出别的场景，按当前 exporter 检查实际参数后使用。
- 导入/生成模型按 world bounding box 调整尺度和接地，不能仅看 local dimensions；记录素材来源与许可。

## 把 pdoom 的导演方法转成三维制作

先定义每个 shot 的 position、target、fov/focal、roll 与起止事件，再落实相机/目标控制器。可用关键帧、驱动器或按绝对帧脚本，但预览和正式输出用同一时间口径。空间文字、前景物体、遮挡和阅读面一起布置，先做灰模机位预览再加材质。

多机位可以在 marker/camera 或脚本层切换；有意硬切单独记录，不让默认插值跨切点。连续 portal/穿屏转场让前镜预先使用下一镜同一场景状态、相机/映射或渲染层；跨工具时用实际投影帧对齐。

物理/粒子/Geometry Nodes 模拟需要可重现缓存或固定步长烘焙；随机 seed、子帧、motion blur、FPS 与缓存区间一致。live scene 操作成功不等于可任意 seek 后得到相同状态。

## 输出

先低分辨率抽样多个机位与切点，再短片验速度/遮挡/文字。EEVEE/Cycles 按镜头需要选，先核对当前引擎和设备；不从“有 RTX”推断 Cycles GPU 已配置。

长片优先 PNG/EXR 序列，明确帧起止、命名、颜色管理、alpha、FPS 与 motion blur。保存原生 .blend 和必要缓存，再使用独立后台 Blender 输出，避免 live MCP 长调用阻塞：
```powershell
& '<BLENDER_BIN>' -b '<PROJECT_ROOT>/shot.blend' -o '<PROJECT_ROOT>/out/frame_####' -F PNG -s 1 -e 120 -a
```
这是使用已配置工程的示例，路径/帧数需替换；项目内先设相机、引擎、尺寸、FPS、色彩和设备。命令参数顺序有作用，-a 最后启动渲染。不要对现有文件盲目执行此示例。

保存后独立重开验证资产、相机/帧和关键设置；检查序列缺帧，再由 FFmpeg 合成用户音轨并验媒体。用 look 或本地图像读取查看实际编码/渲染输出，不能以执行无异常代替视觉验收。
