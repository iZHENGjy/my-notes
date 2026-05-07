# CME 222 MASS TRANSFER

# Diffusive Mass Transfer

Dr. Tan Peng Chee

A4 #407

pengchee.tan@xmu.edu.my

# Table of Content

• Concentrations, Velocities   
• Fluxes & Fick’s First Law

# Concentrations, Velocities

# Concentration

Mass Concentration

$$
\rho_ {A} = \frac {m _ {A}}{V}
$$

???? = mass concentration of species A $m _ { A } = { \mathsf { m a s s } }$ of species A V = total volume of mixture

![](images/45f89dd3fc478a591183b6a2fce512f5bd14f4ec2108e2943b7140dc3642cd97.jpg)

Total Mass Concentration/Density

$$
\rho = \sum_ {i = 1} ^ {n} \rho_ {i}
$$

# Concentration

# Specific Volume (υ)

Volume occupied by a unit mass of the substance.   
Inverse of density

$$
v = \frac {V}{m} = \frac {1}{\rho}
$$

# Mass Fraction (??)

$$
w _ {A} = \frac {m _ {A}}{m} = \frac {\rho_ {A}}{\rho}
$$

$$
m _ {A} = \mathrm {m a s s o f A}
$$

$$
m = t o t a l m a s s o f m i x t u r e
$$

$$
\sum_ {i = 1} ^ {n} w _ {i} = 1
$$

Summation of mass fraction equal to 1

# Concentration

# Molar Concentration

Mole of A per unit volume of mixture

$$
c _ {A} = \frac {n _ {A}}{V} = \frac {\rho_ {A}}{M _ {A}}
$$

• For gas phase, concentration often expressed in term of pressure

$$
c _ {A} = \frac {n _ {A}}{V} = \frac {P _ {A}}{R T}
$$

$$
\sum_ {i = 1} ^ {n} c _ {i} = \mathbf {c}
$$

$$
n _ {A} = \mathrm {m o l e o f A}
$$

$$
P _ {A} = \text {p a r t i a l p r e s s u r e o f A (m o l e f r a c t i o n x t o t a l p r e s s u r e)}
$$

$$
M _ {A} = \mathrm {m o l e c u l a r w e i g h t o f A}
$$

# Concentration

Mole Fraction

For solid and liquid,

$$
x _ {A} = \frac {n _ {A}}{n} = \frac {c _ {A}}{c}
$$

For gas,

$$
y _ {A} = \frac {n _ {A}}{n} = \frac {c _ {A}}{c} = \frac {P _ {A}}{P}
$$

Summation of mole fraction equal to 1

# Concentration

How can I correlate mass fraction and mole fraction?

$$
\begin{array}{l} x _ {A} o r y _ {A} = \frac {\frac {w _ {A}}{M _ {A}}}{\frac {w _ {A}}{M _ {A}} + \frac {w _ {B}}{M _ {B}}} \\ w _ {A} = \frac {x _ {A} M _ {A}}{x _ {A} M _ {A} + x _ {B} M _ {B}} \\ \end{array}
$$

![](images/dcd486a1c891de4b1f393ad56fcb16781c5b44fb4e1ce3d05e6d1e0f500769a0.jpg)

Can be applied for ya

# Example 1

The composition of air is often given in terms of only two principal species in the gas mixture

O2, $y _ { O _ { 2 } } = 0 . 2 1$   
N2, $y _ { N _ { 2 } } = 0 . 7 9$

The molecular weight of oxygen is 0.032 kg/mol and of nitrogen is 0.028 kg/mol. Condition: 25 oC and 101300 Pa abs.

# Determine

i. Mean molecular weight of the air   
ii. Mass fraction of both oxygen and nitrogen   
iii. Molar concentration of both oxygen and nitrogen

# Velocity

• In multicomponent mixture, each species will normally move at different velocities

# Mass-Average Velocity

Defined in term of mass densities

$$
\mathbf {v} = \frac {\sum_ {i = 1} ^ {n} \rho_ {i} \mathbf {v} _ {i}}{\sum_ {i = 1} ^ {n} \rho_ {i}} = \frac {\sum_ {i = 1} ^ {n} \rho_ {i} \mathbf {v} _ {i}}{\rho} = \sum_ {i = 1} ^ {n} \omega_ {i} \mathbf {v} _ {i}
$$

# Velocity

# Molar-Average Velocity

Defined in term of molar concentration

$$
\mathbf {V} = \frac {\sum_ {i = 1} ^ {n} c _ {i} \mathbf {v} _ {i}}{\sum_ {i = 1} ^ {n} c _ {i}} = \frac {\sum_ {i = 1} ^ {n} c _ {i} \mathbf {v} _ {i}}{c} = \sum_ {i = 1} ^ {n} x _ {i} \mathbf {v} _ {i}
$$

# Velocity

# Diffusion Velocity

Velocity of a species in relative to average velocity

Diffusion velocity of species i relative to $v _ { i d = } v _ { i } - \mathrm { v }$ mass-average velocity

Diffusion velocity of species i relative to $V _ { i d = } v _ { i } - \mathsf { V }$ molar-average velocity

# Example 2

A gas mixture containing 65 mol% NH3, 8 mol% N2, 24 mol% H2 and 3 mol% Ar is flowing through a pipe 25 mm in diameter at a total pressure of 4 atm.

The velocities of the components are as follows:

$$
N H _ {3} = 0. 0 3 \mathrm {m / s}
$$

$$
N _ {2} = 0. 0 3 \mathrm {m / s}
$$

$$
H _ {2} = 0. 0 3 5 \mathrm {m / s}
$$

$$
\mathrm {A r} = 0. 0 2 \mathrm {m / s}
$$

Calculate the molar-average, mass-average velocity, and mass diffusion velocity of H2.

# Fluxes & Fick’s First Law

# Fick’s Law - Flux

• In 1855, Fick quantified the observation of diffusion   
• Flux is the amount of the particular species passes through a cross section area at a given time (can be in mass or molar unit)   
• Fick’s first law proposed molar flux for isothermal, isobaric system

$$
\mathbf {J} _ {A} = - D _ {A B} \nabla c _ {A}
$$

• For diffusion of a binary mixture of A and B in z-direction

$$
J _ {A, z} = - D _ {A B} \frac {d c _ {A}}{d z}
$$

????,?? = molar flux of component A (mol/m2.s)

?????? = diffusivity or diffusion coefficient (m2/s)

?????? ???? = concentration gradient ( $\mathtt { C _ { A } }$ in mol/m3, and z in m)

# Fick’s Law - Flux

Can be expressed in another form

$$
J _ {A, z} = - c D _ {A B} \frac {d y _ {A}}{d z}
$$

where cA = cyA

# Fick’s Law - Flux

$$
j _ {A, z} = - \rho D _ {A B} \frac {d \omega_ {A}}{d z}
$$

$$
j _ {A, z} = - D _ {A B} \frac {d \rho_ {A}}{d z}
$$

$$
j _ {A, z} = \mathrm {m a s s f l u x o f A (k g / m ^ {2} . s)}
$$

# Example 3

A mixture of He and N2 gas is contained in a pipe at 298 K and 1 atm total pressure. At one end of the pipe the partial pressure of He is 0.6 atm and the partial pressure of He is 0.2 atm at the other end (0.2m apart). Calculate the molar and mass flux of He at steady state if DHe-N2 is 0.687 x 10-4 m2/s.

# Example 4

A plate of iron is exposed to a carburizing (carbon-rich) atmosphere on one side and a decarburizing (carbon-deficient) atmosphere on the other side at 700 oC. If a condition of steady state is achieved, calculate the mass diffusion flux of carbon through the plate if the concentration of carbon at position of 5 and 10 mm beneath the carburizing surface are 100 and 66.67 mol/m3, respectively. Assume a diffusion coefficient of 3 x 10-7 cm2/s at this temperature.

# Flux

Flux is contributed by:

✓ Concentration gradient   
✓ Bulk motion

• For binary system with constant average velocity, molar diffusion flux can be expressed as

$$
J _ {A, z} = c _ {A} \left(V _ {A d, z}\right) = c _ {A} \left(v _ {A, z} - V _ {z}\right)
$$

??????,?? is the molar diffusion velocity

$v _ { A , z }$ is the velocity of component A

???? is the molar-average velocity of stream

# Flux

Diffusion velocity is measured relative to the moving fluid   
To a stationary observer, A is moving faster than the bulk of the phase since its diffusion velocity $( V _ { A d } )$ is added to that of the bulk phase (V)

$$
v _ {A} = V _ {A d} + \mathsf {V}
$$

![](images/7215c3156aacc9ca67dac0833782f8e8fce44c179e8e81b977dccc6ec1e828fb.jpg)

Multiply with cA

$$
c _ {A} v _ {A} = c _ {A} V _ {A d} + c _ {A} \mathsf {V}
$$

# Flux

Diffusion flux of A relative to moving fluid (concentration gradient contribution)

$$
c _ {A} v _ {A} = c _ {A} V _ {A d} + c _ {A} \mathsf {V}
$$

Total flux of A relative to stationary point

Convective flux of A relative to stationary point (bulk motion contribution)

Let NA = total flux = ????????

# Flux

Introduce new term total flux represented by NA

$$
J _ {A, z} = c _ {A} \left(v _ {A, z} - V _ {z}\right) = - c D _ {A B} \frac {d y _ {A}}{d z}
$$

Rearrangement,

$$
c _ {A} v _ {A, z} = - c D _ {A B} \frac {d y _ {A}}{d z} + c _ {A} V _ {z}
$$

Molar-average velocity,

$$
V _ {z} = \frac {1}{c} \big (c _ {A} v _ {A, z} + c _ {B} v _ {B, z} \big)
$$

$$
c _ {A} V _ {z} = y _ {A} \big (c _ {A} v _ {A, z} + c _ {B} v _ {B, z} \big)
$$

# Flux

![](images/a0a3499c06318d3c30849781b2d9b3e8c0d6617b62eb676c8713b55c123cac51.jpg)

$$
c _ {A} v _ {A, z} = - c D _ {A B} \frac {d y _ {A}}{d z} + y _ {A} \big (c _ {A} v _ {A, z} + c _ {B} v _ {B, z} \big)
$$

$$
\mathsf {N} _ {\mathsf {A}} = c _ {A} v _ {A}
$$

$$
\mathsf {N} _ {\mathtt {B}} = c _ {B} v _ {B}
$$

$$
N _ {A, z} = - c D _ {A B} \frac {d y _ {A}}{d z} + y _ {A} \big (N _ {A, z} + N _ {B, z} \big)
$$

This relation may be generalized and written in vector form

$$
\boldsymbol {N} _ {\boldsymbol {A}} = - c D _ {A B} \nabla \boldsymbol {y} _ {\boldsymbol {A}} + y _ {\boldsymbol {A}} \left(\boldsymbol {N} _ {\boldsymbol {A}} + \boldsymbol {N} _ {\boldsymbol {B}}\right)
$$

If species A were diffusing in a multicomponent mixture,

$$
\pmb {N} _ {\pmb {A}} = - c D _ {A M} \pmb {\nabla} \pmb {y} _ {\pmb {A}} + y _ {A} \sum_ {i = 1} ^ {n} \pmb {N} _ {i}
$$

where $D _ { A M }$ is the diffusivity of A in the mixture

# Flux

Expression for mass flux

$$
\pmb {n} _ {A} = - \rho D _ {A B} \nabla \pmb {w} _ {A} + w _ {A} (\pmb {n} _ {A} + \pmb {n} _ {B})
$$

where

$$
n _ {A} = \rho_ {A} v _ {A}
$$

$$
n _ {B} = \rho_ {B} v _ {B}
$$

# Flux

<table><tr><td>Flux</td><td>Gradient</td><td>Fick rate equation</td></tr><tr><td rowspan="2">nA</td><td>∇ωA</td><td>nA = -ρDAB∇ωA + ωA(nA + nB)</td></tr><tr><td>∇ρA</td><td>nA = -DAB∇ρA + ωA(nA + nB)</td></tr><tr><td rowspan="2">NA</td><td>∇yA</td><td>NA = -cDAB∇yA + yA(NA + NB)</td></tr><tr><td>∇cA</td><td>NA = -DAB∇cA + yA(NA + NB)</td></tr></table>

Consider a dye-filled balloon dropped into the moving stream

• Dye diffuse radially while being carried downstream   
• Two contribution in mass transfer

# Take Home Message

• Mass concentration vs Molar concentration   
Diffusion velocity (Mass/molar basis)

$$
v _ {i d} = v _ {i} - \mathbf {v}
$$

Fick’s first law

$$
J _ {A, z} = - D _ {A B} \frac {d c _ {A}}{d z}
$$

Diffusion flux and convective flux

$$
c _ {A} v _ {A} = c _ {A} V _ {a d} + c _ {A} \mathsf {V}
$$

$$
\boldsymbol {N} _ {\boldsymbol {A}} = - c D _ {A B} \nabla \boldsymbol {y} _ {\boldsymbol {A}} + y _ {A} (\boldsymbol {N} _ {\boldsymbol {A}} + \boldsymbol {N} _ {\boldsymbol {B}})
$$