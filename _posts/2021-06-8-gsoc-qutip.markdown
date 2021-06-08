---
layout: post
title:  "GSoC 2021 : Gate Decomposition for QuTiP"
date:   2021-06-8 11:38:29 -0500
categories: GSoC QuTiP
---
I will be contributing to QuTiP as a part of GSoC 2021. My contributions will be
to the [`qutip-qip`](https://github.com/qutip/qutip-qip) package.

The main goal for this project is to make changes to [how gates are resolved](https://github.com/qutip/qutip-qip/blob/master/src/qutip_qip/circuit.py#L529) when provided a small universal set. The gates can be decomposed
based as a pre-defined sequential gate action. If [`resolve_gates`](https://github.com/qutip/qutip-qip/blob/master/src/qutip_qip/circuit.py#L1025) is used to decompose some $$ n \times n $$ unitary matrix, the process is bound to fail if the gate decomposition sequence is not pre-defined.

While defining a general decomposition scheme for a $$ n \times n $$ unitary is
non-trivial, the process can be broken down based on the number of qubits being
acted upon by the gate. For example, a $$ 2 \times 2 $$ unitary gate will act upon a single qubit. There exist [pre-defined decomposition](https://arxiv.org/abs/quant-ph/9503016) schemes to be able to decompose such a unitary in terms of a Bloch Rotation Vector parameters, as a product of $$ ZY $$ rotations, $$ XY $$ rotations and so on.

As the size of the unitary increases, the number of elementary gates needed in
the decomposition sequence increases as well. The elementary gates or so called universal gates are chosen according to the [Solovay-Kitaev theorem](https://arxiv.org/abs/quant-ph/0505030). Some of the decomposition schemes for higher dimensional gates are described in these papers : [1](https://arxiv.org/abs/quant-ph/0504100), [2](https://www.osti.gov/biblio/889415), [3](https://arxiv.org/abs/quant-ph/0311008) and many more. The hope is to implement as many general decomposition schemes as possible.

If time permits, additional module could be introduced to [optimize the decomposition schemes](https://arxiv.org/abs/1210.0974). Any gate decomposition scheme is useful for fault tolerant quantum computing particularly Steane code where quantum gates are decomposed as a combination of Clifford gates and T-gates. The linked optimization scheme strives to decrease the number of non-fault tolerant T-gates upto a certain depth. As decomposition functions are introduced in `qip`, there will be a need to search for optimization schemes based on the chosen universal gate sets. 
