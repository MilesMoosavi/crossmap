# Efficient Optimization using Quantum Computing
[comment]: <> (TODO: change title of writeup if necessary)

## Table of Contents
* [Table of Contents](#table-of-contents)
* [Introduction](#introduction)
* [Objectives](#objectives)
* [Methods](#methods)
* [Usage](#usage)
* [Authors](#authors)
* [Works Cited](#works-cited)
* [License](#license)


## Introduction
The University of Maryland has a lot of resources that it seeks to ensure every student has easy access to, ranging from facilities like Wi-fi to basic safety measures such as streetlights. However, ensuring these resources are properly distributed amongst campus can grow to be expensive considering the University’s 1,339-acre estate. This optimization algorithm aims to minimize the resources necessary to ensure the entirety of any given area is fully encompassed by whatever facility the user desires. Quantum optimization is the ideal way to accomplish this task as classical optimizers are unable to provide as efficient of a solution due to the risk of getting trapped in local minima and the significantly weaker processing ability. The poorer performance of the classical optimizer is demonstrated in our results.

## Objectives
This project aims to implement and extend the work done in [*Efficient Light Source Placement Using Quantum Computing*](https://doi.org/10.48550/arXiv.2312.01156). The paper uses Quadratic Unconstrained Binary Optimization (QUBO) with Quantum Annealers to optimize the placement of light sources in a grid-like environment, in this case the game Minecraft is tested. The paper's results suggest practical results outside the video game.

Our primary objective is to release an open-source implementation of the methods used to optimize the placement of nodes or points of interest. This project aims to recreate the functionality of the original code, making it freely available for experimentation. While the previous work was designed to run on a Quantum Annealer provided by D-Wave and did not include publicly available code, our implementation offers a more accessible version of the outlined computing techniques.

Moreover, a web app is currently in development to simplify the usability of the paper as a tool for optimizing the placement of nodes throughout the campus of the University of Maryland, College Park. This web tool is designed to make it easier for users to test and experiment with the optimization techniques, even with current hardware limitations. The node optimization can be extrapolated for multiple use cases (network access points, emergency blue poles, etc.). This project aims to lay the groundwork for future developments in quantum computing that are more capable of handling such tasks.

[comment]: <> (TODO: Quantum Advantage: Explain why quantum machine learning is a suitable or advantageous approach for this problem.) 

## Methods
### Implementation
Our implementation follows a similar methodology to the paper.
The problem can be defined as the QUBO formulation:

$$ 
\begin{align}
& \min_{x\in \{0, 1 \}^n } && 1^\intercal x &&& \newline 
& \text{subject to } &&  Dx - 1 - z, &&& z \in \mathbb{N}^n _0
\end{align}
$$

Such that $D\in\{0,1\}^{n\times{n}}$ with entry $d_{ij}$ representing if the location at $(i, j)$ is in range of the closest node, $x$ being a candidate solution, and $z$ being an auxillary vector.

In order to use fewer qubits, the Alternating Direction Method of Multipliers (ADMM) is used to create a hybrid quantum-classical problem. The problem can be rewritten as

$$ 
\begin{align}
& \min_{x\in \{0, 1 \}^n , z \in \mathbb{Z}^n} && 1^\intercal x + \gamma 1^\intercal \Theta (z )&&& \newline 
& \text{subject to } &&  c(x, z) = 0 &&& 
\end{align}
$$

Such that $c(x, y) = Dx - 1 - z$, $\gamma > 0$, and $\Theta (z)$ representing a binary vector if $( \{z_0 < 0\}, ... , \{z_n < 0\} )$. This penalizes negative $z$ values, as from $(2)$  $z \in \mathbb{N}^n _0$.

The augmented lagrangian can then be created as:
$$
\begin{align}
L(x, z, \lambda, \mu) := 1^\intercal x + \gamma 1^\intercal \Theta (z) + \mathbb{\lambda}^\intercal c(x, z) + \frac{\mu}{2}\|c(x, z)\|^2
\end{align}
$$

Such that $\lambda$ and $\mu$ are coefficients and multipliers for the penalty terms. Thus we follow the algorithm:

$$
 \textbf{repeat} \newline 
  x^* \leftarrow \mathbb{\argmin}_x L(x, z^*, \lambda^*, \mu^*) \newline
  z^* \leftarrow \mathbb{\argmin}_z L(x^*, z, \lambda^*, \mu^*) \newline
  \lambda^x \leftarrow \lambda^* + \mu^*c(x, z) \newline
 \mu_{k+1} \leftarrow 
 \begin{cases}
 \rho \mu_k & \text{if} & \|c(x_k, z_k)\| > 10 \|D(z_k - z_{k+1})\| \newline
  \frac{\mu_k}{\rho} & \text{if} &10 \|D(z_k - z_{k+1})\| >  \|c(x_k, z_k)\| \newline
  \mu_k & \text{else}
 
 \end{cases}

 \newline
 \text{until convergence}
$$

Such that $\rho \ge 1$ is a fixed learning rate. A quantum computer is used in our implenentation when updating $x^*$. This implementation uses $\mu_0 = 1$ and $\rho = 1$.

[comment]: <> (TODO: add argument handling?)
[comment]: <> (TODO: containerize so it can be run from source)

### Circuit Explanation



The circuit is made up of Hadamard gates applied on each qubit. Hadamard gates set a qubit into a superposition, essentially meaning that they have no definite value but instead must be “observed” to get the value, either a 0 or a 1. 

Next, a Z rotation is applied first to every qubit. A Z gate rotates the qubit in the Z direction. Usually, this rotation is 180 degrees (pi radians) in the specified direction on the Bloch sphere. In this case, the rotations are instead of an input value (as seen by the t in the exponential) via the usage of an RZ gate. 

Similarly, there are now sets of rotations that affect two qubits simultaneously in the Z direction, going through every possible set of two qubits. 

Finally, X rotations are applied to each qubit using an RX gate, which functions similarly to the RZ gate above, but instead rotates the qubits over the X direction on the Bloch sphere. 

It appears that these rotations are finding what is the minimum energy level of the function, similar to how an annealer works. The Z rotations are the quadratic terms (when applied to a single qubit, these are when the quadratic term is the corresponding variable squared), and they are multiplied by a function of theta. When the value of that multiplication is minimal, then we get the most optimal values. Similarly, the X rotations are the linear terms. 


## Conclusion

[comment]: <> (TODO: Summary: Briefly summarize your project's key findings and their significance.)
[comment]: <> (TODO: Discuss the potential broader impact of your work in the field of quantum machine learning.)

Since this work describes how things can be optimized over discrete spaces, it can be expanded to cover almost any resource. For example, for the idea of optimally spreading out wifi signal sources above, this work can be expanded to include varying ranges for different areas, work with non-square base shapes, and perhaps expand to a much larger area. Since this work does not use quantum annealing, this work can also be expanded to use an actual quantum annealer, or perhaps other algorithms with quantum computers that are more efficient or take less time to compute. Finally, since this program does take a large amount of processing power, having it run passively on a computer (such as a cluster) would be a great way to expand the scale of the project without requiring a large upgrade in the quantum computers that are accessible. 


## Authors
Arnav Dayal \
Raghava Kalidindi \
Miles Moosavi \
Sohan Kosuru

[comment]: <> (TODO: authors and umd credits)

## Works Cited
Mücke, S., & Gerlach, T. (2023). Efficient Light Source Placement using Quantum Computing. ArXiv (Cornell University), 3630. CEUR Workshop Proceedings. https://doi.org/10.48550/arxiv.2312.01156


## License
TBD

[comment]: <> (TODO: Choose license probably MIT)
