# Machine Learning Engineer Nanodegree
## Capstone Project
Simon Jackson
April 24th, 2017

## I. Definition
_(approx. 1-2 pages)_

### Project Overview

To cope with overwhelming options, it is essential for e-commerce sites to implement recommender systems that predict the ratings or preferences of users for products[^c1]. This ensures that users can be provided with personalized results to save them time and improve the user experience. For example, Netflix will try to present movies to you that you will like, rather than you having to sift through their entire database to find what you are looking for. Similarly, Amazon ought to recommend books you're likely to enjoy, rather than you having to read the blurb of every book to find something appealing.

Recommender systems often take the form of collaborative or content-based filtering. The former involves predicting what a user will like based on their similarity to other users[^c2]. The latter involves matching content to a user based on similarities among the content[^c3]. Hybrid approaches, which combine content-based and collaborative filtering, can also be constructed.

In this project I will compare the ability of various recommender systems to predict how different users will rate movies (out of 5-stars) that have **not been rated by any users**. This problem prohibits the use of purely collaborative filtering approaches, instead depending on content-based or hybrid recommender systems. These systems will be trained by combining information from the [MovieLens 20M](https://www.kaggle.com/grouplens/movielens-20m-dataset) and [IMDB 5000 Movie](https://www.kaggle.com/deepmatrix/imdb-5000-movie-dataset) datasets, which are both available via the online machine learning challenge platform, [Kaggle](https://www.kaggle.com/).

### Problem Statement

Imagine you work at Netflix and have added new movies to the service. You'd like to recommend these to people who will like them. To determine whether a particular user might like one of these movies, it's impossible to see if other, similar users like the movie (because it's new and hasn't been rated). It's also challenging to see how the user rated other similar movies, because user ratings are relatively sparse.

This problem of estimating user ratings for new movies can be visualised in the Figure below. In this Figure, users are represented as rows, and movies in the data base as columns. Cells are populated with user ratings (from 1 to 5), which are sparse. The right-most column represents the introduction of a new movie for which no user ratings exist. The problem is to estimate the ratings for these cells denoted "?".

![challenge](https://github.com/drsimonj/machine-learning/blob/master/projects/capstone/imgs/challenge.png?raw=true)

In order to estimate these ratings, a separate source of information is required: features about the movies. In this project, this information will be features scraped from the movie review site, IMDB. The Figure below represents an example of how this data looks.

![imdb](https://github.com/drsimonj/machine-learning/blob/master/projects/capstone/imgs/imdb.png?raw=true)

Unlike the rating data for which user's score are unknown, information from the IMDB data base is known for all movies, including those being newly added for which the ratings wish to be estimated. This information makes it possible to determine how similar the new movie is to movies that already exist in the user-rated data base. These similarity scores can be used to derive rating estimates.

In summary, the goal is to create a recommender system that will predict users' ratings of new movies, given that an external source of information about the movie (e.g., from IMDB) is avialable.

### Metrics

Given a data set of known movie ratings, models investigating this problem can be evaluated by the accuracy of their predictions. For this project, these ratings will be treated as a continuous variable. Therefore, an appropriate metric for evaluating model performance will be the [root mean square error](https://en.wikipedia.org/wiki/Root-mean-square_deviation) (RMSE). This has been the metric used in similar problems such as the famous [Netflix Prize](https://en.wikipedia.org/wiki/Netflix_Prize)).

The RMSE is calculated by squaring the error terms (residuals) for predictions on a given set of data points, calculating the means of these, and taking the square root. For a vector of true values (*y*'s) and corresponding predicted values (*y*-hat), the formula for calculating RMSE is shown below:

![RMSE formula](http://statweb.stanford.edu/~susan/courses/s60/split/img29.png)

A value of zero indicates that all predictions perfectly match the true values. The more positive the value, the worse the performance.

## II. Analysis
_(approx. 2-4 pages)_

### Data Exploration

The data used in this project comes from two open-source projects:

- [MovieLens 20M Dataset](https://www.kaggle.com/grouplens/movielens-20m-dataset): Over 20 Million Movie Ratings and Tagging Activities Since 1995
- [IMDB 5000 Movie Dataset](https://www.kaggle.com/deepmatrix/imdb-5000-movie-dataset): 5000+ movie data scraped from IMDB website

The MovieLens dataset is to be used as the key source for the collaborative filtering component of the model. It contains individual user ratings of movies on a 5-star scale (with 5 being the best and 1 being the lowest). In total, there are 20,000,263 movie ratings given by  138,493 unique users.

The IMDB dataset is to be used as the key source for the content-based filtering component of the model. It contains public information about 5,000 movies and includes the following variables:

> "movie_title" "color" "num_critic_for_reviews" "movie_facebook_likes" "duration" "director_name" "director_facebook_likes" "actor_3_name" "actor_3_facebook_likes" "actor_2_name" "actor_2_facebook_likes" "actor_1_name" "actor_1_facebook_likes" "gross" "genres" "num_voted_users" "cast_total_facebook_likes" "facenumber_in_poster" "plot_keywords" "movie_imdb_link" "num_user_for_reviews" "language" "country" "content_rating" "budget" "title_year" "imdb_score" "aspect_ratio"

Combined, these two data sets can be used to train and test a hybrid recommender model for predicting the ratings that users will give "new" movies.

The major steps taken to initially clean the data and prepare it for analysis are described below:

- **Remove duplicated movie information**
    - The IMDB data set originally contained 5043 rows. However, these included duplicate cases. After removing duplicates, a total of 4919 movies were retained.
- **One-hot encode movie information**
    - Much of the movie information in the IMDB data set was stored as string variables. In such cases, information was converted into one-hot encoded variables. To demonstrate, the genre(s) of each movie, which is a vital piece of information, was recorded in the IMDB dataset as a string variable in a format such as "Action|Adventure|Fantasy|Sci-Fi". This variable was processed so that splits were conducted at the presence of each "|", and then the values were turned into one-hot encodings. That is, instead of a single "genres" variable, a column was created for each genre label appearing in the data set (e.g., "Action") which was given the value 1 for movies that had recieved this label, and 0 otherwise.
- **Imputing missing values**
    - There were a number of missing values in the IMDB dataset. Missing values were present for cateogrical (one-hot encoded) and continuous variables. Missing values were imputed as the mode for categorical variables and as the mean for continuous variables. One exception to this was "gross", which recorded the movie's gross profit. Missing data here was more likley to indicate that no gross profit had been recorded or made. Thus, missing values were set to zero on this variale.
- **Transform continuous variables**
    - Continuous variables relating to the movies (e.g., "gross", "actor_1_facebook_likes") were transformed in two ways. First, variables that did not have a normal/gaussian distribution were operated on to make their distribution more normal. In all cases, the best option was to calculate the log value of the variable. Finally, each continuous variable was normalized to have a value between 0 and 1. This was because these variables were going to be used for computing similarity among movies. If there were on very different scales, then this may bias the simialrity algorithm.
- **Retain data for movies only appearing in both data sets**
    - User ratings were given to many more movies that those included in the IMDB data set. For this project, however, only movies that existed in both data sets were of interest. Therefore, user ratings given to movies that did not exist in the IMDB data set were removed, leaving 13,426,294 ratings. Doing this cleaning involved finding a common identifier for movies across the datasets. This common identifier was the identifier assigned to each movie by IMDB. This information was stored in a hashtable in the MovieLens data set, which could be bound to user ratings. The same ID could be extracted from the movie URL from the IMDB data set in the "movie_imdb_link" column. For example, the URL for James Cameron's Avatar was "http://www.imdb.com/title/tt0499549/?ref_=fn_tt_tt_1". The movie ID is "0499549", which is prefixed by "tt". The regular expression (regex) '(tt[0-9]+)' was used to extract this value from each URL, and then the "tt" was removed. This process resulted in both datasets containing a common movie ID, which was used to drop user ratings for movies that did not exist in the IMDB data set.
- **Retaining data only for users with many ratings**
    - Mentioned above, a significant number of ratings were still kept. When transformed to a user-by-movie-dataset, this matrix was huge. This project had to be executed on a basic laptop that did not possess the hardware needed to operate on such a large data set. Therefore, to reduce the data set size ratings from a sample of users were retained. Given the sparsity of the data, ratings were retained for users who had given at least 1000 movie ratings. This left 487,691 ratings given by 385 unique users.
-  **Converting user ratings into a sparse matrix**
    - The final 487,691 ratings were converted into a sparse 385 (users) x 4919 (movies) matrix.

In this section, you will be expected to analyze the data you are using for the problem. This data can either be in the form of a dataset (or datasets), input data (or input files), or even an environment. The type of data should be thoroughly described and, if possible, have basic statistics and information presented (such as discussion of input features or defining characteristics about the input or environment). Any abnormalities or interesting qualities about the data that may need to be addressed have been identified (such as features that need to be transformed or the possibility of outliers). Questions to ask yourself when writing this section:
- _If a dataset is present for this problem, have you thoroughly discussed certain features about the dataset? Has a data sample been provided to the reader?_
- _If a dataset is present for this problem, are statistics about the dataset calculated and reported? Have any relevant results from this calculation been discussed?_
- _If a dataset is **not** present for this problem, has discussion been made about the input space or input data for your problem?_
- _Are there any abnormalities or characteristics about the input space or dataset that need to be addressed? (categorical variables, missing values, outliers, etc.)_

### Exploratory Visualization
In this section, you will need to provide some form of visualization that summarizes or extracts a relevant characteristic or feature about the data. The visualization should adequately support the data being used. Discuss why this visualization was chosen and how it is relevant. Questions to ask yourself when writing this section:
- _Have you visualized a relevant characteristic or feature about the dataset or input data?_
- _Is the visualization thoroughly analyzed and discussed?_
- _If a plot is provided, are the axes, title, and datum clearly defined?_

### Algorithms and Techniques

This project will compare two methods for estimating users ratings for new movies: content-based filtering and a hybrid approach of content-based and collaborative filtering.

1. Content-based filtering involves a **k-nearest neighbours** algorithm. For example, take a new movie `m` and all of a user's existing ratings `U = {rating-movie.i, rating-movie.j, ...}`.  An estimate of the user's rating for `m` can be obtained by using ratings from the `k`-most similar movies to `m` rated by the user. There are a few major considerations for this algorithm:
    - how similarity is computed
    - the value of `k`
    - how to combine ratings to produce an estimate
2. An issue with the content-based filtering only is the sparsity of the data. For example, what if the user has only rated movies that are very dissimilar to `m`? To solve this, collaborative filtering can be used to estiamte missing data within rated movies. These estimates, in turn, can be used by the content-based filtering appraoch described above. Combined, this approach is a hybrid recommender system. One of the most successful collaborative filtering algorithms for estimating sparse rating data in **non-negative matrix factorisation**. This approach attempts to factorise the user-by-movie rating matrix into two smaller matrices which can be multipled together to reproduce the original matrix. In this way, the smaller matrices can be estimated by ignoring missing values and then, when multipled back together, produce estiamtes in missing cells. The major consideration for this algorithm:
    - How many latent factors to factorise, `p`. That is, if the main matrix is of dimensions  `m x n`, then the two smaller matrices will be of dimensions `m x p` and `p x n`.


In this section, you will need to discuss the algorithms and techniques you intend to use for solving the problem. You should justify the use of each one based on the characteristics of the problem and the problem domain. Questions to ask yourself when writing this section:
- _Are the algorithms you will use, including any default variables/parameters in the project clearly defined?_
- _Are the techniques to be used thoroughly discussed and justified?_
- _Is it made clear how the input data or datasets will be handled by the algorithms and techniques chosen?_

### Benchmark

As a benchmark, the estimated rating of a new movie by a given user will be calculated as the mean of all of the user's existing ratings. This estimate represents the typical rating assigned by a user to movies, and therefore serves as an appropriate benchmark. The Figure below demonstrates this benchmark approach for a new movie and existing ratings:

![benchmark](https://github.com/drsimonj/machine-learning/blob/master/projects/capstone/imgs/benchmark.png?raw=true)

The performance of this benchmark approach was established using the user-by-movie-ratings matrix via a leave-one-out cross validation. That is, for each movie (column) in the matrix, user-wise (row-wise) means were computed for all other available ratings as estimates of users' ratings for that movie. Once the entire matrix was reconstructed via this approach, RMSE (see [Metrics](#metrics])) was calcualted for all known ratings.

**The RMSE for the benchmark model was 0.9396**

In this section, you will need to provide a clearly defined benchmark result or threshold for comparing across performances obtained by your solution. The reasoning behind the benchmark (in the case where it is not an established result) should be discussed. Questions to ask yourself when writing this section:
- _Has some result or value been provided that acts as a benchmark for measuring performance?_
- _Is it clear how this result or value was obtained (whether by data or by hypothesis)?_


## III. Methodology
_(approx. 3-5 pages)_

### Data Preprocessing
In this section, all of your preprocessing steps will need to be clearly documented, if any were necessary. From the previous section, any of the abnormalities or characteristics that you identified about the dataset will be addressed and corrected here. Questions to ask yourself when writing this section:
- _If the algorithms chosen require preprocessing steps like feature selection or feature transformations, have they been properly documented?_
- _Based on the **Data Exploration** section, if there were abnormalities or characteristics that needed to be addressed, have they been properly corrected?_
- _If no preprocessing is needed, has it been made clear why?_

### Implementation

Implementation was split into three main stages:

1. Developing a content-based filtering recommender for new movies using k-nearest-neighbours
2. Developing a collaborative-based filtering recommnder for existing movies using non-negative matrix factorisation
3. Developing a hybrid recommender system for new movies

Details of how each step was executed is described below.

#### Step 1: content-based filtering

This step involved developing a content-based filtering recommender that acts similarly to the benchmark model (by computing the mean of ratings), but in a more informed way. Specifically, rather than computing the mean of all a user's ratings, this approach first calcualtes the similarity of the new movie to each rated movie, and computes the mean based on ratings from the k-most similar rated movies. This step involved

- Finding a method that best captured "similarity" among movies
- Given similarity scores, finding a value of `k` that provided the best predictions

#### Step 2: collaborative filtering

This step involves developing a collaborative-filtering recommender for movies already in the data base (which could be used by a hybrid recommender in step 3). This involved applying non-negative matrix factorisation (using stochastic gradient descent to handle missing values) to the full user-by-movie rating matrix. Tuning was required to determine the optimal number of latent dimensions that best estaimted ratings.

#### Step 3: hybrid

This step involved combining steps 1 and 2. The collaborative filtering method developed in Step 2 was used to fill in the sparse ratings data. Then, the content-based recommender developed in Step 1 was used to estimate ratings for new movies.

In this section, the process for which metrics, algorithms, and techniques that you implemented for the given data will need to be clearly documented. It should be abundantly clear how the implementation was carried out, and discussion should be made regarding any complications that occurred during this process. Questions to ask yourself when writing this section:
- _Is it made clear how the algorithms and techniques were implemented with the given datasets or input data?_
- _Were there any complications with the original metrics or techniques that required changing prior to acquiring a solution?_
- _Was there any part of the coding process (e.g., writing complicated functions) that should be documented?_

### Refinement

Similarity between movies was calcualted as cosine similarity << Should define...>>. A challenge was to determine which variables from the IMDB dataset would work best. Various combinations of variables were tested. To validate this method, a handful of movies known well to the author (especially ones that were part of a series) were selected for analysis. When a set of variables was tried, the 5-10 most similar movies to these validation movies was examined. Based on the authors knowledge of which movies were similar (and those known to be in the same series), a judgement call was made about which variable combinations worked best.

To demonstrate, below are the most similar movies to "Pirates of the Caribbean: At World's End" based on different variable sets:

- *Using all variables*: Carlos; Spy Game; Superman; Street Fighter
- *Using Genres only*: The Legend of Hercules; Indiana Jones and the Last Crusade; Warcraft; Gods of Egypt
- *Using Genre, rating, gross, cast likes*: Pirates of the Caribbean: The Curse of the Black Pearl; Pirates of the Caribbean: Dead Man's Chest; Pirates of the Caribbean: On Stranger Tides

It was clear that using all variables did not produce good results. Selecting too few (e.g., just the one-hot-encoded genre variables) was not accurate either. Rather, a combination of genre, rating, and the continuous variables produced reasonable results.

To further improve this, the twelve continuous variables were submitted to Principal Components Analysis in attempt to reduce the dimensional space being used by the cosine algorithm. It was decided a prior that as many components would be selected as accounted for at least 90% of the variance in these twelve variables. The results was that six components accounted for precisely 90% of the variance.

At this point, the movie similarity approach was tried again using one-hot-encoded variables relating to genre and rating, and the six principle components derived from the continuous variables. Below are the results of some tests:

```
MOVIES THAT ARE MOST SIMILAR TO: Harry Potter and the Half-Blood Prince  

200               Harry Potter and the Sorcerer's Stone 
64     The Chronicles of Narnia: The Lion, the Witch ...
144                                                 Pan 
38                            Oz the Great and Powerful 
374                      Percy Jackson: Sea of Monsters 
193            Harry Potter and the Prisoner of Azkaban 
282             Harry Potter and the Chamber of Secrets 
9                Harry Potter and the Half-Blood Prince 
33                                  Alice in Wonderland 
106                     Alice Through the Looking Glass 
Name: movie_title, dtype: object
******************************************************** 


MOVIES THAT ARE MOST SIMILAR TO: The Avengers  

52                             Pacific Rim 
112                           Transformers 
53          Transformers: Dark of the Moon 
86     Captain America: The Winter Soldier 
65                       X-Men: Apocalypse 
8                  Avengers: Age of Ultron 
10      Batman v Superman: Dawn of Justice 
11                        Superman Returns 
27              Captain America: Civil War 
17                            The Avengers 
Name: movie_title, dtype: object
******************************************************** 


MOVIES THAT ARE MOST SIMILAR TO: The Hunger Games: Catching Fire  

216                   The Day After Tomorrow 
236                            The Wolverine 
97                                 Inception 
29                            Jurassic World 
689                            Jurassic Park 
187                        War of the Worlds 
433                         The Hunger Games 
253                                Insurgent 
204    The Hunger Games: Mockingjay - Part 1 
185          The Hunger Games: Catching Fire 
Name: movie_title, dtype: object
******************************************************** 


MOVIES THAT ARE MOST SIMILAR TO: Pirates of the Caribbean: The Curse of the Black Pearl  

133                                  Wrath of the Titans 
1030                  Indiana Jones and the Last Crusade 
54      Indiana Jones and the Kingdom of the Crystal S...
210                                  Clash of the Titans 
13            Pirates of the Caribbean: Dead Man's Chest 
18           Pirates of the Caribbean: On Stranger Tides 
127                                 Thor: The Dark World 
1               Pirates of the Caribbean: At World's End 
202     Pirates of the Caribbean: The Curse of the Bla...
21                                The Amazing Spider-Man 
Name: movie_title, dtype: object
******************************************************** 


MOVIES THAT ARE MOST SIMILAR TO: Star Wars: Episode VI - Return of the Jedi  

2394                               Conan the Destroyer 
3418                                       Logan's Run 
2876                   Star Trek II: The Wrath of Khan 
40                                        TRON: Legacy 
1068                Journey to the Center of the Earth 
2031    Star Wars: Episode V - The Empire Strikes Back 
237          Star Wars: Episode I - The Phantom Menace 
234       Star Wars: Episode II - Attack of the Clones 
1521        Star Wars: Episode VI - Return of the Jedi 
2974                Star Wars: Episode IV - A New Hope 
Name: movie_title, dtype: object
******************************************************** 
```

These results appear to be excellent. For example, in the final test shown, the most similar movies to the fourth Star Wars movie are the five other movies in the same series.

XXXXXXXXXXXXXXXXXX

Using the simliarty metric defined in the stages above, the k-nearest neighbours approach described above was used to estimate movie ratings. As with the benchmark model, leave-one-out cross validation was conducted to derive performance metrics. The model was fit using five values of `k`: 10, 20, 30, 40, and 50. The results were as follows:

Using 10-most similar movies...	RMSE = 0.8787
Using 20-most similar movies...	RMSE = 0.8717
Using 30-most similar movies...	RMSE = 0.8712
Using 40-most similar movies...	RMSE = 0.8717
Using 50-most similar movies...	RMSE = 0.8732

In all cases, the RMSE was better than the benchmark model, indicating that this was a useful approach to take. Comparing the results for different values of `k`, 30 was determined to be the most suitable.



In this section, you will need to discuss the process of improvement you made upon the algorithms and techniques you used in your implementation. For example, adjusting parameters for certain models to acquire improved solutions would fall under the refinement category. Your initial and final solutions should be reported, as well as any significant intermediate results as necessary. Questions to ask yourself when writing this section:
- _Has an initial solution been found and clearly reported?_
- _Is the process of improvement clearly documented, such as what techniques were used?_
- _Are intermediate and final solutions clearly reported as the process is improved?_


## IV. Results
_(approx. 2-3 pages)_

### Model Evaluation and Validation
In this section, the final model and any supporting qualities should be evaluated in detail. It should be clear how the final model was derived and why this model was chosen. In addition, some type of analysis should be used to validate the robustness of this model and its solution, such as manipulating the input data or environment to see how the model’s solution is affected (this is called sensitivity analysis). Questions to ask yourself when writing this section:
- _Is the final model reasonable and aligning with solution expectations? Are the final parameters of the model appropriate?_
- _Has the final model been tested with various inputs to evaluate whether the model generalizes well to unseen data?_
- _Is the model robust enough for the problem? Do small perturbations (changes) in training data or the input space greatly affect the results?_
- _Can results found from the model be trusted?_

### Justification
In this section, your model’s final solution and its results should be compared to the benchmark you established earlier in the project using some type of statistical analysis. You should also justify whether these results and the solution are significant enough to have solved the problem posed in the project. Questions to ask yourself when writing this section:
- _Are the final results found stronger than the benchmark result reported earlier?_
- _Have you thoroughly analyzed and discussed the final solution?_
- _Is the final solution significant enough to have solved the problem?_


## V. Conclusion
_(approx. 1-2 pages)_

### Free-Form Visualization
In this section, you will need to provide some form of visualization that emphasizes an important quality about the project. It is much more free-form, but should reasonably support a significant result or characteristic about the problem that you want to discuss. Questions to ask yourself when writing this section:
- _Have you visualized a relevant or important quality about the problem, dataset, input data, or results?_
- _Is the visualization thoroughly analyzed and discussed?_
- _If a plot is provided, are the axes, title, and datum clearly defined?_

### Reflection
In this section, you will summarize the entire end-to-end problem solution and discuss one or two particular aspects of the project you found interesting or difficult. You are expected to reflect on the project as a whole to show that you have a firm understanding of the entire process employed in your work. Questions to ask yourself when writing this section:
- _Have you thoroughly summarized the entire process you used for this project?_
- _Were there any interesting aspects of the project?_
- _Were there any difficult aspects of the project?_
- _Does the final model and solution fit your expectations for the problem, and should it be used in a general setting to solve these types of problems?_

### Improvement
In this section, you will need to provide discussion as to how one aspect of the implementation you designed could be improved. As an example, consider ways your implementation can be made more general, and what would need to be modified. You do not need to make this improvement, but the potential solutions resulting from these changes are considered and compared/contrasted to your current solution. Questions to ask yourself when writing this section:
- _Are there further improvements that could be made on the algorithms or techniques you used in this project?_
- _Were there algorithms or techniques you researched that you did not know how to implement, but would consider using if you knew how?_
- _If you used your final solution as the new benchmark, do you think an even better solution exists?_

-----------

**Before submitting, ask yourself. . .**

- Does the project report you’ve written follow a well-organized structure similar to that of the project template?
- Is each section (particularly **Analysis** and **Methodology**) written in a clear, concise and specific fashion? Are there any ambiguous terms or phrases that need clarification?
- Would the intended audience of your project be able to understand your analysis, methods, and results?
- Have you properly proof-read your project report to assure there are minimal grammatical and spelling mistakes?
- Are all the resources used for this project correctly cited and referenced?
- Is the code that implements your solution easily readable and properly commented?
- Does the code execute without error and produce results similar to those reported?

[^c1]: Francesco Ricci and Lior Rokach and Bracha Shapira, Introduction to Recommender Systems Handbook, Recommender Systems Handbook, Springer, 2011, pp. 1-35
[^c2]: Prem Melville and Vikas Sindhwani, Recommender Systems, Encyclopedia of Machine Learning, 2010.
[^c3]: R. J. Mooney & L. Roy (1999). Content-based book recommendation using learning for text categorization. In Workshop Recom. Sys.: Algo. and Evaluation.