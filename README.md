# What Makes a Spotify Song Popular?

By Venkata Mangalampeta

## Introduction

This project analyzes the Spotify Music Tracks dataset, which contains audio and metadata features for Spotify tracks. The original dataset contains 114,000 tracks across 114 genres. For this project, I focused on five musically distinct genres: pop, rock, hip-hop, country, and classical. After filtering to these genres, removing duplicate rows, and dropping rows missing the main analysis columns, my cleaned dataset contains **3,900 rows and 25 columns**.

My main question is: **are more danceable songs generally more popular?**

This question is interesting because Spotify popularity is something artists, listeners, and platforms care about, but it is not obvious whether measurable audio features actually relate to popularity. Popularity is likely affected by many outside factors, such as artist fame, marketing, playlists, and release timing. Still, audio features like danceability, energy, and valence may reveal patterns in what kinds of songs tend to become more popular.

I chose pop, rock, hip-hop, country, and classical because they are musically distinct. Pop and hip-hop are often more mainstream and rhythm-focused, rock and country have different instrumentation and audience patterns, and classical music is structurally different from the other genres. This makes the comparisons more meaningful than comparing five extremely similar genres.

The main columns I use are:

| Column | Description |
| --- | --- |
| `popularity` | Spotify popularity score from 0 to 100. This is my main response variable. |
| `danceability` | A score from 0 to 1 describing how suitable a track is for dancing. |
| `energy` | A score from 0 to 1 describing intensity and activity. |
| `valence` | A score from 0 to 1 describing musical positivity. |
| `tempo` | Estimated beats per minute. |
| `duration_ms` | Duration of the track in milliseconds. |
| `track_genre` | Spotify genre label for the track. |
| `explicit` | Whether the track contains explicit content. |

## Data Cleaning and Exploratory Data Analysis

I cleaned the dataset by dropping duplicate rows and filtering to five genres: pop, rock, hip-hop, country, and classical. I did this because the full dataset contains many genres, and comparing all of them at once would make the project too broad. Focusing on five musically distinct genres made the analysis more interpretable.

I also created new columns to support my analysis. I converted `duration_ms` into minutes, since minutes are easier to interpret than milliseconds. I created a `high_danceability` column based on whether a song's danceability score was above the median, which I later used in my hypothesis test and fairness analysis. I also created `danceability_group`, which splits songs into low, medium, and high danceability groups.

I chose not to join with `artists.csv` because my question is mainly about track-level audio features and track-level popularity. The artist file could be useful for a different project, but joining it correctly is complicated because some tracks have multiple artists. Since my main goal is to study how audio features relate to track popularity, I kept the analysis focused on `music_tracks.csv`.

### Cleaned DataFrame Preview

The table below shows the first few rows of the cleaned dataset used for my analysis. The first few rows happen to come from the classical genre because of the dataset ordering, but the cleaned dataset contains all five selected genres.

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
  width="100%"
  height="520"
  frameborder="0"
></iframe>

This plot shows the distribution of Spotify popularity scores. Many songs have very low popularity, while fewer songs are highly popular. This suggests that popularity is not evenly distributed: most tracks receive relatively little attention, while a smaller group of tracks becomes widely listened to.

### Danceability vs. Popularity

<iframe
  src="assets/danceability_vs_popularity.html"
  width="100%"
  height="520"
  frameborder="0"
></iframe>

This plot compares danceability and popularity. The relationship is noisy, meaning danceability alone does not fully explain popularity. However, the plot still suggests that many higher-popularity songs tend to fall in the medium-to-high danceability range, which motivates my hypothesis test.

### Energy vs. Popularity

<iframe
  src="assets/energy_vs_popularity.html"
  width="100%"
  height="520"
  frameborder="0"
></iframe>

This plot compares energy and popularity. Like danceability, energy alone does not perfectly explain popularity, but it is still useful because it captures another important audio quality of a song. This also supports the modeling section, where I use multiple audio features together instead of relying on just one feature.

### Interesting Aggregates

I grouped songs by danceability level and compared their average popularity. This gives a clearer summary of whether low, medium, and high danceability songs differ in popularity.

| Danceability Group | Average Popularity |
| --- | --- |
| Low | 13.78 |
| Medium | 25.694 |
| High | 34.163 |

The grouped table suggests that songs with higher danceability tend to have higher average popularity. This motivated my later hypothesis test.

I also grouped songs by genre to compare average popularity and average audio feature values across the five genres.

| Genre | Average Popularity | Average Danceability | Average Energy | Average Valence |
| --- | --- | --- | --- | --- |
| classical | 13.45 | 0.38 | 0.21 | 0.40 |
| country | 16.76 | 0.56 | 0.62 | 0.53 |
| hip-hop | 38.13 | 0.74 | 0.69 | 0.56 |
| pop | 46.87 | 0.63 | 0.62 | 0.52 |
| rock | 19.38 | 0.54 | 0.69 | 0.55 |

Pop and hip-hop have the highest average popularity among the selected genres, while classical has the lowest. Hip-hop also has the highest average danceability, which connects back to my main question about danceability and popularity. This also shows why genre is an important feature to include in the final model: popularity differs substantially across genres.

## Assessment of Missingness

I assessed missingness using the original dataset before dropping rows with missing values. The columns with missing values were `tempo`, `artists`, `album_name`, and `track_name`. The most important missing column was `tempo`, because about **19.4%** of its values were missing. The other missing columns had extremely low missingness, so I focused my permutation tests on `tempo`.

I do not believe `tempo` is NMAR. If missingness were NMAR, the probability that `tempo` is missing would depend directly on the missing tempo value itself. For example, this would mean very slow or very fast songs are missing tempo specifically because of their unobserved tempo. I do not have evidence for that. It is more reasonable that tempo missingness is related to observable properties of the track, such as duration or audio structure, which would make it MAR.

Additional data that could help explain this missingness would include information about how Spotify extracted the audio features, whether tempo extraction failed for certain audio files, and whether some genres or track types were harder for Spotify's algorithms to analyze. This could help determine whether the missingness is better explained by observed technical factors.

For the dependence test, I compared tempo missingness against `duration_ms`. My null hypothesis was that tempo missingness does not depend on duration. My alternative hypothesis was that tempo missingness does depend on duration. I used the absolute difference in mean duration between rows with missing tempo and rows with non-missing tempo as my test statistic. The p-value was less than 0.001, so I reject the null hypothesis and conclude that tempo missingness appears to be associated with duration. This suggests that the missingness is not completely random.

For the non-dependence test, I compared tempo missingness against `random_noise`, a randomly generated negative-control column. The p-value was 0.614. Since this p-value is greater than 0.05, I do not reject the null hypothesis, which makes sense because missingness should not depend on random noise.

Overall, I classify the missingness of `tempo` as likely MAR because it appears related to observed columns like `duration_ms`, but not to a random unrelated column.

<iframe
  src="assets/missingness_plot.html"
  width="100%"
  height="520"
  frameborder="0"
></iframe>

This plot compares the distribution of `duration_ms` for tracks with missing and non-missing `tempo`. The distributions are not identical, which is consistent with the permutation test result that tempo missingness depends on duration.

## Hypothesis Testing

Null hypothesis: songs with high danceability and songs with low danceability have the same average popularity.

Alternative hypothesis: songs with high danceability have higher average popularity than songs with low danceability.

I defined high danceability as songs above the median danceability score and low danceability as songs at or below the median. I used the median because it creates two balanced groups and avoids choosing an arbitrary cutoff. My test statistic was the difference in mean popularity between the high-danceability group and the low-danceability group.

I used a permutation test with 1,000 repetitions and a significance level of 0.05. A permutation test is appropriate because I want to test whether the observed difference in average popularity could have happened by chance if danceability group and popularity were unrelated.

The observed difference was about **9.66 popularity points**, meaning high-danceability songs had a higher average popularity in my sample. The p-value was less than 0.001, since none of the 1,000 simulated differences were as large as the observed difference. Since this p-value is less than 0.05, I reject the null hypothesis.

This suggests that high-danceability songs have higher average popularity in this dataset. However, this does not prove that danceability causes popularity. Other factors, such as genre or artist popularity, could also influence the relationship.

<iframe
  src="assets/hypothesis_test.html"
  width="100%"
  height="520"
  frameborder="0"
></iframe>

The dashed vertical line shows the observed difference in mean popularity. Since it is far to the right of the simulated null distribution, the observed result is unlikely under the null hypothesis.

## Framing a Prediction Problem

My prediction problem is to predict a song's Spotify popularity score. This is a regression problem because popularity is a numeric value from 0 to 100.

The response variable is `popularity`. I chose this response variable because popularity directly connects to my main question about what kinds of songs tend to become more popular. I use RMSE as my evaluation metric because RMSE measures prediction error in popularity points and penalizes large mistakes more heavily. This makes sense because being off by 30 popularity points is much worse than being off by 5.

At prediction time, I assume we know the song's audio features and metadata, such as danceability, energy, valence, tempo, duration, explicit status, and genre. These are track-level features that could be known once a song has been processed by Spotify's audio feature system. I avoid using any feature that directly leaks the popularity score. I also do not use artist popularity or follower counts, since those come from a separate artist-level dataset and would change the meaning of the prediction problem.

## Baseline Model

My baseline model uses three original quantitative features: `danceability`, `energy`, and `valence`. These are all quantitative features because each one is a numeric audio score between 0 and 1. The baseline model uses **0 ordinal features** and **0 nominal features**, so no categorical encoding was needed.

I used median imputation for missing values and a linear regression model inside a single sklearn Pipeline. Linear regression is a reasonable baseline because it is simple, interpretable, and gives a starting point for understanding how well basic audio features can predict popularity.

The baseline RMSE was **31.48**. This model is not very strong because an error of about 31 popularity points is large on a 0 to 100 popularity scale. The weak performance makes sense because popularity is influenced by many factors beyond just danceability, energy, and valence.

## Final Model

My final model improves on the baseline by using more audio features, categorical information, engineered features, and a more flexible algorithm.

The final model uses the following quantitative features:

- `danceability`
- `energy`
- `valence`
- `tempo`
- `duration_min`
- `loudness`
- `acousticness`
- `instrumentalness`
- `speechiness`
- `liveness`
- `energy_danceability`
- `valence_energy`

The final model also uses the following nominal features:

- `track_genre`
- `explicit`

For the categorical features, I used one-hot encoding. For numeric features, I used median imputation. All preprocessing and modeling steps were implemented in a single sklearn Pipeline.

I engineered two main interaction features: `energy_danceability` and `valence_energy`. These features make sense because a song's popularity may depend not just on one audio quality alone, but on combinations of qualities. For example, a song that is both energetic and danceable may behave differently from a song that is only high in one of those features. Similarly, a song that is both positive-sounding and energetic may have a different listener appeal than a song that is positive but low-energy.

I used a random forest regressor because it can capture nonlinear relationships better than linear regression. This is useful because the relationship between audio features and popularity is unlikely to be perfectly linear. I tuned `max_depth` and `min_samples_leaf` using GridSearchCV because these hyperparameters control model complexity. `max_depth` controls how deep each tree can grow, while `min_samples_leaf` controls how many samples must be in each leaf. Tuning these helps balance underfitting and overfitting.

The best hyperparameters were:

- `model__max_depth`: `None`
- `model__min_samples_leaf`: `1`

The final RMSE was **23.49**, compared to the baseline RMSE of **31.48**. Since the final model has a lower RMSE, it performs better than the baseline. This suggests that adding more features, engineered interaction features, genre information, and a nonlinear model helped capture more of the structure in the data.

<iframe
  src="assets/final_model_predictions.html"
  width="100%"
  height="520"
  frameborder="0"
></iframe>

This plot compares actual popularity scores to predicted popularity scores. Points closer to the dashed diagonal line are better predictions. The plot shows that the model captures some structure in the data, but there is still error, which is expected because song popularity depends on many factors beyond audio features. The model performs better for low- and mid-popularity songs than for the most popular songs, which makes sense because extremely popular songs are likely influenced by outside factors like artist fame, playlists, and marketing.

## Fairness Analysis

For fairness analysis, I tested whether my final model performs differently for low-danceability songs and high-danceability songs. Since this is a regression problem, I used RMSE as my evaluation metric.

Group X: low-danceability songs, defined as songs at or below the median danceability score.  
Group Y: high-danceability songs, defined as songs above the median danceability score.

Null hypothesis: the model is fair, meaning the RMSE for low-danceability and high-danceability songs is roughly the same, and any observed difference is due to random chance.

Alternative hypothesis: the model is worse for low-danceability songs, meaning the RMSE is higher for low-danceability songs.

The test statistic was:

**RMSE for low-danceability songs − RMSE for high-danceability songs**

I used a permutation test with 1,000 repetitions and a significance level of 0.05. The observed difference was **-2.14**, and the p-value was **0.936**.

Since the p-value is greater than 0.05, I do not reject the null hypothesis. This means there is not evidence that the model performs worse for low-danceability songs. In fact, the observed difference was negative, meaning the model's RMSE was slightly lower for low-danceability songs in this test set.

<iframe
  src="assets/fairness_test.html"
  width="100%"
  height="520"
  frameborder="0"
></iframe>

The dashed vertical line shows the observed difference in RMSE. Since the observed value is not extreme in the direction of the alternative hypothesis, I do not find evidence of unfairness against low-danceability songs.
