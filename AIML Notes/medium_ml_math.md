# 📐 Mathematical Foundations of Machine Learning (Medium Level)

## What is this?
A clearer, slightly deeper version of core ML math. Still intuitive, but with a bit more explanation and structure.

---

# 1. Linear Algebra

## Vectors
A vector is an ordered list of numbers that represents a data point.

Example:
a = [1, 2, 3]  
b = [4, 0, -1]

Operations:
- Addition: [5, 2, 2]
- Scalar multiply (×2): [2, 4, 6]
- Dot product: (1×4 + 2×0 + 3×-1) = 1

👉 Why it matters:
Prediction in ML = weights · features (dot product)

---

## Matrices
A matrix is a 2D table of numbers (rows × columns).

Example:
Dataset with 3 students and 2 features:

X =  
[80, 5]  
[70, 3]  
[90, 6]

Matrix multiplication:
- Combines inputs with weights
- Core operation in neural networks

👉 Key rule:
(m×k) × (k×n) = (m×n)

---

## Systems of Equations
Written as:

Ax = b

Example:
2x + 3y = 7  
x + 2y = 4  

Solution:
x = 2, y = 1

👉 In ML:
Linear regression solves a system like:
Xw ≈ y

Closed-form:
w = (XᵀX)⁻¹ Xᵀy

---

## Eigenvalues & Eigenvectors
Special vectors that only scale when transformed.

Formula:
A v = λ v

- v = direction
- λ = scaling factor

👉 In ML:
Used in PCA to reduce dimensions while keeping important information.

---

# 2. Probability & Statistics

## Probability
Measures likelihood (0 to 1)

Example:
P(heads) = 0.5

Conditional probability:
P(A|B) = P(A and B) / P(B)

---

## Random Variables
A variable whose value depends on randomness.

- Discrete: dice roll
- Continuous: height

---

## Distributions
Common ones:

- Normal (Gaussian): bell curve
- Bernoulli: 0 or 1
- Binomial: number of successes

Example:
Coin flipped 5 times → probability of 3 heads ≈ 0.31

---

## Expectation & Variance

Expectation (mean):
E[X] = average value

Variance:
Var(X) = how spread out values are

Example:
[1,2,3,4,5] → mean = 3

👉 In ML:
MSE = average squared error

---

## Bayes’ Theorem

Formula:
P(A|B) = (P(B|A) × P(A)) / P(B)

Idea:
Update belief after seeing evidence.

👉 In ML:
Used in Naive Bayes classifiers.

---

# 3. Gradient Descent

## Core Idea
Minimize error step by step.

Like going downhill to find the lowest point.

---

## Loss Function
Measures model error.

Example:
MSE = (1/n) Σ (y − ŷ)²

---

## Gradient
Vector of partial derivatives.

Shows direction of steepest increase.

We move in the opposite direction.

---

## Update Rule

θ = θ − η ∇J(θ)

- θ = parameters
- η = learning rate

---

## Learning Rate
Controls step size.

- Too large → unstable
- Too small → slow

---

## Batch vs Mini-Batch

- Batch: full dataset (slow)
- SGD: one sample (noisy)
- Mini-batch: small chunks (best balance)

---

# 🎯 Summary

- Linear Algebra → structure of data
- Probability → uncertainty handling
- Gradient Descent → learning process

👉 Most ML boils down to:
prediction = math(data, parameters)  
learning = reduce error using gradients
