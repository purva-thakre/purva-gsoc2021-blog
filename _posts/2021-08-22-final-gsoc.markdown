---
layout: post
title:  "Summary of work done and description of deliverables"
date:   2021-08-22 10:38:29 -0500
categories: GSoC QuTiP
---
This post is supposed to serve as a permanent link to the work done as a part of
of GSoC 2021 project to add a general quantum gate decomposition in QuTip.

## Goal of the Project

Originally proposed plan was to be able to decompose some matrix describing a
qubit gate into a list of single-qubit and CNOT gates. Since the work is partially
done, this post itemizes the issues faced over the summer.

The deliverables are :

  - A function that is able to decompose a matrix describing a single-qubit
  gate into a description of known single qubit gates ([merged PR](https://github.com/qutip/qutip-qip/tree/master/src/qutip_qip/decompose))
  - Functions decomposing 2, 3 and other higher number of qubit gates ([draft PR - WIP](https://github.com/qutip/qutip-qip/pull/90))
      - Decomposition into two-level arrays and circuit described by gray code
      ordering ([closed PR - code is used above draft](https://github.com/qutip/qutip-qip/pull/86)).
      - Decomposition of two-qubit gates ([closed PR - code is used above draft](https://github.com/qutip/qutip-qip/pull/82)
      - [Other contributions](https://github.com/qutip/qutip-qip/pulls?q=is%3Apr+is%3Aclosed+author%3Apurva-thakre) to documentation

## Decomposition of Function Descriptions

![choice]({{ site.baseurl }}/assets/img/choices_main_decomposition_fn.png)





![sq]({{ site.baseurl }}/assets/img/flowchart_single_qubit.png)

![tq]({{ site.baseurl }}/assets/img/two_qubit_flowchart.png)

![trq]({{ site.baseurl }}/assets/img/three_qubit_flowchart.png)

![gq]({{ site.baseurl }}/assets/img/flowchart_general.png)
## Future Work
