---
layout: post
title: "An Overview of Logistic Regression"
comments: True
---

#### _Why the sigmoid?_

When I first started to learn logistic regression, I realized there was something missing when the 
professor explained why we would want a sigmoid function instead of a 
hard-thresholding 0/1 loss. It was said it’s because sometimes we want the output to be a probability
instead of a yes/no decision for practical purposes, and the logistic sigmoid function maps all real 
numbers to the interval (0, 1). On the other hand, optimizing the log loss is much easier than the 
0/1 loss since the former cost function is convex. I thought, isn’t it strange to have the 
model first, and then justify its legitimacy by the result? Why did people come 
up with the sigmoid function in the first place? Why not use other odd (centered on 0.5) functions 
that map all real numbers to (0, 1), what’s so special about the sigmoid function? 

Up till now I have taken quite some machine learning courses, including the ones from Prof. A. Ihler,
Prof. P. Smyth at UC Irvine, and the famous Andrew Ng course, Learning from Data by Caltech, Data Science 
Specialization courses by JHU, and the BerkeleyX Data Science on Apache Spark series. They all have 
different emphases and are excellent in their own way. However, most of them ignored the simple and 
fundamental question that how sigmoid function is connected with probability and went on explaining 
the optimization. According to the wording by Prof. Y. Abu-Mostafa at Caltech, logistic regression 
gives “genuine probability”, not just any number between 0 and 1. But why the sigmoid form? I found 
the answer in the Bishop book. As I expected, the reason is simple and elegant.

For two-class problems, we adopt a generative approach in which we model the class-conditional 
densities (likelihood) \\( P(x|C\_k) \\), as well as the class prior \\( P(C\_k) \\), and then use these to 
compute posterior probabilities \\( P(C\_k|x) \\) through Bayes Theorem. The posterior for class \\( C\_1 \\) 
can be written as

<br>
<center><img src="/assets/images/logreg/daum_equation_001.png" alt="Drawing" style="width: 400px;"></center>
<br>

This logistic sigmoid function plays an important role in many classification algorithms. It satisfies 
the symmetry property \\( σ(-a) = 1 - σ(a) \\). The inverse of the logistic sigmoid is given by

<br>
<center><img src="/assets/images/logreg/daum_equation_002.png" alt="Drawing" style="width: 400px;"></center>
<br>

and is known as the **logit** function. It represents the log of the ratio of probabilities 
\\( ln[P(C\_1|x)/P(C\_2|x)] \\) for the two classes, also known as the **log odds**. In logistic regression, there 
are two layers for the feature vector x to come to the result probability. The second layer is what 
we have shown - the logistic function - the nonlinear layer. The first layer is linear, which is the 
signal \\( a(x) \\).

<br>
<center><img src="/assets/images/logreg/daum_equation_003.png" alt="Drawing" style="width: 450px;"></center>
<br>

That’s it. This is the relationship between the sigmoid form and probability and the reason why the 
output of logistic regression should be in this form. Many generative models can give rise to this 
logistic posterior, e.g., the class-conditional density (likelihood) is Gaussian or Poisson. In fact, 
logistic regression has much weaker assumptions than the assumptions made by Gaussian Discriminant 
Analysis (GDA) (MLAPP section 8.6 Generative vs Discriminative classifiers).

For the case of more than two classes, we have the **normalized exponential** as shown below, and can be 
regarded as a multiclass generalization of the logistic sigmoid.

<br>
<center><img src="/assets/images/logreg/daum_equation_005.png" alt="Drawing" style="width: 250px;"></center>
<br>

The normalized exponential is also known as the **softmax** function (of a’s), and has the same form 
as the **Boltzmann distribution** in statistical physics (aha!).

<br>
#### An Overview of the Theory

<br>
<center><img src="/assets/images/logreg/caltech1.png" alt="Drawing" style="width: 600px;"></center>
<br>

**Data** vectors: \\( (x\_n, y\_n) \\)

\\( x\_n \\) is the feature vector, \\( y\_n \\) is the label.

**Hypothesis**: \\( h(x) = σ[a(x)] \\), interpreted as a probability

a(x) is a linear function of the feature vector x (some higher order polynomial of the original 
features if higher complexity is needed), and σ(a) is the logistic sigmoid function that maps the 
output of the linear layer to (0, 1) as a probability of that data point being in the class 1.

**Goal**: to compute the probability of each data point being in each class by optimizing the logistic 
cost function. Potentially classify the data points by thresholding on the probability.

Quoting Prof. Y. Abu-Mostafa, “The output of logistic regression is treated *genuinely as probability 
even during learning*. ” The data that is given to us does not tell us the probability, only the labels. 
These labels affect the model we fit and the probability we get from it.

<br>
<center><img src="/assets/images/logreg/daum_equation_101.png" alt="Drawing" style="width: 300px;"></center>
<br>

+1 and -1 represent the class labels. We are trying to learn f from the data, notwithstanding the 
fact that examples are giving us only values of y, that happen to be generated by f. We want to take 
the examples, and then generate h that approximates the hidden target function, f.

Learn:

$$ h(x) = σ(w^T x) \approx f(x) $$

Since we have already shown the legitimacy of using the logistic sigmoid σ here, we are going to make 
the approximation as good as possible according to some error measure. How do we choose the error 
measure? It must have two properties,

1. Something that if we minimize it, we are going to do well. Plausible.
2. Something that will be friendly to the optimizer.

Likelihood is a plausible error measure with the two properties. Now we ask, if h = f, how likely 
will we get a certain y from **x**? Note that \\( σ(-a) = 1 - σ(a) \\), we can write

<br>
<center><img src="/assets/images/logreg/daum_equation_103.png" alt="Drawing" style="width: 300px;"></center>
<br>

Hence, with the assumption of independence, the likelihood of the data 
\\( D = (\mathbf{x}\_1, y\_1), (\mathbf{x}\_2, y\_2), …, (\mathbf{x}\_N, y\_N) \\) is

<br>
<center><img src="/assets/images/logreg/daum_equation_104.png" alt="Drawing" style="width: 250px;"></center>
<br>

Then we show that maximizing the likelihood is equivalent to minimizing error. (The error is always 
of a certain form, namely the mean of the sum of some deviance between the label and the prediction 
given by the hypothesis.)

<br>
<center><img src="/assets/images/logreg/daum_equation_105.png" alt="Drawing" style="width: 300px;"></center>
<br>

Don’t forget it’s a function of the parameter, the weight vector w, with given yn and xn. The summand 
is the in-sample error \\( E\_{in} \\), and it is of the cross-entropy error form. It’s a convex function but 
has no closed-form solution. How to minimize it? Use gradient descent.

We will update the weight vector with step size η and step unit vector v such that 

$$ w(1) = w(0) + ηv $$

Therefore the change of the in-sample error is

<br>
<center><img src="/assets/images/logreg/daum_equation_106.png" alt="Drawing" style="width: 500px;"></center>
<br>

We want the change of \\( E\_{in} \\) to be as negative as possible, hence

<br>
<center><img src="/assets/images/logreg/daum_equation_107.png" alt="Drawing" style="width: 200px;"></center>
<br>

Instead of having a fixed step size, sometimes leads to slow convergence, a better alternative is to 
have a fixed **learning rate**. It basically means that we incorporate the magnitude of the vector 
\\( E\_{in}[w(0)] \\)
into η, so that the step size is proportional to the magnitude of the gradient.

<br>
<center><img src="/assets/images/logreg/caltech2.png" alt="Drawing" style="width: 600px;"></center>
<br>

This makes sense, since the steeper the surface is, we are more confident to take a large step, and 
vice versa. Therefore the expression for the weight vector update is

<br>
<center><img src="/assets/images/logreg/daum_equation_108.png" alt="Drawing" style="width: 150px;"></center>
<br>

Finally, the logistic regression algorithm is shown below.

<br>
<center><img src="/assets/images/logreg/caltech3.png" alt="Drawing" style="width: 600px;"></center>
<br>

##### Some notes on running the algorithm

- Initialization. For logistic regression, **w**(0) can be 0, while starting at σ(0) = 0.5 
makes sense because it’s the most uncertain state.

- Termination. Again, not so much a problem for logistic regression as for other models like neural 
networks, since the logistic optimization problem is nicely convex. Generally we use a combination 
of criteria to serve as a stop condition. For example, 
let Δ**w** < threshold. Pathological behaviors arise when the function looks like

<center><img src="/assets/images/logreg/caltech4.png" alt="Drawing" style="width: 500px;"></center>

- Sometimes a better approach is the _Stochastic Gradient Descent_. It’s remarkably efficient and 
often yields good results.

- The king of all derivative-based methods: 
[conjugate gradient](https://en.wikipedia.org/wiki/Conjugate_gradient_method). The idea is doing 2nd order 
optimization without actually compute the Hessian.
