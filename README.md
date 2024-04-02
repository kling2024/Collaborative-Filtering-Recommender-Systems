# Collaborative Filtering Recommender Systems

Here, I implemented collaborative filtering to build a recommender system for movies. 

##  Packages
I used the now familiar NumPy and Tensorflow Packages.

```python
import numpy as np
import tensorflow as tf
from tensorflow import keras
from recsys_utils import *
```

## Notation

|  General  Notation  |                            Description                                    | Python (if any)|
|:-------------------:|:-------------------------------------------------------------------------:|:--------------:|
|  $r(i,j)$           |  scalar; = 1  if user j rated movie , = 0  otherwise                      |        /       |
|  $y(i,j)$           |  scalar; = rating given by user j on movie  i (if r(i,j) = 1 is defined)  |        /       |
|  $\mathbf{w}^{(j)}$ |  vector; parameters for user j                                            |        /       |
|  $b^{(j)}$          |  scalar; parameter for user j                                             |        /       |
|  $\mathbf{x}^{(i)}$ |  vector; feature ratings for movie i                                      |        /       |
|  $n_u$              |  number of users                                                          |   num_users    |
|  $n_m$              |  number of movies                                                         |   um_movies    |
|  $n$                |  Enumber of features                                                      |   num_features |
|  $\mathbf{X}$       |  matrix of vectors $\mathbf{x}^{(i)}$                                     |   X            |
|  $\mathbf{W}$       |  matrix of vectors $\mathbf{w}^{(j)}$                                     |   W            |
|  $\mathbf{b}$       |  vector of bias parameters $b^{(j)}$                                      |   b            |
|  $\mathbf{R}$       |  matrix of elements $r(i,j)$                                              |   R            |


## Recommender Systems
Here, I implemented the collaborative filtering learning algorithm and apply it to a dataset of movie ratings. The goal of a collaborative filtering recommender system is to generate two vectors: For each user, a 'parameter vector' that embodies the movie tastes of a user. For each movie, a feature vector of the same size which embodies some description of the movie. The dot product of the two vectors plus the bias term should produce an estimate of the rating the user might give to that movie.

$Y$ contains ratings; 0.5 to 5 inclusive in 0.5 steps. 0 if the movie has not been rated. $R$ has a 1 where movies have been rated. Movies are in rows, users in columns. Each user has a parameter vector $w^{user}$ and bias. Each movie has a feature vector $x^{movie}$. These vectors are simultaneously learned by using the existing user/movie ratings as training data. One training example is shown above: $\mathbf{w}^{(1)} \cdot \mathbf{x}^{(1)} + b^{(1)} = 4$. It is worth noting that the feature vector $x^{movie}$ must satisfy all the users while the user vector $w^{user}$ must satisfy all the movies. This is the source of the name of this approach - all the users collaborate to generate the rating set. Once the feature vectors and parameters are learned, they can be used to predict how a user might rate an unrated movie. 

Here, I implemented the function `cofiCostFunc` that computed the collaborative filtering objective function. After implementing the objective function, I used a TensorFlow custom training loop to learn the parameters for collaborative filtering. The first step was to detail the data set and data structures that would be used.

## Movie ratings dataset
The data set is derived from the [MovieLens "ml-latest-small"](https://grouplens.org/datasets/movielens/latest/) dataset.   
[F. Maxwell Harper and Joseph A. Konstan. 2015. The MovieLens Datasets: History and Context. ACM Transactions on Interactive Intelligent Systems (TiiS) 5, 4: 19:1–19:19. <https://doi.org/10.1145/2827872>]

The original dataset has 9000 movies rated by 600 users. The dataset has been reduced in size to focus on movies from the years since 2000. This dataset consists of ratings on a scale of 0.5 to 5 in 0.5 step increments. The reduced dataset has $n_u = 443$ users, and $n_m= 4778$ movies. 

Below, I loaded the movie dataset into the variables $Y$ and $R$.

The matrix $Y$ (a  $n_m \times n_u$ matrix) stores the ratings $y^{(i,j)}$. The matrix $R$ is an binary-valued indicator matrix, where $R(i,j) = 1$ if user $j$ gave a rating to movie $i$, and $R(i,j)=0$ otherwise. 

Throughout this part of the exercise, I would also be working with the matrices, $\mathbf{X}$, $\mathbf{W}$ and $\mathbf{b}$: 

$$\mathbf{X} = 
\begin{bmatrix}
--- (\mathbf{x}^{(0)})^T --- \\
--- (\mathbf{x}^{(1)})^T --- \\
\vdots \\
--- (\mathbf{x}^{(n_m-1)})^T --- \\
\end{bmatrix} , \quad
\mathbf{W} = 
\begin{bmatrix}
--- (\mathbf{w}^{(0)})^T --- \\
--- (\mathbf{w}^{(1)})^T --- \\
\vdots \\
--- (\mathbf{w}^{(n_u-1)})^T --- \\
\end{bmatrix},\quad
\mathbf{ b} = 
\begin{bmatrix}
 b^{(0)}  \\
 b^{(1)} \\
\vdots \\
b^{(n_u-1)} \\
\end{bmatrix}\quad
$$ 

The $i$-th row of $\mathbf{X}$ corresponds to the feature vector $x^{(i)}$ for the $i$-th movie, and the $j$-th row of $\mathbf{W}$ corresponds to one parameter vector $\mathbf{w}^{(j)}$, for the $j$-th user. Both $x^{(i)}$ and $\mathbf{w}^{(j)}$ are $n$-dimensional vectors. For the purposes of this exercise, I would use $n=10$, and therefore, $\mathbf{x}^{(i)}$ and $\mathbf{w}^{(j)}$ have 10 elements. Correspondingly, $\mathbf{X}$ is a $n_m \times 10$ matrix and $\mathbf{W}$ is a $n_u \times 10$ matrix.

I started by loading the movie ratings dataset to understand the structure of the data.I would load $Y$ and $R$ with the movie dataset. I would also loaded $\mathbf{X}$, $\mathbf{W}$, and $\mathbf{b}$ with pre-computed values. These values would be learned later, but I would use pre-computed values to develop the cost model.

```python
#Load data
X, W, b, num_movies, num_features, num_users = load_precalc_params_small()
Y, R = load_ratings_small()

print("Y", Y.shape, "R", R.shape)
print("X", X.shape)
print("W", W.shape)
print("b", b.shape)
print("num_features", num_features)
print("num_movies",   num_movies)
print("num_users",    num_users)
```

    Y (4778, 443) R (4778, 443)
    X (4778, 10)
    W (443, 10)
    b (1, 443)
    num_features 10
    num_movies 4778
    num_users 443

```python
#  From the matrix, we can compute statistics like average rating.
tsmean =  np.mean(Y[0, R[0, :].astype(bool)])
print(f"Average rating for movie 1 : {tsmean:0.3f} / 5" )
```

    Average rating for movie 1 : 3.400 / 5

## Collaborative filtering learning algorithm

Here, I begun implementing the collaborative filtering learning algorithm. I started by implementing the objective function. 

The collaborative filtering algorithm in the setting of movie recommendations considers a set of $n$-dimensional parameter vectors $\mathbf{x}^{(0)},...,\mathbf{x}^{(n_m-1)}$, $\mathbf{w}^{(0)},...,\mathbf{w}^{(n_u-1)}$ and $b^{(0)},...,b^{(n_u-1)}$, where the model predicts the rating for movie $i$ by user $j$ as $y^{(i,j)} = \mathbf{w}^{(j)}\cdot \mathbf{x}^{(i)} + b^{(j)}$ . Given a dataset that consists of a set of ratings produced by some users on some movies, I wished to learn the parameter vectors $\mathbf{x}^{(0)},...,\mathbf{x}^{(n_m-1)}, \mathbf{w}^{(0)},...,\mathbf{w}^{(n_u-1)}$  and $b^{(0)},...,b^{(n_u-1)}$ that produce the best fit (minimizes the squared error).

I completed the code in cofiCostFunc to compute the cost function for collaborative filtering. 

### 4.1 Collaborative filtering cost function

The collaborative filtering cost function is given by
$$J({\mathbf{x}^{(0)},...,\mathbf{x}^{(n_m-1)},\mathbf{w}^{(0)},b^{(0)},...,\mathbf{w}^{(n_u-1)},b^{(n_u-1)}})= \left[ \frac{1}{2}\sum_{(i,j):r(i,j)=1}(\mathbf{w}^{(j)} \cdot \mathbf{x}^{(i)} + b^{(j)} - y^{(i,j)})^2 \right]
+ \underbrace{\left[
\frac{\lambda}{2}
\sum_{j=0}^{n_u-1}\sum_{k=0}^{n-1}(\mathbf{w}^{(j)}_k)^2
+ \frac{\lambda}{2}\sum_{i=0}^{n_m-1}\sum_{k=0}^{n-1}(\mathbf{x}_k^{(i)})^2
\right]}_{regularization}
\tag{1}$$
The first summation in (1) is "for all $i$, $j$ where $r(i,j)$ equals $1$" and could be written:

$$
= \left[ \frac{1}{2}\sum_{j=0}^{n_u-1} \sum_{i=0}^{n_m-1}r(i,j)*(\mathbf{w}^{(j)} \cdot \mathbf{x}^{(i)} + b^{(j)} - y^{(i,j)})^2 \right]
+\text{regularization}
$$

Here, I wrote cofiCostFunc (collaborative filtering cost function) to return this cost.

**For loop Implementation:**   
I Started by implementing the cost function using for loops. Consider developing the cost function in two steps. First, develop the cost function without regularization. A test case that does not include regularization has been provided below to test my implementation. Once that was working, add regularization and run the tests that include regularization.  Here, I was accumulating the cost for user $j$ and movie $i$ only if $R(i,j) = 1$.

```python
# GRADED FUNCTION: cofi_cost_func
# UNQ_C1

def cofi_cost_func(X, W, b, Y, R, lambda_):
    """
    Returns the cost for the content-based filtering
    Args:
      X (ndarray (num_movies,num_features)): matrix of item features
      W (ndarray (num_users,num_features)) : matrix of user parameters
      b (ndarray (1, num_users)            : vector of user parameters
      Y (ndarray (num_movies,num_users)    : matrix of user ratings of movies
      R (ndarray (num_movies,num_users)    : matrix, where R(i, j) = 1 if the i-th movies was rated by the j-th user
      lambda_ (float): regularization parameter
    Returns:
      J (float) : Cost
    """
    nm, nu = Y.shape
    J = 0
    ### START CODE HERE ###  
    for j in range(nu):
        w = W[j,:]
        b_j = b[0,j]
        for i in range(nm):
            x = X[i,:]
            y = Y[i,j]
            r = R[i,j]
            J += r * np.square((np.dot(w,x) + b_j - y ))
    J += (lambda_) * (np.sum(np.square(W)) + np.sum(np.square(X)))
    J = J/2
    ### END CODE HERE ### 

    return J
```

```python
# Reduce the data set size so that this runs faster
num_users_r = 4
num_movies_r = 5 
num_features_r = 3

X_r = X[:num_movies_r, :num_features_r]
W_r = W[:num_users_r,  :num_features_r]
b_r = b[0, :num_users_r].reshape(1,-1)
Y_r = Y[:num_movies_r, :num_users_r]
R_r = R[:num_movies_r, :num_users_r]

# Evaluate cost function
J = cofi_cost_func(X_r, W_r, b_r, Y_r, R_r, 0);
print(f"Cost: {J:0.2f}")
```

    Cost: 13.67

```python
# Evaluate cost function with regularization 
J = cofi_cost_func(X_r, W_r, b_r, Y_r, R_r, 1.5);
print(f"Cost (with regularization): {J:0.2f}")
```

    Cost (with regularization): 28.09

```python
# Public tests
from public_tests import *
test_cofi_cost_func(cofi_cost_func)
```

All tests passed!

**Vectorized Implementation**

It is important to create a vectorized implementation to compute $J$, since it will later be called many times during optimization. The linear algebra utilized is not the focus of this series, so the implementation is provided.

Here, I Ran the code below and verified that it produced the same results as the non-vectorized version.


```python
def cofi_cost_func_v(X, W, b, Y, R, lambda_):
    """
    Returns the cost for the content-based filtering
    Vectorized for speed. Uses tensorflow operations to be compatible with custom training loop.
    Args:
      X (ndarray (num_movies,num_features)): matrix of item features
      W (ndarray (num_users,num_features)) : matrix of user parameters
      b (ndarray (1, num_users)            : vector of user parameters
      Y (ndarray (num_movies,num_users)    : matrix of user ratings of movies
      R (ndarray (num_movies,num_users)    : matrix, where R(i, j) = 1 if the i-th movies was rated by the j-th user
      lambda_ (float): regularization parameter
    Returns:
      J (float) : Cost
    """
    j = (tf.linalg.matmul(X, tf.transpose(W)) + b - Y)*R
    J = 0.5 * tf.reduce_sum(j**2) + (lambda_/2) * (tf.reduce_sum(X**2) + tf.reduce_sum(W**2))
    return J
```


```python
# Evaluate cost function
J = cofi_cost_func_v(X_r, W_r, b_r, Y_r, R_r, 0);
print(f"Cost: {J:0.2f}")

# Evaluate cost function with regularization 
J = cofi_cost_func_v(X_r, W_r, b_r, Y_r, R_r, 1.5);
print(f"Cost (with regularization): {J:0.2f}")
```

    Cost: 13.67
    Cost (with regularization): 28.09


## Learning movie recommendations

After I finished implementing the collaborative filtering cost function, I started training my algorithm to make movie recommendations for myself. 

In the cell below, I entered my own movie choices. The algorithm would then make recommendations for me! I have filled out some values according to my preferences. A list of all movies in the dataset is in the file [movie list](data/small_movie_list.csv).


```python
movieList, movieList_df = load_Movie_List_pd()

my_ratings = np.zeros(num_movies)          #  Initialize my ratings

# Check the file small_movie_list.csv for id of each movie in our dataset
# For example, Toy Story 3 (2010) has ID 2700, so to rate it "5", you can set
my_ratings[2700] = 5 

#Or suppose you did not enjoy Persuasion (2007), you can set
my_ratings[2609] = 2;

# We have selected a few movies we liked / did not like and the ratings we
# gave are as follows:
my_ratings[929]  = 5   # Lord of the Rings: The Return of the King, The
my_ratings[246]  = 5   # Shrek (2001)
my_ratings[2716] = 3   # Inception
my_ratings[1150] = 5   # Incredibles, The (2004)
my_ratings[382]  = 2   # Amelie (Fabuleux destin d'Amélie Poulain, Le)
my_ratings[366]  = 5   # Harry Potter and the Sorcerer's Stone (a.k.a. Harry Potter and the Philosopher's Stone) (2001)
my_ratings[622]  = 5   # Harry Potter and the Chamber of Secrets (2002)
my_ratings[988]  = 3   # Eternal Sunshine of the Spotless Mind (2004)
my_ratings[2925] = 1   # Louis Theroux: Law & Disorder (2008)
my_ratings[2937] = 1   # Nothing to Declare (Rien à déclarer)
my_ratings[793]  = 5   # Pirates of the Caribbean: The Curse of the Black Pearl (2003)
my_rated = [i for i in range(len(my_ratings)) if my_ratings[i] > 0]

print('\nNew user ratings:\n')
for i in range(len(my_ratings)):
    if my_ratings[i] > 0 :
        print(f'Rated {my_ratings[i]} for  {movieList_df.loc[i,"title"]}');
```

    
    New user ratings:
    
    Rated 5.0 for  Shrek (2001)
    Rated 5.0 for  Harry Potter and the Sorcerer's Stone (a.k.a. Harry Potter and the Philosopher's Stone) (2001)
    Rated 2.0 for  Amelie (Fabuleux destin d'Amélie Poulain, Le) (2001)
    Rated 5.0 for  Harry Potter and the Chamber of Secrets (2002)
    Rated 5.0 for  Pirates of the Caribbean: The Curse of the Black Pearl (2003)
    Rated 5.0 for  Lord of the Rings: The Return of the King, The (2003)
    Rated 3.0 for  Eternal Sunshine of the Spotless Mind (2004)
    Rated 5.0 for  Incredibles, The (2004)
    Rated 2.0 for  Persuasion (2007)
    Rated 5.0 for  Toy Story 3 (2010)
    Rated 3.0 for  Inception (2010)
    Rated 1.0 for  Louis Theroux: Law & Disorder (2008)
    Rated 1.0 for  Nothing to Declare (Rien à déclarer) (2010)


 Then, I added these reviews to $Y$ and $R$ and normalize the ratings.


```python
# Reload ratings
Y, R = load_ratings_small()

# Add new user ratings to Y 
Y = np.c_[my_ratings, Y]

# Add new user indicator matrix to R
R = np.c_[(my_ratings != 0).astype(int), R]

# Normalize the Dataset
Ynorm, Ymean = normalizeRatings(Y, R)
```

Then, I prepared to train the model. Initialize the parameters and select the Adam optimizer.


```python
#  Useful Values
num_movies, num_users = Y.shape
num_features = 100

# Set Initial Parameters (W, X), use tf.Variable to track these variables
tf.random.set_seed(1234) # for consistent results
W = tf.Variable(tf.random.normal((num_users,  num_features),dtype=tf.float64),  name='W')
X = tf.Variable(tf.random.normal((num_movies, num_features),dtype=tf.float64),  name='X')
b = tf.Variable(tf.random.normal((1,          num_users),   dtype=tf.float64),  name='b')

# Instantiate an optimizer.
optimizer = keras.optimizers.Adam(learning_rate=1e-1)
```

Here, I trained the collaborative filtering model. This would learn the parameters $\mathbf{X}$, $\mathbf{W}$, and $\mathbf{b}$. 

The operations involved in learning $w$, $b$, and $x$ simultaneously do not fall into the typical 'layers' offered in the TensorFlow neural network package.  Consequently, the flow used in Course 2: Model, Compile(), Fit(), Predict(), are not directly applicable. Instead, I could use a custom training loop.
Recall from earlier labs the steps of gradient descent (repeat until convergence) : 1. compute forward pass; 2. compute the derivatives of the loss relative to parameters; 3.update the parameters using the learning rate and the computed derivatives. 
    
TensorFlow has the marvelous capability of calculating the derivatives for me.  Within the `tf.GradientTape()` section, operations on Tensorflow Variables are tracked. When `tape.gradient()` is later called, it will return the gradient of the loss relative to the tracked variables. The gradients can then be applied to the parameters using an optimizer. This is a very brief introduction to a useful feature of TensorFlow and other machine learning frameworks. Further information can be found by investigating "custom training loops" within the framework of interest.

```python
iterations = 200
lambda_ = 1
for iter in range(iterations):
    # Use TensorFlow’s GradientTape
    # to record the operations used to compute the cost 
    with tf.GradientTape() as tape:

        # Compute the cost (forward pass included in cost)
        cost_value = cofi_cost_func_v(X, W, b, Ynorm, R, lambda_)

    # Use the gradient tape to automatically retrieve
    # the gradients of the trainable variables with respect to the loss
    grads = tape.gradient( cost_value, [X,W,b] )

    # Run one step of gradient descent by updating
    # the value of the variables to minimize the loss.
    optimizer.apply_gradients( zip(grads, [X,W,b]) )

    # Log periodically.
    if iter % 20 == 0:
        print(f"Training loss at iteration {iter}: {cost_value:0.1f}")
```

    Training loss at iteration 0: 2321191.3
    Training loss at iteration 20: 136169.3
    Training loss at iteration 40: 51863.7
    Training loss at iteration 60: 24599.0
    Training loss at iteration 80: 13630.6
    Training loss at iteration 100: 8487.7
    Training loss at iteration 120: 5807.8
    Training loss at iteration 140: 4311.6
    Training loss at iteration 160: 3435.3
    Training loss at iteration 180: 2902.1


## Recommendations
Below, I computed the ratings for all the movies and users and display the movies that were recommended. These were based on the movies and ratings entered as `my_ratings[]` above. To predict the rating of movie $i$ for user $j$, I computed $\mathbf{w}^{(j)} \cdot \mathbf{x}^{(i)} + b^{(j)}$. This could be computed for all ratings using matrix multiplication.


```python
# Make a prediction using trained weights and biases
p = np.matmul(X.numpy(), np.transpose(W.numpy())) + b.numpy()

#restore the mean
pm = p + Ymean

my_predictions = pm[:,0]

# sort predictions
ix = tf.argsort(my_predictions, direction='DESCENDING')

for i in range(17):
    j = ix[i]
    if j not in my_rated:
        print(f'Predicting rating {my_predictions[j]:0.2f} for movie {movieList[j]}')

print('\n\nOriginal vs Predicted ratings:\n')
for i in range(len(my_ratings)):
    if my_ratings[i] > 0:
        print(f'Original {my_ratings[i]}, Predicted {my_predictions[i]:0.2f} for {movieList[i]}')
```

    Predicting rating 4.49 for movie My Sassy Girl (Yeopgijeogin geunyeo) (2001)
    Predicting rating 4.48 for movie Martin Lawrence Live: Runteldat (2002)
    Predicting rating 4.48 for movie Memento (2000)
    Predicting rating 4.47 for movie Delirium (2014)
    Predicting rating 4.47 for movie Laggies (2014)
    Predicting rating 4.47 for movie One I Love, The (2014)
    Predicting rating 4.46 for movie Particle Fever (2013)
    Predicting rating 4.45 for movie Eichmann (2007)
    Predicting rating 4.45 for movie Battle Royale 2: Requiem (Batoru rowaiaru II: Chinkonka) (2003)
    Predicting rating 4.45 for movie Into the Abyss (2011)
    
    
    Original vs Predicted ratings:
    
    Original 5.0, Predicted 4.90 for Shrek (2001)
    Original 5.0, Predicted 4.84 for Harry Potter and the Sorcerer's Stone (a.k.a. Harry Potter and the Philosopher's Stone) (2001)
    Original 2.0, Predicted 2.13 for Amelie (Fabuleux destin d'Amélie Poulain, Le) (2001)
    Original 5.0, Predicted 4.88 for Harry Potter and the Chamber of Secrets (2002)
    Original 5.0, Predicted 4.87 for Pirates of the Caribbean: The Curse of the Black Pearl (2003)
    Original 5.0, Predicted 4.89 for Lord of the Rings: The Return of the King, The (2003)
    Original 3.0, Predicted 3.00 for Eternal Sunshine of the Spotless Mind (2004)
    Original 5.0, Predicted 4.90 for Incredibles, The (2004)
    Original 2.0, Predicted 2.11 for Persuasion (2007)
    Original 5.0, Predicted 4.80 for Toy Story 3 (2010)
    Original 3.0, Predicted 3.00 for Inception (2010)
    Original 1.0, Predicted 1.41 for Louis Theroux: Law & Disorder (2008)
    Original 1.0, Predicted 1.26 for Nothing to Declare (Rien à déclarer) (2010)


In practice, additional information can be utilized to enhance my predictions. Above, the predicted ratings for the first few hundred movies lie in a small range. I can augment the above by selecting from those top movies, movies that have high average ratings and movies with more than 20 ratings. This section uses a [Pandas](https://pandas.pydata.org/) data frame which has many handy sorting features.


```python
filter=(movieList_df["number of ratings"] > 20)
movieList_df["pred"] = my_predictions
movieList_df = movieList_df.reindex(columns=["pred", "mean rating", "number of ratings", "title"])
movieList_df.loc[ix[:300]].loc[filter].sort_values("mean rating", ascending=False)
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>pred</th>
      <th>mean rating</th>
      <th>number of ratings</th>
      <th>title</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1743</th>
      <td>4.030961</td>
      <td>4.252336</td>
      <td>107</td>
      <td>Departed, The (2006)</td>
    </tr>
    <tr>
      <th>2112</th>
      <td>3.985281</td>
      <td>4.238255</td>
      <td>149</td>
      <td>Dark Knight, The (2008)</td>
    </tr>
    <tr>
      <th>211</th>
      <td>4.477798</td>
      <td>4.122642</td>
      <td>159</td>
      <td>Memento (2000)</td>
    </tr>
    <tr>
      <th>929</th>
      <td>4.887054</td>
      <td>4.118919</td>
      <td>185</td>
      <td>Lord of the Rings: The Return of the King, The...</td>
    </tr>
    <tr>
      <th>2700</th>
      <td>4.796531</td>
      <td>4.109091</td>
      <td>55</td>
      <td>Toy Story 3 (2010)</td>
    </tr>
    <tr>
      <th>653</th>
      <td>4.357304</td>
      <td>4.021277</td>
      <td>188</td>
      <td>Lord of the Rings: The Two Towers, The (2002)</td>
    </tr>
    <tr>
      <th>1122</th>
      <td>4.004471</td>
      <td>4.006494</td>
      <td>77</td>
      <td>Shaun of the Dead (2004)</td>
    </tr>
    <tr>
      <th>1841</th>
      <td>3.980649</td>
      <td>4.000000</td>
      <td>61</td>
      <td>Hot Fuzz (2007)</td>
    </tr>
    <tr>
      <th>3083</th>
      <td>4.084643</td>
      <td>3.993421</td>
      <td>76</td>
      <td>Dark Knight Rises, The (2012)</td>
    </tr>
    <tr>
      <th>2804</th>
      <td>4.434171</td>
      <td>3.989362</td>
      <td>47</td>
      <td>Harry Potter and the Deathly Hallows: Part 1 (...</td>
    </tr>
    <tr>
      <th>773</th>
      <td>4.289676</td>
      <td>3.960993</td>
      <td>141</td>
      <td>Finding Nemo (2003)</td>
    </tr>
    <tr>
      <th>1771</th>
      <td>4.344999</td>
      <td>3.944444</td>
      <td>81</td>
      <td>Casino Royale (2006)</td>
    </tr>
    <tr>
      <th>2649</th>
      <td>4.133481</td>
      <td>3.943396</td>
      <td>53</td>
      <td>How to Train Your Dragon (2010)</td>
    </tr>
    <tr>
      <th>2455</th>
      <td>4.175743</td>
      <td>3.887931</td>
      <td>58</td>
      <td>Harry Potter and the Half-Blood Prince (2009)</td>
    </tr>
    <tr>
      <th>361</th>
      <td>4.135287</td>
      <td>3.871212</td>
      <td>132</td>
      <td>Monsters, Inc. (2001)</td>
    </tr>
    <tr>
      <th>3014</th>
      <td>3.967900</td>
      <td>3.869565</td>
      <td>69</td>
      <td>Avengers, The (2012)</td>
    </tr>
    <tr>
      <th>246</th>
      <td>4.897137</td>
      <td>3.867647</td>
      <td>170</td>
      <td>Shrek (2001)</td>
    </tr>
    <tr>
      <th>151</th>
      <td>3.971892</td>
      <td>3.836364</td>
      <td>110</td>
      <td>Crouching Tiger, Hidden Dragon (Wo hu cang lon...</td>
    </tr>
    <tr>
      <th>1150</th>
      <td>4.898892</td>
      <td>3.836000</td>
      <td>125</td>
      <td>Incredibles, The (2004)</td>
    </tr>
    <tr>
      <th>793</th>
      <td>4.874936</td>
      <td>3.778523</td>
      <td>149</td>
      <td>Pirates of the Caribbean: The Curse of the Bla...</td>
    </tr>
    <tr>
      <th>366</th>
      <td>4.843375</td>
      <td>3.761682</td>
      <td>107</td>
      <td>Harry Potter and the Sorcerer's Stone (a.k.a. ...</td>
    </tr>
    <tr>
      <th>754</th>
      <td>4.021778</td>
      <td>3.723684</td>
      <td>76</td>
      <td>X2: X-Men United (2003)</td>
    </tr>
    <tr>
      <th>79</th>
      <td>4.242986</td>
      <td>3.699248</td>
      <td>133</td>
      <td>X-Men (2000)</td>
    </tr>
    <tr>
      <th>622</th>
      <td>4.878342</td>
      <td>3.598039</td>
      <td>102</td>
      <td>Harry Potter and the Chamber of Secrets (2002)</td>
    </tr>
  </tbody>
</table>
</div>

# GitHub and Medium

**GitHub Link** [**here**](https://github.com/kling2024?tab=repositories)

**Medium Link** [**here**](https://medium.com/@kling_7476)
