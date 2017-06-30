---
layout: post
title:  "Complete Guide to Parameter Tuning in Xgboost"
date:   2017-06-20 12:00:00 +0800
categories: jekyll update
---

## Table of Contents

1. [The XGBoost Advantage](#1)
1. [Understanding XGBoost Parameters](#2)
1. [Tuning Parameters](#3)

----------

<a name='1'></a>

## 1. The XGBoost Advantage

1. Regularization:
  - Standard GBM implementation has **no regularization** like XGBoost, therefore it also helps to reduce overfitting.
  - In fact, XGBoost is also known as **'regularized boosting'** technique.
1. Parallel Processing:
  - XGBoost implements parallel processing and is blazingly faster as compared to GBM.
  - Check [this link](http://zhanpengfang.github.io/418home.html) out to explore further.
  - XGBoost also supports implementation on Hadoop.
1. High Flexibility
  - XGBoost allow users to define **custom optimization objectives and evaluation criteria**.
  - This adds a whole new dimension to the model and there is no limit to what we can do.
1. Handling Missing Values
  - XGBoost has an in-built routine to handle missing values.
  - User is required to supply a different value than other observations and pass that as a parameter. XGBoost tries different things as it encounters a missing value on each node and learns which path to take for missing values in future.
1. Tree Pruning:
  - A GBM would stop splitting a node when it encounters a negative loss in the split. Thus it is more of a **greedy algorithm**.
  - XGBoost on the other hand make **splits upto the max_depth** specified and then start **pruning** the tree backwards and remove splits beyond which there is no positive gain.
  - Another advantage is that sometimes a split of negative loss say -2 may be followed by a split of positive loss +10. GBM would stop as it encounters -2. But XGBoost will go deeper and it will see a combined effect of +8 of the split and keep both.
1. Built-in Cross-Validation
  - XGBoost allows user to run a cross-validation at each iteration of the boosting process and thus it is easy to get the exact optimum number of boosting iterations in a single run.
  - This is unlike GBM where we have to run a grid-search and only a limited values can be tested.
1. Continue on Existing Model
  - User can start training an XGBoost model from its last iteration of previous run. This can be of significant advantage in certain specific applications.
  - GBM implementation of sklearn also has this feature so they are even on this point.

<a name='2'></a>

## 2. XGBoost Parameters

1. **General Parameters**: Guide the overall functioning
1. **Booster Parameters**: Guide the individual booster (tree/regression) at each step
1. **Learning Task Parameters**: Guide the optimization performed

### 2.1 General Parameters

1. booster [default=gbtree]
1. silent [default=0]
1. nthread [default to maximum number of threads available if not set]

### 2.2 Booster Parameters

1. eta [default=0.3]
  - Typical final values to be used: 0.01-0.2
1. min_child_weight [default=1]
  - Defines the minimum sum of weights of all observations required in a child.
  - This is similar to **min_child_leaf** in GBM but not exactly. This refers to min "sum of weights" of observations while GBM has min "number of observations".
  - Used to control **over-fitting**. Higher values prevent a model from learning relations which might be highly specific to the particular sample selected for a tree.
  - Too high values can lead to under-fitting hence, it should be tuned using CV.
1. max_depth [default=6]
  - Used to control **over-fitting** as higher depth will allow model to learn relations very specific to a particular sample.
  - Typical values: 3-10
1. max_leaf_nodes
  - The maximum number of terminal nodes or leaves in a tree.
  - Can be defined in place of max_depth. Since binary trees are created, a depth of ‘n’ would produce a maximum of 2^n leaves.
  - If this is defined, GBM will ignore max_depth.
1. gamma [default=0]
  - A node is split only when the resulting split gives a positive reduction in the loss function. Gamma specifies the minimum loss reduction required to make a split.
  - Makes the algorithm conservative. The values can vary **depending on the loss function** and should be tuned.
1. max_delta_step [default=0]
  - In maximum delta step we allow each tree’s weight estimation to be. If the value is set to 0, it means there is no constraint. If it is set to a positive value, it can help making the update step more conservative.
  - Usually this parameter is not needed, but it might help in logistic regression when class is **extremely imbalanced**.
  - This is generally not used but you can explore further if you wish.
1. subsample [default=1]
  - Same as the subsample of GBM. Denotes the fraction of observations to be randomly samples for each tree.
  - Lower values make the algorithm more conservative and prevents overfitting but too small values might lead to under-fitting.
  - Typical values: 0.5-1
1. colsample_bytree [default=1]
  - Similar to max_features in GBM. Denotes the fraction of columns to be randomly samples for each tree.
  - Typical values: 0.5-1
1. colsample_bylevel [default=1]
  - Denotes the subsample ratio of columns for each split, in each level.
  - I don’t use this often because subsample and colsample_bytree will do the job for you. but you can explore further if you feel so.
1. lambda [default=1]
  - L2 regularization term on weights (analogous to Ridge regression)
  - This used to handle the regularization part of XGBoost. Though many data scientists don’t use it often, it should be explored to reduce **overfitting**.
1. alpha [default=0]
  - L1 regularization term on weight (analogous to Lasso regression)
  - Can be used in case of very high dimensionality so that the algorithm runs faster when implemented
1. scale_pos_weight [default=1]
  - A value greater than 0 should be used in case of high class **imbalance** as it helps in faster convergence.

### 2.3 Learning Task Parameters

1. objective [default=reg:linear]
1. eval_metric [ default according to objective ]
1. seed [default=0]

XGBoost Parameters guide: [official](http://xgboost.readthedocs.io/en/latest/parameter.html#general-parameters) [github](https://github.com/dmlc/xgboost/blob/master/doc/parameter.md)

<a name='3'></a>

## 3. Parameter Tuning

2 forms of XGBoost:

1. **xgb** – this is the direct xgboost library. I will use a specific function “cv” from this library.
1. **XGBClassifier** – this is an sklearn wrapper for XGBoost. This allows us to use sklearn’s Grid Search with parallel processing in the same way we did for GBM.

Lets define a function which will help us create XGBoost models and perform cross-validation.

~~~python
def modelfit(alg, dtrain, predictors,useTrainCV=True, cv_folds=5, early_stopping_rounds=50):
    
    if useTrainCV:
        xgb_param = alg.get_xgb_params()
        xgtrain = xgb.DMatrix(dtrain[predictors].values, label=dtrain[target].values)
        cvresult = xgb.cv(xgb_param, xgtrain, num_boost_round=alg.get_params()['n_estimators'], nfold=cv_folds,
            metrics='auc', early_stopping_rounds=early_stopping_rounds, show_progress=False)
        alg.set_params(n_estimators=cvresult.shape[0])
    
    #Fit the algorithm on the data
    alg.fit(dtrain[predictors], dtrain['Disbursed'],eval_metric='auc')
        
    #Predict training set:
    dtrain_predictions = alg.predict(dtrain[predictors])
    dtrain_predprob = alg.predict_proba(dtrain[predictors])[:,1]
        
    #Print model report:
    print "\nModel Report"
    print "Accuracy : %.4g" % metrics.accuracy_score(dtrain['Disbursed'].values, dtrain_predictions)
    print "AUC Score (Train): %f" % metrics.roc_auc_score(dtrain['Disbursed'], dtrain_predprob)
                    
    feat_imp = pd.Series(alg.booster().get_fscore()).sort_values(ascending=False)
    feat_imp.plot(kind='bar', title='Feature Importances')
    plt.ylabel('Feature Importance Score')
~~~

### General Approach for Parameter Tuning

1. Choose a relatively **high learning rate**. Generally a learning rate of 0.1 works but somewhere between 0.05 to 0.3 should work for different problems. Determine the **optimum number of trees** for this learning rate. XGBoost has a very useful function called as “cv” which performs cross-validation at each boosting iteration and thus returns the optimum number of trees required.
1. Tune **tree-specific parameters** ( max_depth, min_child_weight, gamma, subsample, colsample_bytree) for decided learning rate and number of trees. Note that we can choose different parameters to define a tree and I’ll take up an example here.
1. Tune **regularization parameters** (lambda, alpha) for xgboost which can help reduce model complexity and enhance performance.
1. **Lower the learning rate** and decide the optimal parameters.

### Step 1: Fix learning rate and number of estimators for tuning tree-based parameters

In order to decide on boosting parameters, we need to set some initial values of other parameters. 

1. max_depth = 5 : This should be between 3-10.
1. min_child_weight = 1 : A smaller value is chosen because it is a highly imbalanced class problem and leaf nodes can have smaller size groups.
1. gamma = 0 : A smaller value like 0.1-0.2 can also be chosen for starting.
1. subsample, colsample_bytree = 0.8 : This is a commonly used used start value.
1. scale_pos_weight = 1: Because of high class imbalance.

Lets take the **default learning rate** of 0.1 here and check the optimum **number of trees** using cv function of xgboost.

~~~python
predictors = [x for x in train.columns if x not in [target, IDcol]]
xgb1 = XGBClassifier(
    learning_rate =0.1,
    n_estimators=1000,
    max_depth=5,
    min_child_weight=1,
    gamma=0,
    subsample=0.8,
    colsample_bytree=0.8,
    objective= 'binary:logistic',
    nthread=4,
    scale_pos_weight=1,
    seed=27)
modelfit(xgb1, train, predictors)
~~~

### Step 2: Tune other params

1. max_depth, min_child_weight
1. gamma
1. subsample, colsample_bytree
1. reg_alpha
1. reg_lambda

~~~python
param_test1 = {
    'max_depth':range(3,10,2),
    'min_child_weight':range(1,6,2)
}
gsearch1 = GridSearchCV(estimator = XGBClassifier( learning_rate =0.1, n_estimators=140, max_depth=5,
    min_child_weight=1, gamma=0, subsample=0.8, colsample_bytree=0.8,
    objective= 'binary:logistic', nthread=4, scale_pos_weight=1, seed=27), 
    param_grid = param_test1, scoring='roc_auc',n_jobs=4,iid=False, cv=5)
gsearch1.fit(train[predictors],train[target])
gsearch1.grid_scores_, gsearch1.best_params_, gsearch1.best_score_
~~~



