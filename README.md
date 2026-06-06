# What Makes a Spotify Song Popular?

By Venkata Mangalampeta

## Introduction

This project analyzes the Spotify Music Tracks dataset to explore how audio features such as danceability, energy, and valence relate to song popularity. My main question is: **are more danceable songs generally more popular?**

This question is interesting because Spotify popularity is something artists, listeners, and platforms care about, but it is not obvious whether measurable audio features actually relate to popularity. I focused on five musically distinct genres: pop, rock, hip-hop, country, and classical. I chose these genres because they are different enough to make comparisons meaningful.

The main columns I use are `popularity`, `danceability`, `energy`, `valence`, `tempo`, `duration_ms`, and `track_genre`. The `popularity` column is the main response variable, while the audio features describe measurable qualities of each song.

## Data Cleaning and Exploratory Data Analysis

I cleaned the dataset by dropping duplicate rows and filtering to five genres: pop, rock, hip-hop, country, and classical. I also created new columns to support my analysis. For example, I converted duration from milliseconds to minutes and created a `high_danceability` column based on whether a song's danceability score was above the median.

These cleaning steps helped make the dataset easier to analyze and kept the project focused on comparing musically distinct genres.

### Cleaned DataFrame Preview

The table below shows the first few rows of the cleaned dataset used for my analysis.

| popularity | danceability | energy | valence | tempo | track_genre |
| --- | --- | --- | --- | --- | --- |
| 54 | 0.608 | 0.638 | 0.439 | 140.109 | classical |
| 59 | 0.583 | 0.308 | 0.241 | 118.226 | classical |
| 59 | 0.642 | 0.562 | 0.671 | 149.82 | classical |
| 2 | 0.496 | 0.186 | 0.585 | 79.334 | classical |
| 1 | 0.355 | 0.215 | 0.65 | 80.606 | classical |

### Distribution of Song Popularity

<iframe
  src="assets/popularity_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This plot shows the distribution of Spotify popularity scores. Many songs have low to moderate popularity, while fewer songs are extremely popular. This makes sense because Spotify has many tracks, but only a smaller number become widely listened to.

### Danceability vs. Popularity

<iframe
  src="assets/danceability_vs_popularity.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This plot compares danceability and popularity. While the relationship is not perfect, it helps motivate my hypothesis test about whether high-danceability songs tend to have higher average popularity.

### Energy vs. Popularity

<iframe
  src="assets/energy_vs_popularity.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This plot compares energy and popularity. Like danceability, energy alone does not perfectly explain popularity, but it is still useful because it captures another important audio quality of a song.

### Interesting Aggregates

I grouped songs by danceability level and compared their average popularity. This gives a clearer summary of whether low, medium, and high danceability songs differ in popularity.

| danceability_group | popularity |
| --- | --- |
| Low | 13.78 |
| Medium | 25.694 |
| High | 34.163 |

The grouped table suggests that songs with higher danceability tend to have higher average popularity. This motivated my later hypothesis test.

I also grouped songs by genre to compare average popularity and audio features across the five genres.

| track_genre | popularity | danceability | energy | valence |
| --- | --- | --- | --- | --- |
| classical | 13.45 | 0.38 | 0.21 | 0.40 |
| country | 16.76 | 0.56 | 0.62 | 0.53 |
| hip-hop | 38.13 | 0.74 | 0.69 | 0.56 |
| pop | 46.87 | 0.63 | 0.62 | 0.52 |
| rock | 19.38 | 0.54 | 0.69 | 0.55 |

Pop and hip-hop have the highest average popularity among the selected genres, while classical has the lowest. Hip-hop also has the highest average danceability, which connects back to my main question about danceability and popularity.

## Assessment of Missingness

I focused on the missingness of `tempo`, since it had the largest amount of missingness in the original dataset. About 19.4% of the values in `tempo` were missing. I tested whether missingness in `tempo` depended on other observed columns.

For the dependence test, I compared tempo missingness against `duration_ms`. The p-value was less than 0.001, so I reject the null hypothesis and conclude that tempo missingness likely depends on duration. This suggests that the missingness is not completely random.

For the non-dependence test, I compared tempo missingness against `random_noise`, a randomly generated negative-control column. The p-value was 0.614. Since this p-value is greater than 0.05, I do not reject the null hypothesis, which makes sense because missingness should not depend on random noise.

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

At prediction time, I assume we know the song's audio features, such as danceability, energy, valence, tempo, duration, and genre. I avoid using any feature that directly leaks the popularity score.

## Baseline Model

My baseline model uses three original quantitative features: `danceability`, `energy`, and `valence`. I used median imputation for missing values and a linear regression model inside a single sklearn Pipeline.

The baseline RMSE was **31.48**. This model is simple and interpretable, but it is limited because it only uses three audio features and assumes a linear relationship between those features and popularity. I do not believe this model is very strong because an error of about 31 popularity points is large on a 0 to 100 popularity scale.

## Final Model

My final model improves on the baseline by using more audio features, categorical information, and engineered features. I engineered `energy_danceability` and `valence_energy`. These features make sense because a song's popularity may depend not just on one audio quality alone, but on combinations of qualities. For example, a song that is both energetic and danceable may behave differently from a song that is only high in one of those features.

I used a random forest regressor because it can capture nonlinear relationships better than linear regression. I tuned `max_depth` and `min_samples_leaf` using GridSearchCV.

The best hyperparameters were:

- `model__max_depth`: `None`
- `model__min_samples_leaf`: `1`

The final RMSE was **23.49**, compared to the baseline RMSE of **31.48**. Since the final model has a lower RMSE, it performs better than the baseline. This suggests that adding more features, engineered interaction features, and a nonlinear model helped capture more of the structure in the data.

## Fairness Analysis

For fairness analysis, I tested whether my final model performs differently for low-danceability songs and high-danceability songs. Since this is a regression problem, I used RMSE as my evaluation metric.

Group X: low-danceability songs, defined as songs at or below the median danceability score.  
Group Y: high-danceability songs, defined as songs above the median danceability score.

Null hypothesis: the model is fair, meaning the RMSE for low-danceability and high-danceability songs is roughly the same.

Alternative hypothesis: the model is worse for low-danceability songs, meaning the RMSE is higher for low-danceability songs.

The test statistic was RMSE for low-danceability songs minus RMSE for high-danceability songs. The observed difference was **-2.14**, and the p-value was **0.936**.

Since the p-value is greater than 0.05, I do not reject the null hypothesis. This means there is not evidence that the model performs worse for low-danceability songs.

<iframe
  src="assets/fairness_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
