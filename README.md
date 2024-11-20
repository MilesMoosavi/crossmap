# Efficient Optimization using Quantum Computing
[comment]: <> (TODO: change title of writeup if necessary)

## Table of Contents
* [Table of Contents](#table-of-contents)
* [Summary](#summary)
* [Implementation](#implementation)
* [Usage](#usage)
* [Authors](#authors)
* [Works Cited](#works-cited)
* [License](#license)



## Summary
This project extends and implements the work done in [*Efficient Light Source Placement Using Quantum Computing*](https://doi.org/10.48550/arXiv.2312.01156). The paper uses Quadratic Unconstrained Binary Optimization (QUBO) with Quantum Annealers to optimize the placement of light sources in a grid-like environment, in this case the game Minecraft is tested. The paper's results suggest practical results outside the video game.

This web app implements the functionality of the paper to optimize the placement of nodes throughout the campus of the University of Maryland, College Park using our own optimization implementaion inspired by the paper. The node optimization can be extrapolated for multiple use cases (network access points, emergency blue poles, etc.)

The web app's frontend is a simple HTTP/CSS/JS framework to support the map and node functionality. The algorithm is implemented with Python using IBM's Qiskit SDK, and IonQ's cloud quantum computing hardware. Future plans include deploying the web app to a cloud platform with a backend setup as well.

## Implementation
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

## Usage
Clone the repository
```
git clone https://github.com/MilesMoosavi/crossmap
```

[comment]: <> (TODO: add argument handling?)
[comment]: <> (TODO: containerize so it can be run from source)

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
