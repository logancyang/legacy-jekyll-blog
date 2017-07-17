---
layout: post
title: "Logistic Regression with Apache Spark"
comments: True
---

Logistic regression is widely used to predict binary responses. It is a linear method as described 
in my other post 
[An Overview of Logistic Regression](http://loganyc1934.github.io/2015/10/13/an-overview-of-logistic-regression/).

Binary logistic regression can be generalized into multinomial logistic regression to train and 
predict multiclass classification problems. For example, for K possible outcomes, one of the outcomes 
can be chosen as a “pivot”, and the other K−1 outcomes can be separately regressed against the pivot 
outcome. In MLlib, the first class 0 is chosen as the “pivot” class.

For multiclass classification problems, the algorithm will output a multinomial logistic regression 
model, which contains K−1 binary logistic regression models regressed against the first class. Given 
a new data points, K−1 models will be run, and the class with largest probability will be chosen as 
the predicted class.

Spark implemented two algorithms to solve logistic regression: gradient descent and L-BFGS. Here we
focus on gradient descent.

<br>
#### Stochastic Gradient Descent (SGD)

Optimization problems whose objective function f is written as a sum are particularly suitable to be 
solved using stochastic gradient descent (SGD). For logistic regression, the optimization formulation 
commonly used:

$$
f(\mathbf{w})=\frac { 1 }{ n } \sum_{ i=1 }^{ n }{ L(\mathbf{w};{ \mathbf{x} }_{ i },{ y }_{ i }) } 
+\lambda R(\mathbf{w})
$$

where the logistic loss \\( L \\) is

$$
L(\mathbf{w};\mathbf{x},y)=log(1+exp(-y{ \mathbf{w} }^{ T }\mathbf{x}))
$$

The loss is written as an average of the individual losses coming from each data point.

A stochastic subgradient is a randomized choice of a vector, such that in expectation, we obtain a 
true subgradient of the original objective function. Picking one data point \\( i\in [1,...,n] \\) 
uniformly at random, we obtain a stochastic subgradient, with respect to \\(\mathbf{w} \\) as follows:

$$
{ f }'_{ \mathbf{w},i }={ L }'_{ \mathbf{w},i }+\lambda { R }'_{ \mathbf{w},i }
$$

This is determine by the i-th data point. Running SGD simply becomes walking in the direction of the
negative stochastic subgradient \\( { f }'_{ w,i } \\), that is

$$
{ \mathbf{w} }^{ (t+1) }={ \mathbf{w} }^{ (t) }-\gamma { f }'_{ \mathbf{w},i }
$$

**Step-size**. The parameter \\( \gamma \\) is the step-size, which in the default implementation is 
chosen decreasing with the square root of the iteration counter, i.e. \\( \gamma =\frac { s }{ \sqrt { t }  }  \\)
in the \\(t\\)-th iteration, 
with the input parameter s = `stepSize`. Note that selecting the best step-size 
for SGD methods can often be delicate in practice and is a topic of active research.

**Gradients**. Here is a table of (sub)gradients of the machine learning methods implemented in MLlib.

<br>
<center><img src="/assets/images/logreg/lossAndGradients.png" alt="Drawing" style="width: 900px;"></center>
<br>

<br>
#### Update Schemes for Distributed-SGD

The SGD implementation in 
[GradientDescent](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.mllib.optimization.GradientDescent)
uses a simple distributed sampling of the data points. 

Recall that the loss part of the optimization problem
is $ \frac { 1 }{ n } \sum \_{ i=1 }^{ n }{ L(\mathbf{w};{ \mathbf{x} }\_{ i },{ y }\_{ i }) } $,
and therefore $ \frac { 1 }{ n } \sum \_{ i=1 }^{ n }{ { L }'\_{ w,i } } $ would be the true
subgradient. Since this would require the access to the full data set, the parameter `miniBatchFraction`
specifies which fraction of the full data to use instead.
The average of the gradients over this subset, i.e.

$$
\frac { 1 }{ \left| S \right|  } \sum_{ i\in S }^{  }{ { L }'_{ w,i } } 
$$

is a stochastic gradient. Here \\( S \\) is the sampled subset of size 
\\( \left| S \right|= \\) `miniBatchFraction` \\( \cdot n \\)

In each iteration, the sampling over the distributed dataset (RDD), as well as the computation of 
the sum of the partial results from each worker machine is performed by the standard spark routines.

If the fraction of points `miniBatchFraction` is set to 1 (default), then the resulting step in each 
iteration is exact (sub)gradient descent. In this case there is no randomness and no variance in the 
used step directions. 

On the other extreme, if miniBatchFraction is chosen very small, such that only 
a single point is sampled, i.e. \\( \left| S \right|= \\) `miniBatchFraction` \\( \cdot n = 1\\), 
then the algorithm is equivalent to standard SGD. 
In that case, the step direction depends from the uniformly random sampling of the point.

<br>
#### SGD Implementation in MLlib

_class_ pyspark.mllib.classification.**LogisticRegressionWithSGD**

_classmethod_ **train**(_data, iterations=100, step=1.0, miniBatchFraction=1.0, 
initialWeights=None, regParam=0.01, regType='l2', intercept=False, validateData=True_)

Train a logistic regression model on the given data.

Parameters: 

- `data` – The training data, an RDD of LabeledPoint.
- `iterations` – The number of iterations (default: 100).
- `step` – The step parameter used in SGD (default: 1.0).
- `miniBatchFraction` – Fraction of data to be used for each SGD iteration (default: 1.0).
- `initialWeights` – The initial weights (default: None).
- `regParam` – The regularizer parameter (default: 0.01).
- `regType` – The type of regularizer used for training our model. Allowed values: 
  - “l1” for using L1 regularization
  - “l2” for using L2 regularization
  - None for no regularization
  - (default: “l2”)
- intercept – Boolean parameter which indicates the use or not of the augmented representation for 
training data (i.e. whether bias features are activated or not, default: False).
- validateData – Boolean parameter which indicates if the algorithm should validate data before 
training. (default: True)

<br>
#### Click-Through Rate (CTR) Prediction with MLlib

Here I will show an example of utilizing 
[LogisticRegressionWithSGD](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.classification.LogisticRegressionWithSGD)
to predict CTR given some training examples. The dataset is provided by Criteo Labs to a Kaggle competition.
It has roughly 51M data points, splitted into 39M for training, 6M for validation and 6M for test. 
Each data point has 233K one-hot-encoded binary features (originally some features had a huge number of
categories before one-hot-encoding).

In MLlib, each labeled data point is represented as the object
`LabeledPoint(label, featureVector)`. The training set is an RDD of a lot of `LabeledPoint` objects,
`OHETrainData`. 

First, use `LogisticRegressionWithSGD` to train a model using `OHETrainData` with
the given hyperparameter configuration. `LogisticRegressionWithSGD` returns a
[LogisticRegressionModel](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.regression.LogisticRegressionModel).
Next, use the `LogisticRegressionModel.weights` and `LogisticRegressionModel.intercept` attributes to
print out the model's parameters.

```python
from pyspark.mllib.classification import LogisticRegressionWithSGD

# fixed hyperparameters
numIters = 50
stepSize = 10.
regParam = 1e-6
regType = 'l2'
includeIntercept = True

model0 = LogisticRegressionWithSGD.train(OHETrainData, iterations=numIters, step=stepSize, 
                                         regParam=regParam, regType=regType, intercept=includeIntercept)
sortedWeights = sorted(model0.weights)
print sortedWeights[:5], model0.intercept

'''
output:
[-0.4589923685357562, -0.37973707648623972, -0.3699655826675331, -0.36934962879928285, -0.32697945415010637] 0.56455084025
'''

```

log loss is used to evaluate the model. To compute the log loss, we first need the predictions. To
generate predictions from our trained model, we use the model's weight and intercept attributes and 
pass them into the sigmoid function.

```python
from math import exp #  exp(-t) = e^-t

def getP(x, w, intercept):
    """Calculate the probability for an observation given a set of weights and intercept.

    Note:
        We'll bound our raw prediction between 20 and -20 for numerical purposes.

    Args:
        x (SparseVector): A vector with values of 1.0 for features that exist in this
            observation and 0.0 otherwise.
        w (DenseVector): A vector of weights (betas) for the model.
        intercept (float): The model's intercept.

    Returns:
        float: A probability between 0 and 1.
    """
    rawPrediction = w.dot(x) + intercept

    # Bound the raw prediction value
    rawPrediction = min(rawPrediction, 20)
    rawPrediction = max(rawPrediction, -20)
    pred = (1 + exp(-rawPrediction))**(-1)
    return pred

trainingPredictions = OHETrainData.map(lambda point: getP(point.features, model0.weights, model0.intercept))

print trainingPredictions.take(5)

'''
output:
[0.30262882023911125, 0.10362661997434075, 0.283634247838756, 0.17846102057880114, 0.5389775379218853]
'''

```

Now the \\( p \\)'s are calculated (probabilities for the data points to be in class y = 1), the 
log loss can be computed with \\( p \\) and the true labels \\( y \\).

```python
from math import log

def computeLogLoss(p, y):
    """Calculates the value of log loss for a given probabilty and label.

    Note:
        log(0) is undefined, so when p is 0 we need to add a small value (epsilon) to it
        and when p is 1 we need to subtract a small value (epsilon) from it.

    Args:
        p (float): A probabilty between 0 and 1.
        y (int): A label.  Takes on the values 0 and 1.

    Returns:
        float: The log loss value.
    """
    epsilon = 10e-12
    if p == 0:
        p += epsilon
    if p == 1:
        p -= epsilon
    return -y * log(p) - (1 - y) * log(1-p)

classOneFracTrain = OHETrainData.map(lambda point: point.label).mean()
print classOneFracTrain

logLossTrBase = OHETrainData.map(lambda point: computeLogLoss(classOneFracTrain, point.label)).mean()
print 'Baseline Train Logloss = {0:.3f}\n'.format(logLossTrBase)

'''
output:
0.22717773523
Baseline Train Logloss = 0.536
'''

def evaluateResults(model, data):
    """Calculates the log loss for the data given the model.

    Args:
        model (LogisticRegressionModel): A trained logistic regression model.
        data (RDD of LabeledPoint): Labels and features for each observation.

    Returns:
        float: Log loss for the data.
    """
    w = model.weights
    intercept = model.intercept
    logloss = data.map(lambda point: computeLogLoss(getP(point.features, w, intercept), point.label)).mean()
    return logloss

logLossTrLR0 = evaluateResults(model0, OHETrainData)
print ('OHE Features Train Logloss:\n\tBaseline = {0:.3f}\n\tLogReg = {1:.3f}'
       .format(logLossTrBase, logLossTrLR0))

'''
output:
OHE Features Train Logloss:
    Baseline = 0.536
    LogReg = 0.457
'''

classOneFracVal = OHEValidationData.map(lambda point: point.label).mean()
logLossValBase = OHEValidationData.map(lambda point: computeLogLoss(classOneFracTrain, point.label)).mean()

logLossValLR0 = evaluateResults(model0, OHEValidationData)
print ('OHE Features Validation Logloss:\n\tBaseline = {0:.3f}\n\tLogReg = {1:.3f}'
       .format(logLossValBase, logLossValLR0))

'''
output:
OHE Features Validation Logloss:
    Baseline = 0.528
    LogReg = 0.457
'''
```

After feature hashing and training several models for different step-sizes and regularization constants,
the evaluation of the best model is shown below.

```python
logLossTest = evaluateResults(bestModel, hashTestData)

# Log loss for the baseline model
classOneFracTest = hashTestData.map(lambda point: point.label).mean()
logLossTestBaseline = hashTestData.map(lambda point: computeLogLoss(classOneFracTest, point.label)).mean()

print ('Hashed Features Test Log Loss:\n\tBaseline = {0:.3f}\n\tLogReg = {1:.3f}'
       .format(logLossTestBaseline, logLossTest))

'''
output:
Hashed Features Test Log Loss:
    Baseline = 0.537
    LogReg = 0.456
'''
```





