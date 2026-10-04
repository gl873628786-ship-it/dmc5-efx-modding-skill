# 鬼泣5 EFX 特效文件手工编辑 — Agent Skills

把「用 010 Editor 手工编辑 DMC5（RE Engine）`.efx` 特效文件」这套方法，蒸馏成一组可被 AI agent 在真实场景下调用的技能。

**1 个路由入口 + 7 个专项能力 = 8 个可发现入口。**

---

## 这是什么

不是教程的复述，是从 22 个分 P 的教学内容里抽出的**可执行方法**。核心可迁移内核是一句话：

> **改二进制结构时如何维持数字自洽** —— 复制 → 定位后粘贴 → 同步计数或字节长度 → F5 重解析。

## 技能清单

| 文件夹 | 何时触发 |
|---|---|
| `dmc5-efx-modding-router` | 兜底入口。不确定读哪张卡时进这里；内含全部 11 张能力卡、术语表、速查表 |
| `efx-safe-structure-edit` | 增删特效组 / 属性段 / 链接 EFX；改带长度前缀的路径或骨骼名 |
| `efx-crash-troubleshoot` | 改完崩溃 / 报 `*ERROR` / 特效消失 / 改了没反应 |
| `efx-attribute-selection` | 「想让特效 X，但不知道该改哪个属性」 |
| `efx-numeric-semantics` | 填具体数值：Min/Max vs Range、弧度、加速度三段、帧单位、反直觉特殊值 |
| `efx-render-type-selection` | 选 Billboard / Polygon / Mesh / GpuBillboard；Linked 还是 Embedded |
| `efx-texture-chain` | 换贴图 / 皮肤 / 序列帧；efx → UVSequence → uvs → tex 四跳链路 |
| `efx-model-chain` | 换自制模型；FrameCount 填法；TypeMesh 的「子网格数 = 帧数」 |

> `efx-bone-parenting`、`efx-conditional-trigger`、`efx-edit-hygiene`、`efx-file-structure-model` 这 4 项未晋级为独立入口，但都能从 router 命中。

## 安装

每个文件夹都是一个独立的 Skill 包（根目录含 `SKILL.md`）。

**方式一：WorkBuddy** —— 左侧「技能」→「添加技能」→「上传技能」，把文件夹拖进去，然后**重启**（不重启扫描不到）。

**方式二：手动放置** —— 复制到 `~/.workbuddy/skills/`（用户级）或 `<项目>/.workbuddy/skills/`（项目级）。必须是 `skills/<技能名>/SKILL.md`，**不能多套一层目录**。

> 只想要知识、不想装 8 个入口？只装 `dmc5-efx-modding-router` 即可 —— 它是自包含的，11 张能力卡全文都在它的 `references/` 里。

## 质量

通过了 34 条盲测用例的触发精度测试（含 6 条「应触发同包另一个 skill」的近邻诱饵），**34/34**。

首轮 33/34 的失败点暴露了 `efx-texture-chain` 与 `efx-model-chain` 在「换完模型改材质」场景的归属歧义 —— 修的是 skill 的边界条款，不是测试。

## 来源与署名

本仓库内容**蒸馏自**以下来源，非原创教程：

- 视频：**《鬼泣5MOD制作系列教程四：特效修改基础篇》** — B 站 UP 主 **AshToAsh815**（BV1oqBeYFEb4，22 个分 P）
- 原教程文本：**thezippotm**（infernalwarks.boards.net/thread/662）

各能力卡的 `R — 原文` 段落保留了对视频画面文字的直接引用并标注了分 P 与时间点，以便回溯核对。感谢原作者的方法整理与公开分享。

**本仓库为私有仓库。** 若要转公开，请先确认上述作者的授权与署名方式。

## 边界

- 只覆盖 DMC5。其它 RE Engine 作品（RE2 / RE4 / MHR 等）素材中无证据，不外推。
- 不涉及 Unity / UE / Godot 的粒子系统。
- 不涉及 3DS Max / Maya / Blender 的建模与动画制作本身（仅在「导出 FBX 供替换」的必要范围内提及）。
- 画面中的**动作**（鼠标拖动轨迹等）无法从文字重建，相关内容已在生成时标注。
