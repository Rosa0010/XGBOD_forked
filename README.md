# XGBOD (Extreme Boosting Based Outlier Detection)

------------

Zhao, Y. and Hryniewicki, M.K., "XGBOD: Improving Supervised Outlier Detection with Unsupervised Representation Learning," *International Joint Conference on Neural Networks (IJCNN)*, IEEE, 2018.

Please cite the paper as:

    @inproceedings{zhao2018xgbod,
      title={XGBOD: improving supervised outlier detection with unsupervised representation learning},
      author={Zhao, Yue and Hryniewicki, Maciej K},
      booktitle={2018 International Joint Conference on Neural Networks (IJCNN)},
      pages={1--8},
      year={2018},
      organization={IEEE}
    }

[PDF](https://www.andrew.cmu.edu/user/yuezhao2/papers/18-ijcnn-xgbod.pdf) | 
[IEEE Explore](https://ieeexplore.ieee.org/document/8489605) | 
[API Documentation](https://pyod.readthedocs.io/en/latest/pyod.models.html#module-pyod.models.xgbod) | 
[Example with PyOD](https://github.com/yzhao062/pyod/blob/master/examples/xgbod_example.py) 


**Update** (Dec 25th, 2018): XGBOD has been officially released in **[Python Outlier Detection (PyOD)](https://github.com/yzhao062/pyod)** V0.6.6.

**Update** (Dec 6th, 2018): XGBOD has been implemented in **[Python Outlier Detection (PyOD)](https://github.com/yzhao062/pyod)**, to be released in pyod V0.6.6.

------------

Additional notes:
1. Two versions of codes are provided:
   1. **Demo version** (xgbod_demo.py) is refactored for fast execution and reproduction as a proof of concept. The key difference from the full version is TOS are built in once for both training and test data. It could be regarded as a static unsupervised engineering. However, it is noted users should not expose and use the testing data while building TOS in practice. 
   2. **Production version** ([Python Outlier Detection (PyOD)](https://github.com/yzhao062/pyod)) is released with full optimization and testing as a framework. The purpose of this version is to be used in real applications, which should require fewer dependencies and faster execution.
3. It is understood that there are **small variations** in the results due to the random process, such as xgboost and Random TOS Selection. Again, running demo code would only give you similar results but not the exact results. Additionally, specific setups are slightly different for distinct datasets, which we have not published yet.
4. While running *L1_Comb* and *L2_Comb*, EasyEnsemble is used to construct balanced bags. It is noted the demo code uses 10 bags instead of 50, for executing efficiently. Despite, increasing to 50 bags would not change the result too much but just bring better stablity. You are welcomed to change "BalancedBaggingClassifier" parameter for using 50 bags. However, it is very slow and this is also one of the reasons why we propose XGBOD -- it is much more efficient:)

------------

##  Introduction
XGBOD is a three-phase framework (see Figure below). In the first phase, it generates new data representations. Specifically, various unsupervised outlier detection methods are applied to the original data to get transformed outlier scores as new data representations. In the second phase, a selection process is performed on newly generated outlier scores to keep the useful ones. The selected outlier scores are then combined with the original features to become the new feature space. Finally, an XGBoost model is trained on the new feature space, and its output decides the outlier prediction result.

![XGBOD Flowchart](https://github.com/yzhao062/XGBOD/blob/master/figs/flowchart.png "XGBOD Flowchart")

## Dependency
The experiment code is writen in Python 3 and built on a number of Python packages:
- matplotlib==2.0.2
- xgboost==0.7
- pandas==0.21.0
- imbalanced_learn==0.3.2
- scipy==0.19.1
- numpy==1.13.1
- PyNomaly==0.1.7
- imblearn==0.0
- scikit_learn==0.19.1

Batch installation is possible using the supplied "requirements.txt":

````cmd
pip install -r requirements.txt
````

------------


## Datasets
Seven datasets are used (see dataset folder):

| Datasets     | Dimension  | Sample Size  | Number of Outliers  |
| ------------ | -----------| ------------ | ------------------- |
| Arrhythmia   | 351        | 274          | 126 (36%)           |
| Letter       | 1600       | 32           | 100 (6.25%)         |
| Cardio       | 1831       | 21           | 176 (9.6%)          |
| Speech       | 3686       | 600          | 61(1.65%)           |
| Satellite    | 6435       | 36           | 2036 (31.64%)       |
| Mnist        | 7603       | 100          | 700 (9.21%)         |
| Mammography  | 11863      | 6            | 260 (2.32%)         |

All datasets are accessible at http://odds.cs.stonybrook.edu/. Citation Suggestion for the datasets please refer to: 
> Shebuti Rayana (2016).  ODDS Library [http://odds.cs.stonybrook.edu]. Stony Brook, NY: Stony Brook University, Department of Computer Science.

------------


## Usage and Sample Output (Demo Version)
Experiments could be reproduced by running **xgbod_demo.py** directly. You could simply download/clone the entire repository and execute the code by "python xgbod_demo.py".

The first part of the code read in the datasets using Scipy. Five public outlier datasets are supplied. Then various TOS are built by seven different algorithms:
1. KNN 
2. K-Median 
3. AvgKNN 
4. LOF
5. LoOP
6. One-Class SVM 
7. Isolation Forests
**Please be noted that you may include more TOS**

Taking KNN as an example, codes are as below:

```python
# Generate TOS using KNN based algorithms
feature_list, roc_knn, prc_n_knn, result_knn = get_TOS_knn(X_norm, y, k_range, feature_list)
```

Then three TOS selection methods are used to select *p* TOS:

```python

p = 10  # number of selected TOS
# random selection
X_train_new_rand, X_train_all_rand = random_select(X, X_train_new_orig, roc_list, p)
# accurate selection
X_train_new_accu, X_train_all_accu = accurate_select(X, X_train_new_orig, feature_list, roc_list, p)
# balance selection
X_train_new_bal, X_train_all_bal = balance_select(X, X_train_new_orig, roc_list, p)
```

Finally, various classification methods are applied to the datasets. Sample outputs are provided below:

![Sample Outputs on Arrhythmia](https://github.com/yzhao062/XGBOD/blob/master/figs/sample_outputs.png "Sample Outputs on Arrhythmia")
------------
## Figures

Running **plots.py** would generate the figures for various TOS selection algorithms:
![The effect of number of TOS and selection method](https://github.com/yzhao062/XGBOD/blob/master/figs/results.png "The effect of number of TOS and selection method")

