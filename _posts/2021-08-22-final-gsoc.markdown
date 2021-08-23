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
qubit gate into a list of single-qubit and CNOT gates. Since **the work is partially done**, this post itemizes the issues faced over defining a decomposition scheme.

The deliverables are :

  - A function that is able to decompose a matrix describing a single-qubit
  gate into a description of known single qubit gates ([merged PR](https://github.com/qutip/qutip-qip/pull/67))
  - Functions decomposing 2, 3 and other higher number of qubit gate matrices ([draft PR - WIP](https://github.com/qutip/qutip-qip/pull/90))
      - Decomposition into two-level arrays and circuit described by gray code
      ordering ([closed PR - code is used above draft](https://github.com/qutip/qutip-qip/pull/86)).
      - Decomposition of two-qubit gates ([closed PR - code is used above draft](https://github.com/qutip/qutip-qip/pull/82)
      - Other minor contributions to documentation - [link1](https://github.com/qutip/qutip-qip/pull/54), [link2](https://github.com/qutip/qutip-qip/pull/56),
      [link3](https://github.com/qutip/qutip-qip/pull/58), [link4](https://github.com/qutip/qutip-qip/pull/59), [link5](https://github.com/qutip/qutip-qip/pull/65).


## Description of decomposition functions

The main decomposition function uses different methods to be able to decompose
some qubit gate matrix depending on the number of qubits being acted upon. For example, if number of qubits > 3, then ancilla qubits are inserted into the circuit to get the required decomposition. This is mainly because the scheme without ancilla qubits was utilizing a recursion theme but there were some issues
with how the control and target qubits were calculated recursively (discussed at the end of this post).

![choice]({{ site.baseurl }}/assets/img/choices_main_decomposition_fn.png)


When the gate to be decomposed is a single-qubit gate then the matrix can be
decomposed into either of the rotation matrices as described [here](https://purva-thakre.github.io/purva-blog/gsoc/qutip/single-qubit-in-qutip/).

![sq]({{ site.baseurl }}/assets/img/flowchart_single_qubit.png)

For matrices beyond single qubit gates, the matrix is first decomposed into
two-level arrays. A circuit for these two-level gates can be obtained via gray code ordering scheme. This scheme finds the target qubit and control qubits by utilizing the indices where the two-level array acts non-trivially.

<span style="color:red">**NOTE:** </span> Right now, the general method decomposing two-level array matrices into single qubit gates is not in correct order. This is due to the output being a nested dictionary comprising of a mix of
other dictionaries and lists. The output form of intermediate functions was not
consistent and the list of gates for each two-level gates was not correctly appended.

### Two-qubit gate decomposition

![tq]({{ site.baseurl }}/assets/img/two_qubit_flowchart.png)

As discussed briefly [in a previous post](https://purva-thakre.github.io/purva-blog/gsoc/qutip/two-qubit-two-level-unitary/), a two-qubit gate matrix can be described by a product of 6 two-level array matrices. In reversed order, the non-trivial positions in each two-level array are as shown below :

$$U_5 = \begin{pmatrix}1 & 0 & 0 & 0 \\\ 0 & 1& 0 & 0 \\\ 0 & 0 & \textcolor{blue}{\textbf{x}} & \textcolor{blue}{\textbf{x}} \\\ 0 & 0 &  \textcolor{blue}{\textbf{x}} & \textcolor{blue}{\textbf{x}} \end{pmatrix}$$, $$U_4 = \begin{pmatrix}1 & 0 & 0 & 0 \\\ 0 & \textcolor{blue}{\textbf{x}}& 0 & \textcolor{blue}{\textbf{x}} \\\ 0 & 0 & 1 & 0 \\\ 0 & \textcolor{blue}{\textbf{x}} &  0 & \textcolor{blue}{\textbf{x}} \end{pmatrix}$$, $$U_3 = \begin{pmatrix}1 & 0 & 0 & 0 \\\ 0 & \textcolor{blue}{\textbf{x}}& \textcolor{blue}{\textbf{x}} & 0 \\\ 0 & \textcolor{blue}{\textbf{x}} & \textcolor{blue}{\textbf{x}} & 0 \\\ 0 & 0 &  0 & 1 \end{pmatrix}$$

$$U_2 = \begin{pmatrix}\textcolor{blue}{\textbf{x}} & 0 & 0 & \textcolor{blue}{\textbf{x}} \\\ 0 & 1& 0 & 0 \\\ 0 & 0 & 1 & 0 \\\ \textcolor{blue}{\textbf{x}} & 0 &  0 & \textcolor{blue}{\textbf{x}} \end{pmatrix}$$, $$U_1 = \begin{pmatrix}\textcolor{blue}{\textbf{x}} & 0 & \textcolor{blue}{\textbf{x}} & 0 \\\ 0 & 1& 0 & 0 \\\ \textcolor{blue}{\textbf{x}} & 0 & \textcolor{blue}{\textbf{x}} & 0 \\\ 0 & 0 &  0 & 1 \end{pmatrix}$$, $$U_0 = \begin{pmatrix}\textcolor{blue}{\textbf{x}} & \textcolor{blue}{\textbf{x}} & 0 & 0 \\\ \textcolor{blue}{\textbf{x}} & \textcolor{blue}{\textbf{x}}& 0 & 0 \\\ 0 & 0 & 1 & 0 \\\ 0 & 0 &  1 & 0 \end{pmatrix}$$

The gray code ordering scheme for mapping from one non-trivial index to another
are as shown below. If there's only 1 gate from the gray code then a two-level gate can be directly added in the circuit. Otherwise, CNOT gates map from one trivial position to another and the last item in first list will give the required
two-level gate.

| Gate     | Control State | Target State |Gates from Gray Code|
| :----:   |    :----:     |       :----: |     :----:               |
| $$U_5$$  | '10'          | '11'         |(['11', '10'], 1)         |
| $$U_4$$  | '01'          | '11'         |(['01', '11'], 1)         |
| $$U_3$$  | '01'          | '10'         |(['01', '11', '10'], ['11', '01'], 3)|
| $$U_2$$  | '00'          | '11'         |(['00', '01', '11'], ['01', '00'], 3)|
| $$U_1$$  | '00'          | '10'         |(['00', '01', '11', '10'], ['11', '01', '00'], 5)|
| $$U_0$$  | '00'          | '01'         |(['00', '01'], 1)|

The gate controls and targets are as listed below :
{% highlight python linenos%}
# U5
{0: {'controls =': [0], 'control_value =': ['1'], 'targets =': [1]}}

# U4
{0: {'controls =': [1], 'control_value =': ['1'], 'targets =': [0]}}

# U3
[{0: {'controls =': [1], 'control_value =': ['1'], 'targets =': [0]},
  1: {'controls =': [0], 'control_value =': ['1'], 'targets =': [1]}},
 {2: {'controls =': [1], 'control_value =': ['1'], 'targets =': [0]}}]

# U2
[{0: {'controls =': [0], 'control_value =': ['0'], 'targets =': [1]},
  1: {'controls =': [1], 'control_value =': ['1'], 'targets =': [0]}},
 {2: {'controls =': [0], 'control_value =': ['0'], 'targets =': [1]}}]

# U1
[{0: {'controls =': [0], 'control_value =': ['0'], 'targets =': [1]},
  1: {'controls =': [1], 'control_value =': ['1'], 'targets =': [0]},
  2: {'controls =': [0], 'control_value =': ['1'], 'targets =': [1]}},
 {3: {'controls =': [1], 'control_value =': ['1'], 'targets =': [0]},
  4: {'controls =': [0], 'control_value =': ['0'], 'targets =': [1]}}]

# U0
{0: {'controls =': [0], 'control_value =': ['0'], 'targets =': [1]}}

{% endhighlight %}

The circuit diagram for the description in terms of two-level gates is shown. In the diagram, if the control value is 1 then the control circle is not filled.
![tqtlc]({{ site.baseurl }}/assets/img/two_qubit_two_level.png)

We assumed all of the control values should be 1 and not a mix of 0 and 1. So, if there's a gate that's controlled on value 0 then we insert Pauli X gates before and after the gate on the control qubit. An example for $$U_1$$ is shown
below. Once this change is complete, then it is possible to decompose a 2-qubit
two-level into single qubit and CNOT gates.

![tqlcpx]({{ site.baseurl }}/assets/img/u1_paulix.png)

When a two-level is controlled on first qubit and target is last qubit  ($$U_5$$ array) the circuit is as decomposed of Pauli X, rotation matrices formed from Pauli Z and Y and CNOT gates. If the target is first qubit ($$U_4$$ array) then the control and target qubits can be flipped.

$$U_5 = \begin{pmatrix}1 & 0 & 0 & 0 \\\ 0 & 1& 0 & 0 \\\ 0 & 0 & \textcolor{blue}{\textbf{x}} & \textcolor{blue}{\textbf{x}} \\\ 0 & 0 &  \textcolor{blue}{\textbf{x}} & \textcolor{blue}{\textbf{x}} \end{pmatrix}$$

![tqlcu5]({{ site.baseurl }}/assets/img/u5_two_qubit.png)

$$U_4 = \begin{pmatrix}1 & 0 & 0 & 0 \\\ 0 & \textcolor{blue}{\textbf{x}}& 0 & \textcolor{blue}{\textbf{x}} \\\ 0 & 0 & 1 & 0 \\\ 0 & \textcolor{blue}{\textbf{x}} &  0 & \textcolor{blue}{\textbf{x}} \end{pmatrix}$$

![tqlcu5]({{ site.baseurl }}/assets/img/u4_two_qubit.png)

As stated previously, the full decomposition output cannot be verified because of the general function's output being different to iterate over (more detailed discussion towards the end of this post) + the final output missing some gates.

For example, here's what the example output of $$U_1$$ decomposition looks like from the main function's output. This output is missing first 3 gates after the two-level array gate decomposition has been added (see comment in code block).

{% highlight python linenos%}

4: [[[Gate(X, targets=[0], controls=None, classical controls=None, control_value=None)],
   Gate(CNOT, targets=[1], controls=[0], classical controls=None, control_value=None),
   [Gate(X, targets=[0], controls=None, classical controls=None, control_value=None)],
   # upto this point, the gates are not added at the end
   Gate(CNOT, targets=[0], controls=[1], classical controls=None, control_value=None),
   # two-level array decomposition start
   [Gate(RZ, targets=[1], controls=None, classical controls=None, control_value=None),
    Gate(RY, targets=[1], controls=None, classical controls=None, control_value=None),
    Gate(X, targets=[1], controls=None, classical controls=None, control_value=None),
    Gate(CNOT, targets=[1], controls=[0], classical controls=None, control_value=None),
    Gate(X, targets=[1], controls=None, classical controls=None, control_value=None),
    Gate(RY, targets=[1], controls=None, classical controls=None, control_value=None),
    Gate(RZ, targets=[1], controls=None, classical controls=None, control_value=None),
    Gate(X, targets=[1], controls=None, classical controls=None, control_value=None),
    Gate(CNOT, targets=[1], controls=[0], classical controls=None, control_value=None),
    Gate(X, targets=[1], controls=None, classical controls=None, control_value=None),
    Gate(RZ, targets=[1], controls=None, classical controls=None, control_value=None),
    Gate(PHASEGATE, targets=[0], controls=None, classical controls=None, control_value=None)]],
    # two-level array decomposition end
  Gate(CNOT, targets=[0], controls=[1], classical controls=None, control_value=None)]
  # missing first 3 gates here

{% endhighlight %}

#### Future work

Looking at the circuit output above, it can be observed that if we enforce all
gate control values to be 1, then there might be a scenario when two Pauli X gates are acting one after other. If the goal is to reduce the total number of gates in the decomposition then an optimization scheme will have to be introduced. For a 2-qubit case, there's only a couple gates that could be removed using such an optimization scheme but for larger number of qubits, this will be particularly helpful.

This will be easier to do when two-level gate objects could be inserted in a circuit in QuTiP.

### Three-qubit gate decomposition without Toffoli

**Future Work:**

This scheme will describe a 3-qubit gate composed of single-qubit and CNOT gates. Note that this is different from the general decomposition scheme where ancilla qubits are used.

![trq]({{ site.baseurl }}/assets/img/three_qubit_flowchart.png)

Similar to the two-qubit case scenario, if the two-level gate does not describe a gate with either first or last qubit as target then a SWAP gate is inserted after a list of gates describing a decomposition of a two-level gate with last
qubit as target.

### Decomposition scheme for qubits greater than 2

This scheme will decompose some input gate as a combination of single-qubit gates, CNOT and Toffoli. The Toffoli gates could be decomposed further via previously defined decomposition in `resolve_gates`.

![gq]({{ site.baseurl }}/assets/img/flowchart_general.png)

For the case when total number of qubits is 3, there is no need to insert ancilla qubits in the circuit.

![threeq]({{ site.baseurl }}/assets/img/three_qubit_nont.png)

For $$n > 3$$, the total number of qubits in the circuit are increased after some ancilla qubits are added. If the initial total number of qubits is $$n$$ then the number of possible control qubits are $$n-1$$. Thus, number of ancillas introduced are $$n-2$$ and the original target is also shifted by this amount.

## Future Work

If we want to introduce ancilla qubits for the decomposition then there's more than one way to do so.

## Work in progress for the general decomposition method

As stated previously, the full decomposition output cannot be verified because of the general function's output being different to iterate over. The idea was to describe the output in terms of two-level gate arrays for which a dictionary seemed to be a good output format.

But this ended up turning into a mix of nested dictionaries and lists which made it difficult to keep track of the decomposition keys for some two-level gate array. For larger number of qubits, there were cases when some gates were not appended in the final output.

The code of following functions need to be edited to output appended lists:

 - Gray code scheme is used to find control, target qubits and their control values. This should not output a dictionary.
 - The next step is to make all control values to be 1. This function should not
 output a dictionary as well.
 - When CNOT gates are inserted into the gate list, this should also not be a dictionary.
 - For $$n>2$$, if the Toffoli is not targeted on first qubit or last qubit then SWAP gate decomposition is inserted into the gate list along with a Toffoli gate object that's only targeted on the last qubit. This function should also output a 1-dimensional list.
 - For $$n>2$$, if the two-level gate object is not describing a gate with first or last qubit as target then SWAP gates are inserted into the gate list along with a two-level gate with last qubit as target.
 - When the gates are decomposed into smaller gates, this output should also be inserted into the full gate list instead of creating a separate dictionary.
 - Only the final output should be a dictionary.

The issues faced were also due to trying to separate the gray code output into two separate lists (labeled as forward gray code gates and backward gray code gates respectively) - last item of first list is the non-trivial two-level gate and the rest are CNOT gates in the gray code scheme. It would be better to change the output to one list for CNOT gates in gray code scheme and one list for the two-level gates that need decomposition function. Currently, because it is one nested list, it is difficult to iterate over.

## Discussion related to creating a recursion scheme

Because of above itemized issues, we had to resort to use ancilla qubits for describing a decomposition for more than 3 qubits as it was problematic to create a recursion function while still trying to keep track of all different keys. The procedure for recursion is shown below :

**Step 1**
![r1]({{ site.baseurl }}/assets/img/recursion_step1.png)


**Step 2**

![r2]({{ site.baseurl }}/assets/img/recursion_step2.png)

Looking at the circuit above, we can provide the option to decompose Toffoli using known decomposition in `resolve_gates` or repeat the same recursion scheme
until the circuit is only described by single-qubit gates and two-qubit CNOT.

The steps shown below demonstrate how the recursions scheme could still keep on
decomposing the gates in the circuit.

**Step 3**

![r3]({{ site.baseurl }}/assets/img/recursion_cnot1.png)

**Step 4**

![r4]({{ site.baseurl }}/assets/img/recursion_final.png)


**Note :** For now, there's no way to start a discussion under a blogpost. While
I figure out a way to change the default layout of a post, comments/suggestions are welcome [in the repository](https://github.com/purva-thakre/purva-blog/discussions) of this blog.
