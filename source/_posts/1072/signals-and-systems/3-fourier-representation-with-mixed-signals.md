# 傅立葉表示法對混合信號的應用

IC Design:

* baseband IC (digital in general)
* mixed signal IC (digital + continuous-time)
* analog IC (continuous-time)
* RF IC (continuous-time)

Mixed Singal:

* discrete + continuous (sampling)
* periodic + nonperiodic (modulation)

## Why Mixed Signal?

* 將週期訊號應用在 stable LTI system
  * convolution 包含非週期的 impulse response 及週期的 input signal
* Sampling
  * Sample continuous signals to discrete signals

建立傅立葉表示法之間的關係

## 週期訊號的FT表示法

### Relating FT to FS

> Definition of FS：$x(t) = \sum^\infin_{k = -\infin} X[k] e^{j k w_0 t}$
>
> Recalled that $1 \xleftrightarrow{FT} 2 \pi \delta(w)$
> 
> Freq-shift property: $e^{j k w_0 t} \xleftrightarrow{FT} 2 \pi \delta(w - kw_0)$ 代入 FS 的定義中

根據 linearity 的性質：

$$
x(t) = \sum^\infin_{k = -\infin} X[k] e^{j k w_0 t} \xleftrightarrow{FT} X(jw) = 2 \pi \sum^\infin_{k = -\infin} X[k] \delta(w - kw_0)
$$

可以得到 FT 與 FS 的關係式

![](2019-05-06-22-09-24.png)

觀察上圖可以發現：

* $kw_0$：$k \in \Z$，discrete $w \in \R$，continuous
* 高度為 $X[k]$ 乘上 $2 \pi$

歸納 FS 與 FT 間的關係：

1. Kronecker delta $\to$ Dirac delta
2. Strength$\times 2 \pi$
3. Spacing: $k \to kw_0$

#### Example

##### 1. cosine

Q: 求 $x(t) = cos(w_0t)$ 的 FT 表示法？

$$
cos(w_0t) \xleftrightarrow{FS; w_0} X[k] = \begin{cases}
    \frac{1}{2}, k = \pm 1 \\
    0, k ~\cancel{=}~ \pm 1
\end{cases}
$$

代入 FT 與 FS 的關係式可得：

$$
cos(w_0t) \xleftrightarrow{FT} X(jw) = \pi \delta (w - w_0) + \pi \delta (w + w_0)
$$

![](2019-05-06-22-27-26.png)

##### 2. unit impulse train

Q. 求 $p(t) = \sum^\infin_{n = -\infin} \delta(t - nT)$ 的 FT 表示法？

$p(t)$ 為週期性且基頻 $w_0 = 2\pi / T$

FS of $p(t)$：

$$
P[k] = \frac{1}{T} \int^{T/2}_{-T/2} \delta(t) e^{-jkw_0t} dt = \frac{1}{T}
$$

代入 FT 與 FS 的關係式可得：

$$
P(jw) = \frac{2\pi}{T} \sum^\infin_{k = -\infin} \delta(w - kw_0)
$$

![](2019-05-07-01-27-41.png)

可以觀察到 $p(t)$ 在做完 FT 後依然為 impulse train:

1. Kronecker delta $\to$ Dirac delta
2. Strength: $\frac{1}{T} \times 2 \pi$
3. Spacing: $k \frac{2\pi}{T}$

### Relating DTFT to DTFS

手法與前面類似

> Definition of DTFS：$x[n] = \sum^{N-1}_{k=0} X[k] e^{j k \Omega_0 n}$
>
> $\frac{1}{2\pi} e^{jk\Omega_0n} \xleftrightarrow{DTFT} \sum^\infin_{m = -\infin} \delta(\Omega - k\Omega_0 - m2\pi)$ 代入 DTFS 的定義

根據 linearity 的性質：

$$
x[n] = \sum^{N-1}_{k=0} X[k] e^{j k \Omega_0 n} \xleftrightarrow{DTFT} X(e^{j\Omega}) = 2\pi \sum^{N-1}_{k=0} X[k] \sum^\infin_{m=-\infin} \delta(\Omega - k\Omega_0 - m2\pi)
$$

因為 $X[k]$ 週期為 $N$ 且 $N\Omega_0 = 2\pi$，所以可以合併右邊的兩個 $\Sigma$：

$$
x[n] = \sum^{N-1}_{k=0} X[k] e^{j k \Omega_0 n} \xleftrightarrow{DTFT} X(e^{j\Omega}) = 2\pi \sum^\infin_{k = -\infin} X[k] \delta(\Omega - k \Omega_0)
$$

得到 DTFT 與 DTFS 的關係式

![](2019-05-07-00-48-50.png)

歸納 DTFS 與 DTFT 間的關係：

1. Kronecker delta $\to$ Dirac delta
2. Strength$\times 2 \pi$
3. Spacing: $k \to k\Omega_0$

## 週期與非週期訊號的摺積

![](2019-05-06-23-40-24.png)

在真實的 LTI 系統中，impulse response 是 nonperiodic 的

> Recall: convolution property is that both signals are periodic or both signals are nonperiodic
> 
> e.g. $y(t)=x(t)*h(t) \xleftrightarrow{FT} Y(jw)=X(jw)H(jw)$

Solution: <mark>用 FT 來使訊號變為週期性</mark>

$$ x(t) \xleftrightarrow{FT} X(jw) = 2\pi \sum^\infin_{k=-\infin} X[k] \delta(w-kw_0)  $$

其中 $X[k]$ 是 FS 係數，將上式帶入 convolution 的式子中：

$$ y(t)=\underbrace{x(t)}_{\text{periodic}} * \underbrace{h(t)}_{\text{nonperiodic}} \xleftrightarrow{FT} Y(jw) = 2\pi \sum^\infin_{k=-\infin} X[k] \delta(w - kw_0)H(jw) $$

根據 Dirac delta function 的性質：

$$ y(t)=x(t)*h(t) \xleftrightarrow{FT} Y(jw) = 2\pi \sum^\infin_{k=-\infin} H(j\underline{kw_0}) X[k] \delta(w - kw_0) $$

![](2019-05-06-23-52-23.png)

上圖表示出了 $X(jw)$ 與 $H(jw)$ 相乘的情形，$Y(jw)$ 的形式 (nonperiodic, discrete) 對應於一個週期訊號 (continuous, periodic)。因此，$y(t)$ 是一個與 $x(t)$ 有相同週期的週期訊號 ($\because \omega_0$ 相同)。

## 週期與非週期訊號的乘積

> Recall: $y(t)=g(t)x(t) \xleftrightarrow{FT} Y(jw)=\frac{1}{2\pi} G(jw)*X(jw)$

若 $x(t)$ 是週期訊號

根據[前面](#週期訊號的ft表示法)的推導：

$$ x(t) \xleftrightarrow{FT} X(jw) = 2 \pi \sum^\infin_{k = -\infin} X[k] \delta(w - kw_0) $$

代入上式中：

$$ y(t)=g(t)x(t) \xleftrightarrow{FT} Y(jw)=G(jw) * \sum^\infin_{k = -\infin} X[k] \delta(w - kw_0) $$

根據 Dirac delta function 的性質：

$$ y(t)=g(t)x(t) \xleftrightarrow{FT} Y(jw) = \sum^\infin_{k = -\infin} X[k] G(j(w - kw_0)) $$

### AM Radio

AM: Amplitude Modulataion

調變(modulation)原本的訊號 $m(t)$：

$$ r(t) = m(t) cos(w_c t) \xleftrightarrow{FT} R(jw) = \frac{1}{2} (M(w-w_c) + M(w+w_c)) $$

> $R(jw) = \frac{1}{2\pi} M(jw) * C(jw)$
> 
> $\to \frac{1}{2\pi} M(jw) * C(jw)$
> 
> $\to \frac{1}{2 \cancel{\pi}} M(jw) * ( \cancel{\pi}\delta(w-w_0) + \cancel{\pi}\delta(w+w_0) )$
> 
> $\to \frac{1}{2} (M(w-w_0) + M(w+w_0))$

解調(demodulation)：

$$ g(t) = r(t) cos(w_c t) \xleftrightarrow{FT} G(jw) = \frac{1}{2} (R(w-w_c) + R(w+w_c)) $$

用 $M(jw)$ 表示 $G(jw)$：

$$ G(jw) = \underbrace{\frac{1}{4} M(j(w-2w_c))}_{\text{high freqency}} + \underbrace{\frac{1}{2} M(j(w))}_{\text{low frequency}} + \underbrace{\frac{1}{4} M(j(w+2w_c))}_{\text{high frequency}} $$

使用 low pass filter 即可還原回原本的 $m(t)$

![](2019-05-07-00-44-20.png)

## 離散時間訊號的FT表示法

考慮如何將 discrete-time signal 和 continuous-time signal 做轉換，在這邊從 discrete-time signal 推回去 continuous-time signal，整體的流程：

1. discrete-time signal 轉成 temp signal (過度訊號)
2. temp signal 轉成 continuous-time signal

### Relating FT to DTFT

complex sinusoids:

* continuous-time: $x(t) = e^{jwt}$
* discrete-time: $g[n] = e^{j\Omega n}$

#### DTFT

在之前就已經知道，對任意 discrete-time signal $x[n]$ 做 DTFT：$X(e^{j\Omega}) = \sum_{n = - \infin}^{\infin}x[n]e^{-j\Omega n}$

#### 取樣、代換

每隔 $T_s$ 時間間隔去取樣 continuous 的訊號，假設 $x[n]$ 等於取樣過後的 $x(t)$:

* $x(nT_s) = x[n] \Rightarrow t = nT_s$
* $e^{j\Omega n} = e^{jwt} \Rightarrow e^{j\Omega n} = e^{jwT_sn} \Rightarrow \Omega = wT_s$

將 $\Omega = w T_s$ 代入公式：$X(e^{j\Omega})|_{\Omega = w T_s} = \sum_{n = - \infin}^{\infin}x[n]e^{-j w T_s n}$，我們稱這個頻域的訊號叫<mark>過度訊號</mark>：$X_{\delta}(jw)$

#### FT

過渡訊號 $X_{\delta}(jw)$ 在時域的表示可以用逆傅立葉轉換得到，且因為有 linearity 的性質，所以可以看成：

$$
x_\delta(t) = F^{-1}\{X_\delta(jw)\} =  F^{-1}\{ \sum_{n = - \infin}^{\infin}x[n]e^{-j w T_s n} \} = \sum_{n = - \infin}^{\infin} x[n] F^{-1}\{ e^{-j w T_s n} \}
$$

由之前的結論：

$$
\delta (t) \xleftrightarrow{FT} 1 \\
\delta (t - t_0) \xleftrightarrow{FT} e^{-j w t_0} 
$$

上面的 $T_s n$ 可以看成一個 time shift，於是就可以得到：

$$
x_\delta(t) = \sum_{n = - \infin}^{\infin} x[n] \delta (t - n T_s)
$$

觀察 delta function 後可以發現，$x_\delta(t)$ 就是 continuous-time 的 $x[n]$ 表示法，在非 $nT_s$ 的地方都為 0，總結以上：

$$
x_\delta(t) = \sum_{n = - \infin}^{\infin} x[n] \delta (t - n T_s) \xleftrightarrow{FT} X_\delta (j\omega) = \sum_{n = - \infin}^{\infin}x[n]e^{-j w T_s n}
$$

其中 $x_\delta(t)$ 是 continuous-time signal (來自$x[n]$)，$x_\delta(t)$ 在 FT 後得到的結果 $X_\delta(jw)$ 是對應到 $x[n]$ 在 DTFT 後得到的 $X(e^{j\Omega})$

#### 圖例

![](2019-04-30-00-34-18.png)

原先為左上方的 Kronecker delta (discrete-time)，透過 DTFT、取樣、FT 後轉成 Dirac delta (continuous-time)，即為過度訊號

### Relating FT to DTFS

沒教，之後有機會補上

## Sampling

![](2019-04-30-01-21-33.png)

### Sampling (Time)

$$
x_\delta (t) = \sum_{n = - \infin}^{\infin} x[n] \delta (t - nT_s) \\
\xRightarrow{x[n] = x(n T_s)} x_\delta (t) = \sum_{n = - \infin}^{\infin} x(n T_s) \delta (t - nT_s) \\
\xRightarrow{t = nT_s} x_\delta (t) = \sum_{n = - \infin}^{\infin} x(t) \delta (t - nT_s) \\
\Rightarrow x_\delta (t) = x(t) \sum_{n = - \infin}^{\infin} \delta (t - nT_s) \\
\Rightarrow x_\delta (t) = x(t) p(t) ~ where ~ p(t) = \sum_{n = - \infin}^{\infin} \delta (t - nT_s)
$$

由 $p(t)$ 的式子可以觀察到它是一個 **impulse train**，且由推導可以知道 $p(t)$ 是用來關聯 $x(t)$ 與 $x_\delta(t)$ 的關係式，透過 $p(t)$ 及的式子可以從 $x(t)$ 得到 $x_\delta(t)$，再由前面推導的過度訊號轉換成離散訊號即可達成 sampling 的目的

### 圖例 (Time)

![](2019-04-30-01-17-27.png)

$x_\delta(t)$ 為 $x(t)$ 與 $p(t)$ 的 time-domain 做相乘而得

### Sampling (Frequency)

在時域做相乘等於在頻域做convolution

$$
X_\delta (j w) = \frac{1}{2 \pi} X(jw) * P(j w)
$$

對 impulse train $P(jw)$ 做 FT 仍然是 impulse train (詳見[前面](#relating-ft-to-fs))，sample frequency $w_s = 2 \pi / T_s$：

$$
X_\delta (j w) = \frac{1}{\cancel{2\pi}} X(jw) * \frac{\cancel{2\pi}}{T_s} \sum_{k = - \infin}^{\infin} \delta (w - kw_s)
$$

因為 convolution delta function 所以可以簡單地知道結果：

$$
X_\delta (j w) = \frac{1}{T_s} \sum_{k = -\infin}^{\infin} X(jw - jkw_s)
$$

### 圖例 (frequency)

![](2019-04-30-02-01-15.png)

對 sample 後的訊號做 FT 的結果(過度訊號，$X_\delta (jw)$)是將原訊號複製無限多份，且強度變為 $1/T_s$ (右上圖)

對 sample 後的訊號做 DTFT 的結果，可以透過 $x[n] \xleftrightarrow{DTFT} X (e^{j \Omega}) = X_\delta (jw) |_{w = \Omega / T_s}$ 得到 (右下圖)

> time: discrete、frequency: periodic，可以用 sampling 來理解
> 
> $\because \Omega = w_s T_s = 2 \pi f_s T_s = 2 \pi \frac{1}{\cancel{T_s}} \cancel{T_s}$，固定每 $2\pi$ 為一週期所以是 periodic

還原訊號時，取 $X (e^{j \Omega})$ 的 $-\pi \sim \pi$

### Sampling Aliasing

$w_s = 2\pi / T_s$ 當 $T_s$ 提升時會使 $w_s$ 下降，當 $w_s \lt 2 W$ 時會產生 overlap。

![](2019-04-30-01-54-04.png)

當取樣 overlap 時即無法再還原回原訊號，因此取樣頻率要足夠高。

> 取樣頻率也不可過大，原先 1MHz 即可取的用 100MHz 來取，代表多取了 100 倍的點，複雜度上升，所需記憶體也上升

### Example

#### Sampling a Sinusoid

Q: 考慮取樣 $x(t) = cos(\pi t)$

對 $x(t)$ 做 FT：$x(t) \xrightarrow{FT} X(jw) = \pi \delta(w + \pi) + \pi \delta(w - \pi)$

![](2019-05-06-16-54-26.png)

對 sample 後的 $x(t)$ 做 FT：$x_\delta(t) \xrightarrow{FT} X_\delta(jw) = \frac{\pi}{T_s} \sum^\infin_{k = - \infin} \delta(w + \pi - kw_s) + \delta(w - \pi - kw_s)$

![](2019-05-06-17-01-09.png)

$$
T_s = \frac{1}{4} \Rightarrow w_s = \frac{2\pi}{T_s} = 8\pi
$$

> 為了避免 aliasing：$w_s > 2 \pi \Rightarrow T_s < 1$

**取樣頻率太低的例子**：

$T_s = 1 \Rightarrow w_s = 2\pi$

沒辦法還原，因為 sampled signal 變成 $\sum^\infin_{n = -\infin} (-1)^n \delta(t - nT_s)$ (impulse train)，而非原先的弦波。

![](2019-05-06-17-07-41.png)

$T_s = 3/2 \Rightarrow w_s = 4/3\pi$

要還原原本的訊號時，變成取 $\pi / 3$，而非原本的 $\pi$

![](2019-05-06-17-11-43.png)

#### Aliasing in Movies

![Film-based movies](2019-05-06-17-26-58.png)

![Frame space less than π/2](2019-05-06-17-31-03.png)

![Frame space equal to 3π/2](2019-05-06-17-31-11.png)

![Frame space equal to π](2019-05-06-17-31-18.png)

#### Aliasing

若有 aliasing，我們就無法對同一個取樣區分不同的 continuous-time signal

![](2019-05-07-01-30-51.png)

## Sample Reconstruction

### Sampling Theorem

取樣頻率太高需要很多記憶體，太低會產生aliasing，到底要取多少？<mark>用**取樣定理**</mark>

假設 $x(t) \xleftrightarrow{FT} X(jw)$ 表示一個 **band-limited signal** ($X(jw) = 0 ~ for ~ |w| > w_m$)：

![](2019-05-07-01-32-27.png)

若取樣頻率 $w_s > 2 w_m$ 則 $x(t)$ 可以由它的取樣 $x(nT_s), n = 0, \pm 1, \pm 2...$ 還原回來。

#### Definition

* <mark>Nyquist sampling rate(Nyquist rate): minimum sampling frequency $2w_m$</mark>
* Nyquist frequency: actual sampling frequency $w_s$

#### Example

$x(t) = sin(10 \pi t ) / (\pi t)$，求 maximum sampling interval $T_s$？

$$
X = \begin{cases}
    1, ~ |w| \leq 10 \pi \\
    0, ~ |w| > 10 \pi
\end{cases}
$$

![](2019-05-07-01-33-24.png)

$w_m = 10 \pi \Rightarrow w_s > 2 w_m \Rightarrow 2 \pi / T_s > 20 \pi \Rightarrow T_s < (1 / 10)$

### Ideal Reconstruction

![考慮如何從 discrete signal 還原回 continuous signal](2019-05-07-01-34-22.png)

#### Frequency domain

對 sampled signal 做 FT：$X_\delta(jw) = \frac{1}{T_s} \sum^\infin_{k = -\infin} X (jw - jkw_s)$

![](2019-05-06-18-06-29.png)

透過 filter 從 $X_\delta(jw)$ 取出 $X(jw)$

$$
H_r(jw) = \begin{cases}
    Ts, ~ |w| \leq w_s / 2 \\
    0, ~ |w| > w_s / 2
\end{cases}
$$

![](2019-05-07-01-36-32.png)

$$
X(jw) = X_\delta(jw) H_r(jw)
$$

#### Time domain

頻域相乘等於時域做convolution

$$
x(t) = x_\delta(t) * h_r(t)
$$
$$
\Rightarrow x(t) = h_r(t) * \sum^\infin_{n = -\infin} x[n] \delta(t - nT_s)
$$
$$
\Rightarrow x(t) = \sum^\infin_{n = -\infin} x[n] h_r(t - nT_s)
$$

$H_r(jw)$ 在時域為 sinc function (對 $H_r(jw)$ 做 IFT)：

$$
h_r(t) = \frac{sin(\frac{w_s}{2} t)}{\frac{w_s}{2} t} = \frac{sin(\pi \frac{t}{T_s})}{\pi \frac{t}{T_s}} = sinc(\frac{t}{T_s}) = sinc(\frac{w_s t}{2 \pi})
$$

帶入上式：

$$
\Rightarrow x(t) = \sum^\infin_{n = -\infin} x[n] sinc(\frac{w_s(t - nT_s)}{2 \pi})
$$

![](2019-05-06-20-32-44.png)

觀察上圖，可以發現 $x(n T_s) = x[n]$ ($\because h_r(t)$在 $n T_s$ 都為0，不干擾 $x[n]$)，其餘點都為內插求得