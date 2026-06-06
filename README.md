# What Makes a Spotify Song Popular?

By Venkata Mangalampeta

## Introduction

This project analyzes the Spotify Music Tracks dataset to explore how audio features such as danceability, energy, and valence relate to song popularity. My main question is: are more danceable songs generally more popular?

This question is interesting because Spotify popularity is something artists, listeners, and platforms care about, but it is not obvious whether measurable audio features actually relate to popularity. I focus on five musically distinct genres: pop, rock, hip-hop, country, and classical.

The main columns I use are `popularity`, `danceability`, `energy`, `valence`, and `tempo`. `popularity` is the response variable, while the audio features help describe the musical qualities of each song.

## Data Cleaning and Exploratory Data Analysis

I cleaned the data by dropping duplicate rows and keeping the main columns needed for my analysis. I focused on five distinct genres: pop, rock, hip-hop, country, and classical. I chose these genres because they are musically different from each other, which makes comparisons more meaningful. I also created a danceability group column to compare low, medium, and high danceability songs.

### Distribution of Song Popularity

This plot shows the distribution of popularity scores in the Spotify dataset. Popularity is measured from 0 to 100, and the distribution shows that many songs have low to moderate popularity while fewer songs are extremely popular.

<iframe
  src="assets/popularity_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Danceability vs. Popularity

This plot compares danceability and popularity. It helps show whether more danceable songs tend to have higher popularity scores.

<iframe
  src="assets/danceability_vs_popularity.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates

I grouped songs by danceability level and compared their average popularity. This helps summarize whether songs with different danceability levels tend to have different popularity scores.

| Danceability Group | Average Popularity |
|---|---:|
| Low | lower average popularity |
| Medium | moderate average popularity |
| High | higher average popularity |

## Assessment of Missingness

I focused on the missingness of `tempo`, since it had the largest amount of missingness in the original dataset. I tested whether missingness in `tempo` depended on other observed columns.

For the dependence test, I compared tempo missingness against `duration_ms`. The p-value was less than 0.001, so I reject the null hypothesis and conclude that tempo missingness likely depends on duration. This suggests that the missingness is not completely random.

For the non-dependence test, I compared tempo missingness against `random_noise`, a randomly generated negative-control column. The p-value was 0.616. Since this p-value is greater than 0.05, I do not reject the null hypothesis, which makes sense because missingness should not depend on random noise.

I do not think this missingness is NMAR because the missingness does not seem to depend directly on the unseen tempo value itself. It is more reasonable to treat it as MAR because it appears related to observed columns in the dataset.

<iframe
  src="assets/missingness_plot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Hypothesis Testing

Null hypothesis: songs with high danceability and songs with low danceability have the same average popularity.

Alternative hypothesis: songs with high danceability have higher average popularity than songs with low danceability.

I defined high danceability as songs above the median danceability score and low danceability as songs at or below the median. My test statistic was the difference in mean popularity between the two groups. I used a permutation test with 1,000 repetitions and a significance level of 0.05.

The observed difference was about 9.66 popularity points, and the p-value was less than 0.001. Since this p-value is less than 0.05, I reject the null hypothesis. This suggests that high-danceability songs have higher average popularity in this dataset, though this does not prove causation.

<iframe
  src="assets/hypothesis_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Framing a Prediction Problem

My prediction problem is to predict a song's Spotify popularity score. This is a regression problem because popularity is a numeric value from 0 to 100.

The response variable is `popularity`. I use RMSE as my evaluation metric because RMSE measures prediction error in popularity points and penalizes large mistakes more heavily. This makes sense because being off by 30 popularity points is much worse than being off by 5.

At prediction time, I assume we know the song's audio features, such as danceability, energy, valence, tempo, and other track-level information. I avoid using any feature that directly leaks the popularity score.

## Baseline Model

My baseline model uses three original quantitative features: `danceability`, `energy`, and `valence`. I used median imputation for missing values and a linear regression model inside a single sklearn Pipeline.

The baseline RMSE was 31.48. This model is simple and interpretable, but it is limited because it only uses three audio features and assumes a linear relationship between those features and popularity.

## Final Model

My final model improves on the baseline by using more audio features, categorical information, and engineered features. I engineered `energy_danceability` and `valence_energy`. These features make sense because a song's popularity may depend not just on one audio quality alone, but on combinations of qualities. For example, a song that is both energetic and danceable may behave differently from a song that is only high in one of those features.

I used a random forest regressor because it can capture nonlinear relationships better than linear regression. I tuned `max_depth` and `min_samples_leaf` using GridSearchCV.

The best hyperparameters were `{'model__max_depth': None, 'model__min_samples_leaf': 1}`. The final RMSE was 23.49, compared to the baseline RMSE of 31.48. Since the final model has a lower RMSE, it performs better than the baseline.

## Fairness Analysis

For fairness analysis, I tested whether my final model performs differently for low-danceability songs and high-danceability songs. Since this is a regression problem, I used RMSE as my evaluation metric.

Group X: low-danceability songs, defined as songs at or below the median danceability score.  
Group Y: high-danceability songs, defined as songs above the median danceability score.

Null hypothesis: the model is fair, meaning the RMSE for low-danceability and high-danceability songs is roughly the same.

Alternative hypothesis: the model is worse for low-danceability songs, meaning the RMSE is higher for low-danceability songs.

The test statistic was RMSE for low-danceability songs minus RMSE for high-danceability songs. The observed difference was -2.14, and the p-value was 0.934.

Since the p-value is greater than 0.05, I do not reject the null hypothesis. This means there is not evidence that the model performs worse for low-danceability songs.

<iframe
  src="assets/fairness_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
