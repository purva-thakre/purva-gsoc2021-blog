---
layout: post
title:  "Calculations for Single Qubit Gate Decomposition Example"
date:   2021-06-19 11:38:29 -0500
categories: GSoC QuTiP
---

Let's consider decomposing a single qubit gate into a decomposition described by other elementary single qubit gates { $$ \sigma_x,\, \textrm{R}_y(\theta),\, \textrm{R}_z(\theta) $$}. The decomposition is equivalent to the input single qubit gate upto some global phase factor.

Any $$ 1$$-qubit unitary $$ U $$ is of the following form :

<div style="text-align: center"><img src="{{ site.baseurl }}/assets/img/general_single.png" width="50%" height="50%"></div>


Single qubit elementary gates are the set of Pauli matrices along with identity { $$ \sigma_x, \, \sigma_y, \, \sigma_z, \, \sigma_0 $$}
and the set of rotation matrices formed via the matrices in the Pauli set { $$ \textrm{R}_x(\theta), \, \textrm{R}_y(\theta), \, \textrm{R}_z(\theta), \, \textrm{R}_0(\theta) $$}.

![pauli_rotation]({{ site.baseurl }}/assets/img/pauli_rotation.png)

There exist known decomposition schemes to decompose an arbitrary $$ U $$ into a product of rotation matrices $$ \textrm{R}_y(\theta)$$ and $$ \textrm{R}_y(\theta)$$ or into a product of Pauli $$ \textrm{X} $$ and rotation matrices $$ \textrm{R}_y(\theta)$$ and $$ \textrm{R}_z(\theta)$$.

**$$\textrm{\textbf{ZYZ}}$$ Decomposition**

Suppose we want $$ U = \textrm{R}_z(\alpha) \textrm{R}_y(\theta) \textrm{R}_z(\beta)$$[^1].

Simplifying the expression for $$ \textrm{R}_z(\alpha) \textrm{R}_y(\theta) \textrm{R}_z(\beta) $$ gives :

<div style="text-align: center"><img src="{{ site.baseurl }}/assets/img/zyz_matrix.png" width="80%" height="80%"></div>

Rewrite the elements of $$ U $$ as complex numbers.

Let $$ a = x + iy$$ and $$ b = p + iq$$. Thus, $$ Re(a) = x, \, Im(a) = y, \, Re(b) = p, \, Im(b) = q $$.

<div style="text-align: center"><img src="{{ site.baseurl }}/assets/img/u_real_imaginary.png" width="50%" height="50%"></div>

Equating each real and complex element of $$ U $$ and $$ \textrm{R}_z(\alpha) \textrm{R}_y(\theta) \textrm{R}_z(\beta) $$ leads to

<div style="text-align: center"><img src="{{ site.baseurl }}/assets/img/xypq_single.png" width="80%" height="80%"></div>

Using basic trigonometry, the angles are calculated as

<div style="text-align: center"><img src="{{ site.baseurl }}/assets/img/zyz_angle.png" width="40%" height="40%"></div>


In the end, an arbitrary single qubit $$ U $$ can be implemented in a quantum circuit as

![zyz_circuit]({{ site.baseurl }}/assets/img/zyz_circuit.png)

**$$\textrm{\textbf{ABC}}$$ Decomposition**

Suppose we want $$ U = \textrm{A} \sigma_x \textrm{B} \sigma_x \textrm{C}$$ where $$ \textrm{ABC} = \textrm{I}$$.

If $$ \textrm{ZYZ} $$ decomposition has already been calculated then there's an easier workaround to describing $$ \textrm{A}, \, \textrm{B}$$ and $$\textrm{C}$$ in terms of $$ \textrm{R}_z$$ and $$\textrm{R}_y$$ rotation matrices[^2].

![abc_general]({{ site.baseurl }}/assets/img/abc_general.png)

$$ U = \textrm{R}_z(\alpha) \textrm{R}_y \left(\frac{\theta}{2} \right) \sigma_x \textrm{R}_y \left(\frac{-\theta}{2} \right) \textrm{R}_z \left(\frac{-(\alpha+\beta)}{2} \right) \sigma_x \textrm{R}_z \left(\frac{-\alpha+\beta}{2} \right) = \textrm{A} \sigma_x \textrm{B} \sigma_x \textrm{C}$$

Here, $$\alpha, \, \theta$$ and $$ \beta $$ were calculated from $$ \textrm{ZYZ} $$ decomposition.

<div style="text-align: center"><img src="{{ site.baseurl }}/assets/img/ABC_angle.png" width="40%" height="40%"></div>

**Example Calculations for Hadamard**

<div style="text-align: center"><img src="{{ site.baseurl }}/assets/img/hadamard_real_im.png" width="80%" height="80%"></div>

An additional step is needed to multiply above matrix by some common global phase
if the individual elements of the matrix are to be used to find the rotation
matrix angles. Here, we multiply above $$ \textrm{H} $$ by complex $$ i $$.

This is mostly because if $$ a = 1$$ then $$ a^* = 1$$ which does not agree with
the value $$ a^* = -1$$ in the matrix. This, if $$ \textrm{H} $$ is multiplied
by complex $$ i $$, then $$ a = i$$ then $$ a^* = -i$$.

**$$\textrm{ZYZ}$$ Decomposition**

The output provided below is off by some global phase after the matrix was multiplied by some global phase before decomposition.

<div style="text-align: center"><img src="{{ site.baseurl }}/assets/img/zyz_single_qubit.png" width="70%" height="70%"></div>


**$$\textrm{ABC}$$ Decomposition**

The output provided below is off by some global phase after the matrix was multiplied by some global phase before decomposition.

![abc_single]({{ site.baseurl }}/assets/img/abc_single.png)

**Note :** For now, there's no way to start a discussion under a blogpost. While I figure out a way to change the default layout of a post, comments/suggestions are welcome [in the repository](https://github.com/purva-thakre/purva-blog/discussions) of this blog.  

# References
[^1]: *Lemma 4.1* of [Elementary Gates For Quantum Computation](https://arxiv.org/abs/quant-ph/9503016)

[^2]: *Lemma 4.3* of [Elementary Gates For Quantum Computation](https://arxiv.org/abs/quant-ph/9503016)
