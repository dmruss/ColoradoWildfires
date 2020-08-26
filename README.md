# ColoradoWildfires

This project is a data mining analysis of Colorado wildfires.  The dataset is supplied by the Forest Service Data Archive, and was developed and maintained by Karen Short at the US Forestry Science Lab.  The data is geospatial data housed in sqlite database and is available for download on the Forest Service website [0].  The data spans 24 years, from 1992 to 2015, and contains wildfire data from the entire US.  This is a multidimensional database with 1.88 million wildfire entries.  The coordinate point is within 1-square-mile of accuracy.  There are 39 attributes for each fire which span many different types and formats.  To maintain scope and relevance, this report will be focusing on wildfires located in Colorado.
The main questions of this report are:1)  Have wildfire characteristics changed over time?  This report looks to find if there is any meaningful trend in select characteristics over this period of time.  2) Which regions in Colorado are “hotspots” for wildfires?  This report looks to define regions of the state that are historic wildfire hotspots.  These regions may cross county boundaries and may vary greatly in size.  The similarity of the fires in these clusters relative to their dissimilarity to fires outside the cluster will determine the success of their clustering.  3) Can a classification model be trained to predict the cause of a wildfire more accurately than a standard baseline?  If the model can predict a wildfire cause more accurately and precisely than a guess of the most common cause then it will be deemed successful.
The stakeholders in this project include Colorado residents, private landowners, the U.S. Forest Service, the Bureau of Land Management, and environmental recreators and educators.  Federal wildfire suppression alone accounted for over $3 billion worth of expenses in 2018, doubling from close to $1.5 billion in 2008 [1].  With more Coloradans living in the “wildland-urban interface”, development close to or in forested areas[3], than ever before it is of key importance to understand at-risk regions and fire trends in the state.  Also, with a high number of fires originating from man-made causes, potentially 90%,  accurate public understanding and education about the causes of wildfires is more important than ever [2].  
	Due to the large volume of high quality validated entries in this dataset, there have been many different analyses done with it.  One article on Medium[4] uses a Random Forest classifier to try and predict fire class size using the included attributes as well as air temperature data.  The main issue the writer runs into is that a small number of outlier fires are responsible for the largest burn areas.  They adjust for this by tuning their algorithm to increase recall for largest, but most destructive target classes.

#Methods

	To approach the main questions stated above, this analysis uses a combination of regression, clustering, and classification methods.  To look at the change of wildfire behavior over time, key features were selected and represented by either frequency or a mean value per year.  This worked to focus key attributes related to fire activity into single indexes (mean, max, etc.) that could represent each year.  For example, the mean fire size was found for each year in the dataset.  This mean could then be plotted as a bar chart and the tuple containing (year, mean_size) could be used to perform a linear regression.  The line of best fit was then graphed along with the bar chart.  The R-squared value, p-value and standard deviation give an indication as to how meaningful the regression is.  The slope would give an indication as to how much, if at all, the values had changed over time.
	During the preprocessing phase of this project, log transform was used to handle skewed fire size data.  A defining feature of this data set is that there are far more small fires than there are large, destructive fires.  This leads to a heavily left skewed fire size attribute field.  Log transform reduced the skewness coefficient from 63.7 to 2.4 and the standard deviation from ~1266 to ~1.9.  This was done to improve the usefulness of fire size during classification.
	A large part of this report consists of evaluating different clustering methods for the geospatial data.  This serves several purposes within the scope of this study: 1) It helps to answer the second main question, which regions of Colorado are hotspots for wildfire? 2) It serves as feature engineering for the classification methods used later in the report.  Unsupervised learning works as a way to take noisy, but extremely necessary, coordinate points and distill them down to discrete regions.  Three different methods (K-means, DBSCAN, and Agglomerative Hierarchical)  were used with various parameter combinations and compared to a baseline/naive method (clustering by county).  Each clustering iteration’s success was evaluated using a silhouette score.  The higher the score, between -1 and 1, the greater the similarity that a wildfire has to the fires in its cluster and the lower the similarity that it has to other clusters. 
	K-means clustering was an early choice in this report due to its low parameter count (number of clusters) and quick clustering [10, p. 451].  The number of clusters evaluated in this report were 12, 24, 48, and 64.  This choice began with 64 to mimic the baseline method and decreased down to 12 to find if a larger cluster increased silhouette score.
	DBSCAN clustering was the next selection due to its ability to handle non-circular clusters and its outlier detection [10, p. 471].  The DBSCAN parameters are minimum points in a cluster (min_pts) and epsilon (eps), the max distance to evaluate a point’s neighborhood.  The parameters used for this report are: min_pts: 50, 100, and 150, eps: 0.06, 0.08, 0.1.   Testing began at 50 for min_pts, which was chosen initially as an arbitrary number that a cluster would need to classify as a high frequency fire area.  100 and 150 were added to maintain increments of 50.  Testing began with an epsilon of 0.06, as this translates to approximately 7 km (4 mi) at the coordinates 39 N 107 W which are squarely in the Colorado rocky mountain region, near Crested Butte [5].  This was chosen as an arbitrary minimum cutoff for separate clusters and then increased by increments of .02 degrees.  Each combination of min_pts and epsilon values were tested to find which best eliminated outliers and produced the highest silhouette score.
	To round out the partitioning and density based clustering methods, Agglomerative Hierarchical clustering was added to the method list.  This clustering method seemed well suited to grouping geospatial data as it begins with each point as its own cluster and progressively combines clusters located within its threshold as determined by euclidean distance [10, p. 459].  The distance threshold parameters used were 0.2, 0.4, and 0.6.  Initially, the distance threshold parameters were the same values as the epsilon values used by DBSCAN.  These proved to be too low and too many clusters (>1000) were created.  0.2 was then the lowest value which produced a reasonable number of clusters and silhouette score.  0.4 and 0.6 were then chosen to maintain increments of 0.2.
	Five different methods were used for the classification section of this analysis, Linear Discriminant Analysis, K Nearest Neighbor, Decision Tree, Gaussian Naive Bayes, and a Neural Network.  Each of these methods were trained and tested using a 10-fold cross validation which was compared with a baseline/naive method which only selected the most common outcome.  80% of the data was used for training the models while 20% was withheld for testing the models. These classifiers were deliberately run using default parameter settings, then whichever classifier proved to be the most accurate would move on to have its parameters tuned. 
	Linear Discriminant analysis (LDA) was selected because it is known to work well when predicting categorical variables, such as fire cause codes in this case.  It assumes that the independent variables will be normally distributed and then finds a linear combination to predict the target category [10, p. 600].  The parameters used are solver: single value decomposition, priors = none, number of components = none, threshold for rank estimation = 0.0001.  These are all default values.
	K nearest neighbor (KNN) were used to classify the fires based on the euclidean distance of the training data’s inputs.  This method was selected due to the dataset’s multivariable independent and dependent variables [10, p. 423].  The parameters used were number of neighbors = 5, weight of inputs are uniform, auto select algorithm, leaf size of 30, p value of 2 (euclidean distance), and the distance metric for constructing the tree is the minkowski metric.
	A decision tree classifier was the next method used to predict fire cause.  It was chosen because it is a proven method for splitting a dataset into class labels by constructing a tree data structure.  By not needing to set any parameters, the method may give accurate results with minimal domain knowledge [10, p. 330].  Default parameters were used which included an unlimited maximum tree depth, a minimum of 1 sample per leaf, and consideration of up to n features when considering a split.
	The next method used is a Gaussian Naive-Bayes classifier.  This method uses Bayesian probability to predict which features will lead to which class label [10, p. 351].  This method was chosen to add diversity to the classifier choices by including probability.  The default parameters used are to include no priors and a variance smoothing variable of 1e-9.
The final classifier used was a neural network.  This was chosen due to its ability to handle noisy data and continuous input variables.  The downside of using a neural network in this analysis is that it may only become a valid option after parameter tuning due to its high complexity and large variability [10, p. 398].  All default variables were used including hidden layer size of 100, rectified linear unit activation function, stochastic gradient optimizer, L2 penalty of .0001, batch size of the minimum value between 200 and the number of samples, constant learning rate, initial learning rate of .001, inverse scaling learning rate exponent of .5, maximum iterations of 200, shuffle samples each iteration, tolerance for score improvement of 1e-4, exponential decay rate for first moment vector of 0.9, exponential decay rate for second moment vector of 0.999, value for numerical stability of 1e-8, and maximum number of epochs to not meet the tolerance requirement: 10 iterations.
Initially, logistic regression was included as part of the classifier methods.  This was later dropped when it proved not possible to complete classification with multiple class labels using this method.  A support vector machine was also used in initial classifier testing.  After several attempts to train the model, it was dropped due to its overly long training time (no convergence after 20 minutes).  Also, while included in the silhouette score plot, the two DBSCAN iterations with the parameters min_pts = 100, eps = 0.06 and min_pts = 150, eps = 0.08 were both dropped from the project during the classification phase.  This was due to fewer than 10 members, as needed for 10-fold cross validation, per class label in the target training set.  Also, a method to remove entries with NaN values for DBSCAN clusters (outliers) was moved to the bottom of the project and commented out.  This entry removal was done in QGIS’s attribute table instead.

#Results

	The goal of this project is to mine information about wildfires in Colorado, with the purpose of uncovering spatial and temporal knowledge.  With this knowledge, the goal is to then extend this to predict fire cause.  The results of this project were evaluated primarily using accuracy measures.  The two main accuracy measures are the silhouette score, used for evaluating the spatial clustering accuracy, and mean accuracy and f1 scores to evaluate fire cause classification.  When finding trends over time, p-value, R-squared, and standard error were used to evaluate the regression.
