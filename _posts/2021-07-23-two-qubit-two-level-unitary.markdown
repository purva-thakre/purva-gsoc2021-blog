---
layout: post
title:  "Progress towards defining a general decomposition method"
date:   2021-08-02 11:38:29 -0500
categories: GSoC QuTiP
---
## Decomposing to two-level arrays
Suppose we want to describe a circuit for a general $$ n$$-qubit unitary $$ U $$
in terms of $$ 2$$-qubit $$ CNOT and single qubit gates.

 The first step is to decompose the input into a description of two-level unitary
 arrays. Here, a two-level unitary is a $$ n$$-qubit unitary which is similar to
 a $$ n$$-qubit identity gate except for 4 positions that act non-trivially.

 Shown below is a $$ 2$$-qubit gate which we want to decompose into a description of two-level unitaries[^1].

 $$U = \begin{pmatrix}0.5 & 0.5 & 0.5 & 0.5 \\\ 0.5 & 0.5i & -0.5 & -0.5i \\\ 0.5 & -0.5 & 0.5 & -0.5 \\\ 0.5 & -0.5i & -0.5 & 0.5i \end{pmatrix}$$

A private function defined for the general decomposition module is able to do so,
as shown below :

{% highlight python linenos%}
from qutip_qip.decompose.decompose_general_qubit_gate _decompose_to_two_level_arrays
np.set_printoptions(suppress=True)

# define the Qobj for QuTiP
U_array = np.multiply(0.5, np.array([[1,1,1,1],[1,1j,-1,-1j],[1,-1,1,-1],[1,-1j,-1,1j]]))
U = Qobj(U_array, [[2] * 2] * 2)

# Get the list of arrays in reverse order
two_level = _decompose_to_two_level_arrays(U, 2)
print(*two_level, sep ='\n\n')
{% endhighlight %}

The list returned in returned in the reverse order i.e. last gate in the decomposition
process is the first item in the list and so on. As can be observed from one of the outputs shown below, there's a smaller block of single qubit arrays that act non-trivially compared to an identity gate. Here, the non-trivial index basis labels are '10' and '11'.

$$U_6 = \begin{pmatrix}1 & 0 & 0 & 0 \\\ 0 & 1& 0 & 0 \\\ 0 & 0 & 0.707 & 0.707j \\\ 0 & 0 &  -0.707 & +0.707j \end{pmatrix}$$

The rest of the gates in the list and their non-trivial indices are shown below :

| Gate     | Control Index Label | Target Index Label |
| :----:   |    :----:           |       :----:       |
| $$U_6$$  | '10'                | '11'               |
| $$U_5$$  | '01'                | '11'               |
| $$U_4$$  | '01'                | '10'               |
| $$U_3$$  | '00'                | '11'               |
| $$U_2$$  | '00'                | '10'               |
| $$U_1$$  | '00'                | '01'               |

These gates can be decomposed via gray code ordering and decomposing a single qubit
gate from the values at non-trivial indices (complete details for above gate
to be added in another blog post).


**Note :** For now, there's no way to start a discussion under a blogpost. While
I figure out a way to change the default layout of a post, comments/suggestions are welcome [in the repository](https://github.com/purva-thakre/purva-blog/discussions) of this blog.  

## References
[^1]: Exercise 4.37 in Nielsen, M. A., &amp; Chuang, I. L. (2019). Quantum computation   and quantum information. Cambridge University Press.

[^2]: Exercise 4.39 in Nielsen, M. A., &amp; Chuang, I. L. (2019). Quantum computation   and quantum information. Cambridge University Press.
