---
type: lecture
course: CME222 传质 / Mass Transfer
title: Estimation of Diffusion Coefficient (Gas Diffusivity)
lecturer: Dr. Tan Peng Chee
date: 2026-05-07
slides: 32
images_embedded: 5
---

# L03 — 气相扩散系数 D_AB 怎么估

> [!info] 这一讲在干什么
> L02 给了通量公式 $N_A = -c D_{AB}\nabla y_A + y_A(N_A+N_B)$，但**没说 $D_{AB}$ 数值是多少**。L03 给三层方法：① 查表（J.1/J.2/J.3 实验值）→ ② 不在表里就用动力学理论估算（自扩散 / 二元 / 多组分）→ ③ 极性 / 缺数据时用 Fuller / Brokaw 备胎。

---

## 学习目标

- 能根据"有 LJ 参数 / 无 LJ 参数 / 极性分子 / 多组分"四种情形选对公式
- 能跑 Hirschfelder 公式（最常用）— **关键是单位制不能错**
- 能用 P/T 外推关系把 1 atm 25 °C 的 D 改写到目标条件
- 能用 Wilke 公式把多组分体系归约到二元

---

## 决策树（写题前先看这个）

```
拿到题目 →
├─ 直接给 D_AB 实验值？           → 用 P/T 外推到目标条件（§4 末）
├─ 给的是非极性 + LJ 参数？       → Hirschfelder（§4，最准）
├─ 给的是非极性 + 没 LJ 参数？    → Fuller（§5，查扩散体积）
├─ 至少一个是极性分子？           → Brokaw（§6，加 δ 修正）
└─ A 在多组分混合气中？           → Wilke 公式（§7，归约成二元）
```

---

## 知识块 1：实验值速查（教材附录 J）

| 表 | 内容 | 用法 |
|---|---|---|
| Table J.1 | 气-气 D_AB @ 1 atm，多个温度 | 直接查 |
| Table J.2 | 气体在液体中（待 L04 用）|  |
| Table J.3 | 气-气 D_AB 自定温度（细化版）| 内插 |

> [!tip] 量级感
> 常见气体在 1 atm、25 °C 下的 $D_{AB} \sim 10^{-5}$ m²/s（轻气体如 H₂ 大一点 $\sim 10^{-4}$，重气体小一点）。算出来差太多就是单位错了。

---

## 知识块 2：自扩散（kinetic theory 起点）

> [!info] 为什么先讲自扩散
> 自扩散 = A 在自己同位素 A* 中扩散，**没有分子间相互作用力的差异**——这是最理想的"刚球碰撞"模型，所有后续公式都是从这里加修正得到的。

### 2.1 三个核心公式（必背组合）

$$
D_{AA^*} = \frac{1}{3}\lambda u
$$

| 量 | 公式 | 物理意义 |
|---|---|---|
| 平均自由程 $\lambda$ | $\lambda = \dfrac{\kappa T}{\sqrt 2 \,\pi \sigma_A^2 P}$ | 分子两次连续碰撞间走的平均距离 |
| 平均速度 $u$ | $u = \sqrt{\dfrac{8\kappa N T}{\pi M_A}}$ | Maxwell-Boltzmann 分布平均速率 |

合并：

$$
D_{AA^*} = \frac{2 T^{3/2}}{3\pi^{3/2}\sigma_A^2 P}\left(\frac{\kappa^3 N}{M_A}\right)^{1/2}
$$

| 符号 | 单位（**必须 SI**）|
|---|---|
| M_A | kg/mol |
| P | Pa |
| T | K |
| σ_A | m |
| κ | $1.38\times10^{-23}$ J/K（Boltzmann） |
| N | $6.022\times10^{23}$（Avogadro） |

### 2.2 mean free path（平均自由程）

![分子直线运动到碰撞的折线轨迹](_attachments/CME%20222%20Lecture%203/images/CME222_L03_p14_mean-free-path_diagram.jpg)

**直觉**：分子直线跑、碰撞、转向、再跑 ⋯ 两次碰撞之间的距离平均起来就是 λ。

- 单位体积分子数 ↓ → 碰撞少 → λ ↑
- σ ↓（分子小）→ 碰撞截面小 → λ ↑

### 2.3 LJ 碰撞直径 σ（什么是"分子大小"）

![两分子最近接触示意（碰撞直径 σ）](_attachments/CME%20222%20Lecture%203/images/CME222_L03_p15_collision-diameter_diagram.jpg)

**两种等价定义**：
1. 两分子无法再靠近的最小中心距
2. Lennard-Jones 势能曲线上 $U(r)=0$ 的那个 $r$

![Lennard-Jones 势能曲线（U vs r）](_attachments/CME%20222%20Lecture%203/images/CME222_L03_p15_lennard-jones-potential_curve.jpg)

> [!important] 看懂 LJ 势能曲线
> - $r \to 0$：$U \to +\infty$（强排斥，电子云重叠）
> - $r = \sigma$：$U = 0$（无相互作用势，定义为"刚球直径"）
> - $r = r_{\min} \approx 1.122\sigma$：$U = -\varepsilon$（最稳定结合，$\varepsilon$ 是势阱深度）
> - $r \to \infty$：$U \to 0$（远距离无相互作用）
>
> $\sigma$ 和 $\varepsilon$ 是分子的两个**特征参数**，附录 K 表查得到。

### 2.4 4 条隐含假设（kinetic theory 的边界）

1. 理想气体（PV = nRT）
2. 低密度（分子稀疏）
3. 刚球碰撞（无形变）
4. **弹性碰撞**——**无分子间作用力**

⚠️ 假设 4 是 Hirschfelder 要修补的——真实分子不仅有"刚球大小"还有"势阱"。

---

## 知识块 3：二元混合 D_AB（自扩散的扩展）

把 σ²、M 换成"两分子"版本：

$$
D_{AB} = \frac{2}{3}\left(\frac{\kappa}{\pi}\right)^{3/2} N^{1/2} T^{3/2} \cdot \frac{(1/2 M_A + 1/2 M_B)^{1/2}}{P\left(\dfrac{\sigma_A+\sigma_B}{2}\right)^2}
$$

**两个性质**：
- $D_{AB} = D_{BA}$（气相对称）
- $D_{AB} \propto T^{3/2} / P$（温度 3/2 次幂、压力反比）

### 3.1 P/T 外推（量级估算的杀手锏）

只要知道某条件下的 $D_{AB,1}$，外推到新条件：

$$
D_{AB,2} = D_{AB,1}\cdot \frac{P_1}{P_2}\cdot \left(\frac{T_2}{T_1}\right)^{3/2}
$$

> [!tip] 这就是为什么 Table J.1 给 1 atm 的值
> 实验只测一组就够，工程上想算别的 (P, T) 直接外推。**Hirschfelder 在这个基础上还加 Ω_D 比值**（§4.4）。

---

## 知识块 4：Hirschfelder 公式（**工程标准**）

> [!important] 这是这一讲最重要的公式
> 改进点：**考虑分子间吸引/排斥力**（用 LJ 势），多了一个无量纲修正因子 $\Omega_D$（碰撞积分）。

### 4.1 公式（注意单位制完全换了）

$$
\boxed{\,D_{AB} = \frac{0.001858\, T^{3/2}\sqrt{1/M_A + 1/M_B}}{P\,\sigma_{AB}^2\,\Omega_D}\,}
$$

| 符号 | 单位（**和 §2 不一样！混合制**）|
|---|---|
| $D_{AB}$ | **cm²/s** |
| T | K |
| M_A, M_B | **g/mol**（不是 kg/mol） |
| P | **atm**（不是 Pa） |
| σ_AB | **Å**（不是 m） |
| $\Omega_D$ | 无量纲 |

> [!warning] 单位坑（最容易失分的地方）
> 系数 0.001858 已经"吃了"单位换算——**不要乱代单位**。
> - SI 用 §2 的形式
> - 表格 / 教材引用一律用上面这个混合制
> - 给的 D 是 cm²/s 还是 m²/s？算之前先确认

### 4.2 适用范围

✅ **非极性 + 非反应** 气对（O₂、N₂、CO₂、H₂、Ar 等）
❌ 极性分子（H₂O、NH₃、SO₂）—— 用 §6 Brokaw

### 4.3 混合规则（怎么把 A、B 单参数合成 AB 双参数）

![σ_AB 算术平均 + ε_AB 几何平均](_attachments/CME%20222%20Lecture%203/images/CME222_L03_p18_mixing-rules_formula.jpg)

$$
\sigma_{AB} = \frac{\sigma_A + \sigma_B}{2},\qquad \varepsilon_{AB} = \sqrt{\varepsilon_A \cdot \varepsilon_B}
$$

直觉：σ 是"长度量"取算术平均；ε 是"能量量"取几何平均。

### 4.4 碰撞积分 $\Omega_D$（核心查图）

$\Omega_D$ 是**无量纲温度** $T^* = \kappa T / \varepsilon_{AB}$ 的函数。

![Ω_D vs κT/ε_AB 查图](_attachments/CME%20222%20Lecture%203/images/CME222_L03_p19_collision-integral_chart.jpg)

| 横坐标 $T^* = \kappa T / \varepsilon_{AB}$ | $\Omega_D$（近似） |
|---|---|
| 0.5 | 2.5+ |
| 1.0 | 1.6 |
| 2.0 | 1.0 |
| 4.0 | 0.85 |
| 10.0 | 0.74 |

> [!note] 为什么 T ↑ 时 D ↑ 加倍快
> Hirschfelder 公式有两个温度依赖：
> 1. $T^{3/2}$（来自 kinetic theory 平均速度）
> 2. $\Omega_D(T)$ 单调递减（温度越高，分子动能 ≫ 势阱深度，ε_AB 影响越小）
>
> 两者同向贡献 → D 比纯 $T^{3/2}$ 还涨得快。

### 4.5 实验数据外推（Hirschfelder 升级版）

$$
D_{AB,2} = D_{AB,1}\cdot \frac{P_1}{P_2}\cdot \left(\frac{T_2}{T_1}\right)^{3/2}\cdot \frac{\Omega_{D,1}}{\Omega_{D,2}}
$$

比 §3.1 多了 Ω_D 比值修正。

### Example 1：CO₂ 在空气中扩散 @ 20 °C, 1 atm

> 步骤：① 查 σ、ε/κ for CO₂ 和 air → ② 算 σ_AB、ε_AB/κ → ③ 算 $T^* = T/(ε_{AB}/κ)$ → ④ 查 $\Omega_D$ → ⑤ 套公式 → ⑥ 与 Table J.1 实验值对比

---

## 知识块 5：Fuller 公式（**LJ 参数查不到时用**）

$$
D_{AB} = \frac{10^{-3}\, T^{1.75}\sqrt{1/M_A + 1/M_B}}{P\left[(\Sigma v)_A^{1/3} + (\Sigma v)_B^{1/3}\right]^2}
$$

| 符号 | 单位 |
|---|---|
| $D_{AB}$ | cm²/s |
| T | K |
| P | atm |
| $\Sigma v$ | 扩散体积（查表 24.4 / 24.5） |
| M | g/mol |

### 5.1 扩散体积怎么查

| 来源 | 用法 |
|---|---|
| **Table 24.4** | 整分子扩散体积（H₂=14.3, N₂=31.2, O₂=25.6, Air=29.9, CO₂=34.0, NH₃=25.8, H₂O=18.9 等） |
| **Table 24.5** | 原子级扩散体积（如果整分子不在表里，按原子相加：C=16.5, H=1.98, O=5.48, N=5.69, F=9.7, Cl=19.5, Br=24.2, S=17.0；环结构修正：芳环 −20.2、杂环 −20.2） |

> [!warning] PPT OCR 笔误
> Table 24.4 中"$CClF_2$"（115.6）被 MinerU 识别成"$CCIF_2$"（小写 l 错认成大写 I）——查表时认准是 Cl（氯）。

### Example 2：苯（benzene）在空气中扩散 @ 30 °C, 1 atm

苯在 Table 24.4 里查不到 → 用 Table 24.5 原子相加：
$\Sigma v_{C_6 H_6} = 6\times 16.5 + 6\times 1.98 + (-20.2) = 90.7$（含芳环修正 −20.2）

---

## 知识块 6：Brokaw 公式（**极性分子修正**）

> [!info] 适用：至少一个分子有偶极矩（H₂O、NH₃、SO₂、HCl、丙酮等）

$$
\Omega_D = \Omega_{D,0} + \frac{0.196\,\delta_{AB}^2}{T^*}, \qquad \delta_{AB} = \sqrt{\delta_A \delta_B}, \qquad \delta = \frac{1.94\times 10^3 \mu_p^2}{V_b T_b}
$$

| 符号 | 含义 | 单位 |
|---|---|---|
| $\Omega_{D,0}$ | Hirschfelder 的 $\Omega_D$（按非极性算的） | 无量纲 |
| $\delta$ | 极性参数 | 无量纲 |
| $\mu_p$ | 偶极矩 | Debye |
| $V_b$ | 沸点摩尔体积 | cm³/mol |
| $T_b$ | 正常沸点 | K |

**配套 ε/κ、σ 的极性修正**：

$$
\varepsilon/\kappa = 1.18(1 + 1.3\delta^2)\,T_b, \qquad \sigma = \left(\frac{1.585\, V_b}{1+1.3\delta^2}\right)^{1/3}
$$

### 6.1 $\Omega_{D,0}$ 的拟合公式（不查图）

$$
\Omega_{D,0} = \frac{A}{(T^*)^B} + \frac{C}{e^{D T^*}} + \frac{E}{e^{F T^*}} + \frac{G}{e^{H T^*}}
$$

| A | B | C | D | E | F | G | H |
|---|---|---|---|---|---|---|---|
| 1.06036 | 0.15610 | 0.19300 | 0.47635 | 1.03587 | 1.52996 | 1.76474 | 3.89411 |

### Example 3：NH₃ 在空气中扩散 @ 100 °C, 1 atm

给定：$T_{b,NH_3} = 239.81$ K，$T_{b,Air} = 78.8$ K，$\mu_{p,NH_3} = 1.46$ Debye；NH₃ 是极性分子，Air 当非极性。

→ 算 $\delta_{NH_3}$ → $\delta_{Air} = 0$ → $\delta_{AB} = 0$（**几何平均使 air 把极性贡献"中和"了**）→ 实际上仍按 Hirschfelder + Brokaw 修正系数处理。

> [!tip] 为什么 air 一边写 0 仍要做 Brokaw
> 即便 $\delta_{AB} = 0$ 让修正项消失，**Brokaw 还修正了 σ 和 ε/κ 的算法**——比直接用 Hirschfelder 的纯算术/几何平均更准。

---

## 知识块 7：Wilke 公式（**多组分混合**）

A 在 (B, C, D, ...) 多组分混合气中的等效扩散系数：

$$
D_{1\text{-mix}} = \frac{1}{\dfrac{y_2'}{D_{1\text{-}2}} + \dfrac{y_3'}{D_{1\text{-}3}} + \cdots + \dfrac{y_n'}{D_{1\text{-}n}}}
$$

$$
y_i' = \frac{y_i}{y_2 + y_3 + \cdots + y_n} = \frac{y_i}{1 - y_1}
$$

**核心思想**：把"A 在 mix 中"拆成"A 在每个 B_i 中"的二元 D_{1-i}，然后**调和平均**（电阻并联类比）。

### Example 4：SiH₄（硅烷）在 H₂/N₂ 混合气中扩散 @ 900 K, 100 Pa

给定 $y_{SiH_4}=0.0075$, $y_{H_2}=0.015$, $y_{N_2}=0.9775$。

步骤：
1. 算 $y_{H_2}' = 0.015/(1-0.0075) = 0.0151$
2. 算 $y_{N_2}' = 0.9775/(1-0.0075) = 0.9849$
3. Hirschfelder 算 $D_{SiH_4-H_2}$、$D_{SiH_4-N_2}$（用 SiH₄ 的 LJ 参数 $\varepsilon/\kappa = 207.6$ K, $\sigma = 4.08$ Å）
4. $1/D_{SiH_4-mix} = 0.0151/D_{SiH_4-H_2} + 0.9849/D_{SiH_4-N_2}$

---

## 知识块 8：Take Home Message（公式表速查）

| 情形 | 公式 | 备注 |
|---|---|---|
| 自扩散（同位素）| $D_{AA^*} = \dfrac{1}{3}\lambda u$ | SI 单位 |
| 二元（无相互作用）| $D_{AB} = (\ldots)\dfrac{T^{3/2}}{\sigma^2 P}$ | 简单刚球 |
| **二元（有相互作用）**| Hirschfelder + $\Omega_D$ | **首选** |
| 二元（无 LJ 参数）| Fuller + 扩散体积 | 备胎 |
| 至少一极性 | Brokaw + $\delta$ 修正 | NH₃、H₂O、SO₂ 等 |
| 多组分 | Wilke 调和平均 | $y_i' = y_i/(1-y_1)$ |
| **P/T 外推** | $D_2 = D_1 \cdot (P_1/P_2)(T_2/T_1)^{3/2}\cdot\Omega_{D,1}/\Omega_{D,2}$ | 实验数据延伸 |

---

## 易错点 / 我的疑问

1. **单位混乱**：SI（kg/mol、Pa、m）vs 混合制（g/mol、atm、Å、cm²/s）—— 每次套公式前先标记一下。
2. **$\sigma$ vs $r_{\min}$**：σ 是 $U(r)=0$，$r_{\min} \approx 1.122\sigma$ 才是势阱底——不要混。
3. **$T^*$ 横坐标**：是 $\kappa T/\varepsilon$（无量纲），课本通常给 $\varepsilon/\kappa$（K），所以 $T^* = T(\text{K}) / (\varepsilon/\kappa)(\text{K})$。
4. **何时算 D_AB ≠ D_BA？** 气相 = 相等，液相 = 不等（L04 会给 Leffler-Cullinan）。
5. **Wilke 公式的物理类比**：电阻并联——是数学巧合还是物理上每对气都贡献"扩散阻力"？（自查）
6. **Brokaw 当 air 一边 δ=0**：geometric mean 让修正项消失了，那 Brokaw 还修了啥？答：σ 和 ε/κ 的算法（§6 末）。

---

## 与其它讲的连接

- **L02** 给了通量公式 $N_A = -c D_{AB}\nabla y_A + y_A\sum N_i$ —— L03 给 $D_{AB}$ 数值
- **L04** 把 L03 的方法搬到液相（Stokes-Einstein / Wilke-Chang）和固相（Arrhenius）和多孔（Knudsen）
- **L05+** 解扩散方程时直接代 D_AB —— 量级估错就全错
