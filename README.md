## Pauli Encoding for a Single Run Grid Energy Optimization on Rigetti Ankaa-3 QPU!

This is `eQoSystem`'s solution to Rigetti's challenge `Track A: Energy Grid Optimization` in the `Q-Volution` Hackathon powered by `Girls in Quantum`

*"Everything we call real is made of things that cannot be regarded as real!"* -Niels Bohr-

### Authors & Team Members:

- **Ashraf Boussahi**  
  Research Intern @ CQTech (Constantine Quantum Technologies) | Algeria  
  AI Student @ ESI-SBA | Algeria  
  Email: [a.boussahi@esi-sba.dz](mailto:a.boussahi@esi-sba.dz)  
  [LinkedIn](https://www.linkedin.com/in/ashraf-boussahi-53a4731ab/)

- **Abir Chekroun**  
  Student @ ESI-SBA | Algeria  
  Email: [a.chekroun@esi-sba.dz](mailto:a.chekroun@esi-sba.dz)  
  [LinkedIn](https://www.linkedin.com/in/abir-chekroun-a066b52a8/)

- **Zakaria Lourghi**
  AI Student @ ESI-SBA | Algeria  
  Email: [z.lourghi@esi-sba.dz](mailto:z.lourghi@esi-sba.dz)  
  [LinkedIn](https://www.linkedin.com/in/zakaria-lourghi/)
  
*Thank you for reviewing our work. Enjoy reading :)))*

### Abstract

In this work, we explore the power of **Pauli Correlation Encoding (PCE)** to solve a Grid Energy Optimization task, which can be formulated as the well-known weighted Max-Cut problem. We proved that PCE helps drastically reduce the number of necessary qubits $n$ to encode an $m$-node graph, with an asymptotic scaling of $n = \mathcal{O}(m^{1/k})$, where $k$ is a factor of polynomial compression. We explored this technique over two different parts of a public dataset representing South Carolina's energy grid, and we showed that this can be beneficial in terms of running a **single optimization circuit** over the whole graph with a fewer number of qubits that can fit current NISQ-era QPUs, without compromising the general knowledge of the full graph that we might otherwise lose when partitioning the problem into smaller graphs fed into several QAOA runs.

Instead of a simple Ising mapping that is typically used in standard Quantum Approximate Optimization Algorithm (QAOA) solutions and scales linearly with the number of nodes ($n = m$), PCE presents another paradigm of encoding the graph through the **expected value** of an observable of a variational quantum state, whose variables we tune through a **Variational Quantum Eigensolver (VQE)** that aims to minimize a non-linear loss function defined for the problem. We found, using **Quantum Virtual Machine QVM** (and also Qiskit) simulations and **Ankaa-3 real Quantum Processing Unit** experiments with access provided by Rigetti, that the proposed approach converged to—and sometimes outperformed—pure classical approaches, and it can provide a warm start for other solvers and reduce their complexity and execution time.

### Introduction


### The Typical QAOA Circuit & Resources Estimation



### The Optimization Challenge: Our Solution:

Many of the solutions that insist on the use of QAOA circuits have to partition the massive graphs they need to optimize into smaller subgraphs and treat each of them independently. This creates a generalization bottleneck, since the optimization of each subgraph is unconscious of the existence of its siblings. Thus, theoretically, a single optimization round over the whole graph should be better, if and only if we can overcome the qubits number scaling problem. For this, we propose tailoring the Pauli Correlation Encoding technique proposed by [Sciorilli et al](https://arxiv.org/pdf/2401.09421). to the challenge we have at hand, and making it hardware-aware for the **Rigetti Ankaa-3 QPU**.

#### Pauli Correlation Encoding:

This method allows us to encode $m = \mathcal{O}(n^{k})$ nodes of the graph into only $n$ qubits by creating a different observable as a Pauli string that contains $k$ Pauli terms from $\{X, Y, Z\}$ and $n-k$ identity terms $I$ for each node.

Each node $i$ is associated with an observable:

$$
O_i = \bigotimes_{j=1}^{n} P_{ij}, \qquad P_{ij} \in \{I, X, Y, Z\}
$$

with exactly $k$ non-identity Pauli operators.

The assignment of each node to one of the two cuts is retrieved through a sign function over the expectation value of the node’s observable:

$$
x_i = \text{sign}\left( \langle \psi(\theta) | O_i | \psi(\theta) \rangle \right).
$$

where $x_i \in \{-1, +1\}$.

Creating $m$ observables reduces the number of qubits required, but it would naively require $m$ rounds of measurement, one for each observable, which creates another bottleneck, especially on noisy real hardware executions. To overcome this, we exploit the commutativity feature of the observables. If two observables are commutative, they can be measured within the same basis, for this, we create three distinct sets of observables, each one continsonly $k$ of only one of the three Pauli terms $\{X, Y, Z\}$, since each gate is commutative to itslef and to the Identity, we can measure all observables from the same set at once, which will reduce the number of required measurments from $n$ to just 3 rounds!


To optimize the problem towards a solution, we need to fine-tune our parameterized quantum state encoded within our ansatz circuit. Acting as a VQE, we aim to minimize the following non-linear loss function:

$$\mathcal{L}(\theta) = \sum_{(i,j) \in E} W_{ij} \tanh(\alpha | O_i \rangle) \tanh(\alpha | O_j \rangle) + \mathcal{L}^{(reg)}$$

With a scaling facture of $\alpha$, and $\tanh$ being used instead of the sign function as a smooth, differentiable relaxation to enable gradient descent, and $\mathcal{L}^{(reg)}$ being a regulation term.

The nature of this formalism treats the best solution as the ground-state energy that our VQE is trying to reach, and the Rayleigh–Ritz Variational Principle ensures that

$$
\langle \psi(\boldsymbol{\theta}) | H | \psi(\boldsymbol{\theta}) \rangle \geq E_0,
$$

meaning that the optimization can approach the true ground-state energy $E_0$ from above but never go below it.

To optimize the parameters of our VQE, we use the Adaptive Moment Estimation method (ADAM optimizer) combined with the parameter-shift rule and apply the chain rule over our loss function.

For our ansatz, we used Qiskit’s EfficientSU2 with $\text{reps} = 2, 3, 4$ and `"linear"` and `"pairwise"` entangling structures.

We also considered the use of the Armijo rule inside our optimization loops to adapt the learning rate accordingly.

Once the quantum circuit is fully trained, we read out the output state to obtain the optimal continuous correlations and extract the binary string $x^*$ via the sign function:

$$
x_i^* = \text{sign}\left( \langle O_i \rangle_{\theta^*} \right).
$$

We then pass the obtained cut into a classical post-processing step. We tried one round of single-bit swap search and a greedy improvement algorithm.

Benchmarking and results:











