---
layout: post
title:  "QuTiP Function to Decompose a Single Qubit Gate"
date:   2021-07-19 11:38:29 -0500
categories: GSoC QuTiP
---
## Description of QuTiP function

If provided a matrix representing a single-qubit gate, the equivalent circuit
can be determined by implementing one of the methods defined in
[`qutip_qip/decompose`](https://github.com/qutip/qutip-qip/tree/master/src/qutip_qip/decompose) module. Currently, any $$1$$-qubit gate can be decomposed into a description described by rotation matrices or rotation matrices $$\textrm{ZYZ}$$ and a Pauli $$\textrm{X}$$.

Suppose we want to decompose following random unitary acting on a single qubit :

 $$U = \begin{pmatrix}+0.94798071+0.00789819i & -0.08545385-0.30654173i\\\ -0.05739092-0.31301194i & +0.84882368-0.42217078i\end{pmatrix}$$

The possibilities for decomposing above $$U$$ are as shown below. Other decompsoitions to be added soon[^1].

**$$\textrm{\textbf{ZYZ}}$$ Example**

$$ U = \textrm{R}_z(\alpha) \textrm{R}_y(\theta) \textrm{R}_z(\beta)$$[^2]

{% highlight python linenos%}
# Initialize the quantum circuits for the 1-qubit rotation gates
quantum_circuit = QubitCircuit(1, reverse_states=False)

# Decompose the matrix
gate_list = decompose_one_qubit_gate(input_gate, "ZYZ")

# Create a circuit of the gates forming input matrix
quantum_circuit.add_gates(gate_list)

# Circuit diagram of above decomposition
quantum_circuit.png
{% endhighlight %}

The circuit diagram of this decomposition is

![zyz_output]({{ site.baseurl }}/assets/img/dec_one_qubit_zyz.png)

The output of above circuit is $$U_{out}$$ which when compared to the input
has an average fidelity of $$1.0000000000000002$$.

$$U_{out} = \begin{pmatrix}0.948+0.008𝑗 & −0.085−0.307𝑗\\\ −0.057−0.313𝑗 & 0.849−0.422𝑗\end{pmatrix}$$

**$$\textrm{\textbf{ZXZ}}$$ Example**

By mapping $$\textrm{R}_y(\theta)$$ to $$\textrm{R}_x(\theta)$$, result from \textrm{ZYZ} decomposition[^2] can be extended to another combination of rotation matrices.

$$ U = \textrm{R}_z(\alpha) \textrm{R}_x(\theta) \textrm{R}_z(\beta)$$

Change method string `ZYZ` to `ZXZ` in previously described decomposition.

{% highlight python linenos%}
# Decompose the matrix
gate_list = decompose_one_qubit_gate(input_gate, "ZXZ")
{% endhighlight %}

The circuit diagram of this decomposition is

![zxz_output]({{ site.baseurl }}/assets/img/dec_one_qubit_zxz.png)

The output of above circuit is $$U_{out}$$ which when compared to the input
has an average fidelity of $$1.0$$.

$$U_{out} = \begin{pmatrix}+0.948+0.008𝑗 & −0.085−0.307𝑗\\\ −0.057−0.313𝑗 & +0.849−0.422𝑗\end{pmatrix}$$

**$$\textrm{\textbf{ABC}}$$ Example**

$$ U = \textrm{A} \sigma_x \textrm{B} \sigma_x \textrm{C}$$[^3]

Change method string `ZXZ` to `ZYZ_PauliX` in previously described decomposition.

{% highlight python linenos%}
# Decompose the matrix
gate_list = decompose_one_qubit_gate(input_gate, "ZYZ_PauliX")
{% endhighlight %}

The circuit diagram of this decomposition is

![abc_output]({{ site.baseurl }}/assets/img/dec_one_qubit_abc.png)

The output of above circuit is $$U_{out}$$ which when compared to the input
has an average fidelity of $$1.0$$.

$$U_{out} = \begin{pmatrix}+0.948+0.008𝑗 & −0.085−0.307𝑗\\\ −0.057−0.313𝑗 & +0.849−0.422𝑗\end{pmatrix}$$

**Note :** For now, there's no way to start a discussion under a blogpost. While
I figure out a way to change the default layout of a post, comments/suggestions are welcome [in the repository](https://github.com/purva-thakre/purva-blog/discussions) of this blog.  

# References
[^1]: Other possible combinations will be added after a general decomposition method
      has been added. This is mostly due to lack of weeks needed before the projected end
      of the GSoC program.

[^2]: *Lemma 4.1* of [Elementary Gates For Quantum Computation](https://arxiv.org/abs/quant-ph/9503016)

[^3]: *Lemma 4.3* of [Elementary Gates For Quantum Computation](https://arxiv.org/abs/quant-ph/9503016)
