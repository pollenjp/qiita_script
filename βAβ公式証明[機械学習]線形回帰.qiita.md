# β^t Aβ公式証明[機械学習]線形回帰

```math
証明する公式:
\frac{\partial({}^t \boldsymbol{\beta A \beta})}{\partial(\boldsymbol{\beta})} = (\boldsymbol{A} + {}^t \boldsymbol{A}) \boldsymbol{\beta} \\
\\
A \in \boldsymbol{R}^{(1+p) \times (1+p)}, \beta \in \boldsymbol{R}^{(1+p)}
```

## 証明

```math
\begin{align}
\frac{\partial({}^t \boldsymbol{\beta A \beta})}{\partial(\boldsymbol{\beta})}

&= 
\left[
\begin{array}{c}
\frac{\partial({}^t \boldsymbol{\beta A \beta})}{\partial(\beta_1)}\\
\vdots \\
\frac{\partial({}^t \boldsymbol{\beta A \beta})}{\partial(\beta_i)} \\
\vdots \\
\frac{\partial({}^t \boldsymbol{\beta A \beta})}{\partial(\beta_{(1+p)})}
\end{array}
\right] 
\\

&=
\left[
\frac{\partial({}^t \boldsymbol{\beta A \beta})}{\partial(\beta_i)} 
\right]_i 
\tag{略記}

\end{align}

```

```math
\begin{align}
\left[
\frac{\partial \left(
{}^t \boldsymbol{\beta A \beta}
\right)}{\partial(\beta_i)} 
\right]_i 

&=
\left[
\frac{\partial \left(
\sum_{j=0}^{1+p} {}^t \boldsymbol{\beta} \boldsymbol{a}^{(j)} \beta_j
\right)}{\partial(\beta_i)} 
\right]_i 
\left(
\boldsymbol{a}^{(j)}は行列\boldsymbol{A}のj列目ベクトル
\right)
\\

&=
\left[
\frac{\partial \left(
\sum_{j=0}^{1+p} \left(
\sum_{k=0}^{1+p} \beta_{k} a_{k, j}
\right) \ \beta_j
\right)}{\partial(\beta_i)} 
\right]_i
\\

&=
\left[
\left(
\sum_{j=0}^{1+p} a_{i, j} \ \beta_j
\right)
+
\left(
\sum_{k=0}^{1+p} \beta_{k} a_{k, i}
\right)
\right]_i
\\

&=
\left[
\left(
\boldsymbol{a}_{(i)} \ \boldsymbol{\beta}
\right)
+
\left(
{}^t \boldsymbol{a}^{(i)} \boldsymbol{\beta}
\right)
\right]_i
\left(
\boldsymbol{a}_{(i)}は行列\boldsymbol{A}のi行目ベクトル
\right)
\\

&=
\left(
\boldsymbol{A} + {}^t \boldsymbol{A}
\right) \ \boldsymbol{\beta}
\tag{証明終}

\end{align}

```
