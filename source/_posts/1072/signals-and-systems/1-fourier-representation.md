# 傅立葉表示法

$$ a_1cos(w_1t) \xrightarrow{\text{LTI System}} \underbrace{\alpha_1}_{\text{magnitude change}} cos( \underbrace{w_1}_{\text{frequency}} t + \underbrace{\beta_1}_{\text{phase shift}} ) $$

把一組 sinusoid 訊號丟進LTI系統後，magnitude, phase 改變，frequency 不變

## 頻率響應

> 頻率響應（英語：Frequency response，簡稱頻響）是當向電子儀器系統輸入一個振幅不變，頻率變化的信號時，測量系統相對輸出端的響應。通常與電子放大器、擴音器等聯繫在一起，頻響的主要特性可用系統響應的**幅度**（用分貝）和**相位**（用弧度）來表示。-- [wiki](https://zh.wikipedia.org/zh-tw/%E9%A2%91%E7%8E%87%E5%93%8D%E5%BA%94)

### Discrete-time

若輸入訊號 $x[n] = e^{j \Omega n}$ (unit amplitude complex sinusoidal)

$$
\begin{aligned}
    y[n] &= \sum^{\infin}_{k = -\infin} h[k] x[n-k] \\
    &= \sum^{\infin}_{k = -\infin} h[k] e^{j \Omega (n-k)} \\
    &= \underbrace{e^{j \Omega n}}_{\text{x[n]}} \sum^{\infin}_{k = -\infin} h[k] e^{- j \Omega k} \\
    &= e^{j \Omega n} \underbrace{H(e^{j \Omega})}_{\text{frequency response}} \\
    &= e^{j \Omega n} \underbrace{| H(e^{j \Omega}) |}_\text{magnitude} \underbrace{e^{j \angle H(e^{j \Omega})}}_\text{phase}
\end{aligned}
$$

> $H(e^{j\Omega})$ 是在強調 $H(j\Omega)$ 的週期為 $2\pi$
>
> $H(e^{j\Omega}) = \sum^{\infin}_{k = -\infin} h[k] e^{-j\Omega k}$
>
> $H(e^{j(\Omega + 2\pi)}) = \sum^{\infin}_{k = -\infin} h[k] e^{-j (\Omega+2\pi) k} = \sum^{\infin}_{k=-\infin} h[k] e^{-j\Omega k} \cancel{e^{-j 2\pi k}} = H(e^{j\Omega})$

$$ H(e^{j \Omega}) = \sum^{\infin}_{k = -\infin} h[k] e^{- j \Omega k} $$

### Continuous-time

$$ H(jw) = \int^{\infin}_{-\infin} h(\tau) e^{-jw\tau} d\tau $$

過程類似，換成積分而已

$$ 
\begin{aligned}
    y(t) &= \int^{\infin}_{-\infin} h(\tau) e^{jw(t-\tau)} d\tau \\
    &= e^{jwt} \int^{\infin}_{-\infin} h(\tau) e^{-jw\tau} d\tau \\
    &= e^{jwt} H(jw)
\end{aligned}
$$

### Polar Form

$$ H(jw) = |H(jw)| e^{j \angle H(jw)} $$

$$ \text{Magnitude response (} \alpha_i \text{) : } |H(jw)|$$

$$ \text{Phase response (} \beta_i \text{) : } e^{j \angle H(jw)}$$

### Example - RC Circuit

![](2019-05-11-16-38-35.png)

$$ \text{Impulse response: } h(t) = \frac{1}{RC} e^{-\frac{t}{RC}} u(t) $$

$$
\begin{aligned}
    \text{Frequency response: } H(jw) &= \frac{1}{RC} \int^{\infin}_{\infin} e^{-\frac{\tau}{RC}} u(\tau) e^{-j w \tau} d\tau \\
    &= \frac{1}{RC} \int^{\infin}_0 e^{-(jw + \frac{1}{RC})\tau} d\tau \\
    &= \frac{\frac{1}{RC}}{jw + \frac{1}{RC}}
\end{aligned}
$$

$$ \text{Magnitude response: } |H(jw)| = \frac{\frac{1}{RC}}{\sqrt{w^2 + (\frac{1}{RC})^2}} $$

$$ \text{Phase response: } \angle H(jw) = tan^{-1}(wRC) $$

![Magnitude and Phase response](2019-05-11-16-55-27.png)

## 訊號表示法

如果我們把訊號用 M 個 complex sinusoids 加起來表示：

$$ x(t) = \sum^{M}_{k=1} a_k e^{j w_k t} $$

$a_k$ 為 $e^{j w_k t}$ 在頻率為 $w_k$ 時的強度：

$$ a_k = X(j w_k) $$

把 $x(t)$ 丟進LTI系統中：

$$ \sum^{M}_{k=1} a_k e^{j w_k t} \xRightarrow{H(jw)} \sum^{M}_{k=1} \underbrace{a_k H(jw_k)}_{Y(jw_k)} e^{j w_k t} $$

可以觀察到如果在時域丟到系統後要做 convolution：

$$ y(t) = x(t) * h(t) $$

在頻域中就可以直接用相乘的：

$$ Y(jw_k) = X(jw_k) H(jw_k) $$

## 基底與投影

### 基底

定義：給定一個向量空間 $V \in R^n$，$V$ 的基底 $\text{\ss}$ 是指在 $V$ 裡面可以線性生成 $V$ 且彼此線性獨立的子集。

以下圖($R^2$)為例：

![](2019-05-11-17-25-11.png)

$$ \text{basis: } \{ A_\theta e_1, A_\theta e_2 \} $$

$$ \text{subspace: } V = \alpha A_\theta e_1 + \beta A_\theta e_2 $$

$$ 
\text{projection: }
\begin{pmatrix}
    \alpha = V^T A_\theta e_1 \\
    \beta = V^T A_\theta e_2
\end{pmatrix}
$$

### 正交基底

> 一組彼此正交的向量集必定是線性獨立的，但是線性獨立未必正交。-- [基底一定是正交嗎？](https://ccjou.wordpress.com/2010/06/15/%E5%9F%BA%E5%BA%95%E8%88%87%E7%B6%AD%E5%BA%A6-%E5%B8%B8%E8%A6%8B%E5%95%8F%E7%AD%94%E9%9B%86/#comment-3631)

基底中的向量彼此互相正交(內積為0)，$V_i^T V_j = 0, i ~\cancel{=}~ j$

### 正則基底

該基底是正交基底且長度均為1

$$ 
V_i^T V_j = \begin{cases}
   1, i = j \\
   0, i ~\cancel{=}~ j
\end{cases}
$$

### 使用基底表示向量

假設用 $\text{\ss} = \{ V_1, V_2, ... , V_k \}$ 表示向量 $u$

當 $\text{\ss}$ 是正交基底：

$$ u = \underbrace{\frac{u \cdot V_1}{||V_1||^2}}_{\text{projection scalar}} V_1 + \frac{u \cdot V_2}{||V_2||^2} V_2 + ... + \frac{u \cdot V_k}{||V_k||^2} V_k$$

> projection scalar: $u$ 投影到 normalize 後的 $V_i$ 所產生的投影量
> 
> $u \cdot V_i = V_i^T \cdot u$

當 $\text{\ss}$ 是正則基底：

$$ u = (u \cdot V_1) V_1 + (u \cdot V_2) V_2 + ... + (u \cdot V_k) V_k $$

#### 複數型向量

> 共軛轉置(hermitian conjugate): 取共軛並轉置
>
> $V_1 = \begin{bmatrix}
>     a + jb \\
>     c + jd
> \end{bmatrix}
> , V_1^H = \begin{bmatrix}
>     a - jb, c - jd
> \end{bmatrix}$

當 $\text{\ss}$ 是正交基底：

$$
u = \frac{V_1^H \cdot u}{||V_1||^2_2} V_1 + \frac{V_2^H \cdot u}{||V_2||^2_2} V_2 + ... + \frac{V_k^H \cdot u}{||V_k||^2_2} V_k
$$

把 projection scalar 視為一個一個的變量 $C_i$，於是式子就變成：

$$ u = C_1 V_1 + C_2 V_2 + ... + C_k V_k $$

$$ C_i = \frac{V_i^H \cdot u}{||V_i||^2_2} $$

> $||V_i||_2$ -- [two-norms](https://en.wikipedia.org/wiki/Norm_(mathematics)#Euclidean_norm)

$$ $$

當 $\text{\ss}$ 是正則基底：

$$ u = (V_1^H \cdot u) V_1 + (V_2^H \cdot u) V_2 + ... + (V_k^H \cdot u) V_k $$

#### 連續時間向量

假設用 $\text{\ss} = \{ ...~ g_{-1}(t), g_{0}(t), g_{1}(t) ~... \}$ 表示向量 $f(t)$

且 $\text{\ss}$ 為正交基底 (兩兩積分一段週期為0)

$$ f(t) = ... + C_{-1}g_{-1}(t) + C_{0}g_{0}(t) + ... $$

$C_i$ 在 discrete-time 為內積，在 continuous-time 為積分，廣義的內積：

$$ C_i = \frac{\int_t g_i^*(t) f(t) dt}{\int_t |g_i(t)|^2 dt} $$

## 傅立葉表示法

有 4 種傅立葉表示方式：

|                  | Discrete-Time                   | Continuous-Time   |
| ---------------- | ------------------------------- | ----------------- |
| **periodic**     | Discrete-Time Fourier Series    | Fourier Series    |
| **non-periodic** | Discrete-Time Fourier Transform | Fourier Transform |

時域與頻域的關係：

| Time-domain  | Freqency-domain | Explain                                                                                                           |
| ------------ | --------------- | ----------------------------------------------------------------------------------------------------------------- |
| discrete     | periodic        | [Sampling](/NCTU-Coursenote/1072/signals-and-systems/3-fourier-representation-with-mixed-signals/#圖例-frequency) |
| continuous   | non-periodic    |                                                                                                                   |
| periodic     | discrete        | 很好想像                                                                                                          |
| non-periodic | continuous      |                                                                                                                   |

### DTFS

$$
\text{DTFS Pair} \begin{cases}
    \text{Time: discrete, periodic} (n, \Omega)\\
    \text{Frequency: periodic, discrete} (k\Omega_0)
\end{cases}
\text{Sinusoid: } e^{j k \Omega_0 n}, \Omega_0 = \frac{2\pi}{N}
$$

因為 Frequency 為 discrete 且 periodic，所以我們可以將 $x[n]$ 用 N 個 complex sinusoids 疊加組合而成 (詳見[訊號表示法](#訊號表示法))：

$$ x[n] = \sum^{N-1}_{k=0} X[k] e^{j k \Omega_0 n} $$

是不是有點眼熟？再回去看看[使用基底表示複數型向量](#複數型向量)中提到的：

$$ u = C_1 V_1 + C_2 V_2 + ... + C_k V_k $$

觀察一下上式後發現這個一個向量一個向量加起來的形式，很像上面求和 $\sum$ 的式子！其中 projection scalar 就是我們感興趣的 $X[k]$

$$ C_i = \frac{V_i^H \cdot u}{||V_i||^2_2} $$

然後我們就發現了一件事：我們可以把  $e^{j k \Omega_0 n}$ 看成是一個基底，之後再將 $x[n]$ 投影到基底上，如此變可以求得 $X[k]$ 了！

首先，將 $e^{j k \Omega_0 n}$ 看成是基底：

$$
\begin{Bmatrix}
    \begin{matrix}
        & k = 0 & k = 1 & ... & k = N-1 \\
        \begin{matrix}
            n = 0 \\
            n = 1 \\
            \vdots \\
            n = N-1 \\
        \end{matrix}
        &
        \begin{bmatrix}
            1 \\
            1 \\
            \vdots \\
            1 \\
        \end{bmatrix}
        &
        \begin{bmatrix}
            e^{j \Omega_0 0} \\
            e^{j \Omega_0 1} \\
            \vdots \\
            e^{j \Omega_0 k} \\
        \end{bmatrix}
        &
        ...
        &
        \begin{bmatrix}
            e^{j (N-1) \Omega_0 0} \\
            e^{j (N-1) \Omega_0 1} \\
            \vdots \\
            e^{j (N-1) \Omega_0 k} \\
        \end{bmatrix} \\
        & V_0 & V_1 & ... & V_{N - 1} \\
    \end{matrix}
\end{Bmatrix}_{N \times N}
$$

而且這個基底還是[正交基底](#正交基底)，證明如下：

$$
\begin{aligned}
    V_l^H V_m &= \sum^{N-1}_{n=0} e^{-j l \Omega_0 n} e^{j m \Omega_0 n} \\
    &= \sum^{N-1}_{n=0} e^{j (m-l) \frac{2\pi}{N} n}
    = \begin{cases}
        \frac{ 1 - e^{j (m-l) \frac{2\pi}{\cancel{N}} \cancel{N}} }{ 1 - e^{j (m-l) \frac{2\pi}{N}} } = 0 &, l ~\cancel{=}~ m \\
        N &, l = m
    \end{cases}
\end{aligned}
$$

同時可以得到 2-norms 值，$||V_k||^2_2 = N$

把 $x[n]$ 也寫成向量的形式：

$$
\underline{x} = \begin{bmatrix}
    x[0] \\
    x[1] \\
    \vdots \\
    x[N-1]
\end{bmatrix}
$$

之後把 $\underline{x}$ 投影到基底上：

$$ X[k] = \frac{V_k^H \cdot \underline{x}}{||V_k||^2_2} = \frac{1}{N} \sum^{N-1}_{n=0} x[n] e^{-j k \Omega_0 n} $$

可以發現有一個 nomarlization factor $N$ 以及 sinusoid 上有一個負號(因為 hermitian conjugate)

到此我們變找到了 DTFS Pair：

$$ x[n] = \sum^{N-1}_{k=0} X[k] e^{j k \Omega_0 n} \xleftrightarrow{DTFS} X[k] = \frac{1}{N} \sum^{N-1}_{n=0} x[n] e^{-j k \Omega_0 n} $$

### Normalization factor

為了證明 normalization factor，我們要證明 $x[n'] = IDTFS\{DTFS\{x[n]\}\}$：

$$
\begin{aligned}
    &IDTFS\{DTFS\{x[n]\}\} \\
    &= IDTFS\{ \frac{1}{N} \sum^{N-1}_{n=0} x[n] e^{-j k \Omega_0 n} \} \\
    &= \color{blue}{\sum^{N-1}_{k=0}} \color{black}{\frac{1}{N} \sum^{N-1}_{n=0} x[n] e^{-j k \Omega_0 n}} \color{blue}{e^{j k \Omega_0 n'}} \\
    &= \frac{1}{N} \sum^{N-1}_{n=0} x[n] \sum^{N-1}_{k=0} e^{-j k \Omega_0 (n-n')} \\
    &= \begin{cases}
        \frac{1}{N} \sum^{N-1}_{n=0} x[n] \frac{1 - e^{-j \cancel{N} \frac{2\pi}{\cancel{N}} (n-n')}}{1 - e^{-j \frac{2\pi}{N} (n-n')}} = 0 &, n ~\cancel{=}~ n' \\
        \frac{1}{\cancel{N}} x[n'] \cancel{N} = x[n'] &, n = n'
    \end{cases}
\end{aligned}
$$

若要反求 normalization factor，可以將 $\frac{1}{N}$ 設成任意未知數，帶入上方的證明，並令結果 $x[n'] = x[n]$ 即可求得

### FS

$$
\text{FS Pair} \begin{cases}
    \text{Time: continuous, periodic} (t, w)\\
    \text{Frequency: non-periodic, discrete} (kw_0)
\end{cases}
\text{Sinusoid: } e^{j k w_0 t}, w_0 = \frac{2\pi}{T}
$$

因為 Frequency 為 discrete 且 non-periodic，所以我們要用所有 complex sinusoids 疊加才能組成 $x(t)$：

$$ x(t) = \sum^{\infin}_{k = -\infin} X[k] e^{j k w_0 t} $$

將 $\{ e^{j k w_0 } \}^\infin_{k=-\infin}$ 視為正交基底

證明，任取兩個向量積分一段週期為0：

$$
\begin{aligned}
    &\int_t e^{j k_1 w_0 t} e^{-j k_2 w_0 t} dt \\
    &= \int_t e^{j w_0 t (k_1 - k_2)} dt \\
    &= \begin{cases}
        \frac{T}{j (k_1 - k_2) 2\pi} e^{j (k_1 - k_2) \frac{2\pi}{T} T} = 0 &, k_1 ~\cancel{=}~ k_2 \\
        \int^T_{t=0} 1 dt = T &,k_1 = k_2
    \end{cases}
\end{aligned}
$$

同時可以得到 normalization factor 為 $T$

把 $x(t)$ 投影到基底可以得到 $X[k]$：

$$ x(t) = \sum^{\infin}_{k = -\infin} X[k] e^{j k w_0 t} \xleftrightarrow{FS} X[k] = \frac{1}{T} \int^T_0 x(t) e^{-j k w_0 t} dt $$

Trigonometric Fourier Series:

$$ x(t) = \underbrace{B[0]}_{\text{DC Term}} + \sum^\infin_{k=1} B[k]cos(kw_0t) + A[k]sin(kw_0t) $$

### DTFT

$$
\text{DTFT Pair} \begin{cases}
    \text{Time: discrete, non-periodic} (n, \Omega)\\
    \text{Frequency: periodic, continuous}
\end{cases}
\text{Sinusoid: } e^{j \Omega n}
$$

投影的技巧與 normalization factor 證明手法與前面相同，這邊省略

$$ x[n] = \frac{1}{2\pi} \int^{2\pi}_0 X(e^{j\Omega}) e^{j \Omega n} d\Omega \xleftrightarrow{DTFT} X(e^{j\Omega}) = \sum^\infin_{n = -\infin} x[n] e^{-j \Omega n} $$

### FT

$$
\text{FT Pair} \begin{cases}
    \text{Time: continuous, non-periodic} (t, w)\\
    \text{Frequency: non-periodic, continuous}
\end{cases}
\text{Sinusoid: } e^{j w t}
$$

$$ x(t) = \frac{1}{2\pi} \int^{\infin}_{-\infin} X(jw) e^{jwt} dw \xleftrightarrow{FT} X(jw) = \int^{\infin}_{-\infin} x(t) e^{-j w t} dt$$

## Examples

### DTFS

#### Periodic signal

Q: 求 $X[k]$ ?

![](2019-05-11-22-26-58.png)

週期: 5

$$
\begin{aligned}
    X[k] &= \frac{1}{5} \sum^{2}_{-2} x[n] e^{-j k \frac{2\pi}{5} n} \\
    &= \frac{1}{5} (1 + \frac{1}{2} e^{j k \frac{2\pi}{5}} - \frac{1}{2} e^{- j k \frac{2\pi}{5}}) \\
    &= \frac{1}{5} (1 + j sin(k \frac{2\pi}{5}))
\end{aligned}
$$

![X[k] is periodic](2019-05-11-22-42-30.png)

#### Sinusoid

Q: 求 $x[n] = cos(n\frac{\pi}{3} + \phi)$ 的 DTFS？

週期: $\frac{\pi}{3}$

根據[尤拉公式](https://en.wikipedia.org/wiki/Euler%27s_formula#Relationship_to_trigonometry)，可以把 $x[n]$ 看成：

$$ x[n] = \frac{e^{j(n\frac{\pi}{3} + \phi)} + e^{-j(n\frac{\pi}{3} + \phi)}}{2} = \frac{1}{2} e^{-j \phi} e^{-j \frac{\pi}{3} n} + \frac{1}{2} e^{j \phi} e^{-j \frac{\pi}{3} n} $$

根據 DTFS 的定義：

$$ x[n] = \sum^{3}_{k = -2} X[k] e^{j k \pi \frac{n}{3}} $$

可以發現

$$ X[-1] e^{j \pi \frac{n}{3}} = \frac{1}{2} e^{-j \phi} e^{-j \frac{\pi}{3} n} $$
$$ X[1] e^{j \pi \frac{n}{3}} = \frac{1}{2} e^{j \phi} e^{-j \frac{\pi}{3} n} $$

![](2019-05-11-23-17-07.png)

$$
x[n] \xleftrightarrow{DTFS} X[k] = \begin{cases}
    e^{\frac{-j \phi}{2}} &,k = -1 \\
    e^{\frac{j \phi}{2}} &,k = 1 \\
    0 &,\text{otherwise on} -2 \leq k \leq 3
\end{cases}
$$

#### Impulse Train

Q: 求 $x[n] = \sum^{\infin}_{l = -\infin} \delta[n - Nl]$ 的 DTFS？

![](2019-05-11-23-19-13.png)

$$ X[k] = \frac{1}{N} \sum^{N-1}_{n = 0} \delta[n] e^{-j k n \frac{2\pi}{N}} = \frac{1}{N} $$

因為 delta function 變化太大，所以所有頻率都要有值才能表示出 $x[n]$

#### Square Wave

$$
x[n] = \begin{cases}
    1 &, -M \leq n \leq M \\
    0 &, M < n < N - M
\end{cases}
$$

Q: 求 $X[k]$？

![](2019-05-11-23-26-53.png)

$$ \Omega_0 = \frac{2\pi}{N} $$

$$
\begin{aligned}
    X[k] &= \frac{1}{N} \sum^{N-M-1}_{n=-M} x[n] e^{-j k \Omega_0 n} \\
    &= \frac{1}{N} \sum^{M}_{n=-M} e^{-j k \Omega_0 n} \\
    &= \frac{1}{N} \sum^{2M}_{m=0} e^{-j k \Omega_0 (m-M)} \\
    &= \frac{1}{N} e^{j k \Omega_0 M} \sum^{2M}_{m=0} e^{-j k \Omega_0 m}
\end{aligned}
$$

若 k 為 N 的整數倍：

$$
X[k] = \frac{1}{N} \sum^{2M}_{m=0} 1 = \frac{2M+1}{N}
$$

若 k 不為 N 的整數倍：

$$
\begin{aligned}
    X[k] &= \frac{e^{j k \Omega_0 M}}{N} (\frac{ 1 - e^{-j k \Omega_0 (2M + 1)} }{ 1 - e^{- j k \Omega_0} }) \\
    &= \frac{1}{N} (\frac{ e^{j k \Omega_0 (2M + 1) / 2} }{ e^{j k \Omega_0 / 2} })(\frac{ 1 - e^{-j k \Omega_0 (2M + 1)} }{ 1 - e^{-j k \Omega_0} }) \\
    &= \frac{1}{N} (\frac{ e^{j k \Omega_0 (2M + 1) / 2} - e^{-j k \Omega_0 (2M + 1) / 2} }{ e^{j k \Omega_0 / 2} - e^{-j k \Omega_0 / 2} }) \\
    &= \frac{1}{N} \frac{sin( k \Omega_0 (2M + 1) / 2 )}{ sin(k \Omega_0 / 2) }
\end{aligned}
$$

![X[k]](2019-05-11-23-44-48.png)

用 cos 來組合方波，把 DTFS 的式子改寫回 cos 的形式，假設 N 都是偶數：

$$
\begin{aligned}
    x[n] &= \sum^{N/2}_{k = -N/2 + 1} X[k] e^{j k \Omega_0 n} \\
    &= X[0] + X[N/2] e^{j N \Omega_0 n / 2} + \sum^{N/2-1}_{m = 1} (X[m] e^{jm\Omega_0n} + X[-m] e^{-jm\Omega_0n}) \\
    &= X[0] + X[N/2] e^{j \pi n} + \sum^{N/2-1}_{m = 1}2X[m](\frac{e^{jm\Omega_0n} + e^{-jm\Omega_0n}}{2}) \\
    &= X[0] + X[N/2] cos(\pi n) + \sum^{N/2-1}_{m = 1} 2X[m] cos(m \Omega_0 n) \\
    &= \sum^{N/2}_{k=0} B[k] cos(k\Omega_0n) ,B[k] = \begin{cases}
        X[k] &, k = 0, N/2 \\
        2X[k] &, k = 1, 2, ... , N/2-1
    \end{cases}
\end{aligned}
$$

$$ 
\hat{x}_j[n] = \sum^J_{k=0} B[k] cos(k\Omega_0n)
$$

![](2019-05-11-23-57-57.png)

### FS

#### Periodic signal

Q: 求 $X[k]$ ?

![](2019-05-12-15-21-15.png)

週期：2

基頻：$w_0 = 2\pi / 2 = \pi$

$$
\begin{aligned}
    X[k] &= \frac{1}{2} \int_0^2 e^{-2t} e^{-j k \pi t} dt \\
    &= \frac{1}{2} \int_0^2 e^{-(2 + jk\pi) t} dt \\
    &= \frac{1 - e^{-4}}{4 + jk2\pi}
\end{aligned}
$$

![](2019-05-12-15-26-16.png)

#### Impulse Train

Q: 求 $x(t) = \sum^{\infin}_{l = -\infin} \delta(t - 4l)$ 的 FS？

週期：4

基頻：$w_0 = 2\pi / 4 = \pi / 2$

$$
\begin{aligned}
    X[k] &= \frac{1}{4} \int^2_{-2} \delta(t) e^{-jk\frac{\pi}{2}t} dt \\
    &= \frac{1}{4}
\end{aligned}
$$

#### Square Wave

Q: 求 $X[k]$ ?

![](2019-05-12-15-31-32.png)

週期：$T$

基頻：$w_0 = 2\pi / T$

$$
\begin{aligned}
    X[k] &= \frac{1}{T} \int^{T/2}_{-T/2} x(t) e^{-jkw_0t} dt \\
    &= \frac{1}{T} \int^{T_0}_{-T_0} e^{-jkw_0t} dt \\
    &= \frac{-1}{Tjkw_0} e^{-jkw_0t} |^{T_0}_{-T_0}, k ~\cancel{=}~ 0 \\
    &= \frac{2}{Tkw_0} (\frac{e^{jkw_0T_0} - e^{-jkw_0T_0}}{2j}), k ~\cancel{=}~ 0 \\
    &= \frac{2sin(kw_0T_0)}{Tkw_0}, k ~\cancel{=}~ 0
\end{aligned}
$$

$$ \text{For k = 0, } X[0] = \frac{1}{T} \int^{T_0}_{-T_0} dt = \frac{2T_0}{T} $$

$k = 0$ 的 case 用羅必達也能得到相同的結果，所以 $X[k] = \frac{2sin(kw_0T_0)}{Tkw_0}$

在這邊我們定義 $sinc(u) = \frac{sin(\pi u)}{\pi u}$，所以 $X[k] = \frac{2T_0}{T} sinc(k \frac{2T_0}{T})$

![sinc(u)](2019-05-12-15-56-28.png)

![X[k]](2019-05-12-16-06-30.png)

若要組合回原本的波形，用 Trigonometric FS 觀察 $X[k]$ 各項的貢獻：

$$ x(t) = B[0] + \sum^\infin_{k=1} B[k]cos(kw_0t) + A[k]sin(kw_0t) $$

因為 $x(t)$ 是偶函數，所以 $A[k]$ 為 0，因此式子變成：

$$ x(t) = B[0] + \sum^\infin_{k=1} B[k]cos(kw_0t) $$

因為 $B[0]$ 也滿足 $B[k]$：

$$ x(t) = \sum^\infin_{k=0} B[k]cos(kw_0t) $$

但現實中不可能有無限大的項，所以我們改寫上界為：

$$ x(t) = \sum^J_{k=0} B[k]cos(kw_0t) $$

![](2019-05-12-16-16-31.png)

觀察上圖，當 J = 99 時，仍無法完美的表示方波，方波兩端有 Overshot

#### RC Circuit

![RC Circuit](2019-05-11-16-38-35.png)

![x(t)](2019-05-12-15-31-32.png)

Q: 求 $y(t)$ 的 FS？

我們已經有系統的 [Frequency Response](#example-rc-circuit) 以及輸入訊號的 [FS](#square-wave-v2)

根據[訊號表示法](#訊號表示法)討論出來的結果：$Y(jw_k) = X(jw_k)H(jw_k)$，我們可以知道在 FS 中：$Y[k] = H(jkw_0) X[k]$

假設 $RC = 0.1, w_0 = 2\pi , T_0/T = 1/4$：

$$ Y[k] = \frac{10}{j2\pi k+10} \frac{sin(k\pi/2)}{k\pi} $$

![](2019-05-12-16-36-08.png)

觀察左上圖，可以發現高頻都被濾掉了，因此 $y(t)$ 是平滑的，沒有 Overshot

### DTFT

#### Exponential Sequence

Q: 求 $x[n] = \alpha^n u[n]$ 的 DTFT？

$$
\begin{aligned}
    X(e^{j\Omega}) &= \sum^\infin_{n=-\infin} \alpha^n u[n] e^{-j \Omega n} \\
    &= \sum^\infin_{n = 0} (\alpha e^{-j \Omega})^n \\
    &= \frac{1}{1 - \alpha e^{-j \Omega}}, |\alpha e^{-j \Omega}| \equiv |\alpha| < 1
\end{aligned}
$$

![](2019-05-12-17-00-03.png)

#### Rectangular Pulse

![](2019-05-12-17-02-48.png)

Q: 求 DTFT？

$$
\begin{aligned}
    X(e^{j\Omega}) &= \sum^{2M}_0 e^{-j\Omega(m-M)} = e^{j\Omega M} \sum^{2M}_0 e^{-j\Omega m} \\
    &= \begin{cases}
        e^{j\Omega M} \frac{1 - e^{-j \Omega 2(M+1)}}{1 - e^{-j\Omega}} &, \Omega ~\cancel{=}~ 0, \pm 2\pi, \pm 4\pi... \\
        2M + 1 &, \Omega = 0, \pm 2\pi, \pm 4\pi...
    \end{cases}
\end{aligned}
$$

上式通過化簡，可以得到：

$$ X(e^{j\Omega}) = \frac{sin(\Omega(2M+1)/2)}{sin(\Omega/2)} $$

![](2019-05-12-17-14-10.png)

#### Rectangular Spectrum

![](2019-05-12-17-19-46.png)

Q: 求 Invert DTFT？

$$
\begin{aligned}
    x[n] &= \frac{1}{2\pi} \int^W_{-W} e^{j \Omega n} d\Omega \\
    &= \frac{1}{2 \pi n j} e^{j \Omega n} |^W_{-W} &, n ~\cancel{=}~ 0 \\
    &= \frac{1}{\pi n} sin(Wn) &, n ~\cancel{=}~ 0
\end{aligned}
$$

$$ \lim_{n \to 0} \frac{1}{n\pi} sin(Wn) = \frac{W}{n} $$

轉成 sinc function 的形式，可以得到：

$$ x[n] = \frac{W}{\pi} sinc(\frac{Wn}{\pi}) $$

![](2019-05-12-17-27-23.png)

#### Unit Impulse

Q: 求 $x[n] = \delta[n]$ 的 DTFT？

$$ X(e^{j\Omega}) = \sum^\infin_{n = -\infin} \delta[n] e^{-j \Omega n} = 1 $$

![](2019-05-12-17-31-09.png)

$$ \delta[n] \xleftrightarrow{DTFT} 1 $$

#### Unit Impulse Spectrum

Q: 求 $X(e^{j\Omega}) = \delta(\Omega), -\pi < \Omega < \pi$ 的 Invert DTFT？

$$ x[n] = \frac{1}{2\pi} \int^\pi_{-\pi} \delta(\Omega) e^{j \Omega n} d\Omega = \frac{1}{2\pi} $$

![](2019-05-12-17-34-21.png)

$$ \frac{1}{2\pi} \xleftrightarrow{DTFT} \delta(\Omega), -\pi < \Omega < \pi $$

### FT

#### Real Decaying Exponential

Q: 求 $x(t) = e^{-at} u(t), a > 0$ 的 FT？

![](2019-05-12-17-39-39.png)

$$
\begin{aligned}
    X(jw) &= \int^{\infin}_{-\infin} e^{-at} u(t) e^{-jwt} dt \\
    &= \int^{\infin}_0 e^{-(a + jw)t} dt \\
    &= - \frac{1}{a + jw} e^{-(a+jw)t} |^{\infin}_0 \\
    &= \frac{1}{a + jw}
\end{aligned}
$$

![](2019-05-12-17-42-53.png)

#### Rectangular Pulse

![](2019-05-12-17-45-02.png)

Q: 求 FS？

$$
\begin{aligned}
    X(jw) &= \int^{\infin}_{-\infin} x(t) e^{-jwt} dt \\
    &= \int^{T_0}_{-T_0} e^{-jwt} dt \\
    &= -\frac{1}{jw} e^{-jwt} |^{T_0}_{-T_0} &, w ~\cancel{=}~ 0 \\
    &= \frac{2}{w} sin(wT_0) &, w ~\cancel{=}~ 0
\end{aligned}
$$

$$ \text{For w = 0, } \lim_{w \to 0} \frac{2}{w} sin(wT_0) = 2T_0 $$

![](2019-05-12-17-49-53.png)

#### Rectangular Spectrum

![](2019-05-12-17-51-03.png)

Q: 求 Invert FS？

$$
\begin{aligned}
    x(t) &= \frac{1}{2\pi} \int^W_{-W} e^{jwt} dw \\
    &= - \frac{1}{j\pi t} e^{jwt} |^W_{-W} &, t ~\cancel{=}~ 0 \\
    &= \frac{1}{\pi t} sin(Wt) &, t ~\cancel{=}~ 0 \\
\end{aligned}
$$

$$ \text{For t = 0, } \lim_{t \to 0} \frac{1}{\pi t} sin(wT) = \frac{W}{\pi} $$

![](2019-05-12-17-55-16.png)

#### Unit Impulse

Q: 求 $x(t) = \delta(t)$ 的 FT？

$$ X(jw) = \int^{\infin}_{-\infin} \delta(t) e^{-jwt} dt = 1 $$

$$ \delta(t) \xleftrightarrow{FT} 1 $$

#### Impulse Spectrum

Q: 求 $X(jw) = 2\pi \delta(w)$ 的 Invert FT？

$$ x(t) = \frac{1}{2\pi} \int^{\infin}_{-\infin} 2\pi \delta(w) e^{jwt} dw = 1 $$

$$ 1 \xleftrightarrow{FT} 2\pi \delta(w) $$