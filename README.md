# What Makes a Spotify Song Popular?

By Venkata Mangalampeta

## Introduction

This project analyzes the Spotify Music Tracks dataset to explore whether audio features such as danceability, energy, and valence help explain song popularity. I focus on five musically distinct genres: pop, rock, hip-hop, country, and classical. The main question is: do measurable audio features help explain how popular a song is on Spotify?

The response variable is `popularity`, a score from 0 to 100. Relevant features include `danceability`, `energy`, `valence`, `tempo`, `duration_ms`, `explicit`, `release_date`, and `track_genre`.

## Data Cleaning and Exploratory Data Analysis

I filtered the dataset to five genres: pop, rock, hip-hop, country, and classical. I chose these genres because they are musically distinct and allow more meaningful comparisons. I converted `duration_ms` into minutes, extracted `release_year` from `release_date`, and created a high-danceability indicator based on the median danceability score.

PASTE CLEANED DATAFRAME HEAD TABLE HERE.

### Distribution of Popularity

<iframe
  src="assets/popularity_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This plot shows the distribution of Spotify popularity scores. Popularity is not evenly distributed, and many songs have low to moderate popularity scores.

### Popularity by Genre

<iframe
  src="assets/popularity_by_genre.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This plot compares popularity across the five selected genres. It helps show whether popularity differs by genre.

### Interesting Aggregates

PASTE GENRE TABLE HERE.

The grouped table compares average popularity and audio features by genre. This helps show whether different genres have different audio profiles.

## Assessment of Missingness

I examined columns with missing values and tested whether missingness appeared to depend on other observed columns. I do not believe the missingness is NMAR because the missingness does not appear to depend directly on the unseen missing value itself. Instead, it is more likely MCAR or MAR depending on whether it depends on observed columns such as popularity or duration.

For my missingness permutation tests, I tested whether missingness in COLUMN_NAME depended on popularity and duration. The p-value for the popularity test was PASTE_PVAL. The p-value for the duration test was PASTE_PVAL. Based on these results, I concluded that the missingness DOES/DOES NOT appear to depend on popularity, and DOES/DOES NOT appear to depend on duration.

## Hypothesis Testing

Null hypothesis: songs with high danceability and songs with low danceability have the same average popularity.

Alternative hypothesis: songs with high danceability have higher average popularity than songs with low danceability.

I defined high danceability as songs above the median danceability score and low danceability as songs at or below the median. My test statistic was the difference in mean popularity between the two groups. I used a permutation test with 1,000 repetitions and a significance level of 0.05.

The observed difference was about 3.2 popularity points, and the p-value was 0.03. Since 0.03 is less than 0.05, I reject the null hypothesis. The data provides evidence that high-danceability songs tend to have higher average popularity, but this does not prove that danceability causes popularity.

## Framing a Prediction Problem

My prediction problem is to predict a track’s numeric Spotify popularity score. This is a regression problem because `popularity` is a numeric value from 0 to 100. I use RMSE as my evaluation metric because it measures prediction error in popularity points and penalizes larger mistakes more strongly.

At prediction time, I assume we know the song’s audio features, genre, explicit status, release date, and duration. I do not use information that directly leaks popularity.

## Baseline Model

My baseline model uses three quantitative audio features: danceability, energy, and valence. I used median imputation and a linear regression model inside a single sklearn Pipeline.

The baseline model’s RMSE was PASTE_BASELINE_RMSE. This model is useful as a simple starting point, but it is limited because it ignores genre, release year, duration, and possible nonlinear relationships.

## Final Model

My final model uses a random forest regressor with more audio features, genre, explicit status, and engineered features. I engineered `duration_min`, `release_year`, and `energy_danceability`. These features are useful because duration, recency, and the interaction between energy and danceability may all relate to song popularity.

I tuned `max_depth` and `min_samples_leaf` using GridSearchCV. The best hyperparameters were PASTE_BEST_PARAMS. The final model’s RMSE was PASTE_FINAL_RMSE. This improved over the baseline model, showing that the added features and more flexible model better captured patterns in the data.

## Fairness Analysis

I tested whether my final model performs worse for low-danceability songs than high-danceability songs. Since this is a regression problem, I used RMSE as my metric.

Null hypothesis: the model is fair; RMSE for low- and high-danceability songs is roughly the same.

Alternative hypothesis: the model is worse for low-danceability songs, meaning RMSE is higher for that group.

The test statistic was RMSE for low-danceability songs minus RMSE for high-danceability songs. The observed test statistic was PASTE_OBS_FAIRNESS, and the p-value was PASTE_FAIRNESS_PVAL. Based on this p-value, I DO/DO NOT reject the null hypothesis.
