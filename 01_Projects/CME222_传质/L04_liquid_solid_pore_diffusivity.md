---
type: lecture
course: CME222 传质 / Mass Transfer
title: Estimation of Diffusion Coefficient (Liquid / Solid / Pore)
lecturer: Dr. Tan Peng Chee
date: 2026-05-07
slides: 48
images_embedded: 11
---

# L04 — 液相 / 固相 / 多孔介质 D_AB 怎么估

> [!info] 这一讲在干什么
> L03 处理气相 D_AB（kinetic theory + Hirschfelder）。L04 把"算 D_AB"扩展到三类**非气相场景**：
> 1. **液相**（水液 / 有机液 / 电解质 / 互扩散）—— 物理机制：分子推开邻居挤进空穴（Eyring）
> 2. **固相**（金属自扩散 / 半导体掺杂）—— 物理机制：vacancy / interstitial + Arrhenius
> 3. **多孔介质 / 微孔膜**（催化剂 / 凝胶）—— Knudsen + bulk 串联 + 受限（hindered）扩散

---

## 决策树（写题前先看这个）

```
拿到题目 →
├─ 液相？
│   ├─ 稀溶液 + 普通溶剂 + 知道关联因子    → Wilke-Chang（§1.2）
│   ├─ 水溶液（专门拟合）                  → Hayduk-Laudie（§1.3）
│   ├─ 大溶质（V_A > 2 V_B）              → Scheibel（§1.4）/ Tyn-Calus（§1.5）
│   ├─ 互扩散（不是稀的，A/B 浓度都重要）   → Leffler-Cullinan（§1.6）
│   └─ 电解质（单价稀溶液）                 → Nernst（§1.7）
├─ 固相？
│   ├─ 金属自扩散 / 间隙原子                → Arrhenius D = D_o e^{-Q/RT}（§2.1）
│   └─ 半导体掺杂                           → 同上 + 查 dopant 表（§2.4）
└─ 多孔介质？
    ├─ Kn ≪ 1（孔大）                       → 纯 bulk D_AB
    ├─ Kn ≫ 1（孔小）                       → Knudsen D_KA（§3.2）
    ├─ 中间                                  → 串联：1/D = 1/D_AB + 1/D_KA（§3.3）
    └─ 溶质大小接近孔径（hindered）           → +F₁F₂ 修正（§4）
```

---

## 知识块 1：液相扩散

### 1.0 物理机制：Eyring "hole" theory（页 5）

液相不是连续介质——分子排成准晶格，**夹杂随机分布的"空穴"（hole）**，扩散就是分子跳进相邻空穴。

| ![Eyring 准晶格 + 散布空穴](_attachments/CME%20222%20Lecture%204/images/CME222_L04_p05_eyring-hole-lattice_concept.jpg) | ![分子跳进空穴 → 留下新空穴](_attachments/CME%20222%20Lecture%204/images/CME222_L04_p05_eyring-hole-jump_concept.jpg) |
|:--:|:--:|
| **Step 1**：液体的"准晶格 + 空穴"模型 | **Step 2**：邻近分子跳进空穴，原位置成新空穴 |

> [!important] 为什么液相 D 远小于气相
> - 气相：$D \sim 10^{-5}$ m²/s
> - 液相：$D \sim 10^{-9}$ m²/s（**慢 4 个数量级**）
>
> 因为液相分子要"等空穴出现"才能跳，气相分子直接自由飞——这是 Eyring 模型给的物理直觉。

### 1.1 Stokes-Einstein（页 7）— 大球溶质在小溶剂中

$$
D_{AB}^\circ = \frac{\kappa T}{6\pi\mu_B r_A}
$$

| 符号 | 含义 | 单位 |
|---|---|---|
| $\mu_B$ | 溶剂粘度 | Pa·s |
| $r_A$ | 溶质球半径 | m |
| $\kappa T$ | $kT$（Boltzmann × 温度） | J |

**物理**：把溶质 A 当成在粘性流体里运动的小球（Stokes 阻力 = 浮力来源）。
**适用**：A 比 B 大很多（蛋白质在水里）。

### 1.2 Wilke-Chang（页 8）— 最常用的稀溶液公式

$$
\boxed{\,D_{AB}^\circ = \frac{7.4\times 10^{-8}\,(\phi_B M_B)^{1/2}\,T}{\mu_B\,V_A^{0.6}}\,}
$$

| 符号 | 含义 | 单位 |
|---|---|---|
| $D_{AB}^\circ$ | A 在 B 中**无穷稀释**扩散系数 | cm²/s |
| $\phi_B$ | 溶剂关联因子（**水 = 2.6, 甲醇 = 1.9, 乙醇 = 1.5, 苯 = 1.0, 其它 = 1.0**） | 无量纲 |
| $M_B$ | 溶剂分子量 | g/mol |
| T | 温度 | K |
| $\mu_B$ | 溶剂粘度 | cP（厘泊；1 cP = $10^{-3}$ Pa·s） |
| $V_A$ | 溶质沸点摩尔体积 | cm³/mol |

> [!warning] 单位坑（液相全用 cgs 混合制）
> Wilke-Chang 给 cm²/s，**μ 用 cP 不是 Pa·s**，V_A 用 cm³/mol。系数 $7.4\times10^{-8}$ 已经吃了单位转换。

> [!warning] **液相 $D_{AB}^\circ \neq D_{BA}^\circ$**
> 这是液相和气相最关键的差别——A 在 B 中（稀溶液）和 B 在 A 中（稀溶液）算出来不一样，因为 Wilke-Chang 中 $\phi$、μ、M 都换了。**Leffler-Cullinan（§1.6）就是处理 D_{AB}（非稀）需要 D°_AB 和 D°_BA 两个独立量的根本原因**。

### 1.3 Hayduk-Laudie（页 15，PPT 误写 "Kayduk"）— 水溶液专用

> [!note] 拼写说明
> 原 PPT 写 "Kayduk"，学界标准名是 **Hayduk-Laudie (1974)**，专门拟合非电解质在水中的扩散。

$$
D_{AB}^\circ = 13.26\times 10^{-5}\,\mu_B^{-1.14}\,V_A^{-0.589}
$$

仅适用：B = 水。

### 1.4 Scheibel（页 16）— 大溶质修正

$$
D_{AB}^\circ = \frac{K\,T}{\mu_B V_A^{1/3}}
$$

K 取值：
| 条件 | K |
|---|---|
| 一般 | $8.2\times 10^{-8}\,[1+(3V_B/V_A)^{2/3}]$ |
| **苯**作溶剂 + $V_A < 2 V_B$ | $18.9\times 10^{-8}$ |
| 其它有机溶剂 + $V_A < 2.5 V_B$ | $17.5\times 10^{-8}$ |

### 1.5 Tyn-Calus（页 19，PPT 简写 "Tyne"）— 含汽化潜热修正

$$
D_{AB}^\circ = 8.93\times 10^{-8}\left(\frac{V_B}{V_A^2}\right)^{1/6}\left(\frac{P_B}{P_A}\right)^{0.6}\frac{T}{\mu_B}
$$

含 $P = (\Delta H_v)$ 修正项，对极性溶剂更准。

### 1.6 Leffler-Cullinan（**互扩散，关键**）

> [!important] 用什么场合
> 当 A 和 B 浓度**都不是稀的**（比如 50:50 混合），单组分稀溶液公式（Wilke-Chang）失效，必须用互扩散：

$$
D_{AB} = (D_{AB}^\circ)^{x_B}\cdot (D_{BA}^\circ)^{x_A}\cdot \frac{1}{\mu_{\text{mix}}}
$$

需要先用 Wilke-Chang 算两个稀溶液端点 $D_{AB}^\circ$（A 在 B 中稀）和 $D_{BA}^\circ$（B 在 A 中稀），再几何平均加粘度修正。

### 1.7 Nernst 方程（电解质稀溶液 / 单价离子）

$$
D_{AB} = \frac{2 RT}{\left(\dfrac{1}{\lambda_+^\circ} + \dfrac{1}{\lambda_-^\circ}\right) F_a^2}
$$

| 符号 | 含义 |
|---|---|
| $\lambda_\pm^\circ$ | 极限离子电导率（25 °C 水溶液中查表，单位 cm²·S/equiv，PPT 写 A/cm² 是简写） |
| $F_a$ | Faraday 常数 96485 C/mol |

> [!note] 适用范围
> 仅限**单价**（univalent）离子（NaCl、KBr）的稀水溶液。多价离子要把系数 2 换成 $(1/n_+ + 1/n_-)$。

### Example 1-3（液相扩散典型题）

| Example | 体系 | 公式选择 | 关键提示 |
|---|---|---|---|
| 1 | A 在 B 中（题给 V_A、V_B）| Wilke-Chang | μ_B 用 cP |
| 2 | 乙醇在水稀溶液 | Hayduk-Laudie | water-only 公式 |
| 3 | NaCl 水溶液 | Nernst | 25 °C 查电导率 |

---

## 知识块 2：固相扩散

### 2.0 物理机制：vacancy + interstitial（页 25-26）

固体里原子怎么"动"？两种机制：

#### Vacancy diffusion（空位扩散）

![原子跳进相邻空位](_attachments/CME%20222%20Lecture%204/images/CME222_L04_p25_vacancy-diffusion-lattice_concept.jpg)

晶格里偶尔有空位（vacancy），相邻原子跳进去 → 它原来的位置变成新空位 → 链式传递。

**典型场景**：金属自扩散、铁碳合金中 Fe 原子扩散。

#### Interstitial diffusion（间隙扩散）

| ![小原子在 lattice 间隙位置（before）](_attachments/CME%20222%20Lecture%204/images/CME222_L04_p26_interstitial-diffusion-before_concept.jpg) | ![小原子跳到相邻间隙位置（after）](_attachments/CME%20222%20Lecture%204/images/CME222_L04_p26_interstitial-diffusion-after_concept.jpg) |
|:--:|:--:|
| **before**：小原子（红）在 lattice 间隙 | **after**：跳到相邻间隙 |

**典型场景**：C/N/H 在 Fe 中扩散（小原子可塞进 lattice 间隙）。

### 2.1 Arrhenius 形式（**固相扩散的统一公式**）

$$
\boxed{\,D_{AB} = D_o\,\exp\left(-\frac{Q}{RT}\right)\,}
$$

| 符号 | 含义 | 单位 |
|---|---|---|
| $D_o$ | 频率因子（前指因子） | m²/s 或 cm²/s |
| Q | 扩散激活能（跳出 lattice 势阱的能量） | J/mol |
| R | 气体常数 8.314 J/(mol·K) |  |

物理：原子跳进空位 / 间隙需要克服势垒 Q，**温度越高跳得越频繁** → D 指数式增大。

### 2.2 Si 半导体掺杂（页 28）

![Si 中各 dopant 的 Arrhenius 直线（log D vs 1000/T）](_attachments/CME%20222%20Lecture%204/images/CME222_L04_p28_si-dopants-arrhenius_data.jpg)

> [!tip] 看图要点
> - 横坐标 **1000/T (K⁻¹)** 不是 T——温度高的在**左**
> - 纵坐标 D (cm²/s)，**对数尺度**（10⁻⁹ ~ 10⁻¹⁴）
> - 五条直线斜率 = $-Q/(R\cdot 1000)$，截距 = log D_o
> - **B、P 扩散最快**（斜率最缓）；Sb 最慢

### 2.3 BCC vs FCC 自扩散（页 30）

| ![bcc 单胞](_attachments/CME%20222%20Lecture%204/images/CME222_L04_p30_bcc-unit-cell_concept.jpg) | ![fcc 单胞](_attachments/CME%20222%20Lecture%204/images/CME222_L04_p30_fcc-unit-cell_concept.jpg) |
|:--:|:--:|
| **BCC**（体心立方）：边长 a，1+8/8 = 2 个原子 | **FCC**（面心立方）：边长 a，6/2+8/8 = 4 个原子 |

教材 Table 24.6 给两种结构的金属自扩散 D_o 和 Q（α-Fe / γ-Fe / Cu / Ni 等）。

> [!note] 为什么分 bcc/fcc
> 不同晶格结构的间隙位置数和大小不同 → 激活能 Q 不同 → D 不同。bcc（如 α-Fe）间隙较松，FCC（如 γ-Fe）原子排列更密。

---

## 知识块 3：多孔介质扩散

### 3.0 关键无量纲数：Knudsen number

$$
\text{Kn} = \frac{\lambda}{d_{\text{pore}}}
$$

| Kn | 物理 | 主导机制 |
|---|---|---|
| Kn ≪ 1 | 孔比平均自由程**大** | 分子-分子碰撞（**纯 bulk D_AB**）|
| Kn ≈ 1 | 同量级 | 串联（bulk + Knudsen）|
| Kn ≫ 1 | 孔比平均自由程**小** | 分子-孔壁碰撞（**纯 Knudsen D_KA**）|

> [!info] 为什么孔越小 Knudsen 越主导
> 孔小到分子还没碰到另一个分子就先撞墙了——这时候"扩散阻力"来自壁碰撞，与孔径有关，与对方气浓度无关。

### 3.1 四类多孔扩散对比图（**这一讲的概念骨干**）

| ![Pure molecular（Kn≪1）](_attachments/CME%20222%20Lecture%204/images/CME222_L04_p38_porous-pure-molecular_concept.jpg) | ![Pure Knudsen（Kn≫1）](_attachments/CME%20222%20Lecture%204/images/CME222_L04_p38_porous-pure-knudsen_concept.jpg) |
|:--:|:--:|
| **Pure molecular**：分子-分子碰撞主导，孔大 | **Pure Knudsen**：分子-孔壁碰撞主导，孔小 |
| ![Knudsen + molecular（Kn~1）](_attachments/CME%20222%20Lecture%204/images/CME222_L04_p38_porous-knudsen-molecular_concept.jpg) | ![Random porous（真实多孔体）](_attachments/CME%20222%20Lecture%204/images/CME222_L04_p38_porous-random_concept.jpg) |
| **Knudsen + molecular**：两种机制同量级 | **Random porous**：真实多孔（迂曲、随机） |

### 3.2 Knudsen 扩散系数

$$
D_{KA} = 4850\, d_{\text{pore}}\sqrt{\frac{T}{M_A}}
$$

| 符号 | 单位 |
|---|---|
| $D_{KA}$ | cm²/s |
| $d_{\text{pore}}$ | cm |
| T | K |
| $M_A$ | g/mol |

**物理**：分子撞壁后随机散射，运动 = 平均速度 × 孔径 → 系数 4850 来自 kinetic theory + 单位换算。

### 3.3 串联（bulk + Knudsen）：电阻类比

$$
\boxed{\,\frac{1}{D_{Ae}} = \frac{1}{D_{AB}} + \frac{1}{D_{KA}}\,}
$$

> [!important] 为什么是阻力相加
> 类比串联电阻：分子-分子碰撞和分子-壁碰撞是**两种独立的"减速机制"**——总扩散阻力（1/D）等于两个阻力之和。这是化工里"electrical analogy"的经典应用。

### 3.4 真实多孔体的 effective diffusivity

$$
D_{Ae}' = \varepsilon^2\, D_{Ae}
$$

| ε | 含义 |
|---|---|
| ε（孔隙率）| 孔体积 / 总体积 |

> [!important] 为什么是 ε² 不是 ε
> 一个 ε 来自**截面占比**（横截面只有 ε 是孔，剩下是固体）；另一个 ε 来自**路径迂曲度**（孔不直，τ ≈ 1/ε，分子要绕路）。两个 ε 相乘 → ε²。
>
> 严格写法是 $D' = \varepsilon\, D / \tau$，工程近似 $\tau \approx 1/\varepsilon$ 给出 ε²。

### Example 4：光纤 SiH₄ CVD 沉积

![光纤反应器示意图（d_pore=10μm，900 K，100 Pa）](_attachments/CME%20222%20Lecture%204/images/CME222_L04_p39_optical-fiber-cvd_diagram.jpg)

**条件**：T=900 K, P=100 Pa, d_pore=10 μm, SiH₄+He 进料。**问 Knudsen 是否重要？**

步骤：
1. 算平均自由程 $\lambda = \kappa T/(\sqrt 2\,\pi\sigma^2 P)$（理想气体公式，**P=100 Pa 是低压**，λ 会很大）
2. 算 $\text{Kn} = \lambda/d_{\text{pore}}$
3. 100 Pa 低压 + 10 μm 小孔 → 通常 Kn ≫ 1 → **Knudsen 主导**

> [!warning] 量级感是关键
> 这个题不算 D 也能定性回答——L02 的 mean free path 公式 + L04 的 Kn 判据。**先算量级再决定用哪个 D**。

---

## 知识块 4：受限扩散（Hindered Diffusion in Pores）

> [!info] 触发场景
> 溶质大小（直径 $d_s$）**接近**孔径 $d_{\text{pore}}$ —— 比如蛋白质过凝胶过滤膜。这时候溶质既被挤进孔（partition）又在孔内被壁拖慢（hydrodynamic）。

定义比例参数：

$$
\varphi = \frac{d_s}{d_{\text{pore}}}
$$

### 4.1 修正公式

$$
D_{Ae} = D_{AB}^\circ\cdot F_1\cdot F_2
$$

| 修正因子 | 公式 | 物理 |
|---|---|---|
| $F_1$ 立体配分（steric partition）| $F_1 = (1-\varphi)^2$ | 溶质中心要离壁至少 $r_s$，可用孔截面缩小 |
| $F_2$ 流体力学阻碍 | $F_2 = 1 - 2.104\varphi + 2.09\varphi^3 - 0.95\varphi^5$ | 孔壁附近流速被抑制 |

> [!warning] $F_2$ 的适用范围
> 拟合公式仅在 $0 \leq \varphi \leq 0.6$ 可靠。$\varphi > 0.6$ 时**外推不可靠**，要换更复杂关联式（Renkin / Bungay-Brenner）。

> [!note] 拼写说明（PPT 笔误）
> 原 PPT 写 "Stearic Partition Coefficient"——**正确应为 "Steric"**（空间位阻）。Stearic 是硬脂酸，与扩散无关。

### Example 5：凝胶过滤膜分离酶 A 和 B

![Gel filtration 膜分离两种酶](_attachments/CME%20222%20Lecture%204/images/CME222_L04_p43_gel-filtration-membrane_diagram.jpg)

**给定**：$d_{s,A} = 4.12$ nm, $d_{s,B} = 10.44$ nm, $d_{\text{pore}} = 30$ nm。

| 量 | A | B |
|---|---|---|
| φ | 4.12/30 = 0.137 | 10.44/30 = 0.348 |
| $F_1 = (1-\varphi)^2$ | 0.745 | 0.425 |
| $F_2$（多项式）| 0.728 | 0.388 |
| **$F_1 F_2$** | **0.542** | **0.165** |

→ 大酶 B 的有效扩散系数被压到了原来的 16.5%，小酶 A 还有 54.2%——**两者跨膜扩散速率差 3 倍多** → 这是膜分离的物理基础。

---

## 知识块 5：Take Home Message（公式速查）

### 液相

| 场景 | 公式 | 备注 |
|---|---|---|
| 大球 / 小溶剂 | Stokes-Einstein | $D = kT/(6\pi\mu r)$ |
| **常用稀溶液** | **Wilke-Chang** | 关联因子 $\phi$（水 2.6） |
| 水溶液专用 | Hayduk-Laudie | 仅 B=H₂O |
| 互扩散（非稀）| Leffler-Cullinan | 需要 $D^\circ_{AB}$ 和 $D^\circ_{BA}$ |
| 单价电解质 | Nernst | $D = 2RT/[(1/\lambda_+ + 1/\lambda_-) F^2]$ |

### 固相

| 公式 | 备注 |
|---|---|
| $D = D_o e^{-Q/RT}$ | Arrhenius，查 D_o 和 Q |
| 机制：vacancy / interstitial | 大原子 vacancy，小原子 interstitial |

### 多孔

| 场景 | 公式 |
|---|---|
| 单孔 Knudsen | $D_{KA} = 4850\, d_{\text{pore}}\sqrt{T/M_A}$ |
| 单孔混合 | $1/D_{Ae} = 1/D_{AB} + 1/D_{KA}$ |
| 真实多孔 | $D'_{Ae} = \varepsilon^2 D_{Ae}$ |
| **Hindered**（大溶质）| $D = D^\circ_{AB} F_1 F_2$（$F_1=(1-\varphi)^2$, $F_2$ 多项式） |

---

## 易错点 / 我的疑问

1. **液相 D_AB ≠ D_BA**（与气相不同）—— 何时差异显著？什么时候非要做 Leffler-Cullinan？
2. **Wilke-Chang 单位**：μ 用 cP（不是 Pa·s）—— 每次套公式前先单位检查。
3. **Si dopants 横轴是 1000/T 不是 T**——读图时温度高在左。
4. **Knudsen 系数 4850** 怎么推？kinetic theory + 单位换算（自查教材）。
5. **ε² 还是 ε/τ**：为啥工程近似 $\tau \approx 1/\varepsilon$？（粗略：孔越多，曲折越少？真实情况复杂得多）
6. **Hindered diffusion** φ > 0.6 时要换什么公式？教材给的拟合外推到底什么时候失效？
7. **Brokaw（L03）** 和 **Wilke-Chang（L04）** 的 association factor 是不是一回事？（不是——前者是分子偶极矩 δ，后者是溶剂自缔合因子 φ）
8. **Example 4 量级估算**——平均自由程 λ 在 100 Pa, 900 K 下大约是多少？（自算：σ 取空气 3.7 Å，结果 λ ≈ 几十 μm，所以 d_pore=10 μm 时 Kn 约 1-10，Knudsen 显著）

---

## 与其它讲的连接

- **L02** 给通量框架 N_A —— 不论液相 / 固相 / 多孔，都用同一框架，只是 D 不同
- **L03** 给气相 D_AB —— L04 是它的"非气相版"
- **L05+** 解扩散方程时直接代 D —— 这一讲的 11 个公式决定了具体场景用哪个
- **课程后期**会用到 effective diffusivity（多孔催化剂）和 hindered diffusion（生物膜）—— L04 是基础
