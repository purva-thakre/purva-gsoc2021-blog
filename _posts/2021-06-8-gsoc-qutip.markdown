---
layout: post
title:  "GSoC 2021 QuTiP"
date:   2021-06-8 11:38:29 -0500
categories: [GSoC]
---
I will be contributing to QuTiP as a part of GSoC 2021. My contributions will be
to the [`qutip-qip`](https://github.com/qutip/qutip-qip) package.

The main goal for this project is to make changes to [how gates are resolved](https://github.com/qutip/qutip-qip/blob/master/src/qutip_qip/circuit.py#L529) when provided a small universal set. The gates can be decomposed
based as a pre-defined sequential gate action. If [`resolve_gates`](https://github.com/qutip/qutip-qip/blob/master/src/qutip_qip/circuit.py#L1025) is used to decompose some $n \by n$ unitary matrix, the process is bound to fail if the
gate decomposition is not pre-defined.

While 
