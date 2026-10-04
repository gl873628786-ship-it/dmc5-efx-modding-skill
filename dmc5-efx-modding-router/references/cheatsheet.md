# 决策规则速查 — 鬼泣5 EFX 特效文件手工编辑（RE Engine / 010 Editor 模板）

| 能力 | 一句话规则 |
|---|---|
| 安全增删 EFX 结构（计数与字节长度同步） | 增删结构后必须同步对应的 Count / size，并按 F5 重解析确认无 *ERROR 才算改完。 |
| EFX 崩溃与失效排错（三条故障通道） | 先判故障属于哪条通道——编辑器报错看行号反推层级，编辑器干净看值域与引用，视图不刷新属缓存例外。 |
| 属性选型（需求维度 → Segment 映射） | 先问"要改的是整体还是单个元素、是时长还是运动还是外观"，再落到对应 Segment；动画段必须寄生在渲染段上。 |
| EFX 数值语义与安全边界 | 填值前先认这个字段属于哪一套语义（Min/Max 区间、Range 浮动、弧度、加速度三段、帧单位），再算目标值并自查安全边界。 |
| 渲染类型选型（Type* / Linked vs Embedded） | 按三问选类型（要不要正对摄像机 / 要不要挂模型 / 是不是海量粒子），并连带接受它的限制；引用别的 efx 优先用 Linked。 |
| 贴图资源链路改造（efx → UVSequence → uvs → tex） | 贴图是一条链，逐跳改（efx 路径 → uvs 位置 → uvs 内的 tex 路径 → 彩色图还要改 tex_light）；路径变长先改 pathsize 再插 2 倍字符数字节。 |
| 模型/网格资源链路改造（TypeMesh / MeshEmitter） | 同一份几何数据语义随段而变——TypeMesh 里子网格数=帧数，MeshEmitter 里顶点分布=粒子分布；材质在 mdf 里改，不在这里。 |
| 骨骼跟随与父级配置（ParentOptions） | 十个 0/1 开关按"旋转/移动/缩放 × 三轴 + FollowBone"逐维决策；Position 置 0 = 飞到世界原点（不是关闭跟随），粒子类 FollowBone 应为 0。 |
| 条件触发与事件触发特效 | 触发分三类——状态条件靠 ukn(0/2) + Condition Block，生命周期靠 PtLife 的 ukn3 且只认 Duration，碰撞靠 PtColliderAction/PtCollision；往条件表里加东西一律粘末尾。 |
| 编辑前纪律与环境准备 | 开工前先做三件事——把文件复制出 natives 之外、确认全路径无中文、把要动的字段标为"已知 / ukn"，然后一次只改一处。 |
| EFX 文件结构认知模型 | EFX 是四层结构（Header → Name Buffer → Effect[] → Linked/Embedded EFX）；Id 是类型码不是序号；Effect Count 是文件级的、Segment Count 是每个 Effect 各自的。 |

> 速查只给结论；需要原文依据、案例或反例时读对应能力卡。
