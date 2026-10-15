# GazeGLIDE project context / 项目背景

Last updated: 2026-10-15.
最后更新：2026-10-15。

This is a portable project handoff distilled from the user's background brief, subsequent discussion, and a read-only repository inspection.
本文档汇总用户提供的项目背景、后续讨论及对仓库的只读检查结果，供跨设备和跨会话接续工作。

## 1. Current phase and scope / 当前阶段与范围

- **Current phase: background alignment and documentation.** The user explicitly stopped feature implementation and then authorized creation of this document and the root `AGENTS.md`.
  **当前阶段：背景沟通与文档整理。** 用户明确叫停了功能实现，随后授权创建本文档及仓库根目录的 `AGENTS.md`。
- Working codebase: focus on `gazeglide/` and its vendored `third_party/i2rt/`; leave the sibling standalone `../i2rt/` checkout outside the current development scope. GazeGLIDE still uses the `i2rt` Python package, whose installed source should match the vendored runtime when configuring the robot environment.
  工作代码库：以 `gazeglide/` 及其内置的 `third_party/i2rt/` 为主；旁边独立的 `../i2rt/` 仓库不纳入当前开发范围。GazeGLIDE 仍使用 `i2rt` Python 包，配置机器人环境时应确保安装来源与内置运行代码一致。
- Earlier gaze implementation drafts were deleted at the user's request; no gaze feature from that work remains or has been validated.
  此前的 gaze 实现草稿已按用户要求删除；该部分没有留下已交付或已验证的 gaze 功能。
- Gaze logging, display-to-image mapping, and four-view replay remain a **proposed future milestone**, not an active implementation task.
  Gaze 记录、显示坐标到图像坐标的映射及四视角回放仍是**拟议的后续阶段**，不是当前正在执行的实现任务。
- The user has now supplied `calibration/meta/`; camera calibration and dataset metadata were inspected on 2026-10-14 (see the supplementary findings in section 7). The standalone `i2rt/` checkout and GazeGLIDE's vendored runtime were compared on 2026-10-15; neither contains the named native-gaze recording script or camera-calibration source. Headset setup remains unresolved.
  用户现已提供 `calibration/meta/`；已于 2026-10-14 检查相机标定和数据集元数据（见第 7 节补充发现）。已于 2026-10-15 比较独立 `i2rt/` 仓库和 GazeGLIDE 内置运行代码；两者均未找到元数据引用的原生 gaze 录制脚本或相机标定源代码。头显配置仍待确认。

## 2. Research goal / 研究目标

**User-stated goal:** extend GLIDE with visual perception and human gaze to help operators collect successful demonstrations for precise bimanual manipulation, initially pen capping and connector insertion.
**用户陈述的目标：** 在 GLIDE 中加入视觉感知与人类 gaze，帮助操作者为精细双臂操作采集成功示范，初始任务为盖笔帽和插接件插入。

The intended assistance combines gaze-based intent estimation, multi-view perception, and task-dependent constraints that update with the scene and manipulation phase.
预期辅助结合基于 gaze 的意图估计、多视角感知，以及随场景和操作阶段更新的任务约束。

These research directions are not evidence that the corresponding capabilities already exist in the code.
这些研究方向不代表代码已经具备相应能力。

## 3. Hardware and observations / 硬件与观测

The following describes the user's reported setup; it has not been validated against live hardware or a supplied recording.
以下内容来自用户对实际配置的描述，尚未通过现场硬件或提供的录制数据验证。

- Four views: head, left wrist, right wrist, and a small tabletop camera at approximately waist height.
  四个视角：头部、左腕、右腕，以及约腰部高度附近的小型桌面相机。
- The headset displays these views; the operator may switch gaze between them because of occlusion.
  头显显示这些视角；操作者可能因遮挡而切换注视视角。
- The user reports that camera calibration has been performed. The supplied metadata now contains intrinsics, static-camera and wrist-mount transforms, and frame labels; its overall calibration quality flag is false. Transform semantics, applicability to the current setup, and moving-camera pose computation still need verification against the original source and recordings.
  用户表示已完成相机标定。现已提供的元数据包含内参、静态相机及腕部安装变换和坐标系标签；整体标定质量标志为 false。变换含义、对当前配置的适用性及运动相机位姿计算仍需结合原始代码和录制验证。
- Depth estimates may be unreliable.
  深度估计可能不可靠。
- The exact headset model, gaze API/payload, actual display arrangement, and timestamp availability remain unresolved.
  实际头显型号、gaze API／数据格式、实际显示布局及时间戳可用性仍待确认。

## 4. Proposed gaze representation / 拟采用的 gaze 表示

- Start with 2D gaze: camera-view ID, normalized image coordinates, timestamp, validity, and confidence when available.
  先使用二维 gaze：相机视角编号、归一化图像坐标、时间戳、有效性，以及可获取的置信度。
- Map headset display coordinates into the correct source camera image, accounting for viewport placement, resizing, cropping, and any padding.
  将头显显示坐标映射到正确的源相机图像，考虑视口位置、缩放、裁剪及可能存在的留边。
- Preserve raw gaze, filtered/calibrated gaze, and inferred interaction targets separately.
  分别保存原始 gaze、滤波／标定后的 gaze，以及推断出的交互目标。
- Avoid smoothing image coordinates across different camera views.
  避免跨不同相机视角直接平滑图像坐标。
- Explicit 3D gaze reconstruction and projection into all cameras are optional later extensions.
  显式三维 gaze 重建及向全部相机投影属于可选的后续扩展。

## 5. Proposed assistance / 拟议辅助方式

A view switch may still concern the same physical interaction; eventually, preserve the selected object pair across views and combine gaze history with tracking, robot states, and operator commands.
切换视角后，操作者可能仍在关注同一个物理交互；后续希望跨视角保持选中的对象组合，并结合 gaze 历史、跟踪、机器人状态和操作者指令。

For pen capping, investigate axis alignment, lateral-drift reduction, and relative-pose stabilization; keyed connectors also require attention to rotational alignment.
对于盖笔帽，研究轴线对齐、横向偏移抑制和相对位姿稳定；对于具有方向要求的插接件，还需关注旋转对齐。

Compose reusable constraint primitives and update their parameters according to task, phase, geometry, and confidence.
组合可复用的基础约束模块，并根据任务、阶段、几何和置信度更新参数。

## 6. Future policy deployment / 后续策略部署

Human gaze is intended to be available during collection; the planned VLA policy would predict gaze during deployment.
采集阶段预期可获取人类 gaze；计划中的 VLA 策略将在部署阶段预测 gaze。

Future interfaces should be able to accept either source while retaining source identity, validity, and uncertainty information.
后续接口应能接收这两类来源，并保留来源标识、有效性及不确定性信息。

Policy training and predicted-gaze integration belong to later milestones and are not current deliverables.
策略训练及预测 gaze 的接入属于后续阶段，不是当前交付内容。

## 7. Repository inspection / 仓库检查结果

**Inspection snapshot:** 2026-10-14, branch `peiran-gaze`, HEAD `8ebb33a` (`update`); the working tree was clean immediately before these documentation files were created.
**检查快照：** 2026-10-14，分支 `peiran-gaze`，HEAD 为 `8ebb33a`（`update`）；创建这两个文档文件前工作区干净。

The fork's `origin` fetch/push URL was `https://github.com/lpr1376375365-tech/gazeglide.git`; verify the current checkout on each device rather than assuming this snapshot is still current.
该 fork 的 `origin` 拉取／推送地址为 `https://github.com/lpr1376375365-tech/gazeglide.git`；每台设备上都应核实当前仓库状态，不能假设此快照始终有效。

In the inspected workspace, `gazeglide/` was the fork's Git root and a sibling `i2rt/` was a separate checkout; no existing `AGENTS.md` was found in the fork.
检查时，`gazeglide/` 是该 fork 的 Git 根目录，旁边的 `i2rt/` 是另一个独立仓库；该 fork 内没有找到已有的 `AGENTS.md`。

### Architecture map / 架构索引

| Component / 组件 | Repository evidence / 仓库依据 |
| --- | --- |
| Public entry points / 公开入口 | [scripts/teleop.py](../scripts/teleop.py), [scripts/teleop_craft.py](../scripts/teleop_craft.py), [scripts/policy_client.py](../scripts/policy_client.py), [scripts/policy_client_craft.py](../scripts/policy_client_craft.py) |
| Launcher and camera arguments / 启动与相机参数 | [glide_runtime/launcher.py](../glide_runtime/launcher.py) |
| Two-gripper teleoperation, camera acquisition, recording / 双夹爪遥操作、相机采集与记录 | [third_party/i2rt/glide_runtime/teleop_gripper.py](../third_party/i2rt/glide_runtime/teleop_gripper.py): `RealSenseStream`, `fit_frame`, `VuerControllerTeleopCam`, `add_dataset_frame` |
| Marker and plate guardrails / 马克笔与托盘约束 | [teleop_gripper_marker_glide.py](../third_party/i2rt/glide_runtime/teleop_gripper_marker_glide.py): `PenHandoverTeleopFilter`; [teleop_gripper_plate_glide.py](../third_party/i2rt/glide_runtime/teleop_gripper_plate_glide.py): `PlateLiftTeleopFilter` |
| CRAFT teleoperation and guardrails / CRAFT 遥操作与约束 | [teleop_craft.py](../third_party/i2rt/i2rt-craft-hand/teleop_craft.py), [teleop_craft_glide.py](../third_party/i2rt/i2rt-craft-hand/teleop_craft_glide.py) |
| Guarded policy runtimes / 带约束的策略运行程序 | [policy_gripper_marker_guarded.py](../glide_runtime/policy_gripper_marker_guarded.py), [policy_gripper_plate_guarded.py](../glide_runtime/policy_gripper_plate_guarded.py), [policy_craft.py](../glide_runtime/policy_craft.py) |
| Headset rendering and controller input / 头显渲染与控制器输入 | [TeleVision.py](../third_party/i2rt/TeleVision/TeleVision.py), [teleop_dual_arm.py](../third_party/i2rt/TeleVision/teleop_dual_arm.py) |
| Hand input and receipt timestamps / 手部输入与接收时间戳 | [quest_source.py](../third_party/i2rt/i2rt-hand/helpers/quest_source.py), [types.py](../third_party/i2rt/i2rt-hand/helpers/types.py), [dual_arm_helpers.py](../third_party/i2rt/i2rt-hand/helpers/dual_arm_helpers.py) |

### Findings and boundaries / 已发现的情况与边界

- The runtime source paths inspected in the initial review support head, left-wrist, and right-wrist camera streams; no tabletop-camera integration or eye-gaze input handler was found in those paths. The subsequently supplied metadata describes four camera streams and gaze fields, but the recording script it names was not found in the inspected checkout.
  初次检查的运行程序源代码路径支持头部、左腕和右腕相机流；这些路径中未发现桌面相机接入或眼动输入处理接口。后来提供的元数据描述了四路相机流及 gaze 字段，但在已检查的仓库中未找到其引用的录制脚本。
- The base gripper preview passes the head image through centered cropping/padding and duplicates it for both eyes; Vuer then subsamples rows and uses image backgrounds with a fixed aspect value.
  基础夹爪预览对头部图像进行居中裁剪／填充，并将其复制给双眼；Vuer 随后进行行降采样，并使用固定宽高比参数的图像背景。
- The hand-teleoperation helper offers a `--vr-image-background` option defaulting to false for passthrough; the actual four-view headset UI remains to be supplied and checked.
  手部遥操作辅助程序提供 `--vr-image-background` 选项，默认关闭以使用透视；实际四视角头显界面仍需用户提供后核查。
- Quest/Vuer code and comments mentioning Quest 3 do not establish the actual headset model or available eye tracking.
  Quest／Vuer 代码及提到 Quest 3 的注释不能确认实际头显型号或眼动能力。
- `RealSenseStream` stores a host `time.monotonic()` receipt timestamp in memory, while `get_latest_frame()` returns only the image; the inspected gripper `add_dataset_frame()` does not save that camera timestamp explicitly.
  `RealSenseStream` 在内存中保存主机 `time.monotonic()` 接收时间戳，而 `get_latest_frame()` 只返回图像；已检查的夹爪 `add_dataset_frame()` 没有显式保存该相机时间戳。
- `QuestSource` uses host `time.time()` for sample/receipt times; CRAFT recording includes tracking validity, event counts, and packet ages.
  `QuestSource` 使用主机 `time.time()` 作为采样／接收时间；CRAFT 记录包含跟踪有效性、事件计数及数据包年龄。
- LeRobot recording contains images, states, actions, and teleoperation data; loop-rate recording alone does not establish synchronized camera exposure, gaze, headset presentation, or robot measurements.
  LeRobot 记录包含图像、状态、动作及遥操作数据；仅按循环频率记录不能证明相机曝光、gaze、头显显示及机器人测量已经同步。
- Vuer transforms are reshaped using column-major order, and controller transforms use `grd_yup2grd_zup`; this is not a verified camera-to-robot calibration chain.
  Vuer 变换按列优先方式重排，控制器变换使用 `grd_yup2grd_zup`；这不等于已经核实的相机到机器人标定链。
- No camera calibration assets or real episode recordings were found during the initial inspection. The subsequently supplied `calibration/` directory contains calibration and episode metadata only; actual episode tables, RGB videos, and depth recordings are still absent from that directory.
  初次检查时未找到相机标定资料或真实回合录制。后来提供的 `calibration/` 目录仅包含标定和回合元数据；该目录仍未包含实际回合数据表、RGB 视频或深度录制。
- Existing marker handover/stand guardrails are not evidence that pen-capping or connector-insertion assistance is implemented.
  已有的马克笔交接／竖放约束不代表盖笔帽或插接件插入辅助已经实现。

### Supplied calibration metadata, inspected 2026-10-14 / 新提供的标定元数据，检查于 2026-10-14

These are findings from local JSON/JSONL contents and consistency checks, not live-hardware or real-recording validation. At inspection, Git reported `calibration/` as untracked on branch `peiran-gaze`.
以下发现来自本地 JSON／JSONL 文件内容及一致性检查，不代表现场硬件或真实录制验证。检查时分支为 `peiran-gaze`，Git 显示 `calibration/` 尚未跟踪。

- [camera_calibration.json](../calibration/meta/camera_calibration.json) embeds a `charuco_calibration` bundle and a recording-time `runtime` snapshot; `created_by` names `record_dual_arm_lerobot_native_gaze.py`. It contains intrinsics (`fx`, `fy`, `cx`, `cy`, distortion), 4×4 transforms, quality checks, depth scales, and 3D-gaze configuration.
  [camera_calibration.json](../calibration/meta/camera_calibration.json) 内嵌 `charuco_calibration` 标定包及录制时的 `runtime` 快照；`created_by` 为 `record_dual_arm_lerobot_native_gaze.py`。内容包括内参（`fx`、`fy`、`cx`、`cy`、畸变）、4×4 变换、质量检查、深度比例及三维 gaze 配置。

| File-reported role / 文件记录的视角 | File-reported model / 文件记录的型号 | Serial / 序列号 | Runtime image size / 运行时图像尺寸 |
| --- | --- | --- | --- |
| Head / 头部 | Intel RealSense D435I | `346522070736` | 848×480 |
| Left wrist / 左腕 | Intel RealSense D405 | `335122270643` | 480×270 |
| Right wrist / 右腕 | Intel RealSense D405 | `335122270449` | 480×270 |
| Waist / 腰部 | Intel RealSense D415 | `013422062138` | 424×240 |

- Frame labels are `world_frame=left_base_link`, `ee_frame=link_6`, and `camera_frame=opencv_optical`, with `pixel_convention=realsense_pixel_center`. The file treats head/waist cameras as static and records `T_leftbase_cam`, wrist mounts as `T_link6_cam`, and the inter-base relation as `T_leftbase_rightbase`. This describes the stored setup; the transform direction and the live wrist-camera pose chain have not been independently verified in the generating code.
  坐标系标签为 `world_frame=left_base_link`、`ee_frame=link_6`、`camera_frame=opencv_optical`，像素约定为 `realsense_pixel_center`。文件将头部／腰部相机作为静态相机并记录 `T_leftbase_cam`，腕部安装关系为 `T_link6_cam`，双基座关系为 `T_leftbase_rightbase`。这描述的是文件保存的配置；尚未在生成代码中独立核实变换方向及腕部相机的实时位姿链。
- Both `bundle.quality_valid` and `bundle.content.quality.valid` are false. Recorded failure reasons are wrist agreement 2.024° > 2°, world-board translation drift 2.238 mm > 1 mm, and world-board rotation drift 0.9436° > 0.1°. Individual wrist-camera `quality_pass` values are true; this does not override the overall failure. The file also warns about unevaluated alternative head/waist estimates and deviations from the nominal base mounting.
  `bundle.quality_valid` 和 `bundle.content.quality.valid` 均为 false。记录的失败原因是腕部结果一致性角度 2.024° > 2°、世界标定板平移漂移 2.238 mm > 1 mm、旋转漂移 0.9436° > 0.1°。两台腕部相机单独的 `quality_pass` 为 true，但这不改变整体未通过的结果。文件还提示头部／腰部缺少其他方法估计的交叉验证，以及基座安装相对名义值的偏差。
- The embedded bundle uses 640×480 intrinsics for every camera, whereas `runtime.stream_intrinsics` uses the image sizes in the table. Runtime intrinsics and dimensions match [depth_streams.json](../calibration/meta/depth_streams.json) and the image dimensions in [info.json](../calibration/meta/info.json) for all four roles. Use parameters matched to the actual stream; do not apply the bundle's 640×480 intrinsics directly to these recorded image sizes without the appropriate mapping.
  内嵌标定包的四台相机内参均对应 640×480，而 `runtime.stream_intrinsics` 对应表中的图像尺寸。四个视角的运行时内参与尺寸均与 [depth_streams.json](../calibration/meta/depth_streams.json) 一致，尺寸也与 [info.json](../calibration/meta/info.json) 相符。使用时应匹配实际图像流；不能未经相应映射就将标定包的 640×480 内参直接用于这些录制尺寸。
- Dataset metadata declares 30 episodes, 18,064 frames, four RGB streams at 30 fps, and the task “Use both arms to cap the marker”; episode lengths sum to the declared frame count. The schema includes gaze state, active source, world gaze, per-view projections, and camera poses. Schema/configuration and aggregate statistics do not establish per-frame validity, synchronization, display mapping, or successful assistance. `display_latency_ms=0.0` is a stored setting, not a measured latency result.
  数据集元数据声明有 30 个回合、18,064 帧、四路 30 fps 的 RGB 图像，以及双臂盖马克笔帽任务；各回合长度总和与声明帧数一致。数据结构包含 gaze 状态、当前来源、世界坐标 gaze、各视角投影和相机位姿。数据结构／配置及汇总统计不能证明逐帧有效性、同步、显示映射或辅助效果；`display_latency_ms=0.0` 是保存的设置，不是实测延迟结果。

### Two local i2rt trees, compared 2026-10-15 / 两份本地 i2rt 代码，比较于 2026-10-15

This comparison uses tracked-file inventories, content hashes, source diffs, and a launcher `--print-command` check; no robot or headset was started.
本次比较使用 Git 跟踪文件清单、内容哈希、源代码差异及启动入口的 `--print-command` 检查；未启动机器人或头显。

- The sibling `../i2rt/` is a separate Git checkout: origin `https://github.com/Yuchen-Song/i2rt.git`, branch `peiran-gaze`, HEAD `eeedbbc` (`Clean up dual arm record scripts`). GazeGLIDE was on `peiran-gaze` at `4375d75` (`add camera calibration`); both working trees were clean before this documentation update. `third_party/i2rt/` is tracked as ordinary files in GazeGLIDE, with no independent Git root or submodule registration. Changes in one copy do not automatically update the other.
  旁边的 `../i2rt/` 是独立 Git 仓库：origin 为 `https://github.com/Yuchen-Song/i2rt.git`，分支为 `peiran-gaze`，HEAD 为 `eeedbbc`（`Clean up dual arm record scripts`）。GazeGLIDE 当时位于 `peiran-gaze`，HEAD 为 `4375d75`（`add camera calibration`）；此次文档更新前两者工作区均干净。`third_party/i2rt/` 由 GazeGLIDE 作为普通文件跟踪，没有独立 Git 根目录或 submodule 登记。修改其中一份不会自动更新另一份。
- The standalone tree has 99 tracked files; the vendored tree has 62. They share 31 relative paths, of which 25 are byte-identical and 6 differ; there are 68 standalone-only and 31 vendored-only paths. These counts compare paths, so relocated files count as unique paths.
  独立仓库有 99 个跟踪文件，内置版本有 62 个。共有 31 个相同相对路径，其中 25 个内容完全相同、6 个不同；独立仓库特有路径为 68 个，内置版本特有路径为 31 个。这些数字按路径统计，因此迁移位置的文件会计入各自特有路径。
- [The vendored README](../third_party/i2rt/README.md) describes a curated runtime. The standalone tree retains device setup, motor tools, examples, dataset utilities, and older recording/policy entry points. The vendored tree adds marker/plate GLIDE filters, manual task variants, CRAFT-hand runtimes and helpers, Quest hand-input helpers, and OpenTeleVision preprocessing support.
  [内置版本 README](../third_party/i2rt/README.md) 将其定义为经过整理的运行代码。独立仓库保留设备配置、电机工具、示例、数据集工具及较早的录制／策略入口。内置版本增加马克笔／托盘 GLIDE 过滤器、手工任务变体、CRAFT 手运行程序及辅助模块、Quest 手部输入辅助模块和 OpenTeleVision 预处理支持。
- Standalone `scripts/record_dual_arm_lerobot.py` and vendored `glide_runtime/teleop_gripper.py` both contain 1,880 lines and differ only in the three camera-serial defaults, replaced with placeholders in the vendored file. The shared YAM URDF/XML files, `pink_kinematics.py`, and `get_robot.py` are byte-identical.
  独立版本 `scripts/record_dual_arm_lerobot.py` 和内置版本 `glide_runtime/teleop_gripper.py` 均为 1,880 行，仅三处相机序列号默认值不同，内置版本改用了占位符。共有的 YAM URDF／XML 文件、`pink_kinematics.py` 和 `get_robot.py` 内容完全相同。
- The six changed common paths are `README.md`, `pyproject.toml`, `TeleVision/TeleVision.py`, `TeleVision/teleop_dual_arm.py`, `i2rt/motor_drivers/dm_driver.py`, and `i2rt/robots/motor_chain_robot.py`. Changes include optional input-only VR rendering, workspace clamping and virtual IK tracking in the direct dual-arm entry point, gripper-force handling, CAN retry/communication-error handling, initialization for no-gripper configurations, and expanded/changed dependencies. Do not treat the two trees as interchangeable.
  六个内容不同的共有路径为 `README.md`、`pyproject.toml`、`TeleVision/TeleVision.py`、`TeleVision/teleop_dual_arm.py`、`i2rt/motor_drivers/dm_driver.py` 和 `i2rt/robots/motor_chain_robot.py`。差异包括可选的仅输入 VR 渲染、直接双臂入口中的工作空间裁剪和虚拟 IK 跟踪、夹爪力处理、CAN 重试／通信错误处理、无夹爪配置的属性初始化，以及扩充／调整的依赖；两份代码不能直接视为可互换。
- [scripts/teleop.py](../scripts/teleop.py) selects scripts inside GazeGLIDE's `third_party/i2rt/glide_runtime/`; `--glide` selects the marker or plate filter. It also supplies `--fix-realsense-controls` and `--gripper-force-threshold 0` before advanced overrides. Low-level `import i2rt` still depends on the Python environment: both projects declare package name `i2rt` and version `0.0.1`. The documented GazeGLIDE installation uses `pip install -e "third_party/i2rt[spacemouse]"`; selecting a runtime script alone does not prove the installed driver package comes from the same tree.
  [scripts/teleop.py](../scripts/teleop.py) 选择 GazeGLIDE 内 `third_party/i2rt/glide_runtime/` 下的脚本；`--glide` 选择马克笔或托盘过滤器。它还会在高级参数覆盖之前提供 `--fix-realsense-controls` 和 `--gripper-force-threshold 0`。底层 `import i2rt` 仍由 Python 环境决定：两个项目的包名均为 `i2rt`、版本均为 `0.0.1`。GazeGLIDE 文档要求使用 `pip install -e "third_party/i2rt[spacemouse]"` 安装；只确认所选运行脚本不能证明安装的驱动包也来自同一份目录。
- The standalone recorder defaults to left wrist `335122270449` and right wrist `335122270643`, opposite to the supplied calibration metadata (left `335122270643`, right `335122270449`). This is a recorded configuration discrepancy; actual physical assignments require confirmation. Neither inspected tree contains `record_dual_arm_lerobot_native_gaze.py` or a camera-calibration implementation, and searches for gaze/waist/ChArUco calibration identifiers in Python/Markdown source returned no matches.
  独立录制脚本默认左腕为 `335122270449`、右腕为 `335122270643`，与已提供标定元数据（左腕 `335122270643`、右腕 `335122270449`）相反。这是已发现的配置差异；实际物理对应关系仍需确认。两份已检查代码均未包含 `record_dual_arm_lerobot_native_gaze.py` 或相机标定实现，在 Python／Markdown 源文件中搜索 gaze／waist／ChArUco 标定标识也未找到匹配。

## 8. Open inputs and discussion / 待补充信息与讨论

Use the supplied calibration metadata as the current file evidence; the following inputs still need confirmation from setup details, generating code, or actual recordings.
以已提供的标定元数据作为当前文件依据；以下信息仍需通过配置详情、生成代码或实际录制确认。

- Headset model, gaze access method, example raw payload, coordinate origin/units, validity/confidence fields, and timestamp clock domain.
  头显型号、gaze 获取方式、原始数据示例、坐标原点／单位、有效性／置信度字段，以及时间戳的时钟域。
- Four-view layout, per-eye presentation if applicable, source image sizes, viewport rectangles, resizing/cropping, and whether the layout changes during operation.
  四视角布局、适用时的双眼显示方式、源图像尺寸、视口矩形、缩放／裁剪规则，以及操作中布局是否变化。
- Confirm that the supplied camera intrinsics/extrinsics apply to the current setup, resolve the recorded quality failures, and verify transform directions/units, robot/base/tool conventions, and the time-varying wrist-camera pose computation in the generating code.
  确认已提供的相机内外参适用于当前配置，核查记录的质量检查失败，并在生成代码中验证变换方向／单位、机器人／基座／工具坐标约定及腕部相机随时间变化的位姿计算。
- A short recording with camera frames, gaze, available timestamps, robot states, and operator commands; identify which times are exposure, receipt, presentation, or nominal dataset times.
  包含相机帧、gaze、可用时间戳、机器人状态和操作者指令的短录制，并明确时间分别对应曝光、接收、显示还是名义数据集时间。
- Actual pen-capping/connector workflow, how the two arms are controlled, common failure points, and the assistance behavior the operator wants.
  实际盖笔帽／插接件操作流程、双臂控制方式、常见失败位置，以及操作者希望获得的辅助行为。

## 9. Candidate milestone, when requested / 用户提出实现请求后的候选阶段

The previously discussed smallest useful milestone is gaze logging, display-to-image mapping, and four-view replay with gaze overlays, runnable offline while preserving the existing robot command path.
此前讨论的最小可用阶段是 gaze 记录、显示坐标到图像坐标的映射，以及带 gaze 叠加的四视角回放；应可离线运行，并保持现有机器人指令路径不变。

If implementation is requested, validate viewport mapping, view switching, invalid samples, and available synchronization; clearly separate synthetic checks from real-data validation and report unresolved inputs.
若用户提出实现请求，应验证视口映射、视角切换、无效采样及可用同步情况；明确区分人工构造数据检查与真实数据验证，并报告未解决的输入缺失。

No such functional or hardware validation has been completed in this work.
本次工作尚未完成此类功能或硬件验证。

## 10. Continuing on another device / 在另一台设备继续

Commit and push `AGENTS.md` and this document to the intended branch, then clone/pull that branch on the other device; creating files locally alone does not synchronize them.
将 `AGENTS.md` 和本文档 commit 并 push 到目标分支后，在另一台设备克隆／拉取该分支；仅在本地创建文件不会同步。

Open the actual repository root and have the assistant read both files, summarize its understanding, and follow the current user request before proceeding.
打开实际仓库根目录，让助手阅读这两个文件、总结理解，并根据用户当前请求继续工作。

These files preserve project context, not the entire conversation or machine-local runtime setup; update them as decisions and progress change.
这两个文件保存项目背景，不包含完整聊天记录或机器本地运行配置；应随决定与进度的变化持续更新。
