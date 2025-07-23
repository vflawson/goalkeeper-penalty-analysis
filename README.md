# What Makes a Great Penalty Stopper? An Analysis of the Data

Previously, I explored some factors that are correlated with penalty kick success. A player’s age was found to be statistically significant, as well as a fairly good predictor of penalty success, especially in high pressure matches such as cup games or international events. 

But what determines whether a goalkeeper makes a save? This event happens much less frequently and may prove more difficult to model accurately. Still, if we can create a model for predicting or increasing the likelihood of saving a penalty, these insights could be very valuable indeed.

## A Quick Look at Our Data

For this analysis, I will again be leveraging the rich dataset of 67,286 penalty kicks made available by the research team of Vollmer, Schoch, and Brandes (2024), shared alongside their article, “Penalty shoot-outs are tough, but the alternating order is fair”. 

Just as I did in the last post, I will be joining this dataset with information on player attributes from the European Soccer Database on Kaggle. This time, I will be focusing in on the attributes of the goalkeepers involved in the penalty kicks, instead of the kick takers.  When combining these sources together, I was left with over 13,000 penalty kick events across Men’s European Football from 2012-2021, with attribute data focused on the goalkeeper in each instance. 

## The “Imposing” Keeper: Fact or Fiction?

The first hypothesis I wanted to test on penalty kicks is around the importance of the goalkeeper’s height. It is a common belief among football fans that height is a clear advantage in penalty shootouts. Gianluigi Donnarumma, who stands at 196 cm (6’ 5’’), is particularly successful in saving penalties, and much of this is attributed to his height.

Is a goalkeeper’s height actually a good predictor of penalty saving ability? Let’s take a look at the data.

In order to test for the statistical significance of our height variable, I decided to remove any missed penalty kicks where the goalkeeper did not make a save. That way, I could isolate only directly keeper-influenced outcomes (either the kick taker scores his penalty or the goalkeeper saves it). In doing so, I’m making the assumption that a goalkeeper’s height did not indirectly influence the kick taker when he fluffed his shot wide or kicked it over. 

While in the previous post I defined a successful penalty kick as a “1” in the binary mapping of the data, this time I am setting a penalty save to “1”. From the goalkeeper’s perspective, a saved penalty is a success. 

Using a simple logistic regression model, we can measure the predictive power of goalkeeper height on a penalty save:

<img width="3059" height="1111" alt="Logistic Regression" src="https://github.com/user-attachments/assets/cbf1611b-d5b1-4793-9ffc-457a14cdf2a2" />

Assuming an alpha of 0.05, our model finds that goalkeeper height is not, in fact, statistically significant in terms of predicting whether a keeper will save a penalty or not. 

![ab97cbdb-1ab1-4830-ae42-9b39e5a56436_857x576](https://github.com/user-attachments/assets/adfe9918-f427-4dd8-961c-a8d7446ea99e)

Visualizing the coefficient of the goalkeeper height variable, there is an indication that certain height levels might increase the odds of a keeper saving a penalty. However, the high p-value of the variable means that the variable is not statistically significant. The bump in the graph for the 195-200 cm height range might suggest a non-linear relationship between the two variables. 

To test for a possibility of a non-linear relationship between goalkeeper height and penalty save success, I created a second model, including a 2nd order polynomial variable as well:

<img width="3059" height="1245" alt="Non-Linear Logistic Regression" src="https://github.com/user-attachments/assets/d5c2925a-c090-40b3-a614-a3792c627cde" />

However, the height variable when expressed in a 2nd order polynomial form still did not show statistical significance. Therefore, based on the data available, the common belief that the “imposing” goalkeeper, whose height helps him save penalty kicks more frequently, is not supported by this dataset.

## Fifa Goalkeeper Ratings: Shorthand for Real Skills?

So if a goalkeeper’s height is not a good determinant for penalty save success, what skills do make a difference? Intuitively, a keeper’s reaction time could be important. Perhaps a player’s diving ability as well? 

Fortunately, the European Soccer Database I used to gather player attributes includes FIFA (now EA FC) player ratings, with values for specific goalkeeper skills. There is an overall player rating, and individual keeper skills for diving, handling, kicking, positioning, and reflexes. Short of using video analysis to try to measure something as abstract as positioning, FIFA ratings are a qualitative approach to rating a players’ inate abilities. Which of these are good measures for predicting whether a goalkeeper will save a penalty? Let’s take a look at our regression model:

<img width="3059" height="1780" alt="Fifa ratings logistic model" src="https://github.com/user-attachments/assets/350c7c52-0b18-4c62-a748-f74cfcb308c1" />

Interestingly, only the overall FIFA rating was found to be statistically significant in predicting whether a goalkeeper would make a penalty save or not. The relatively low p-value scores for positioning and reflexes could indicate potential, but the coefficient for positioning (the lower the FIFA rating, the better the odds of saving a penalty) is definitely off. There is likely multicolinearity going on between the variables as well. Overall, judging from this model, the rating development team at FIFA know what a good goalkeeper looks like but not necessarily what makes them so good.

Finally, let’s see if there are any non-linear patterns in our FIFA goalkeeper ratings that a machine learning model can pick out. For this test, I used a simple random forest model (100 estimators without any parameter tuning) to see how well the model could predict if a penalty kick would be saved or not based on the six FIFA rating variables in our dataset. 

<img width="2945" height="694" alt="random forest results" src="https://github.com/user-attachments/assets/030454d2-522d-4066-bb6a-b81ff4501dfd" />

The results of the model indicate a fairly poor predictive performance. Essentially, the model predicted that every penalty kick would be successfully scored (class 0), which resulted in a high F-1 score for that class. However, the model greatly struggled to actually predict our target class, a penalty being saved. This issue is common when training models on imbalanced datasets in binary classification problems. The model will often default to the majority class. 

We can improve the model performance by applying a statistical method known as SMOTE (synthetic minority oversampling technique) to the training data, which creates synthetic interpolated samples of the target class for the random forest model. That way, the model will have more examples of when a penalty is saved and can learn the patterns that predict this class. Additionally, I used class weighting in the model to compensate for the class imbalance, giving more importance to the minority class (a penalty being saved) during training.

<img width="2945" height="694" alt="SMOTE random forest model" src="https://github.com/user-attachments/assets/d1603de1-8356-4428-a0e1-1c8a17be8def" />

While we lost some overall accuracy in our model (a lower F-1 score in the 0 class), the model performed much better at predicting whether a penalty would be saved or not. 

Still, this simple random forest model won’t be winning any major bets off of its predictive power. If I were to continue developing this model, I would look at incorporating additional variables on the penalty kick taker side, as well as examining data on shot placement and power. 

My main takeaway, however, is that good goalkeepers can be measured in terms of their statistical success in penalties, but what makes them special is much harder to quantify and replicate.
