# ClusteringAlgorithmsDTL

This repository contains code to experiment the K Medoids and K Centers algorithms with real data from the Tree of Life Data set. This work accompanies *Clustering Algorithms for Maximum Parsimony DTL Reconciliations*. 

* `n.b.:` The paper talks about using Multidimensional FFT for a faster implementation, that is **not** done in this code base.
* This work is supported by the NSF Grant IIS-1419739.
* Summer 2016 Researchers: Dani Bork, Ricson Cheng, Jean Sung and Jincheng Wang (aka South Pawns!!).
* Advisor: Prof Ran Libeskind-Hadas.
* Email `jeansung94@gmail.com` to get access to past [experimental results](https://drive.google.com/drive/u/0/folders/0B9z84Or5GzOnX21VcjhlbXdYS0E).


## For the casual user

### Testing individual Tree of Life files
The main algorithms, k centers, and k medioids as described in the paper, are in `cluster/`. For k medoids, you can run the algorithm with random intial points or points as generated from a [point collection algorithm](https://en.wikipedia.org/wiki/Lloyd%27s_algorithm).

Run as:
```
python {TEST_FILE} {TREE_LIFE_FILE} {MAX_K}
```

For example:
```
python k_centers.py TreeLifeData/COG1304.newick 4 
python k_medoids_random.py TreeLifeData/COG1304.newick 4 
python k_medoids_pc.py TreeLifeData/COG1304.newick 4 
```

* More information about testing individual files, including what the run outputs (debugging, results) are can be found at `cluster/README.md` and `cluster/README_old.md.`


### Testing batch Tree of Life files
To run the experiment on mutiple files, be in directory `run_experiments/.` Add the list of files you want to run the programs in a space seperated list in `test.sh.`

Run as:
```
bash test.sh {PYTHON_FILE} {RESULTS_FILE} >> {DEBUG_FILE}
```

For example:
```
# nohup forces the program to run even if hangup signal given by terminal
# nice -n 1 puts the program on lower priority 
# & puts the process in the background
# >> to append, > to write to 
nohup nice -n 1 bash test.sh cluster/k_centers.py run_experiments/experiment_info/kcenters_result.out >> run_experiments/experiment_info/kcenters_info.out &
nohup nice -n 1 bash test.sh cluster/k_medoids_random.py run_experiments/experiment_info/km_random_result.out >> run_experiments/experiment_info/km_random_info.out &
nohup nice -n 1 bash test.sh cluster/k_medoids_pointcollect.py run_experiments/experiment_info/km_pc_result.out >> run_experiments/experiment_info/km_pc_info.out &
```

More information about testing batch files, including what you can do once you have the output (i.e. error checking and processing the data) can be found at `run_experiments/README.md.`


### Extra tools 
There are tools to find the reconciliation count and graphs of various Tree of Life files, to verify the validity of the graphs generated, and to plot the file size against the recocniliation count. These are found in `cluster/README.md.`


## For the next researcher 

This details the major changes that were made to the resposistory as cloned from:
https://github.com/alex-ozdemir/phylogenetic-reconciliation

### Known issues
* In k medoids from random, the function does not crash with memory error in its debugging output when it does not finish, but that is most likely what is going on. 
* When the likelihood is more than 1 by a round error, the dictgraph is not fixed and the error can be propagated such that the ratios for end/start are not exactly correct. This has been documented in rare cases for k centers, when `k=1`. This is not mission critical as it is most likely due to a float error compounding, and `k=1` is not a meaningful results. These are the cases that were found:

```
							End	Start	End/Start Normalized
TreeLifeData/COG1344.newick	81	80.6	1.004962779
TreeLifeData/COG1944.newick	29	28.8	1.006944444
TreeLifeData/COG4965.newick	49	48.2	1.01659751
TreeLifeData/COG1433.newick	39	37.6	1.037234043
TreeLifeData/COG0630.newick	44	42.2	1.042654028
```
* `TreeLifeData/COG500.newick` is a file that is an order of magnitude larger than any other file -- skip it for a typical test case
* Some of the naming in the debugging output and the files is really outdated, for example when running k medoids, the output refers to running the k means algorithm which is not what is happening
* There is code that is repeated between k centers, k medoids from point collect and k medoids from random. If rewritten, inputs should be piped through one file, then split into the relevant functions. 
* `TreeLifeData/COG1296.newick` Has a number of reconciliations that are not consistent between the roots and counting through the graph, which should not have been changed with the modifications of this repository. 


###  Added

* `nnic` 
A directory that contains code to test the NNI Conjecture.

* `run_experiments/`
This directory contains bash scripts to run the k centers, k medoids (from random, from point collect), data from the run, and output (including crash data). 

* `run_experiments/processed_data`
This directory contains scripts to format the data that comes back from running the experiments into CSV for google sheets to plot histograms. 

* `cluster/cluster_debug`
This directory contains scripts and result files that are used to calculated the number of reconciliations for files in the TreeLifeData/ and verify that the new version of KMeans.py the DP.py work (produce frequencies that are less than 1). Use `verify_graphs.py` to check for the frequency likelihood that is more than 1.  

* `cluster/k_centers.py`, `cluster/k_medoids_pointcollect.py`, `cluster/k_medoids_random.py`
These files replace the original `FromNewick.py` file that was here, which created reconciliation graphs and calculcated reconciliation counts. The k centers algorithm as described in the paper (Summer 2016 in Share LaTex). Run as `python pythonFile newickFile maxK`. 


* `cluster/calc_recon.py`
Create cache files for graphs and reconciliation counts for the files in `TreeLifeData/` directory. 



### Changed

* `DP.py`
The way that the preorder check was done is updated to check for same map nodes across different depth levels -- preorderCheck(preOrderList). Minor formatting (naming) and readability refactoring in addScores(treeMin, DTLDict, ScoreDict), no functional changes. Removed `import DrawDTL` as it is never used.

* `cluster/cache` 
More reconciliation graphs constructed during experiment running. Also, added a corresponding set of `.count` files that hold the associated number of reconciliations. 

* `cluster/KMeans.py`
Implemented a function `get_weighted_template()` that returns a randomly chosen node that is weighted by frequency as opposed to true random choice. Used in `cluster/k_centers.py`, `cluster/k_medoids_pointcollect.py`, `cluster/k_medoids_random.py`


### Removed 
* `alignlib.py`
* `app.py`
* `calcCostscapeScore.py`
* `cgi.py`
* `commonAnalytic.py`
* `compbio/`
* `CostVector.py`
* `costscape`
* `costscapeNew`
* `costscapeScore.py`
* `cycleCheckingGraph.py`
* `detectCycles.py`
* `DPcostscape.py`
* `DrawDTL.py`
* `getInput.py`
* `make.sh`
* `MasterReconciliation.py`
* `newApp.py`
* `newickData/`
* `newickToVis.py`
* `RandomGenerator.py`
* `README`
* `rasmus/`
* `ReconConversion.py`
* `onlineFolder/`
* `orderGraph.py`
* `plotRecon.py`
* `reconcile.py`
* `reconciliationGraph.py`
* `static/`
* `svgFiles/`
* `templates/`
* `testFiles/`
* `testy.svg`
* `testytesty.svg`
* `unitScore.py`
* `updateFiles/`
* `Vidua/`
* `vistrans`
* `vistrans.py`
* `xscape/`


### Not Changed
* `Greedy.py`
* `__init__.py`
* `newickFormatReader.py`
* `TreeLifeData/`

Documentation written by Jean!
