---
title: 訊號與系統
---

# 訊號與系統

課程名稱：訊號與系統 (Signals and Systems)

授課教師：蔡尚澕

開課單位：電資共同

教科書：[Signals and Systems (2nd Edition)](https://www.amazon.com/Signals-Systems-2nd-Simon-Haykin/dp/0471164747)

## 傅立葉轉換入門

什麼是傅立葉轉換？

1. [But what is the Fourier Transform? A visual introduction.](https://www.youtube.com/watch?v=spUNpyF58BY)

傅立葉轉換公式推導

1. [工程數學三 7-2 傅立葉指數轉換式](https://www.youtube.com/watch?v=5zsTgO5p1j0)

## 傅立葉表示法對混合信號的應用

### Relating FT to DTFT

考慮如何將 discrete-time signal 和 continuous-time signal 做轉換，在這邊從 discrete-time signal 推回去 continuous-time signal，整體的流程：

1. discrete-time signal 轉成 temp signal (過度訊號)
2. temp signal 轉成 continuous-time signal

#### 離散訊號轉為過度訊號

complex sinusoids:

* continuous-time: $x(t) = e^{jwt}$
* discrete-time: $g[n] = e^{j\Omega n}$

##### DTFT

在之前就已經知道，對任意 discrete-time signal $x[n]$ 做 DTFT：$\chi(e^{j\Omega}) = \sum_{n = - \infin}^{\infin}x[n]e^{-j\Omega n}$

##### 取樣

每隔 $T_s$ 時間間隔去取樣 continuous 的訊號，假設 $x[n]$ 等於取樣過後的 $x(t)$:

* $x(nT_s) = x[n] \Rightarrow t = nT_s$
* $e^{j\Omega n} = e^{jwt} \Rightarrow e^{j\Omega n} = e^{jwT_sn} \Rightarrow \Omega = wT_s$

將 $\Omega = w T_s$ 帶入公式：$\chi(e^{j\Omega})|_{\Omega = w T_s} = \sum_{n = - \infin}^{\infin}x[n]e^{-j w T_s n}$，我們稱這個頻域的訊號叫<mark>過度訊號</mark>：$\chi_{\delta}(jw)$

##### FT

過渡訊號 $\chi_{\delta}(jw)$ 在時域的表示可以用逆傅立葉轉換得到，且因為有 linearity 的性質，所以可以看成：

$$
x_\delta(t) = F^{-1}\{\chi_\delta(jw)\} =  F^{-1}\{ \sum_{n = - \infin}^{\infin}x[n]e^{-j w T_s n} \} = \sum_{n = - \infin}^{\infin} x[n] F^{-1}\{ e^{-j w T_s n} \}
$$

由之前的結論：

$$
\delta (t) \xleftrightarrow{FT} 1 \\
\delta (t - t_0) \xleftrightarrow{FT} e^{-j w t_0} 
$$

上面的 $T_s n$ 可以看成一個 time shift $t_0$，於是就可以得到：

$$
x_\delta(t) = \sum_{n = - \infin}^{\infin} x[n] \delta (t - n T_s)
$$

觀察 delta function 後可以發現，$x_\delta(t)$ 就是 continuous-time 的 $x[n]$ 表示法，在非 $nT_s$ 的地方都為 0，而它的 fourier transform pair 是：

$$
\chi_\delta (j\omega) = \sum_{n = - \infin}^{\infin}x[n]e^{-j w T_s n}
$$

##### 圖例

![](2019-04-30-00-34-18.png)

原先為左上方的 Kronecker delta (discrete-time)，透過 DTFT、取樣、FT 後轉成 Dirac delta (continuous-time)，即為過度訊號

#### 過度訊號轉為連續訊號

![](2019-04-30-01-21-33.png)

##### Sampling (Time)

$$
x_\delta (t) = \sum_{n = - \infin}^{\infin} x[n] \delta (t - nT_s) \\
\xRightarrow{x[n] = x(n T_s)} x_\delta (t) = \sum_{n = - \infin}^{\infin} x(n T_s) \delta (t - nT_s) \\
\xRightarrow{t = nT_s} x_\delta (t) = \sum_{n = - \infin}^{\infin} x(t) \delta (t - nT_s) \\
\Rightarrow x_\delta (t) = x(t) \sum_{n = - \infin}^{\infin} \delta (t - nT_s) \\
\Rightarrow x_\delta (t) = x(t) p(t) \,\, where \,\, p(t) = \sum_{n = - \infin}^{\infin} \delta (t - nT_s)
$$

由 $p(t)$ 的式子可以觀察到它是一個 **impulse train**，且由推導可以知道 $p(t)$ 是用來關聯 $x(t)$ 與 $x_\delta(t)$ 的關係式，透過 $p(t)$ 及的式子可以從 $x(t)$ 得到 $x_\delta(t)$，再由前面推導的過度訊號轉換成離散訊號即可達成 sampling 的目的

##### 圖例 (Time)

![](2019-04-30-01-17-27.png)

$x_\delta(t)$ 為 $x(t)$ 與 $p(t)$ 的 time-domain 做相乘而得

##### Sampling (Frequency)

在時域做相乘等於在頻域做convolution

$$
\chi_\delta (j w) = \frac{1}{2 \pi} \chi(jw) * P(j w)
$$

對 impulse train $P(jw)$ 做 FT 仍然是 impulse train (詳見前面)，sample frequency $w_s = 2 \pi / T_s$：

$$
\chi_\delta (j w) = \frac{1}{\cancel{2\pi}} \chi(jw) * \frac{\cancel{2\pi}}{T_s} \sum_{k = - \infin}^{\infin} \delta (w - kw_s)
$$

因為 convolution delta function 所以可以簡單地知道結果：

$$
\chi_\delta (j w) = \frac{1}{T_s} \sum_{k = -\infin}^{\infin} \chi(jw - jkw_s)
$$

##### 圖例 (frequency)

![](2019-04-30-02-01-15.png)

對 sample 後的訊號做 FT 的結果(過度訊號)是將原訊號複製無限多份，且強度變為 $1/T_s$ (右上圖)

對 sample 後的訊號做 DTFT 的結果，可以透過 $x[n] \xleftrightarrow{DTFT} \chi (e^{j \Omega}) = \chi_\delta (jw) |_{w = \Omega / T_s}$ 得到 (右下圖)

> time: discrete、frequency: periodic，可以用 sampling 來理解
> 
> $\because \Omega = w_s T_s = 2 \pi f_s T_s = 2 \pi \frac{1}{\cancel{T_s}} \cancel{T_s}$，固定每 $2\pi$ 為一週期所以是 periodic

##### Sampling Aliasing

$w_s = 2\pi / T_s$ 當 $T_s$ 提升時會使 $w_s$ 下降，當 $w_s \lt 2 W$ 時會產生 overlap。

![](2019-04-30-01-54-04.png)

當取樣 overlap 時即無法再還原回原訊號，因此取樣頻率要足夠高。

> 取樣頻率也不可過大，原先 1MHz 即可取的用 100MHz 來取，代表多取了 100 倍的點，複雜度上升，所需記憶體也上升