# Modularity on the Karate Club Graph

**Course:** DSC212 – Graph Theory Module  
**Assignment:** Research Assignment – *Modularity on the Karate Club Graph*  
**Submission:** this repository (notebook + assets), submitted as a GitHub URL

A from-scratch implementation of **spectral modularity maximisation** (Newman, 2006) on Zachary's Karate Club network, extended to multiple communities by **recursive bisection**, with network-metric tracking after every split.

---

## 1. The assignment in one paragraph

Zachary's Karate Club (34 members, 78 friendships) split into two factions after a dispute between the instructor ("Mr. Hi") and the club president ("Officer"). The task is to see whether a community-detection method that sees **only the graph structure** can recover that split, using the *modularity* quality function, which compares the observed within-community edges against a degree-preserving random (null) model.

## 2. Method

**Modularity matrix**

$$\mathbf{B} = \mathbf{A} - \frac{\mathbf{k}\mathbf{k}^\top}{2m}, \qquad Q = \frac{1}{4m}\,\mathbf{s}^\top \mathbf{B}\,\mathbf{s}, \quad \mathbf{s}\in\{-1,+1\}^n$$

**Spectral bipartition of a community `C`**

1. Restrict to the block `B^(C)` (entries `A_ij − k_i k_j / 2m` for `i, j ∈ C`).
2. Compute the leading eigenpair `(λ₁, u₁)`.
3. If `λ₁ ≤ 0` → **stop** (Theorem 3: no split can increase modularity).
4. Otherwise split by sign: `s_i = +1` if `(u₁)_i > 0`, else `−1`.
5. Recurse on both halves.

Maximising `sᵀBs` over `{±1}ⁿ` needs `2ⁿ` checks, so it is relaxed to `‖s‖₂ = 1`; by the Rayleigh–Ritz theorem the maximiser is the leading eigenvector `u₁`, which is rounded back to `±1` by sign.

**Metrics after every split** (NetworkX built-ins, computed inside each node's current community subgraph): degree centrality, betweenness centrality, closeness centrality, clustering coefficient.

## 3. What the notebook contains

`karate_modularity.ipynb` — runs top to bottom with no manual edits and is saved **with all outputs**.

| Section | Content |
|---|---|
| 1 | Load the graph (unweighted), build `A`, `k`, `m`; ground-truth plot; fixed spring layout |
| 2 | Modularity matrix + numerical checks of Propositions 1–2, Theorem 1, Corollary 1, Theorem 2 |
| 3 | One spectral split (eigenpair, sign rule, stopping rule); comparison with the real split |
| 4 | Recursive bisection and reconstruction of the partition after each split |
| 5 | Graph plots after every split (fixed layout, one colour per community, node-ID labels) |
| 6 | Computation of the four metrics for every node after every split |
| 7 | Evolution plots (line plots and heatmaps) |
| 8 | Quantitative analysis of which nodes stay central |
| 9 | Written discussion, including limitations |
| 10 | Summary and reference |

## 4. Key results

| Quantity | Value |
|---|---|
| First split vs. real club division | **33 / 34 members correct (97.1%)**; only node 8 differs |
| Modularity after first split | `Q = 0.372` |
| Splits performed / final communities | 4 splits → **5 communities**, `Q = 0.409` |
| Modularity path | 0 → 0.372 → 0.390 → 0.391 → 0.409 |
| NetworkX greedy modularity (reference) | 3 communities, `Q = 0.381` |

Final communities: `{0,1,2,3,7,12,13,17,19,21}`, `{11}`, `{4,5,6,10,16}`, `{24,25,27,28,31}`, `{8,9,14,15,18,20,22,23,26,29,30,32,33}`.

**Headline finding:** nodes **0** and **33** (the two faction leaders) stay in the top 5 for degree, betweenness and closeness at every iteration, while bridge-like nodes such as node **2** lose their betweenness once each community is viewed on its own.

## 5. How to run

```bash
git clone <your-repo-url>
cd <your-repo>
pip install -r requirements.txt
jupyter notebook karate_modularity.ipynb      # then: Kernel → Restart & Run All
```

Only `numpy`, `networkx` and `matplotlib` are needed. The Karate Club data ships with NetworkX, so there is nothing to download, and the notebook is deterministic (fixed layout seed, no random sampling in the algorithm).

## 6. Implementation notes and assumptions

* **Unweighted graph.** NetworkX's copy of the dataset carries an edge `weight` attribute; the handout uses `A_ij ∈ {0,1}`, so weights are removed.
* **Restricted modularity matrix** follows the handout: global degrees `k_i` and global `m` are used inside `B^(C)`.
* **Constant-sign eigenvector.** If `λ₁ > 0` but all entries of `u₁` share one sign, the sign rule cannot split the community, so it is treated as indivisible.
* **Metrics "after each split"** are computed on the induced subgraph of each community (iteration 0 equals the plain global metrics, which the notebook asserts). Singleton communities get metrics of `0`.
* **Colours** are stable across iterations: on a split, one child keeps the parent's colour and the other receives a new one.

## 7. Repository layout

```
.
├── karate_modularity.ipynb   # main deliverable (code + explanations + outputs)
├── README.md                 # this file
└── requirements.txt          # Python dependencies
```

## 8. Submission checklist

- [ ] Notebook runs with *Restart & Run All* and no manual edits
- [ ] Outputs and figures are visible in the committed notebook
- [ ] Newman (2006) is cited in the report (see below)
- [ ] Repository is public (or accessible to the grader) and the **repository URL** is submitted

## 9. Reference

M. E. J. Newman (2006). *Modularity and community structure in networks.* Proceedings of the National Academy of Sciences, 103(23):8577–8582.
