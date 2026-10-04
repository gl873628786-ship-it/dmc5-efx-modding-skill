# EFX 文件结构认知模型

## R — 原文 (Reading)

> EFX 文件 (File) ├── EFX Header（头部）│ ├── Effect Count（特效数量）│ └── Linked EFX Count（链接/嵌入 EFX 数量）├── Name Buffer（名称缓冲区）├── Linked/Embedded EFX（链接/嵌入的特效文件）└── Effect（特效）※可多个

— 原教程 §1「文件结构层级」（thezippotm 原教程文本）

> 目前只需了解 effect count，数字 18 表示这个 efx 文件里有 18 个特效组……现在的 efx 模板比较完善，已经写出了各个特效组的名字。

— 视频分 P 02《增加、减少特效组》[00:21]–[00:42]（画面文字 OCR）

> 推荐按 Id 排序：例如 ScaleAnim 的 Id 为 2Ah、Life 的 Id 为 2Ch，应把 ScaleAnim 放在 Life 之上。

— 原教程 §7（thezippotm 原教程文本）

---

## I — 方法论骨架 (Interpretation)

把这些看不出内容的二进制文件，抽象成**四层**：

```
第 1 层  EFX Header     —— 只放数字的元数据头（Effect Count、Linked EFX Count…）
第 2 层  Name Buffer    —— 放名字字符串的缓冲区（也是粘贴 Linked/Embedded EFX 的位置）
第 3 层  Effect[]       —— 真正的效果内容：每个 Effect 里套若干 Segment
第 4 层  Linked/Embedded EFX —— 指向"另一个 efx"的外链/内嵌
外加：指向贴图/模型的外链，全在 tex / uvs / mdf / fbx 里，不在本文件内。
```

定位任何字段时，**先问"它属于哪一层"，再决定改法**（改计数 / 改字节长度 / 直接改值）。

**三个必须建立的术语区分（最容易搞错）：**

1. **`Id` 是类型码，不是序号**。同一种 Segment 在任何 Effect 里 Id 都相同（`2Eh` 永远是 UVSequence、`2Ch` 永远是 Life）。它同时承担两个职能：**排序键**（建议按 Id 升序摆放）与**冲突仲裁**（两个段冲突时 **Id 大者覆盖 Id 小者**）。
   ⚠ 但"大者覆盖小者"这一条**样本单一**（仅在 Transform3D vs TypeXxxx 大小一处演示过），**不可当普遍定理**。
2. **`Segment` / `Attribute` / `Property` 是同义词**（同一层级的不同叫法），**不是**"段 ⊃ 属性 ⊃ 属性值"的三层嵌套，也不是"颜色=红"这类键值对。它是一个**带类型 Id 的二进制结构**，内部可含多个字段。所以"删掉这个属性"的实际动作是"**删整段二进制 + Segment Count 减 1**"。
3. **`Header` 里的 count 与 `Effect` 局部的 count 属于不同层级**。`Effect Count` 是文件级的；`Segment Count` 是**每个 Effect 各自一份**的局部计数；`Override Hash Count` 只在 TypeMesh 段里；`Condition Block` 的 size 在 ConditionBlocks 里。**不要以为"所有计数都在 header"。**
   —— 这条正好解释了报错行号的差异：`*ERROR Line 2432`（Effect 级）远大于 `*ERROR Line 153`（Segment 级）。

**唯一事实来源：010 Editor 模板解析出的变量树**
不要把字段偏移、名称、类型 Id 背在脑子里。**在树里按名字展开**，看到 `ItemType_UVSequence (2Eh)` 就知道这是哪个段、Id 是多少。所有定位都"在树里找名字"，所有结论都"以树为准"。
—— 但记住：字段名是**社区反推**出来的（大量 `ukn`），"字段名 = 引擎真实语义"只是一个假设（见 `efx-edit-hygiene`）。

---

## A1 — 源素材中的应用 (Past Application)

### 案例 1：第一步就是读 Effect Count

- **问题**：第一次打开一个 efx 文件，不知从哪看起。
- **方法的使用**：打开 `pl0800`，指出 EFX Header 里的 `effect count = 18`，代表该文件含 18 个特效组（模板已能显出各特效组名字）；随后任选一个（light）操作。
- **结论**：文件结构的入口是"先读 header 计数"。
- **结果**：确立了"先看计数、再展开 Effect"的阅读顺序。

### 案例 2：报错行号反推错误层级

- **问题**：两次结构错误，报错行号量级差很多。
- **方法的使用**：把 `*ERROR Line 2432`（Effect Count 未同步）与 `*ERROR Line 153`（Segment Count 未同步）对照。
- **结论**：**行号大小 ≈ 错误所在层级**——文件级计数在文件后段报错，局部计数在 Effect 内报错。
- **结果**：形成了"按行号先判层级，再去找对应 count"的定位法。

### 案例 3：属性按 Id 排序

- **问题**：往 Effect 里加了一个 ScaleAnim 段，该放哪。
- **方法的使用**：按 Id 升序摆放——`2Ah`(ScaleAnim) 排在 `2Ch`(Life) 之前。
- **结论**：按 Id 排序与"Id 大者覆盖小者"的冲突规则配套，是既定纪律。
- **结果**：作者同时承认"排序有无影响暂且不明"，即约定与实测事实并存。

---

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill？

1. **第一次打开 efx 文件看不懂**（"这一堆数字和 `ItemType_xxx` 是什么意思"）。
2. 不确定某个东西**属于文件哪一层**（"这是 Effect 还是 Segment？"）。
3. 看到 `2Eh` / `2Ch` 这类数字，**以为是序号或数组下标**。
4. 分不清 `Segment` / `Attribute` / `Property` 是不是三个层级。
5. 以为**所有计数都在 header**，结果漏改了 Effect 局部的 `Segment Count`。
6. 不确定"该以模板变量树为准，还是该记偏移量"。

### 语言信号（用户的话里出现这些就应激活）

- 中文：「EFX 文件结构」「Header 里都是什么」「Effect Count 是什么」「Segment 和属性是一回事吗」「Id 是编号吗」「2Eh 是什么」「为什么所有计数不在 header」「模板变量树」「ItemType」
- English: "EFX file structure", "what's in the header", "Effect Count", "segment vs attribute", "is Id an index", "what is 2Eh", "template variable tree", "ItemType"

### 与相邻能力的区分

- 与 `efx-safe-structure-edit` 的区别：本能力是**认知地图**（理解层级）；那个是**动作规则**（增删后同步哪些数字）。本能力是它的前提。
- 与 `efx-crash-troubleshoot` 的区别：本能力解释"为什么报错行号能反映层级"这一原理；具体排错动作找那个。
- 与 `efx-edit-hygiene` 的区别：本能力的变量树是"事实来源"，但**字段名是反推的**这一点由 `efx-edit-hygiene` 展开为纪律。

---

## E — 可执行步骤 (Execution)

1. **先读 header 计数**：
   - 打开文件 → 展开 EFX Header → 读 `Effect Count`（= 特效组个数）。
   - 完成标准：能说出"这个文件有几个 Effect"。

2. **在变量树里展开，定位目标字段**：
   - 按名字逐层展开：Header → Name Buffer → Effect[] → Segment[]。
   - 看到 `ItemType_XXX` 时，读出它的类型码，映射到已知属性表（0x01–0x79）。
   - 完成标准：能说出该字段属于**四层中的哪一层**、是哪个 Segment 的什么。
   - 判停条件：若树里显示 `*ERROR ... 模板读取超出文件末尾` → **停止手动推理**，转 `efx-crash-troubleshoot`（计数/长度已不一致）。

3. **判定要改的是"计数 / 长度 / 值"中的哪一类**：
   - 增删结构 → 计数（注意是文件级还是 Effect 局部级）。
   - 改字符串长度 → 字节长度（pathsize、骨骼名 size、Condition Block size…）。
   - 调表现 → 直接改值（转 `efx-numeric-semantics`）。
   - 完成标准：写出"本次改的是哪一类、对应哪个声明的数字"。

4. **区分"已证"与"猜测"字段**：
   - 名称含 `ukn` 或作者标注"不明" → 归入"猜测"清单，不当作规格使用。
   - 完成标准：产出"已证 / 猜测"两栏清单。
   - 判停条件：若结论只依赖"猜测"字段 → 明确标注为未证实，不向下游传播为事实。

---

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill

- 已经知道结构、只想问"某个数值怎么填" → 用 `efx-numeric-semantics`。
- 已经知道结构、要执行增删 → 用 `efx-safe-structure-edit`。
- 编辑器报错/游戏崩溃 → 直接转 `efx-crash-troubleshoot`。

### 源素材中警告的失败模式

- **把 Id 当序号/下标** → 排序与插入位置全乱，也无法解释"冲突时 Id 大者赢"。
- **把 Segment/Attribute/Property 当三层嵌套** → 删"属性"时只改字段值而不删整段、不减 Segment Count。
- **以为所有计数都在 header** → 漏改 Effect 局部的 `Segment Count`（这是 `*ERROR Line 153` 的典型成因）。
- **把"Id 大者覆盖小者"当普遍定理** → 该规则仅一处演示，外推会误判冲突结果。
- **靠背偏移量而不看变量树** → 模板/文件版本一变就全错。
- **把变量树字段名当引擎真实语义** → `ukn` 会被误当已知知识。

### 作者的盲点 / 局限

- "Id 大者覆盖小者"**样本单一**，作者自己也在"排序是否有影响"上存疑（"暂且不明"）。
- 属性 Id 表（0x01–0x79）是**社区反推**的，**不保证完整也不保证精确**，且模板版本更新可能变化。
- 四层模型是作者对 DMC5 efx 的归纳；作者未验证其它 RE Engine 资源文件是否同构（虽在 f03 中被推及，但**本能力不作外推主张**）。

### 容易混淆的邻近方法

- **文件级 count vs Effect 局部 count**：Effect Count（文件）≠ Segment Count（每个 Effect 各一份）。
- **Id 的两种职能**：排序键 + 冲突仲裁；二者都不等于"序号"。
- **"变量树" vs "官方 schema"**：前者是反推视图，后者不存在——不要给反推字段加权威性。

---

## 相关能力

- 结构增删动作：`efx-safe-structure-edit`
- 故障分层排错：`efx-crash-troubleshoot`
- 字段可信度纪律：`efx-edit-hygiene`
- 数值语义：`efx-numeric-semantics`

---

## 审计信息

- **验证通过**：V1 ✓（结构地图在五处独立位置被反复验证：p02 Effect Count=18、逐层展开 Effect→Segment、TypeMesh 专属 Override Hash、Name Buffer 作粘贴位置、Condition Block 独立挂 Effect 内）/ V2 ✓（推导 `0x2E` 落在属性表内 → 是 Segment 类型码（UVSequence）、属"体"层第三级，增删应同步该 Effect 的 Segment Count 而非文件头）/ V3 ✓（Id 是类型码不是序号、Segment/Attribute/Property 是同义词而非三层、header count 与 Effect 局部 count 属不同层级）
- **合并候选**：f03, f20, g01, g02, g03, g04, g05, g06, g07, g08, g09
- **晋级去向**：`router`（经来源路由入口可达）
- **蒸馏时间**：2026-10-04
