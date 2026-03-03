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

## Maximum Power Energy Section (MPES)

## MPES Summary

The Maximum Power Energy Section (MPES) focuses on optimizing large-scale power grid energy distribution using quantum-enhanced optimization techniques. The problem is formulated as a Weighted Max-Cut optimization task over a sparse 3-regular graph representing a 180-node grid with 226 weighted edges.

The objective is to maximize

$$
\max_{x \in \{-1,1\}^m} \sum_{i,j} W_{ij}(1 - x_i x_j)
$$

where \( W_{ij} \) represents edge weights and \( x_i \) denotes node partition assignments.

The optimization is implemented using the Quantum Approximate Optimization Algorithm (QAOA), where each node is encoded into a single qubit. The interaction term \( x_i x_j \) is mapped to ZZ gates, which are decomposed into CNOT and rotation gates since ZZ interactions are not native hardware operations.

However, large-scale deployment faces major hardware limitations due to connectivity constraints in two-dimensional lattice architectures. When embedding the sparse graph onto physical qubit layouts, additional SWAP operations are required to route quantum information between distant qubits.

For lattice sizes such as a \( 14 \times 14 \) structure accommodating 180 logical qubits, routing overhead increases circuit depth significantly. Since two-qubit gate fidelity is approximately 99%, the accumulation of errors across numerous SWAP and interaction gates causes the overall circuit fidelity to decay exponentially as system size and QAOA layer depth increase.

This demonstrates the scalability limitation of running large-scale variational quantum optimization algorithms on current noisy intermediate-scale quantum hardware.


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

## Benchmarking and results:

# Experimental Results and Benchmarking

The following sections detail the performance of our **Pauli Correlation Encoding (PCE)** solver on both simulated environments and the **Rigetti Ankaa-3 QPU**. Our results demonstrate the effectiveness of polynomial compression in solving large-scale grid optimization tasks within a single quantum circuit execution.

### 1. Simulation and Classical Deep-Optimization
Initial testing was performed using high-fidelity simulations to establish a baseline for the PCE-VQE performance. The integration of a **greedy search** post-processor significantly enhanced the raw quantum output through iterative refinement. 

| Phase | Cut Size |
| :--- | :--- |
| **Initial Cut (Pre-Local Search)** | 6561.39 |
| **Final Optimized Cut** | 6635.43 |
| **Final Deep-Optimized Cut (12 iterations)** | **6853.43** |

---

### 2. Rigetti Ankaa-3 QPU Evaluation
We successfully deployed the solver on the **Rigetti Ankaa-3 QPU**. To minimize noise and gate infidelities, we utilized a specific hardware-aware qubit mapping tailored to the device's connectivity. 

**Primary Qubit Mapping:**
`{0:2, 1:9, 2:8, 3:7, 4:14, 5:21, 6:22, 7:15, 8:16, 9:23, 10:30, 11:31}`

* **Initial QPU Cut Size (Before Local Search):** 4023.80
* **Final Optimized Cut (Post-Execution):** 6340.69
* **Greedy Search Refinement:** **6600.00**

---

### 3. k=3 Compression and Pauli Twirling
[cite_start]To further explore the limits of qubit-efficient encoding, we implemented **cubic ($k=3$) compression** with `reps=2`. To mitigate hardware noise on the Ankaa-3, we utilized **Pauli Twirling** across 6 twirled circuits per rank, with 1666 shots each.

| Rank | Best Mapping (Nodes to Qubits) | Initial Cut | Final Optimized Cut |
| :--- | :--- | :--- | :--- |
| **Rank 1** | `{0:2, 1:3, 2:4, 3:11, 4:18, 5:19, 6:12, 7:5, 8:6}` | 4183.02 | 6024.04 |
| **Rank 2** | `{0:2, 1:3, 2:4, 3:5, 4:12, 5:19, 6:18, 7:11, 8:10}` | 2834.98 | 6156.95 |
| **Rank 3** | `{0:2, 1:3, 2:4, 3:5, 4:12, 5:11, 6:18, 7:19, 8:20}` | 3455.81 | 6105.12 |
| **Rank 4** | `{0:8, 1:9, 2:2, 3:3, 4:4, 5:5, 6:12, 7:19, 8:20}` | 3465.79 | 5832.58 |
| **Rank 5** | `{0:8, 1:9, 2:2, 3:3, 4:4, 5:11, 6:12, 7:19, 8:20}` | 3486.88 | 5696.84 |
| **Rank 6** | `{0:20, 1:19, 2:12, 3:5, 4:4, 5:11, 6:18, 7:25, 8:24}` | 3329.89 | 5915.87 |
| **Rank 7** | `{0:8, 1:9, 2:2, 3:3, 4:4, 5:5, 6:12, 7:19, 8:18}` | 3533.30 | **6228.15** |
| **Rank 8** | `{0:10, 1:17, 2:18, 3:11, 4:4, 5:5, 6:12, 7:19, 8:20}` | 4019.71 | 6163.49 |

> **Note:** Peak performance during edge exchange iterations reached a maximum cut size of approximately **6700**.



### 4. Project Sandboxes and Documentation
Comprehensive execution logs, iterative mapping trials, and the full implementation logic are available in the attached **Sandboxes (Jupyter Notebooks)**. These documents contain the full history of our optimization loops, including Adam optimizer parameters and twirling configurations used to achieve these results. 

# Extra Results:

# MaxCut Quantum Optimization Experiment Results & Parameter Analysis

## 1. Classical Baseline
- **Method**: Local Search (`maxcut.one_exchange` heuristic)
- **Random Restarts**: 100
- **Best Classical Cut Found**: **6407.29**

## 2. Quantum Experiments Summary
The following table summarizes the different runs, tracking the changes in circuit parameters, learning rates, gradient techniques, and the resulting performance.

| Experiment | Notebook | Qubits | Reps | k | Gradient Method | LR | Iters | Max Cut Size |
|:---|:---|:---:|:---:|:---:|:---|:---:|:---:|:---|
| **Exp 1** | QPU_Exp 1 | 9 | 2 | 3 | Finite Difference (`approx_fprime`) | 0.05 | 50 | 5024.32 |
| **Exp 2** | QPU_Exp 1 | 9 | 4 | 3 | Finite Difference (Padded Warm Start) | 0.05 | 50 | 4929.90 |
| **Exp 3** | QPU_Exp 1 | 9 | 4 | 3 | Finite Difference | 0.05 | 20 | 5222.74 |
| **Exp 4** | QPU_Exp 1 | 9 | 4 | 3 | Finite Difference (LR Decay = 0.98) | 0.05 | 20 | 5378.43 |
| **Exp 5** | QPU_Exp 1 | 9 | 4 | 3 | Finite Difference | 0.04 | 100 | 5400.46 |
| **Exp 6** | QPU_Exp 1 | 9 | 4 | 3 | Finite Difference | 0.20 | 100 | 5301.94 |
| **Exp 7** | QPU_Exp 1 | 9 | 4 | 3 | Finite Difference | 0.20 | 100 | 5183.74 |
| **Exp 8** | QPU_Exp 2 | 12 | 4 | 2 | Parameter Shift Rule + Chain Rule | 0.10 | 50 | **6767.71*** |
| **Exp 9** | QPU_Exp 2 | 12 | 3 | 2 | Parameter Shift Rule + Chain Rule | 0.10 | 100 | 6507.96 |

*\* Result after Deep Decoding & Multi-Sweep Bit-Swap Post-Processing.*

## 3. Best Performing Configuration
The most successful quantum run successfully outperformed the classical baseline algorithm limit.

- **Best Quantum-Backed Cut**: **6767.71**
- **Ansatz Architecture**: `EfficientSU2`, Entanglement = `pairwise`
- **Qubits**: 12
- **Circuit Depth (Reps)**: 4
- **Encoding Scale (k)**: 2 (Quadratic Pauli-Correlation)
- **Gradient Strategy**: Batched Exact Parameter Shift Rule
- **Optimizer**: Custom Adam (Learning Rate = 0.1, Iterations = 50)
- **Post-Processing Phase**: Deep 1-Bit and 2-Bit Swap Iterative Local Search.

## 4. Key Findings & Significant Parameter Impacts

1. **Gradient Computation Technique**: 
   - Using finite-difference Euclidean gradients (`approx_fprime`) resulted in sub-optimal cuts peaking around `~5400`. 
   - Upgrading the pipeline to use exact, analytical gradients via the **Parameter Shift Rule combined with Chain Rule gradients** (`QPU_Exp 2`) drastically improved optimization direction. This caused the base cut (before any local search) to instantly jump from ~5400 to **6260.28**.

2. **Encoding Order (k) & Qubit Count**:
   - Extremely high compression (180 nodes onto 9 qubits with `k=3`) severely limited the model's expressibility.
   - Expanding to **12 qubits** and reducing correlation complexity to **`k=2`** captured the graph structure much better and resulted in significantly better performance.

3. **Learning Rate & Stabilization (Decay)**:
   - High learning rates (`lr=0.2`) proved highly unstable and degraded the final results over 100 iterations (dropping scores from `5400` down to `5183`).
   - Conversely, utilizing a **smooth learning rate decay** (`0.98` per step in Exp 4) yielded a highly efficient run—achieving `5378.43` in just 20 iterations, proving that optimization step-sizing is crucial for the non-linear hyperbolic tangent loss landscape.

4. **Depth (Reps)**:
   - When equipped with the Parameter Shift Rule, deeper circuits successfully utilized their parameter counts (`reps=4` outperformed `reps=3` by reaching **6767** vs **6507** respectively). To prevent barren plateaus when transferring from shallow networks, "padding" with random noise uniformly between -0.01 and 0.01 successfully bridged the warm starts.

5. **Local Search Post-Processing**:
   - The quantum expectation values naturally guide the bitstring vector close to the optimum but contain slight statistical noise. The implementation of deep, 1-bit and 2-bit continuous sweeping pushed the raw quantum vector (`6260`) past the classical baseline limit up to the maximum score of `6767.71`.









