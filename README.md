# MentalHealthRedditAnalysis
Analysis of data from reddit for my Flatiron Capstone Project
Problem Statement:  A nonprofit agency is interested in new ways to assess mental health using social media.  They want a way to assess someone's mental health using their social media posts.

Can we predict if someone has a certain mental illness based on their social media posts about things unrelated to their mental illness?

## Data Overview

Data will be gathered from Reddit.   Users who post in the following subreddits: PTSD, ADHD, Anxiety, BPD, BipolarReddit, and Depression, as well as a control group chosen randomly from other subreddits that those in the experimental group also post to.
Users will be labeled based on flair in subreddits that require users to flag themselves as friend/family member or undiagnosed (BipolarReddit and bpd). Those without that flair will be labeled as having the disorder.   In the other 4 subreddits, self-reported diagnoses will be used to identify users with the disorder (such as "I was diagnosed with depression", "my depression", etc). Anxiety will not be based on an official diagnosis, but instead on those users who speak about struggling with anxiety (as the problem statement specifies mental health in general, not necessarily diagnosable mental health).  Data will then be gathered from these users' posts in unrelated subreddits.

## Models
### Word2Vec with Random Forest
I train a word2vec model using gensim and obtain word embeddings from it. I then use those embeddngs with a Random Forest Classifier. This model performs very poorly, even with paramater hypertuning. In addition to data limitations, we do not know if there is enough of a language difference to be able to classify someone as struggling with a certain mental health diagnosis based solely on how they communicate online about non-diagnose subjects.  The final version of the Random Forest Classifier does not ignore the smaller classes. Interestingly, anxiety has a precision of 13%, even though recall is only 1%.  Depression only has a precision of 1% but has a recall score of 93.7%. PTSD has a precision of 11% and a recall of 2%.

### LSTM 
I create encodings for the 50,000 most common words in the comment data, and train them using Keras LSTM. I do this because the vocabulary is very large, with numerous strange "words" that are only used once.  I then try to classify indviduals as neutral (not-diagnosed), or in one of the 6 diagnoses.  The final version of this model tries to identify each class, but struggles to do so well. The neutral class has a recall of 24.87%, and a precision of 83%.

ADHD - 23.8% recall, 4% Precision
Anxiety - Recall 9.89%, Precision 5%
Bipolar - Recall 7% Precision 2.4%
BPD - Recall 6.8%, Precision 2.3%
Depression - Recall 18.88%, Precision 1.8%
PTSD - Recall 19.4%, Precision 6.2%

### Strategy Change and Success! 
I decide that comment data alone is not enough, so I use both the submission and comment data. I also use SBERT to increase the accuracy of my word embeddings. In addition, I undersample from the "control" group. Finally, I break the process into 2 different classifications - first a binary classification of whether someone is struggling with their mental health or not, and then for those that are, a second model to classify which mental illness they may be struggling with.  

#### Binary Classifier
The overall macro f1 score is .69, precision is .68, and recall is .7. For the mental health class, the f1-score is .59, precision is .55, and recall is .64. 

#### Which Mental Health Illness?
I now train a model that takes only individuals who are strugglng with their mental health, and predicts which mental illness they are struggling with. 

The first version of this model has a f1 score of .297.  It does best with anxiety (recall of .65 and precision of .36) and PTSD (recall of .53 and precision of .36). Also interestingly, PTSD is misclassified as anxiety 473 times, and anxiety is misclassified as PTSD 358 times. These disorders are closely linked, so it makes sense the model is picking up on this.

![alt text](https://github.com/kstrickland680/MentalHealthRedditAnalysis/blob/main/first_confusionmatrix.png)

I then retrain the model using class weights. This model has a weighted f1-score of .27, precision of .32, and recall of .27.  ADHD's recall and precision are similiar to each other at .25 and .27.  Anxiety's precision is .47 and recall is .17.  Bipolar's recall is .43 but precision is .2.  BPD's recall is .29 and precision is .19.  Depression has a recall of .18 and precision of .17.  PTSD's recall is .34 and precision is .35. Interestingly, in this model PTSD is linked more closely to bipolar than anxiety. 

![alt text](https://github.com/kstrickland680/MentalHealthRedditAnalysis/blob/main/final_confusionmatrix.png)

## Conclusions: 
Posts in non-mental health subreddits can also show a lot about whether someone is struggling with their mental health, whether from patterns in speech or from the content of the posts. I was able to train a model on non-mental health related social media posts that could classify users as struggling with their mental health or not, but was unable to train a model solely on non-mental health posts that could classfy which mental health illness someone is likely to have with a high degree of precision or recall. The models struggled to distinguish between the disorders. 

### Business Recommendations
*   Create an online support group: Many people form strong relationships and groups online.  Some people feel more comfortable opening up when not having to speak out loud or show their face.  Creating a support group that's entirely online could be very successful and helpful for individuals. 
*   Respond to users: Talk about the benefits of counseling, programs the agency offers, post information.  Always obey subreddit/community rules! 
*   Subreddits of note: AskReddit, Relationships, RelationshipAdvice, AmITheAsshole - had high post rates for users struggling wth their mental health.  Monitor those subreddits as well as the subreddits focused on mental health. 

### Future Work
*   Continue to refine the models to analyze language
*   Creating an informational and encouraging (not too positive) bot 
*   Examine comorbidity and how it could affect language
