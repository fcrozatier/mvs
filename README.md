# NodeRank+

Here I propose an algorithm for massive competion judging. See the [contributing](#contributing) section.

In a big competition context, a few judges cannot rank all the entries, so we need a **peer review** algorithm.

Also, assigning an absolute number to an entry is quite hard, and we often need to reevaluate the number after judging a few entries. It is far easier to estimate whether an entry is better than another entry. So the ranking will be based on **pairwise comparisons**, following other algorithms like [Gavel](https://www.anishathalye.com/2015/03/07/designing-a-better-judging-system/)

In this way entries correspond quite naturally to the nodes of a directed graph where a comparison between two nodes is an arrow pointing to the better entry. The algorithm needs to describe how to construct this **comparison graph** and how to rank entries from this graph.

## Principles

We want the comparison graph to have the following properties:
1. It needs to be **connected**, since otherwise some islands of entries are not, even indirectly, comparable.
2. It needs to have the smallest **diameter** possible. Information quality decreases over long distances, so if two entries are hundreds of comparisons appart, how to reliably tell which one is better?
3. Nodes need to have the **same order**. This is a fairness principle. Every entry should receive the same amount of attention and have the same number of comparisons: it would not be fair if an entry were compared only once while another entry had dozens of comparisons.
4. Constructing the graph should be easy to scale up with more and more contributions (arrows) while keeping these four properties true.

Since we are talking about a peer review algorithm, these properties are realistic because the graph will have many times more arrows than nodes by asking competitors to each contribute a few comparisons.

### Ideal case

The best graph satisfying these properties is the **complete graph** which has diameter 1 with all nodes connected.

A complete graph with $N$ nodes has $\frac{N(N-1)}{2}$ arrows so in practice we cannot expected to construct this graph, since each of the $N$ competitors would need to rank about $\frac{N-1}{2}$ entries which in pratice is way too much.

So the principles above allow to relax the constaints of a complete graph, while keeping nice properties. Indeed the following algorithm creates a graph with $N$ nodes whose diameter is roughly divided by 2 after each iteration.

## Comparison graph overview

This paragraph is a general overview of the algorithm, see the [details](details.md) for more.

Let say we have N nodes and assume this is a big number.

### Step 0

Randomly order the nodes in a list of size N and create a cycle graph from this list comparing nodes 0 and 1, nodes 1 and 2 etc. until nodes N-1 and 0.

### Step 1

Compare nodes $i$ and $i+\frac{N}{2}$ for all $i$.

This step is nuanced in the [details](details.md), it is the only one with two cases $N$ even or odd. But the idea is to compare nodes far appart.


### Step $k$

Compare nodes $i$ and $i+\frac{N}{2^k}$ for all $i$.

Continue while $\mathrm{ceil}(\frac{N}{2^k})>1$ then start another round at step 0, doubling the comparisons etc. In pratice there will be only one round. An equivalent condition is $k<\log_2(N)$


## Example

Starting with a graph with $2^{13}=8192$ nodes, the sequence of diameters of the comparison graph is:

<div style="display: flex; justify-content: center;">

|  step   | diameter |
| :-----: | :------: |
| step 0  |   4096   |
| step 1  |   2048   |
| step 2  |   1025   |
| step 3  |   513    |
| step 4  |   258    |
| step 5  |   130    |
| step 6  |    67    |
| step 7  |    35    |
| step 8  |    20    |
| step 9  |    12    |
| step 10 |    9     |
| step 11 |    7     |
| step 12 |    7     |

</div>

Notice the diameter is not exactly divided by two, and cannot go below 7 since at step 13 we would connect nodes $i$ and $i+N/N=i+1$ which were already connected at step 0.

<!-- TODO -->
If a competition has about ~1k competitors (let's say $2^{10}$ for convenience), and each competitor contributes 5 comparisons, then we can iterate the algorithm up to step 9 since each step after the first only needs half the competitors to complete.


## Naive ranking

A naive way to select the best nodes would be to pick the ones with the most wins.

Following the previous example, after step 9 each node has order 10 and the probability for a given node to win 9 or 10 over 10 comparisons is, using a binomial distribution with parameters $n=10$ $p=\frac{1}{2}$:

$$
\frac{1}{2^{10}}\left(\binom{10}{9}+\binom{10}{10}\right)=\frac{11}{2^{10}}
$$

So in a competition with $2^{10}$ competitors this would select about 11 entries.

But it's easiser to win against a competitor who lost all his comparisons that to win against a competitor who wins most his comparisons. So it's not enough to count to number of wins. We must take into account the relative strength of the loosing nodes.


## NodeRank

All nodes start with 1 point, so there are $N$ points in the graph. This is a constant (the sum of all points) but points will flow in the graph: at each step the points of a given node are divided among its outgoing arrows. So a node who lost 10 out of 10 comparisons will only contribute 0.1 point to each winner. This node tends to lose often so it's not so meaningul to win against it. On the contrary a node who only lost one out of 10 comparisons will contribute one point to the winner. It means a lot more to win against this node.

We apply this procedure $\mathrm{diam}(G)$ times (the diameter of the graph) to allow the information to flow between any two nodes of the graph. This way we get a more faithful representation of the value of nodes. The best ones are the ones with more points after $\mathrm{diam}(G)$ steps of this procedure.

The name is inpired from the PageRank algorithm. But there is more in the next section. As-is this algorithm promotes nodes with many wins against nodes with many wins themselves. But couldn't it be that a node only happened to be compared to weaker entries, while still being a very good entry ? This kind of node would be stuck with only a few points while still being a very good entry.

It would be nice to avoid this situation by design while keeping the other properties.

### Complexity

- At step $k$ the judge $i$ must compare nodes $i$ and $i+\frac{N}{2^{k-1}}$, so knowing what the next judge should do is $O(1)$.
- After $k$ steps the graph consists of $N+(k-1)\frac{N}{2}$ arrows and to compute the winners we need to flow points $\mathrm{diam}(G)$ times along these edges so the complexity is $O(Nk)$

## NodeRank+

Coming soon!


## Contributing

If you would like to make a suggestion, correct a typo or improve the algorithm/explanation/graphics, you can send a pull request, they are welcome !