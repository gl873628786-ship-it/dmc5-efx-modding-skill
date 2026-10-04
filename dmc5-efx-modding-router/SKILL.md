---
name: dmc5-efx-modding-router
description: |
  鬼泣5 EFX 特效编辑方法的来源路由入口。当问题落在本方法域内、但不确定该读哪张能力卡，或该能力未晋级为独立 Skill 时使用。进入后按意图路由表加载 1 张能力卡。
metadata:
  cangjie.generated-by: cangjie-tools v2.5.0
  cangjie.variant: router
  cangjie.bundle-id: bundle.dmc5-efx-modding
  cangjie.capability-count: 11
  cangjie.entrypoint-count: 8
---
# 鬼泣5 EFX 特效文件手工编辑（RE Engine / 010 Editor 模板） — 来源路由入口（compact pack）

## 触发与不触发

**适用**：与本书能力域相关的咨询与任务（见下方路由表的意图列）。
**不适用**：
- 非 RE Engine 游戏的粒子/特效系统（Unity、UE、Godot 等）。
- 其它 RE Engine 作品（RE2/RE4/MHR 等）——素材中无证据，不外推。
- 3DS Max/Maya/Blender 的建模与动画制作本身（仅在"导出 FBX 供替换"的必要范围内提及）。
- 直接编辑 .tex/.dds 图像内容本身（本工作流只处理引用路径与元数据联动）。
- 纯操作轨迹（快捷键与点击顺序）——见各能力卡的 E 段，不作为独立能力。

## 核心原则（常驻速览，概览类问题读到这里即可回答）

1. 结构增删必须同步计数与字节长度——能过 F5 无 *ERROR 才算改完。
2. 崩溃先分层：编辑器报错的看 ERROR 行号大小反推层级；编辑器干净却在游戏里异常的，走"值域非法 / 引用失效"通道。
3. 贴图是一条链（efx → UVSequence → uvs → tex），模型是另一条（TypeMesh → mdf + fbx）；改资源要顺着整条链改，不能只改一跳。
4. 渲染类型决定能力边界，选型是"功能 ↔ 代价"的交换，不要靠加数值绕过。
5. 一次只改一处，改完保存重进关卡观察——本工作流没有离线校验器，归因能力等于调试能力。
6. 字段名不等于语义；模板字段是社区反推的，ukn/不明字段只划禁区、不当已知参数使用。

## 能力路由（先读本表，按意图加载 1 张能力卡）

| 用户意图 | 先读 | 补读/备注 |
|---|---|---|
| 增删特效组/属性段/链接 EFX 后该改哪个计数；改了路径或字符串长度后错位/崩溃；不确定某个数字字段要不要跟着改 | references/capabilities/efx-safe-structure-edit.md | 已晋级为独立 Skill `efx-safe-structure-edit`（已安装时优先直接使用；本卡仅作原文与背景补充） |
| 改完游戏崩溃 / 编辑器报 *ERROR；改完特效消失或没反应但编辑器没报错；改完进游戏看不到变化 | references/capabilities/efx-crash-troubleshoot.md | 已晋级为独立 Skill `efx-crash-troubleshoot`（已安装时优先直接使用；本卡仅作原文与背景补充） |
| 想实现某个效果但不知道该改哪个属性段；分不清 Transform3D 与 Type* 段的职责；加了动画段却没效果 | references/capabilities/efx-attribute-selection.md | 已晋级为独立 Skill `efx-attribute-selection`（已安装时优先直接使用；本卡仅作原文与背景补充） |
| 某个字段该填多少、什么单位；想随机化效果，在 Min/Max 和 Range 之间犹豫；填完崩了，怀疑是数值违规 | references/capabilities/efx-numeric-semantics.md | 已晋级为独立 Skill `efx-numeric-semantics`（已安装时优先直接使用；本卡仅作原文与背景补充） |
| 该用哪种渲染类型（Billboard / Polygon / Mesh / GpuBillboard）；某个轴缩放或旋转填了没反应；Linked 还是 Embedded | references/capabilities/efx-render-type-selection.md | 已晋级为独立 Skill `efx-render-type-selection`（已安装时优先直接使用；本卡仅作原文与背景补充） |
| 换贴图/换皮肤/换序列帧动画；换了图进游戏没变化或显示异常；资源路径改长度、打包 MOD 路径 | references/capabilities/efx-texture-chain.md | 已晋级为独立 Skill `efx-texture-chain`（已安装时优先直接使用；本卡仅作原文与背景补充） |
| 替换特效的模型/网格；FrameCount 该填 1 还是子网格数；换完模型贴图还是旧的 / 粒子扎堆 | references/capabilities/efx-model-chain.md | 已晋级为独立 Skill `efx-model-chain`（已安装时优先直接使用；本卡仅作原文与背景补充） |
| 把特效绑定到某个骨骼 / 让它跟随角色；特效不跟随、或跟着角色乱翻；特效跑到关卡某个角落去了 | references/capabilities/efx-bone-parenting.md | references/capabilities/efx-conditional-trigger.md、references/capabilities/efx-attribute-selection.md、references/capabilities/efx-safe-structure-edit.md |
| 特效在游戏里不触发 / 只能在某些状态下触发；让子特效在父特效消失时出现；做碰撞触发（被打中冒火花） | references/capabilities/efx-conditional-trigger.md | references/capabilities/efx-bone-parenting.md、references/capabilities/efx-safe-structure-edit.md、references/capabilities/efx-numeric-semantics.md |
| 开始改 MOD 之前要注意什么；卸载 MOD 会不会把文件删掉 / 文件丢了；这些 ukn 未知字段能不能改 | references/capabilities/efx-edit-hygiene.md | references/capabilities/efx-crash-troubleshoot.md、references/capabilities/efx-safe-structure-edit.md、references/capabilities/efx-file-structure-model.md |
| 第一次打开 efx 文件看不懂结构；某个字段属于哪一层 / 2Eh 这类数字是什么；Segment 和 Attribute 是不是一回事 | references/capabilities/efx-file-structure-model.md | references/capabilities/efx-safe-structure-edit.md、references/capabilities/efx-crash-troubleshoot.md、references/capabilities/efx-edit-hygiene.md |

**非能力类查询**：
- 书名/作者/章节/整书概览 → references/overview.md
- 术语解释 → references/glossary.md
- 决策规则速查（不需要原文依据时） → references/cheatsheet.md
- 完整意图与关键词索引（本表未覆盖的意图先查这里） → references/capability-index.md

## 加载规则

- 每次任务先读本文件，再按路由表加载 **1** 张能力卡；任务明确跨域时最多加载 2 张。
- 概览/书名类问题不加载能力卡，用「核心原则」与 overview.md 回答。
- 路由表与 capability-index.md 都无法命中的意图，明确告知超出本书范围，不要硬套。

## 边界与判停

- 编辑器报 `*ERROR ... 模板读取超出文件末尾` → 停止手动推理，转 efx-crash-troubleshoot。
- 关键改动只能落在 ukn / 标注"不明"的字段上 → 停手，明确告知结果不可归因，不要盲改。
- 需求超出 DMC5 / 本教程覆盖范围（如其它游戏） → 明确告知超出范围，不要硬套。
