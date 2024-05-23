# Clique-Cover
Step-by-Step Guide

1. **Define the Graph**: Use `networkx` to create the graph.
2. **Formulate the Ising Model**: Convert the clique cover constraints into an Ising Hamiltonian.
3. **Submit to D-Wave**: Use D-Wave’s tools to solve the Ising problem.
Step 1: Define the Graph
Import Required Libraries

from dimod import BinaryQuadraticModel
from collections import defaultdict
pip install dwave-system networkx

Step 2: Define the Function to Create QUBO

- `graph`: The input graph for which we want to find the clique cover.
- `num_cliques`: The number of cliques to partition the graph into.
- `n`: The number of nodes in the graph.
- `Q`: A defaultdict to hold the QUBO matrix, initialized to float.
- `x = lambda i, k: i * num_cliques + k`: A lambda function to map the 2D variable indices (i, k) to a single index for the QUBO.


Step 3: Ensure Each Node is in Exactly One Clique
python
    # Each node must be in exactly one clique
    for i in range(n):
        for k in range(num_cliques):
            Q[(x(i, k), x(i, k))] -= 1
            for k2 in range(k + 1, num_cliques):
                Q[(x(i, k), x(i, k2))] += 2

**For each node `i` and each clique `k`**:
  - `Q[(x(i, k), x(i, k))] -= 1`: Add a bias term to the QUBO for including the node in a clique.
  - **For each pair of different cliques `(k, k2)`**:
    - `Q[(x(i, k), x(i, k2))] += 2`: Add a penalty term to prevent the node from being in more than one clique.

Step 4: Ensure Each Edge is Covered by At Least One Clique

    # Each edge must be covered by at least one clique
    for u, v in graph.edges():
        for k in range(num_cliques):
            Q[(x(u, k), x(v, k))] -= 1

- **For each edge `(u, v)` and each clique `k`**:
  - `Q[(x(u, k), x(v, k))] -= 1`: Add a term to reward the inclusion of both endpoints of an edge in the same clique.

#### Step 5: Convert QUBO to Dict and Return

    # Convert defaultdict to dict
    return dict(Q)

- `return dict(Q)`: Convert the defaultdict `Q` to a standard dictionary before returning it.

#### Step 6: Define Parameters and Create QUBO
# Parameters
num_cliques = 2  # Adjust based on the graph's structure
Q = clique_cover_qubo(G, num_cliques)

- `num_cliques = 2`: Set the number of cliques to partition the graph into.
- `Q = clique_cover_qubo(G, num_cliques)`: Call the function to generate the QUBO matrix for the given graph and number of cliques.

#### Step 7: Convert QUBO to BQM

bqm = BinaryQuadraticModel.from_qubo(Q)

- `bqm = BinaryQuadraticModel.from_qubo(Q)`: Convert the QUBO matrix `Q` to a Binary Quadratic Model (BQM) using `dimod`.

#### Step 8: Finally, using D'wave sampler to solve the model
