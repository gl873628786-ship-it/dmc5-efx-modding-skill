# 能力索引（完整版）

| capability_id | 标题 | 重要度 | 意图 | 关键词 | 能力卡 |
|---|---|---|---|---|---|
| cap.dmc5-efx-modding.efx-safe-structure-edit | 安全增删 EFX 结构（计数与字节长度同步） | critical | 增删特效组/属性段/链接 EFX 后该改哪个计数；改了路径或字符串长度后错位/崩溃；不确定某个数字字段要不要跟着改 | 计数同步、Effect Count、Segment Count、pathsize、字节长度、count、size、增删特效、增删属性 | capabilities/efx-safe-structure-edit.md |
| cap.dmc5-efx-modding.efx-crash-troubleshoot | EFX 崩溃与失效排错（三条故障通道） | critical | 改完游戏崩溃 / 编辑器报 *ERROR；改完特效消失或没反应但编辑器没报错；改完进游戏看不到变化 | 崩溃、排错、ERROR、模板读取超出文件末尾、静默失败、缓存、troubleshooting、crash | capabilities/efx-crash-troubleshoot.md |
| cap.dmc5-efx-modding.efx-attribute-selection | 属性选型（需求维度 → Segment 映射） | high | 想实现某个效果但不知道该改哪个属性段；分不清 Transform3D 与 Type* 段的职责；加了动画段却没效果 | 属性选型、Segment、Transform3D、Life、Spawn、Velocity3D、EmitterShape3D、UVSequence、该改哪个 | capabilities/efx-attribute-selection.md |
| cap.dmc5-efx-modding.efx-numeric-semantics | EFX 数值语义与安全边界 | high | 某个字段该填多少、什么单位；想随机化效果，在 Min/Max 和 Range 之间犹豫；填完崩了，怀疑是数值违规 | 数值、单位、弧度、Min、Max、Range、加速度、帧、崩了、nits、ukn | capabilities/efx-numeric-semantics.md |
| cap.dmc5-efx-modding.efx-render-type-selection | 渲染类型选型（Type* / Linked vs Embedded） | high | 该用哪种渲染类型（Billboard / Polygon / Mesh / GpuBillboard）；某个轴缩放或旋转填了没反应；Linked 还是 Embedded | 渲染类型、TypeBillboard3D、TypePolygon、TypeMesh、TypeGpuBillboard、MeshEmitter、Linked、Embedded、能力边界 | capabilities/efx-render-type-selection.md |
| cap.dmc5-efx-modding.efx-texture-chain | 贴图资源链路改造（efx → UVSequence → uvs → tex） | high | 换贴图/换皮肤/换序列帧动画；换了图进游戏没变化或显示异常；资源路径改长度、打包 MOD 路径 | 贴图、换皮肤、UVSequence、uvs、tex、tex_light、pathsize、序列帧、8x8、dds、打包路径 | capabilities/efx-texture-chain.md |
| cap.dmc5-efx-modding.efx-model-chain | 模型/网格资源链路改造（TypeMesh / MeshEmitter） | medium | 替换特效的模型/网格；FrameCount 该填 1 还是子网格数；换完模型贴图还是旧的 / 粒子扎堆 | 模型、网格、TypeMesh、MeshEmitter、FrameCount、子网格、顶点动画、mdf、fbx、顶点分布 | capabilities/efx-model-chain.md |
| cap.dmc5-efx-modding.efx-bone-parenting | 骨骼跟随与父级配置（ParentOptions） | medium | 把特效绑定到某个骨骼 / 让它跟随角色；特效不跟随、或跟着角色乱翻；特效跑到关卡某个角落去了 | ParentOptions、FollowBone、骨骼跟随、绑定骨骼、Position、世界原点、骨骼名、大小写 | capabilities/efx-bone-parenting.md |
| cap.dmc5-efx-modding.efx-conditional-trigger | 条件触发与事件触发特效 | medium | 特效在游戏里不触发 / 只能在某些状态下触发；让子特效在父特效消失时出现；做碰撞触发（被打中冒火花） | 触发、ukn、Condition Block、限制条件、PtLife、PtColliderAction、Index Count、消亡触发、粘末尾 | capabilities/efx-conditional-trigger.md |
| cap.dmc5-efx-modding.efx-edit-hygiene | 编辑前纪律与环境准备 | high | 开始改 MOD 之前要注意什么；卸载 MOD 会不会把文件删掉 / 文件丢了；这些 ukn 未知字段能不能改 | 备份、natives、路径中文、ukn、未知字段、纪律、最小改动、checklist | capabilities/efx-edit-hygiene.md |
| cap.dmc5-efx-modding.efx-file-structure-model | EFX 文件结构认知模型 | high | 第一次打开 efx 文件看不懂结构；某个字段属于哪一层 / 2Eh 这类数字是什么；Segment 和 Attribute 是不是一回事 | 文件结构、EFX Header、Effect、Segment、Attribute、Id、ItemType、变量树、模板 | capabilities/efx-file-structure-model.md |
