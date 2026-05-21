软盘爆炸图
----------
(CC-BY-NC-SA 4.0 by karminski-牙医)


你现在是一位拥有 10 年经验的高级 WebGL / Three.js 专家，精通使用纯代码构建复杂的 3D 场景和数据可视化。

我需要你使用原生的 Three.js (无 React Three Fiber，只需纯 HTML/JS) 还原一张"3.5英寸软盘爆炸图"的工程蓝图设计。

请严格遵守以下设计规范、视觉风格和组件结构，编写完整且可运行的代码（包含在一个 HTML 文件中，通过 CDN 引入 Three.js、OrbitControls 和 CSS2DRenderer）。

### 1. 视觉风格与场景设置 (Art Direction)
*   **背景**: 浅灰白色背景 (如 `#F5F7FA`)，使用 Canvas 绘制一个点阵网格(Dot Grid)作为背景图案。
*   **相机**: 必须使用 **正交相机 (OrthographicCamera)**，position 设为 `(100, 80, 100)` 并 `lookAt(0,0,0)`，产生等距视角 (Isometric Perspective)。
*   **材质风格 (Blueprint Wireframe)**: 
    *   除磁性圆盘(Magnetic Disk)外，其他几何体**不应该**有实体填充颜色。
    *   **例外 — SHUTTER（金属滑盖，必须）**：快门本体须呈现**不透明金属板**（视觉上无透视、无蓝图幽灵半透明）；可用不透明 `MeshBasicMaterial`（如深蓝灰 `#4a5568` 或略浅于线框蓝）+ `EdgesGeometry` 描边，或用不透明面片 + 孔洞几何表达。**禁止**将快门做成半透明玻璃感或与「爆炸态半透明填充」相同的低 opacity 薄片。
    *   使用 `EdgesGeometry` 配合 `LineSegments` 描绘边缘线，线宽适中。
    *   线条颜色统一使用 `#4169E1`（工程蓝）。
    *   半透明部件使用 `MeshBasicMaterial`，**爆炸展开后的目标**为 `opacity: 0.08`，`transparent: true`，颜色为蓝白色，叠加蓝色边缘线（**不适用于快门本体**，快门在 exploded 态仍可保持不透明，仅整体随 Group 位移即可）。
*   **组装态 vs 爆炸态透明度（必须）**:
    *   **合体 / 组装 (`assembled`、`collapsing` 回到合体、以及 `exploding` 进度为 0 时)**：所有可设置透明度的 Mesh（含上述半透明填充、**含 MAGNETIC DISK 的实体材质**）在视觉上须为**不透明**：将对应 `material.opacity` 设为 **1**（`transparent` 可保持 `true` 以避免切换时重编译着色器，或按性能需求设为 `false` 直至需要淡出标注）。`LineSegments` / `Line` / `EdgesGeometry` 的线条保持原有可见度，无需因透明度规则被隐藏。
    *   **爆炸完全展开后 (`exploded`，以及 `exploding` 接近结束、`collapsing` 已离开合体)**：恢复蓝图规范中的半透明参数（填充约 **0.08**，磁片约 **0.5** 等），与文档其余章节一致。
    *   **推荐实现**：在 `parts` 或各部件 Group 上保存 `{ assembledOpacity, explodedOpacity }`（磁片与填充目标不同），在 `updateAnimation` 内与位移使用**同一 progress**（或略延迟 0～150ms 的 progress）做 **lerp**，使“变透明”与分层飞出同步；收回动画则反向插值回 1。
*   **交互 — 摄像机（OrbitControls，必须）**:
    *   使用 `OrbitControls`，并**显式**设置鼠标按键行为，保证与常见 DCC 习惯一致：
        *   **鼠标左键**：旋转视角（`THREE.MOUSE.ROTATE`）。
        *   **鼠标滚轮**：缩放 / 推拉相机（Dolly，`controls.enableZoom = true`，默认即滚轮缩放；勿把中键独占为唯一缩放方式）。
        *   **鼠标右键**：平移场景（Pan，`THREE.MOUSE.PAN`）。
    *   示例（在创建 `OrbitControls` 后设置）：
        *   `controls.mouseButtons.LEFT = THREE.MOUSE.ROTATE;`
        *   `controls.mouseButtons.MIDDLE = THREE.MOUSE.DOLLY;`（中键仍可缩放，与滚轮二选一或并存均可，但滚轮必须可用）
        *   `controls.mouseButtons.RIGHT = THREE.MOUSE.PAN;`
    *   在 `renderer.domElement` 上监听 `contextmenu` 并 `preventDefault()`，避免浏览器右键菜单遮挡平移操作。
    *   `controls.enablePan = true`，`controls.enableRotate = true`，并限制缩放与平移范围：若使用 **OrthographicCamera**，优先用 `controls.minZoom` / `controls.maxZoom`（及合理的 `zoom` 初值）限制滚轮缩放；若将来改用透视相机，则用 `minDistance` / `maxDistance`。平移范围可通过缩小 `controls.target` 的移动半径或自定义 `change` 回调钳制，避免把软盘拖出画面过多。
    *   在底部提示 UI 附近增加一行小字说明操作方式，例如：`LMB: ORBIT · WHEEL: ZOOM · RMB: PAN`（Courier New，同色半透明），不遮挡主提示即可。

### 2. 几何组件构建准则（重要！）
**每个 Shell 都不是单一 Box，而是由多个子几何体通过 Group 组合而成的复合结构。** 
请用 `THREE.Group()` 将所有子部件组合，并对每个子部件单独生成 EdgesGeometry。
对于"凹槽"效果：在主体之上叠加一个略低 Y 值的薄盒子或圆柱并描边即可（不需要真正布尔运算）。
对于"镂空"效果：用 `Shape` + `holes` + `ExtrudeGeometry` 实现真正的孔洞。

### 3. 几何组件层级 (双状态位置定义)
**每个部件必须定义两个 Y 坐标状态**：
- `assembledY`: 组装状态下的 Y 值（所有部件几乎贴合，整体厚度约 4 单位，模拟真实软盘合体外观）。
- `explodedY`: 爆炸状态下的 Y 值（自上而下间隔约 15 单位）。

参考数值（assembled → exploded）：
| 部件              | assembledY | explodedY                                  |
| ----------------- | ---------- | ------------------------------------------ |
| TOP SHELL         | 1.5        | 45                                         |
| DUST LINER (上)   | 1.0        | 30                                         |
| MAGNETIC DISK     | 0.5        | 15                                         |
| HUB               | 0.7        | 18                                         |
| DUST LINER (下)   | 0.0        | 0                                          |
| BOTTOM SHELL      | -1.5       | -15                                        |
| SHUTTER           | -1.5       | -15（同时 X/Z 偏移到下壳左前方 30 单位外） |
| WRITE PROTECT TAB | -1.0       | -15（同时 X/Z 偏移到下壳右后方外侧）       |

**初始渲染时所有部件必须位于 `assembledY`**（即看上去是一个完整的软盘）。
SHUTTER 和 WRITE PROTECT TAB 在组装状态下要嵌入到下壳对应槽位中（X/Z 位移也参与动画插值）。

#### 3.1 TOP SHELL (上壳) —— 复合结构
基础尺寸约 60 x 2 x 60。使用 `THREE.Shape` 绘制带切角的多边形外轮廓（四角倒角 3 单位），然后 ExtrudeGeometry 挤压厚度 2。
**必须包含以下子部件（全部用 EdgesGeometry 描边叠加在主体上表面）**：
- **(a) HD NOTCH**: 左上角一个 4x4 的方形镂空（用 Shape 的 holes 实现真正的洞）。
- **(b) 标签凹槽**: 上半部分中央一个大矩形（约 36 x 22），表现为下沉的浅凹槽——叠加一个薄 Box 略低于主体表面。
- **(c) 内部凹槽**: 标签凹槽下方一个较小的矩形（约 20 x 10），代表金属滑盖滑动的凹陷区。
- **(d) Shutter 开口槽（与快门窗口对位，必须）**: 在主体**前边缘**（插入驱动器朝向磁头的一侧）开**长方形贯穿镂空**，尺寸与方位须与 **3.7 SHUTTER** 上的矩形开孔、以及 **3.6 (g) 下壳开口** 在组装态下**三处共线、同宽同高**（同一磁头通道）；允许与参考工程图一致地略偏一侧，但**不得以独立随意尺寸**画成与快门孔错位。镂空须用 `Shape` + `holes` 或等价方式做出真孔，而非仅贴一条线框矩形。
- **(e) 箭头标记**: 在 Shutter 开口附近用几条 Line 画一个小三角形箭头指示方向。

#### 3.2 DUST LINER (上层防尘布)
C 字形（带切口的圆环），用 `Shape` 画外圆 + 内圆 holes，并在某处切出一个矩形缺口模拟磁头窗口，ExtrudeGeometry 极薄挤压。

#### 3.3 MAGNETIC DISK (磁片)
**唯一有实体填充的部件**。CylinderGeometry（半径约 22，高 0.3），材质为半透明蓝色 `#7b9df8`（opacity 0.5），叠加深蓝色 EdgesGeometry。可在表面用两三条短斜线 Line 表现数据轨道反光。

#### 3.4 HUB (中心金属盘)
小圆柱（半径约 5，高 0.4），位于磁片中心。中央叠加一个更小的圆形孔, 然后旁边叠加一个方形孔（这两个是驱动主轴卡口）。

#### 3.5 DUST LINER (下层防尘布)
与上层防尘布相同形状。

#### 3.6 BOTTOM SHELL (下壳) —— 复合结构
基础形状与 Top Shell 相同（带切角矩形，ExtrudeGeometry）。
**必须包含以下子部件**：
- **(a) 中央大圆形凹陷**: 一个直径约 44 的薄圆柱叠加在上表面，描边——表示磁片容纳区。
- **(b) 中央圆孔**: 在大圆凹陷正中心，一个直径约 14 的镂空孔（用 Shape holes 实现），表示主轴穿过的孔。
- **(c) 四角定位柱**: 在四个角附近各放置一个极小的圆柱（半径 1.5，高 1），表示螺丝柱/定位销。
- **(d) LIFTER**: 中下方一个小矩形凸起（约 8 x 4 x 1）。
- **(e) SHUTTER SPRING 区**: 前边缘中部一个小矩形凹槽（约 10 x 4）。
- **(f) WRITE PROTECT NOTCH**: 右后角一个小方形镂空（约 3 x 3）。
- **(g) Shutter 开口槽（与上壳、快门对位，必须）**: 前边缘与 **3.1 (d)** 相同位置与尺寸的**长方形贯穿镂空**，与 Top Shell 镜像共轴；组装态下须与滑盖上的矩形窗口对齐，形成连续磁头通道。

#### 3.7 附属小部件 (位于下壳外侧，不在垂直堆叠轴上)
*   **SHUTTER (金属滑盖) —— 几何与材质（必须严格遵守）**  
    *   **位置**：爆炸展开时位于下壳左前方约 30 单位处；组装态嵌入下壳前缘滑槽（与表格中 `assembledY` / XZ 一致）。  
    *   **不透明**：见 §1「例外 — SHUTTER」；本体为实体金属板视觉，**不做**半透明填充。  
    *   **开孔形状与方位**：在快门板面上开一个**矩形通孔**（`Shape` + `holes` + `ExtrudeGeometry` 或布尔等价实现真镂空）。  
        *   **偏心位置**：矩形开孔位于快门板面沿**滑轨长度方向的一侧**，参考真实 3.5" 软盘为**靠左**（从盘片朝前缘看，孔区偏向观察者左手侧 / 盘体该侧）；**禁止**做成居中对称大窗，除非与上下壳开孔同时居中且仍与参考图一致。  
        *   **朝向约束（与参考图一致）**：设快门板面较长的一对对边为「快门长边」（一般沿磁盘插入/滑动方向或前缘平行方向），较短对边为「快门短边」。矩形开孔的**短边必须与快门长边平行**，长边与快门短边平行（即开孔呈「横置条带」相对长条板面，或等价描述：**开孔矩形的长轴 ⟂ 快门长轴**）。  
    *   **与壳体配合**：开孔在组装态下必须与 **3.1 (d)**、**3.6 (g)** 的矩形镂空**对齐重合**（同一宽度、高度与沿前缘的位置关系），以便示意「滑开快门后磁头对准磁片」。实现时建议先固定「磁头通道」矩形尺寸与在前缘上的局部坐标，再在 Top/Bottom/Shutter 三处复用同一组参数生成孔洞。  
    *   **尺寸参考（可微调但须三处一致）**：快门整体约 **22 × 1 × 16**（厚约 1）；矩形窗约 **14 × 6** 时，应满足「6 为与快门长边平行的边、14 为与快门短边平行的边」之关系（若整体比例调整，须保持该平行关系及左偏位置）。U 形包边若与参考一致可保留，但**开孔仍为单矩形**，不要用圆角大洞替代。  
*   **WRITE PROTECT TAB (防写入滑块)**: 位于下壳右后方外侧。一个极小的方块（约 3 x 2 x 3）。

### 4. 连接线与标注
*   **垂直虚线**: 用 `LineDashedMaterial` 画 4 条垂直虚线，分别穿过主体的四个切角位置，贯穿 Top Shell → Bottom Shell（务必调用 `computeLineDistances()`）。Shutter 和 Write Protect Tab 用斜向虚线连到下壳对应位置。
*   **文字标注**: 
    *   使用 `CSS2DRenderer` 和 `CSS2DObject`。
    *   等宽字体 (Courier New)，字号 12px，颜色 `#4169E1`。
    *   每个标签用一条短 Line 作为引线连接到部件。
    *   标签：HD NOTCH, TOP SHELL, DUST LINER (×2), MAGNETIC DISK, HUB, BOTTOM SHELL, LIFTER, SHUTTER SPRING, WRITE PROTECT NOTCH, SHUTTER, WRITE PROTECT TAB。
    *   画面左侧用 HTML 绝对定位垂直排版 "FIG_001"，右侧垂直排版 "[ 3.5\" FLOPPY DISK ]"。

### 5. 输出要求
请直接输出完整的 `<!DOCTYPE html>` 代码，结构清晰、注释充分。每个 Shell 的复合构建过程用一个独立函数（如 `createTopShell()` / `createBottomShell()`）封装，返回 Group。重点是还原科技感、线框化、等距视角的蓝图美学，且 Shell 上的细节凹槽/镂空必须可见。

**快门与开孔自检（实现完成后在注释或 README 中逐条确认）**：
1. SHUTTER 本体为**不透明**金属感，非半透明幽灵面。  
2. SHUTTER 上为**矩形真镂空**，**偏左**（相对前缘/参考图），**开孔短边与快门长边平行**。  
3. TOP SHELL (d) 与 BOTTOM SHELL (g) 的矩形贯穿孔与快门孔**同一套尺寸与局部坐标**，组装态对齐。

### 6. 动画与交互时序 (Animation Sequence)

#### 6.1 状态机
定义一个全局状态变量 `animState`，可取以下值：
- `'assembled'`: 初始状态，部件合体，无标注。
- `'exploding'`: 爆炸动画进行中。
- `'exploded'`: 爆炸完成，标注淡入中或已显示。
- `'collapsing'`: 反向动画（可选，再次按空格键收回）。

#### 6.2 触发方式
- 监听 `keydown` 事件，按下 **空格键 (Space)** 时切换动画方向。
- 第一次按下：`assembled` → `exploding` → `exploded`
- 再次按下：`exploded` → `collapsing` → `assembled`（同时标注先淡出）
- 在动画进行中再次按空格应被忽略，避免状态混乱。

#### 6.3 爆炸动画细节
- **总时长**: 约 3 秒。
- **缓动函数**: 使用 easeInOutCubic 缓动（请手写实现，不依赖外部库）：
- **错峰启动 (Stagger)**: 各部件不要同时启动，从上到下依次延迟 80ms 启动，让 TOP SHELL 先飞出，BOTTOM SHELL 最后下沉，产生层次感。
- **插值方式**: 在每一帧 `requestAnimationFrame` 中根据 `elapsed` 时间计算 progress，对每个部件 Group 的 `position.y`（必要时 position.x / position.z）进行 lerp 插值。
- **透明度同步**: 与上文「组装态 vs 爆炸态透明度」一致，在同一 `updateAnimation` 内根据同一套 progress（可按部件 delay 做轻微错峰）对各 Mesh 材质的 `opacity` 做 lerp：**合体端为 1，展开端为各部件的目标透明度**；收回动画反向插值。
- 推荐架构：维护一个 `parts` 数组，每项包含 `{ group, fromPos, toPos, delay, fromOpacity, toOpacity }`（或材质引用数组），统一在 animate 循环中推进。

#### 6.4 标注与虚线的显示时序
- **初始状态**: 所有 CSS2DObject 标签、引线 Line、垂直虚线 LineDashedMaterial 均设为 `visible = false`，并把对应的 DOM 元素 opacity 设为 0、CSS `transition: opacity 0.4s ease`。
- **爆炸完成后** (状态变为 `exploded`)：
  - 延迟 200ms 后，先让 4 条垂直虚线 `visible = true` 并补间 material.opacity 从 0 到 1（用同样的 easeInOutCubic）。
  - 再延迟 200ms，逐个标签按从上到下顺序，每个间隔 60ms，将 DOM opacity 设为 1（CSS 过渡自动处理淡入）。
  - 引线 Line 跟随其对应标签同步淡入。
- **收回时**: 先把所有标签和虚线 opacity 设回 0（约 0.3s 完成），再启动收回动画。

#### 6.5 提示 UI
- 在画面底部居中放置一个 HTML 元素，文字 "PRESS [ SPACE ] TO EXPLODE"，使用 Courier New，颜色 `#4169E1`，opacity 0.6，带一个缓慢的呼吸闪烁动画（CSS `@keyframes`）。
- 当 `animState !== 'assembled'` 时，该提示淡出隐藏。

#### 6.6 代码结构建议
请将动画相关逻辑封装为：
- `setupParts()`: 收集所有可动部件及其起止位置。
- `setupControls()`: 创建 `OrbitControls`，配置 `mouseButtons`（左旋转、右平移、中键/滚轮缩放）、`minZoom`/`maxZoom`（正交相机）及平移/旋转开关。
- `triggerExplode()` / `triggerCollapse()`: 启动正/反向动画。
- `updateAnimation(deltaTime)`: 在 render loop 中调用，推进所有部件的插值（**含位置与材质 opacity**）。
- `showLabels()` / `hideLabels()`: 控制标注组的淡入淡出。

并在代码注释中标记动画相关代码段，例如 `// === ANIMATION: explode trigger ===`。
