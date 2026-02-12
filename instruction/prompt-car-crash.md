# Role: Three.js & Cannon-es 物理仿真专家 (Anti-Clipping Edition)

**任务目标：** 编写一个基于 Three.js 和 Cannon-es 的 3D 网页应用，模拟“玩具小车跑步机淘汰赛”。重点在于**绝对的物理稳定性**，防止小车穿模、卡死或掉出世界。

---

### 1. 物理世界的基石 (核心防穿模设置)

为解决 Cannon-es 常见的穿模问题，**必须**严格执行以下初始化设置：

*   **世界设置 (World Setup)：**
    *   `gravity`: (0, -9.82, 0)
    *   `broadphase`: 使用 `CANNON.SAPBroadphase(world)` (Sweep and Prune)，这对多物体性能更好且碰撞检测更准。
    *   **关键设置 A (精度)：** `world.solver.iterations = 20;` (默认为10，提高到20以减少误差)。
    *   **关键设置 B (容差)：** `world.defaultContactMaterial.contactEquationStiffness = 1e8;` (高刚度) 和 `contactEquationRelaxation = 3;`。
    *   **时间步长：** 在 `requestAnimationFrame` 中使用 `world.step(1/60, deltaTime, 10)`。注意第三个参数 `maxSubSteps` 设为 10，这能防止掉帧时物体穿墙。

### 2. 跑步机：视觉与物理分离 (Deep Floor Technique)

*   **视觉模型 (Visual Mesh)：**
    *   创建一个扁平的 BoxGeometry 履带，尺寸例如 (10, 0.5, 20)。
    *   添加明显的纹理或条纹，并在每一帧根据速度滚动纹理偏移量 (UV Offset)，模拟转动视觉。
*   **物理实体 (Physics Body) - 防穿模核心：**
    *   **不要**使用和视觉模型一样薄的 Box。
    *   创建一个**极厚**的 Static (或 Kinematic) Body。
    *   **形状：** `new CANNON.Box(new CANNON.Vec3(5, 50, 10))` —— 注意 Y 轴的一半高度设为 50（即总厚度100）。
    *   **位置：** 将物理体位置向下偏移，使其**上表面**刚好与视觉模型的上表面对齐。
    *   **原理：** 让履带在物理上变成一个深不见底的基座，小车绝对无法穿透 100米厚的方块。
    *   **运动逻辑：** 设置为 `Kinematic` 类型，每一帧强制设置 `.velocity.set(0, 0, speed)`。

### 3. 车辆结构 (Constraints & Initialization)

*   **多体结构：** 1个车身 Box + 4个车轮 Cylinder + 4个 HingeConstraint。
*   **物理修正 (防止卡死)：**
    *   约束中必须设置 **`collideConnected: false`**。
    *   车轮刚体的 **`angularDamping` 设为 0**。
    *   车轮材质摩擦力 `friction > 10`，弹性 `restitution = 0`。
*   **生成策略 (Drop Strategy)：**
    *   **不要**尝试计算车轮接触地面的精确 Y 值（容易导致生成时卡进地板）。
    *   **策略：** 将所有小车在 **Y = 3.0** (或其他安全高度) 生成。
    *   让小车自然**掉落**到跑步机上。
    *   为了防止落地弹跳过乱，生成后的前 1 秒内可以将重力暂时调大，或者给车身一点向下的初速度。

### 4. 场景边界与交互

*   **空气墙 (Invisible Walls)：**
    *   在跑步机的前方、左侧、右侧创建静态的隐形物理墙 (`mass: 0`)，防止小车一开始就掉下去，只留后方作为出口。
*   **淘汰逻辑：**
    *   检测 `chassisBody.position.z` > 跑步机后端边缘。
    *   标记淘汰，不移除刚体，让其自然坠落。
*   **UI：** 滑块控制速度，按钮切换视角 (包含 OrbitControls)。

### 5. 代码实现要求

*   **代码结构：** 单一 HTML 文件结构，模块化函数 (`initPhysics`, `createCar`, `animate`)。
*   **错误处理：** 确保 Three.js 和 Cannon-es 的 CDN 链接正确（使用 unpkg 或 cdnjs）。
*   **主要目标：** 优先保证小车落地后稳稳地抓地并在跑步机上保持位置，不穿模，不陷入地下。
