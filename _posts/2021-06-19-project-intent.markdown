---
layout: post
title:  "Decomposing an arbitrary qubit gate.."
date:   2021-06-19 11:38:29 -0500
categories: GSoC QuTiP
---

## General Scenario

Assume some error in a quantum system has changed the initial state $$ \rho $$
to $$ \rho' $$. As long this evolution can be expressed in terms of an operator-sum representation (OSR), operators $$ \{ E_i\} $$ can be used to revert the system's state back to the original state $$ \rho $$ [^1].  

\begin{equation}
\mathcal{E} (\rho') = \sum_{i} E_i \rho {E_i}^\dagger
\end{equation}

One of the main goals of OSR is to find the operators $$ \{ E_i\} $$ that will act on a $$ n- $$ qubit system i.e. each  $$ E_i $$ is going to also be a $$ n \times n $$ unitary. At the end of this process, we want to implement error correction operator $$ E_i $$ in a quantum circuit with $$ \rho' $$ as the input and $$ \rho $$ as the output.


![error correction]({{ site.baseurl }}/assets/img/error_correction.png)

There's an issue implementing a quantum circuit [on a quantum device  due to the inability to create a circuit with an arbitrary $$ n \times n $$ matrix](https://quantumcomputing.stackexchange.com/questions/4975/how-do-i-build-a-gate-from-a-matrix-on-qiskit) if the matrix does not describe a known gate. Each of the $$ n \times n $$ matrix in $$ \{ E_i\} $$ has to be described in terms of known quantum gates (or rather by using universal quantum gates[^2]).

Different decomposition schemes exist to describe a gate in terms of smaller gates. These schemes differ in terms of the size of the qubit gates to be decomposed, the number of gates needed for the decomposition, the type of gates needed for the decomposition, optimized schemes for each scheme  etc.

### In QuTiP

The decomposition schemes are going to be added as a part of the [`qutip-qip`](https://github.com/qutip/qutip-qip) package. We describe a tentative design for a general decomposition function below.


**Steps taken to implement some decomposition scheme :**

A decomposition function will need a $$n-$$ qubit unitary gate as input. The decomposition function will also need an option to either provide an optimized decomposition or output the decomposition in terms of a particular set of gates.
    - An optimized decomposition scheme could either try to decrease the total number of gates in the decomposition or decrease gates that are expensive to implement (like $$CNOT$$ or $$T$$ gates)[^3].
    - If no option is chosen then the default option to decompose the gate will be to choose a scheme using the least number of gates.

```python
def n_qubit_decomposition(Qobj,'optmization_scheme','decomposition_scheme'):

  if 'optmization_scheme' in list_of_optimization_schemes:
    return(optmization_scheme(Qobj))

  elif 'decomposition_scheme' in list_of_decomposition_schemes:
    return(decomposition_scheme(Qobj))

  else:
    decomposition_name =[]
    length_of_outputs =[]

    for 'decomposition_scheme' in list_of_decomposition_schemes:
      number_of_gates_needed_for_decomposition = decomposition_scheme(Qobj)
      decomposition_name.append('decomposition_scheme')
      length_of_outputs.append(  number_of_gates_needed_for_decomposition)

    all_decomposition_outputs = dict(zip(decomposition_name,length_of_outputs))

    for min_value in all_decomposition_outputs:
      return(decomposition_scheme(Qobj))

```

The output from the decomposition function will be provided in the form of a list. This list could be used to create a circuit diagram.
    - In order to create a circuit diagram, a function will need to be defined to figure out the number of qubits in the circuit based on the input array size.
    - Then, to be able to create an object of the [`Gate`](https://qutip-qip.readthedocs.io/en/latest/_apidoc/qutip_qip.operations.html) class, subclasses of the `Gate` class will need to be introduced.
      For example, if a single qubit gate is decomposed in terms of rotation gates { $$ R_y(\theta), R_z(\theta) $$} then their respective arrays have to be recognized as particular types via `isinstance(some_array,SingleQubitRotation)`.

To verify the decomposition is equivalent to the input quantum gate except for some global phase factor, a separate module will be introduced.As of right now, we do not have a well formed plan of ignoring the global phase. This is mostly due to a function defined to do so has failed to extract the expected global phase via [arctan](https://numpy.org/doc/stable/reference/generated/numpy.arctan.html) despite introducing special conditions for boundary values etc.


We think this is due to the global phase equation being expressed as $$ \theta = arctan(-y/x) $$ whereas numpy's function calculating the values as $$ \theta = arctan(y/x) $$. The special boundary conditions need to be changed due to the quadrants for $$ arctan(-y/x) $$ being different than those for $$ arctan(y/x) $$ as shown below. Quadrants for $$ arctan(y/x) $$ were taken from [here.](http://scipp.ucsc.edu/~haber/ph116A/arg_11.pdf)

<div style="text-align: center"><img src="{{ site.baseurl }}/assets/img/arctanyx_quadrants.png"></div>


As new decomposition functions are introduced, it provides a way to compare the performance of the QuTiP functions to those already defined in other packages[^4].  

### Single Qubit Example

<div style="text-align: center"><img src="{{ site.baseurl }}/assets/img/hadamard.png"></div>

Let's consider decomposing a Hadamard gate ($$ \mathcal{H} $$) into a decomposition described by either single qubit rotation gates { $$ R_y(\theta), R_z(\theta) $$} or a combination of single qubit rotation gates { $$ R_y(\theta), R_z(\theta) $$}  and Pauli X gate ($$ \sigma_x $$). The former is called $$ZYZ$$ decomposition whereas the latter is known as $$ABC$$ decomposition.

Detailed calculations for each are available in [another post]({% post_url 2021-06-19-single-qubit-example %}).

  - If it's desired to decompose $$ \mathcal{H} $$ by the least number of gates then $$ZYZ$$ decomposition is used.
  - If it's desired to decompose $$ \mathcal{H} $$ in Pauli X gates in addition to those in $$ZYZ$$ decomposition then $$ABC$$ decomposition is used.

```python

H_gate = Qobj([[1/np.sqrt(2),1/np.sqrt(2)],[1/np.sqrt(2),-1/np.sqrt(2)]])

def single_qubit_decomposition(H_gate, 'lowest_gate_number', 'ABC_decomposition'):

  if 'lowest_gate_number' in list_of_optimization_schemes:
    return(ZYZ_decomposition(H_gate))

  elif 'ABC_decomposition' in list_of_decomposition_schemes:
    return(ABC_decomposition(H_gate))

```

Expected circuit diagram output for above:

**ZYZ_decomposition**

  Input $$ \mathcal{H} $$ is supposed to be decomposed as $$ R_z(\alpha) R_y(\theta) R_z(\beta)$$

  ![zyz_single]({{ site.baseurl }}/assets/img/zyz_single_qubit.png)

**ABC_decomposition**

  Input $$ \mathcal{H} $$ is supposed to be decomposed as

  $$ R_z(\alpha) R_y(\theta/2) \sigma_x R_y(-\theta/2) R_z(-(\alpha+\beta)/2) \sigma_x R_z(-\alpha+\beta)/2) = A \sigma_x B \sigma_x C$$

  ![abc_single]({{ site.baseurl }}/assets/img/abc_single.png)


Both decompositions are equivalent to $$ \mathcal{H} upto some global phase factor $\delta$ as

\begin{equation}
U \left| \psi \right> \left< \psi \right| U^\dagger = e^{i \delta} U \left| \psi \right>  \left< \psi \right| U^\dagger e^{-i \delta}
\end{equation}

**Note :** For now, there's no way to start a discussion under a blogpost. While I figure out a way to change the default layout of a post, comments/suggestions are welcome [in the repository](https://github.com/purva-thakre/purva-blog/discussions) of this blog.  

# References
[^1]: *Theorem 8.1* in Nielsen, M. A., &amp; Chuang, I. L. (2019). Quantum computation and quantum information. Cambridge University Press.

[^2]: A post will be added soon describing how the gates in a universal gate set are chosen. This will go over the results described in [Dawson, Nielsen (2005)](https://arxiv.org/abs/quant-ph/0505030) and many more.

[^3]: A post going over the lower and upper bounds for the schemes will be added soon.  

[^4]: Post describing the functions already defined in packages like Cirq, Qiskit, ProjectQ, XACC , Pyquil-grove, UniversalQCompiler etc. will be added soon.
