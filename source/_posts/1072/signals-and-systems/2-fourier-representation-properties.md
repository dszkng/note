# 傅立葉表示法的性質

## Linearity

$$ z(t)=ax(t)+by(t) \xleftrightarrow{FT} Z(jw)=aX(jw)+bY(jw) $$

$$ z(t)=ax(t)+by(t) \xleftrightarrow{FS;w_0} Z[k]=aX[k]+bY[k] $$

$$ z[n]=ax[n]+by[n] \xleftrightarrow{DTFT} Z(e^{j\Omega})=aX(e^{j\Omega})+bY(e^{j\Omega}) $$

$$ z[n]=ax[n]+by[n] \xleftrightarrow{DTFS;\Omega_0} Z[k]=aX[k]+bY[k] $$

## Symmetry

$$
\begin{aligned}
    X^*(jw) &= [\int^{\infin}_{-\infin} x(t) e^{-jwt} dt]^* \\
    &= \int^{\infin}_{-\infin} x^*(t) e^{jwt} dt \\
    &= \int^{\infin}_{-\infin} x(t) e^{-j(-w)t} dt \text{ x(t) is real}
\end{aligned}
$$

| Representation | Real Signal                          | Imaginary Signal                      |
| -------------- | ------------------------------------ | ------------------------------------- |
| FT             | $X^*(jw) = X(-jw)$                   | $X^*(jw) = -X(-jw)$                   |
| FS             | $X^*[k] = X[-k]$                     | $X^*[k] = -X[-k]$                     |
| DTFT           | $X^*(e^{j\Omega}) = X(e^{-j\Omega})$ | $X^*(e^{j\Omega}) = -X(e^{-j\Omega})$ |
| DTFS           | $X^*[k] = X[-k]$                     | $X^*[k] = -X[-k]$                     |

## Convolution

$$ x(t) * z(t) \xleftrightarrow{FT} X(jw) Z(jw) $$

$$ x(t) \otimes z(t) \xleftrightarrow{FS;w_0} T X[k] Z[k] $$

$$ x[n] * z[n] \xleftrightarrow{DTFT} X(e^{j\Omega}) Z(e^{j\Omega}) $$

$$ x[n] \otimes z[n] \xleftrightarrow{DTFS;\Omega_0} N X[k] Z[k] $$

$$ \otimes: \text{circular convolution} $$

### Gibbs Phenomenon

在前面FS提到的[例子](/NCTU-Coursenote/1072/signals-and-systems/1-fourier-representation/#square-wave-v2)中，會有Overshot的原因，就是因為在頻域中乘上一個 windows，所以在時域就要 convolution 一個 sinc function，所以才導致有明顯的邊緣

## Time Shift

$$ x(t - t_0) \xleftrightarrow{FT} e^{-jwt_0} X(jw) $$

$$ x(t - t_0) \xleftrightarrow{FS;w_0} e^{-jkw_0t_0} X[k] $$

$$ x[n - n_0] \xleftrightarrow{DTFT} e^{-j\Omega n_0} X(e^{j\Omega}) $$

$$ x[n - n_0] \xleftrightarrow{DTFS;\Omega_0} e^{-jk\Omega_0 n_0} X[k] $$

## Frequency Shift

$$ e^{j \gamma t} x(t) \xleftrightarrow{FT} X(j(w-\gamma)) $$

$$ e^{j k_0 w_0 t} x(t) \xleftrightarrow{FS;w_0} X[k - k_0] $$

$$ e^{j \Gamma n} x[n] \xleftrightarrow{DTFT} X(e^{j(\Omega - \Gamma)}) $$

$$ e^{j k_0 \Omega_0 n} x[n] \xleftrightarrow{DTFS;\Omega_0} X[k - k_0] $$

## Multiplication

$$ x(t) z(t) \xleftrightarrow{FT} \frac{1}{2\pi} X(jw) * Z(jw) $$

$$ x(t) z(t) \xleftrightarrow{FS;w_0} X[k] * Z[k] $$

$$ x[n] z[n] \xleftrightarrow{DTFT} \frac{1}{2\pi} X(e^{j\Omega}) \otimes Z(e^{j\Omega}) $$

$$ x[n] z[n] \xleftrightarrow{DTFS;\Omega_0} X[k] \otimes Z[k] $$

$$ \otimes: \text{periodic convolution} $$

## Scaling

$$ z(t) = x(at) \xleftrightarrow{FT} \frac{1}{|a|} X(\frac{jw}{a}) $$

* If $x(t)$ is a periodic signal
  * $z(t) = x(at)$ is also periodic
* If $x(t)$ fundamental period $T$
  * $z(t)$ has fundamental period $T/a$
* If $x(t)$ fundamental frequency $w_0$
  * $z(t)$ has fundamental frequency $aw_0$

$$ Z[k] = \frac{a}{T} \int^{T/a}_0 z(t) e^{- j k w_0 t} dt $$

透過變數變換可以發現 $Z[k] = X[k]$ (兩者基頻不同，k 指的是第幾倍頻)

$$ z(t) = x(at) \xleftrightarrow{FS; aw_0} Z[k] = X[k], a > 0 $$

## Parseval

能量守恆

$$
\begin{aligned}
    &\int^{\infin}_{-\infin} |x(t)|^2 dt \\
    &= \int^{\infin}_{-\infin} x(t) x^*(t) dt \\
    &= \int^{\infin}_{-\infin} x(t) (\frac{1}{2\pi} \int^{\infin}_{-\infin} X^*(jw) e^{-jwt} dw) dt \\
    &= \frac{1}{2\pi} \int^{\infin}_{-\infin} X^*(jw) X(jw) dw
\end{aligned}
$$

| Representations | Parseval Relationships                                                                                                    |
| --------------- | ------------------------------------------------------------------------------------------------------------------------- |
| FT              | $\int^{\infin}_{-\infin} \vert x(t) \vert ^2 dt = \frac{1}{2\pi} \int^{\infin}_{-\infin} \vert X(jw) \vert ^2 dw$         |
| FS              | $\frac{1}{T} \int^{T}_{0} \vert x(t) \vert ^2 dt = \sum^{\infin}_{k = -\infin} \vert X[k] \vert^2$                        |
| DTFT            | $\sum^{\infin}_{k = -\infin} \vert X[n] \vert^2 = \frac{1}{2\pi} \int^{\pi}_{-\pi} \vert x(e^{j\Omega}) \vert ^2 d\Omega$ |
| DTFS            | $\frac{1}{N} \sum^{N-1}_{n=0} \vert X[n] \vert^2 = \sum^{N-1}_{n=0} \vert X[k] \vert^2$                                   |

## Duality

$$
\begin{aligned}
    f(t) \xleftrightarrow{FT} F(jw) &\xLeftrightarrow{Duality} F(jt) \xleftrightarrow{FT} 2\pi f(-w) \\
    x[n] \xleftrightarrow{DTFS; w_0} X[k] &\xLeftrightarrow{Duality} X[n] \xleftrightarrow{DTFS; w_0} (1/N) x[-k] \\
    x[n] \xleftrightarrow{DTFT} X(e^{j\Omega}) &\xLeftrightarrow{Duality} X(e^{jt}) \xleftrightarrow{FS; 1} x[-k]
\end{aligned}
$$

DTFT 與 FS 因為頻域與時域沒有同步(一個是離散一個是連續的)

FS: $z(t) = \sum^{\infin}_{k=-\infin} Z[k] e^{jkw_0t}$，$z(t)$ 以 $T$ 為週期

DTFT: $X(e^{j\Omega}) = \sum^{\infin}_{k = -\infin} x[n] e^{-j\Omega n}$，$X(e^{j\Omega})$ 以 $2\pi$ 為週期

若要滿足 duality，必須 $T = 2\pi \Rightarrow w_0 = 2\pi / T = 1$