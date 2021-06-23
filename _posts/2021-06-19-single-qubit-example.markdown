---
layout: post
title:  "Calculations for Single Qubit Gate Decomposition Example"
date:   2021-06-19 11:38:29 -0500
categories: GSoC QuTiP
---

Let's consider decomposing a single qubit gate into a decomposition described by other elementary single qubit gates { $$ \sigma_x, R_y(\theta), R_z(\theta) $$}. The decomposition is equivalent to the input single qubit gate upto some global phase factor.

Any $$ 1- $$ qubit unitary $$ U $$ is of the following form :

<div style="text-align: center"><img src="/assets/img/general_single.png" width="50%" height="50%"></div>


Single qubit elementary gates are the set of Pauli matrices along with identity { $$ \sigma_x, \sigma_y, \sigma_z, \sigma_0 $$}
and the set of rotation matrices formed via the matrices in the Pauli set { $$ R_x(\theta), R_y(\theta), R_z(\theta), R_0(\theta) $$}.

![pauli_rotation](/assets/img/pauli_rotation.png)

There exist known decomposition schemes to decompose an arbitrary $$ U $$ into a product of rotation matrices $$ R_y(\theta)$$ and $$ R_y(\theta)$$ or into a product of Pauli $$ X $$ and rotation matrices $$ R_y(\theta)$$ and $$ R_y(\theta)$$.

# ZYZ Decomposition

Suppose we want $$ U = R_z(\alpha) R_y(\theta) R_z(\beta)$$[^1].

Simplifying the expression for $$ R_z(\alpha) R_y(\theta) R_z(\beta) $$ gives :

![zyz_matrix](/assets/img/zyz_matrix.png)

Rewrite the elements of $$ U $$ as complex numbers. Let $$ a = x + iy$$ and $$ b = p + iq$$. Thus, $$ Re(a) = x, Im(a) = y, Re(b) = p, Im(b) = q $$.

<div style="text-align: center"><img src="/assets/img/u_real_imaginary.png" width="50%" height="50%"></div>

Comparing the element on each side leads to

![xypq_single](/assets/img/xypq_single.png)

Using basic trigonometry, the angles are calculated as

<div style="text-align: center"><img src="/assets/img/zyz_angle.png" width="50%" height="50%"></div>


In the end, an arbitrary single qubit $$ U $$ can be implemented in a quantum circuit as

![zyz_circuit](/assets/img/zyz_circuit.png)

# ABC decomposition
Suppose we want $$ U = A \sigma_x B \sigma_x C$$ where $$ ABC = \mathbb{I}$$.

If $$ ZYZ $$ decomposition has already been calculated then there's an easier workaround to describing $$ A, B$$ and $$C$$ in terms of $$ Rz$$ and $$ Ry$$ rotation matrices[^2].

![abc_general](/assets/img/abc_general.png)

$$ U = R_z(\alpha) R_y(\theta/2) \sigma_x R_y(-\theta/2) R_z(-(\alpha+\beta)/2) \sigma_x R_z(-\alpha+\beta)/2) = A \sigma_x B \sigma_x C$$

Here, $$\alpha, \theta$$ and $$ \beta $$ were calculated from $$ ZYZ $$ decomposition.

<div style="text-align: center"><img src="/assets/img/ABC_angle.png" width="50%" height="50%"></div>

## Example Calculations for Hadamard

![hadamard_real_im](/assets/img/hadamard_real_im.png)

Using above derived equations :

<div style="text-align: center"><img src="/assets/img/hadamard_angle.png" width="65%" height="65%"></div>

**ZYZ Decomposition**

![zyz_single](/assets/img/zyz_single_qubit.png)


**ABC Decomposition**

![abc_single](/assets/img/abc_single.png)

**Note :** For now, there's no way to start a discussion under a blogpost. While I figure out a way to change the default layout of a post, comments/suggestions are welcome [in the repository](https://github.com/purva-thakre/purva-blog/discussions) of this blog.  

# References
[^1]: *Lemma 4.1* of [Elementary Gates For Quantum Computation](https://arxiv.org/abs/quant-ph/9503016)

[^2]: *Lemma 4.3* of [Elementary Gates For Quantum Computation](https://arxiv.org/abs/quant-ph/9503016)
