# OracleNet
Code, animations, and supplementary material for OracleNet 


[![OracleNet](https://img.youtube.com/vi/2KYesBrx2kk/0.jpg)](https://youtu.be/2KYesBrx2kk "OracleNet")

[![Baxter - Left](https://img.youtube.com/vi/a440JJRSzy4/0.jpg)](https://www.youtube.com/watch?v=a440JJRSzy4)

[![Baxter - Right](https://img.youtube.com/vi/oXUeDx4AHYU/0.jpg)](https://youtu.be/oXUeDx4AHYU "OracleNet Baxter Right")


### On comparing roadmap-based methods

For the 2D environments presented in our paper, with a sparse precomputed roadmap, PRMs performs comparably to A* and OracleNet, but PRM’s path quality is far from being near-optimal/optimal. In the case of the densely precomputed roadmap,  path qualities of both methods are similar, but OracleNet performs significantly faster than PRMs. As even with the precomputed roadmap, for every query, PRMs require the execution of nearest neighbor search and Dijkstra algorithm whereas OracleNet computes path in single un-rolling of RNNs. Furthermore, randomized roadmaps, while appearing to solve the sample-size issue by decoupling the number of samples required to construct a graph from the dimensionality of the space, suffer from the same fate of requiring an exponential number of points in dimension to maintain a consistent quality. In fact, any sampling scheme, random or deterministic, suffers from this [1, 2]. With this in mind, we argue that even though PRM might be faster than A* in higher dimensional spaces, search times will inevitably scale exponentially in dimensions and path optimality will not be any better than RRT on average. Figure 9 in the paper already compares OracleNet’s performances when scaling in dimensions against A*; we expect PRM to have a very similar profile. Also note that in our results, OracleNet consistently retains low computation time irrespective of planning problem dimensionality.

[1] LaValle, Steven M., Michael S. Branicky, and Stephen R. Lindemann. "On the relationship between classical grid search and probabilistic roadmaps." The International Journal of Robotics Research 23.7-8 (2004): 673-692.

[2] Sukharev, Aleksandr G. "Optimal strategies of the search for an extremum." USSR Computational Mathematics and Mathematical Physics 11.4 (1971): 119-137.

The following table shows timings and optimality comparisons for PRM in the 2D cases presented in the paper. The PRM implementation used here uses a sample set of 1000 random points to build the roadmap with Dijkstra as the planner.

|Environment | A* (s)     | PRM (s)       | OracleNet (s) | OracleNet / A*  | OracleNet / PRM |
|------------|-----------|------------|---------------|------------------|----------------|
|Simple 1    |   0.08 (0.06)| 0.09 (0.05)|  0.13 (0.18) | 0.97 (0.08)   |0.8 (0.2)| 
|Simple 2 |  0.09 (0.07)|	0.09 (0.08)|	0.24 (0.18)	|	0.96 (0.03)|0.82 (0.14)|
|Simple 3 | 0.07 (0.06) | 0.10 (0.09) | 0.16 (0.19)| 0.96 (0.04)| 0.83 (0.13)|
|Simple 4 | 0.08 (0.05)|0.07 (0.09) |0.18 (0.20)|		0.98 (0.05)|0.84 (0.14)|
|Difficult 1|0.07 (0.05)|1.01 (0.08)|0.18 (0.10)|		0.97 (0.07)|0.87 (0.12)|
|Difficult 2| 0.07 (0.05)|0.09 (0.06)   | 0.18 (0.12)|  0.99 (0.05)| 0.86 (0.11)|
|Difficult 3|  0.09 (0.06) | 0.07 (0.08) | 0.17 (0.12)  |0.96 (0.22) | 0.85 (0.10)|
|Difficult 4| 0.05 (0.04)|0.05 (0.05) | 0.18 (0.09)|  0.96 (0.17)|0.85 (0.13)   |

Performance trends with variable dataset and network sizes
----------------------------------------------------------

To analyze the learning capabilities of OracleNet, we performed experiments on environment 'Difficult 2' (refer to Fig. 5 in the paper) with variable dataset and network sizes. Success rates and path optimality (benchmarked against A*) are used to measure performance trends. For obvious reasons, it is not possible to compare all dataset sizes to all network sizes, therefore the trends presented here are meant to serve as an approximate guide to selecting reasonable datasets and network configurations. Note that all trends presented here do not involve the use of the repair and rewire modules, since the purpose is to (1) study the role of dataset and network sizes in generating successful paths, and (2) to show that with sufficient data and an appropriately chosen well-trained network, OracleNet is able to generate near-optimal feasible paths on its own and repair is needed only to handle the edge cases while rewire takes it closer to being absolutely optimal. Thus, the paths generated for the analysis presented here are directly generated by the network and do not involve any correction/processing. 

### Variable Dataset Sizes ###

![alt text](../master/Trends/trend_po_d.png?raw=true "Dataset Trends")


The network size is kept fixed here (refer to the network architecture described in Section IV.A) and the dataset size is incrementally increased to a maximum of 100,000 valid A* generated paths. It is to be expected that the network will predict paths more accurately when a larger training set is provided. Note that success rates reach greater than 90% with 20,000 training paths and continue on an upward trend till it reaches ~99% when 60,000 paths are considered for training. Path optimality for OracleNet climbs quickly to reach A*'s level of optimality and does not seem to be as dependent on dataset size as success rate (only the optimality statistics of successful paths are considered). 

From the graph, it can be inferred that selecting a dataset size between the two aforementioned values to get a good raw success rate while relying on repair/rewire to handle the remaining 5 % of the cases is the reasonable strategy to choose. Also, the environment chosen here has a large number of arbitrarily placed obstacles leading to a high degree of non-linearity in path shapes, hence a larger than average dataset may be required. 


### Variable Network Sizes ###

![alt text](../master/Trends/trend_po_n.png?raw=true "Network Trends")

The dataset size is kept fixed here at 100,000 valid paths while the network size (number of parameters, with each increment indicating an additional layer with 256 units) is kept variable. Note that the network configuration used in the paper and in the previous section has 2 million (2 M) parameters. It has already been established in the previous section that using 100,000 paths with 2 M parameters leads to >99% success rate, which is corroborated here as well. It is interesting to note the falling performance metrics when network size grows beyond a certain range. This can be attributed to the network having too many trainable weights as compared to training data which causes it to overfit quickly and performly poorly with unseen test sets. 