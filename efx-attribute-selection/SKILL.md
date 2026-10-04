---
name: efx-attribute-selection
description: |
  当用户说"我想让特效 X（去某处 / 更久 / 散开 / 变快 / 跟着走），但不知道该改哪个属性"时使用。把需求维度映射到 Segment 类型，并给出三条结构性陷阱（双缩放点、动画段必须挂渲染段、Id 的双重职能）。不适用于：已经知道改哪个段、只问值怎么填（用 efx-numeric-semantics）、要选渲染类型（用 efx-render-type-selection）、要选骨骼跟随开关（用 efx-bone-parenting）。
metadata:
  cangjie.generated-by: cangjie-tools v2.5.0
  cangjie.capability-id: cap.dmc5-efx-modding.efx-attribute-selection
  cangjie.capability-revision: 1
  cangjie.bundle-id: bundle.dmc5-efx-modding
  cangjie.source-title: 鬼泣5 EFX 特效文件手工编辑（RE Engine / 010 Editor 模板）
  cangjie.tags: decision, taxonomy, navigation, high
---
# EFX 属性选型（想改什么 → 去哪个段）

## R — 原文 (Reading)

> 变换（位置、旋转、大小）。要注意的是大小在属性 typeXxxx 中也有更改的地方，需要把 transform3D 这边大小改为 1 再去更改 typeXxxx 中的值，反过来把 typexxxx 里的大小改为 1 再去改 transform3D 中也可以。

— 视频分 P 07《属性：Transform 3D》[00:06]（画面文字 OCR）

> Rotate Anim：旋转动画，一般是要跟纹理特效或者网格特效搭配的，不然没东西可以拿来旋转。

— 视频分 P 13《属性：Rotate Anim》[00:06]（画面文字 OCR）

> Transform 3D：基础位置/旋转/缩放。注意：作用于整个特效，而非单个元素。若想作用于单个元素，需编辑特效类型段（如 TypeBillboard3D、TypeRibbonFollow）。

— thezippotm 原教程 §4

---

## I — 方法论骨架 (Interpretation)

改特效时最大的效率损失不是不会改，而是**改错了地方**：改了半天没反应，或者一改全变形。避免它的办法是先把需求归到一个**维度**，再去找管这个维度的 Segment。

一套工作映射（按需求 → 段）：

| 你想要的效果 | 去哪个段 |
|---|---|
| 活多久、淡入淡出 | Life |
| 整体在哪、整体多大、整体朝哪 | Transform3D |
| 什么时候生成、生成几个、隔多久、延迟多久 | Spawn |
| 跟着角色骨骼动 / 不跟着动 | ParentOptions |
| 往哪飞、多重、飘不飘 | Velocity3D |
| 数量很多时撒开、呈球形还是环形 | EmitterShape3D |
| 某个元素自己转、自己放大缩小、速度怎么变 | RotateAnim / ScaleAnim |
| 长什么样（朝向、形状、粒子量） | Type* 渲染段 |
| 皮是什么（贴图 / 序列帧动画） | UVSequence |
| 扭曲 | Distortion |
| 什么时候被触发（状态/生命周期/碰撞） | ukn + Condition Block / PtLife / PtCollision |

但光有映射表还不够，这套体系里有**三条会让人改错地方的结构性陷阱**，它们才是这个方法论的核心：

**陷阱一：整体与元素是两级，且缩放会相乘。**
Transform3D 管**整个特效**；Type* 段管**这个段自己的元素**。位置/旋转一般只在 Transform3D 改；但**缩放两处都有**，而且是乘法叠加。所以规则是：**先把不动的一边置 1，再改另一边**。调 ScaleAnim 之前也要先把基础大小归 1，否则动画倍率会被基准值放大。

**陷阱二：动画段不产生效果，必须寄生在渲染段上。**
RotateAnim / ScaleAnim 本身什么都不生成，它需要有"作用对象"。如果特效里没有 TypeBillboard3D / TypePolygon / TypeMesh 这类渲染段，加了动画段等于空转。反之，一个没有皮肤的网格特效，是因为缺 UVSequence 而不是缺动画。

**陷阱三：同一个段在不同语境里有不同职责。**
TypeMesh 既可以是静态网格（FrameCount=1），也可以是顶点动画（FrameCount=子网格数）；MeshEmitter 是把网格当"顶点采样器"用。判断依据不是段本身，而是**你给它配了什么**。

Id 是这套体系里的第四个线索：它既是**排序键**（建议按 ID 升序摆），又承担**冲突仲裁**（冲突时 Id 大者覆盖 Id 小者）。但要注意——作者只在一个场景（Transform3D 与 Type* 的大小）演示过"大者覆盖小者"，且他自己说"属性排序有无影响暂且不明"。所以：**把排序当纪律（低成本无害），不要当定律**。

---

## A1 — 源素材中的应用 (Past Application)

### 案例 1：缩放改了两处，尺寸失控

- **问题**：想调特效大小，在 Transform3D 和 Type* 段（如 TypeBillboard3D）里都填了缩放值。
- **方法的使用**：发现两处是乘法叠加之后，改为"把 Transform3D 那边的大小改成 1，只改 Type* 里的值"（反向也可以）。
- **结论**：不确定改哪一层时，先问"我要改的是整个特效，还是这个段里的元素"。
- **结果**：尺寸回到可控范围；这条规则后来也被用在 ScaleAnim 上——调缩放动画前先把特效大小改成 1 倍。

### 案例 2：给基础纹理特效加 ScaleAnim 却没反应

- **问题**：给一个只有 6 个属性的基础纹理特效加了 ScaleAnim，看到的只有"多了一段"。
- **方法的使用**：检查该特效的成员——它有渲染段（Type 类）与 UVSequence，所以动画有作用对象；真正的问题是没有把基础大小归 1，导致动画倍率被基准缩放污染。
- **结论**："加了动画没效果"往往不是动画段选错，而是**作用对象**或**基准值**的问题。
- **结果**：先把大小改成 1 倍后，缩放动画的表现恢复正常。

### 案例 3：想改单个元素却动了整体

- **问题**：一个特效组里有多个元素，只想让其中一个偏一点。
- **方法的使用**：意识到 Transform3D 作用于整个特效，于是改为编辑对应的类型段（TypeBillboard3D / TypeRibbonFollow 等）。
- **结论**："改整体"和"改元素"是两个不同的入口，选错了会全特效一起动。
- **结果**：只改目标元素，其余元素不受影响。

---

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill？

1. 拿着一个现成的 efx，想实现某个效果，但**不知道该动哪个属性**（"我想让它转起来""我想让它散开""我想让它跟着手走"）。
2. 想改的东西**改了半天没反应**，开始怀疑是不是改错了地方。
3. 想**去掉**某个效果（"怎么让它变慢""怎么让它不跟着动"），需要先找到管这件事的段。
4. 一个特效里有多个元素，只想动**其中一个**。
5. 加了一段新属性，但不知道它会不会和已有的段**打架**。

### 语言信号（用户的话里出现这些就应激活）

- 中文：「我想让特效……该改哪里」「该改哪个属性」「改了半天没反应」「加了这个属性怎么没效果」「怎么让它跟着角色动」「哪个属性控制大小/旋转/时长/透明度」
- English: "which attribute controls X", "where do I change the size/duration/rotation", "I added a segment but nothing changed", "how do I make it follow the character"

### 与相邻能力的区分

- 与 `efx-numeric-semantics` 的区别：本能力回答"**改哪个段**"；那个回答"**这个段里的值该怎么填**"（弧度、Min/Max、加速度）。用户说"该改哪里"找本能力，说"这个值填多少/为什么崩"找那个。
- 与 `efx-render-type-selection` 的区别：本能力是**全维度**映射（时长、位置、运动、触发…）；那个只解决"外观与渲染形态用哪种 Type"这一个维度，并且会展开各类型的代价与限制。
- 与 `efx-safe-structure-edit` 的区别：本能力解决"改哪个"，那个解决"选定之后怎么改不出错"。

---

## E — 可执行步骤 (Execution)

1. **把需求写成一句"改什么维度"的话**：不写"让它变酷"，而写"让它 2 秒后消失""让它整体放大 3 倍""让它绕 Z 轴转"。
   - 完成标准：需求里只有一个维度（时长 / 位置 / 缩放 / 旋转 / 运动 / 散布 / 外观 / 触发）。

2. **查映射表定位段**（见 I 段的表）。找不到对应项时，先确认它是不是属于"外观"（那就去 Type* 段）或"皮"（UVSequence）。
   - 完成标准：说得出目标段的名字与它的 Id（如 Life = 2Ch、UVSequence = 2Eh）。

3. **判断作用范围：整体还是元素？**
   - 整体（位置/旋转/整体大小）→ Transform3D
   - 单个元素（某个面片/某个网格自己的变换与缩放）→ 对应的 Type* 段
   - 完成标准：明确回答"我要动的是整体还是元素"。

4. **处理"两处都有"的字段（缩放）**：
   - 把**不动的那一边置 1**，只改另一边。
   - 完成标准：Transform3D 与 Type* 两处中，恰好一处 ≠ 1。

5. **检查依赖是否满足**（这一步最容易被跳过）：
   - 动画段（RotateAnim / ScaleAnim）→ 确认特效里有渲染段。
   - 有皮的段（Billboard / Polygon / Mesh）→ 确认有 UVSequence。
   - MeshEmitter → 确认渲染类型是 GpuBillboard。
   - 子特效跟随（PtLife ukn2=3）→ 确认 ParentOptions 的 FollowBone = 1。
   - 完成标准：每个新加的段都能指出它的"作用对象"或"宿主"是谁。

6. **动手改（并遵守同步与刷新）**：改动结构时同步计数，改长字符串时同步 size，改完按 F5。
   - 完成标准：模板无 `*ERROR`。

---

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill

- 已经知道改哪个段，只是不清楚**数值怎么填** → 用 `efx-numeric-semantics`。
- 问题在"用哪种渲染类型"这一个维度上 → 用 `efx-render-type-selection`。
- 想改的是**贴图/序列帧/模型**这些外部资源 → 用 `efx-texture-chain` / `efx-model-chain`。
- 模板没有解析出相关字段时 → 本映射表无能为力，说明该维度不在模板覆盖范围内。

### 源素材中警告的失败模式

- **两处缩放都不设 1** → 尺寸成倍失控（作者演示里齿轮"一下就变成 15 倍"）。
- **把动画段当成"独立效果"** → 没有宿主，加了等于没加。
- **想改元素却改了 Transform3D** → 全特效一起动，多余的元素也被带走。
- **在 colorcodes 里改颜色** → 作者明确劝退（"最好不要在 colorcodes 里改颜色"），因为 colorcodes 是跨特效组共享的映射，改它会波及其他组；正确入口是具体类型段里展开的颜色/亮度栏。
- **把 Id 大小当成"重要性"** → 只在冲突仲裁里有意义，不代表功能的强弱。

### 作者的盲点 / 局限

- 属性选型表是作者从**DMC5 这一个模板**反推的，换到 RE2/RE3/RE4/MHR 属性表会不同。
- 属性表列了 0x01–0x79 共上百个段，但教程**只展开了约 20 个**（Life、Transform3D、Spawn、ParentOptions、RotateAnim、ScaleAnim、Velocity3D、UVSequence、EmitterShape3D、Distortion、PtLife、PtCollision、各 Type* 等）。映射表未覆盖的段（如 TypeRibbon* 族、TypeNoDraw、FadeBy*、VectorField*）语义**未展开**，不要凭名字猜。
- "按 Id 排序"是纪律而非定律（见 I 段）。

### 容易混淆的邻近方法

- **Type* 段里的变换字段 vs Transform3D**：前者作用于本段元素，后者作用于整个特效；**只有缩放会相乘**，位置与旋转不叠加。
- **UVSequence vs 具体 Type 段**：Type 段决定"长什么样/朝不朝摄像机"，UVSequence 决定"皮从哪来"。两者是搭配关系而非替代关系。
- **Distortion 与 ProceduralDistortion**：前者是把一个贴图特效"转为扭曲"，后者是程序化版本（Id 77h/78h），不可混用。

---

## 相关能力

- 数值细节：`efx-numeric-semantics`
- 渲染形态：`efx-render-type-selection`
- 资源链路：`efx-texture-chain` / `efx-model-chain`
- 执行保障：`efx-safe-structure-edit`

---

## 审计信息

- **验证通过**：V1 ✓（10+ 分 P 独立讲解同一"需求→段"结构）/ V2 ✓（推导"火在角色周围散开并自转"的三段组合方案）/ V3 ✓（RE Engine 特有的维度切分、缩放相乘、动画寄生）
- **合并候选**：f05, f06, f12, f15, p22, p23, p26, p33, ce19
- **蒸馏时间**：2026-10-04
