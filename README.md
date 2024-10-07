
<!-- README.md is generated from README.Rmd. Please edit that file -->

# collinear: R Package for Seamless Multicollinearity Management

<!-- Development badges 
&#10;[![R-CMD-check](https://github.com/BlasBenito/collinear/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/BlasBenito/collinear/actions/workflows/R-CMD-check.yaml)
[![Devel-version](https://img.shields.io/badge/devel%20version-1.0.1-blue.svg)](https://github.com/blasbenito/collinear)
&#10;
&#10;<!-- badges: start -->

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.10039489.svg)](https://doi.org/10.5281/zenodo.10039489)
[![CRAN
status](https://www.r-pkg.org/badges/version/collinear)](https://cran.r-project.org/package=collinear)
[![CRAN_Download_Badge](http://cranlogs.r-pkg.org/badges/grand-total/collinear)](https://CRAN.R-project.org/package=collinear)
[![R-CMD-check](https://github.com/BlasBenito/collinear/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/BlasBenito/collinear/actions/workflows/R-CMD-check.yaml)

<!-- badges: end -->

## Summary

The R package `collinear` combines four methods to offer a comprehensive
tool for automated multicollinearity management:

- **Target Encoding**: transforms categorical variables to numeric by
  using a numeric response variable as reference to facilitate
  multicollinearity management in dataset with mixed data types
  (categorical and numeric).
- **Preference Order**: ranking method based on the association between
  a response variable and a set of predictors to facilitate the
  preservation of important predictors during a ulticollinearity
  filtering.
- **Pairwise Correlation Filtering**: automated multicollinearity
  filtering based on pairwise correlations between mixed types of
  predictors.
- **Variance Inflation Factor Filtering**: automated multicollinearity
  filtering based on Variance Inflation Factors.

These methods are integrated in the `collinear()` function:

``` r
x <- collinear(
  df, #data frame
  response, #name of a response variable
  predictors, #names of predictors to select,
  encoding_method, #method to convert categorical predictors into numerics
  max_cor, #maximum pairwise correlation
  max_vif #maximum variance inflation factor
)
```

The function returns a character vector with the names of the selected
predictors.

The package contains other functions that may be useful during
multicollinearity management:

- `cor_select()`: automated pairwise correlation filtering.
- `vif_select()`: automated VIF-based multicollinearity filtering.
- `preference_order()`: to rank predictors based on their univariate
  association with a response variable.
- `target_encoding_lab()`: to transform categorical predictors into
  numeric by using a numeric response variable as reference.
- `cor_df()`: to generate a data frame of pairwise correlation scores.
- `cor_matrix()`: to convert a correlation data frame into matrix or
  generate a correlation matrix from a training data frame.
- `cor_clusters()`: to cluster predictors in groups, based on their
  pairwise correlations.
- `vif_df()`: to generate a data frame with variance inflation factors.

## Citation

If you found this package useful during your research work, please cite
it as:

*Blas M. Benito (2024). collinear: R Package for Seamless
Multicollinearity Management. Version 1.2.0. doi:
10.5281/zenodo.10039489*

## Install

The package `collinear` can be installed from CRAN.

``` r
install.packages("collinear")
library(collinear)
```

The development version can be installed from GitHub.

``` r
remotes::install_github(
  repo = "blasbenito/collinear", 
  ref = "development"
  )
```

## Multicollinearity management with the `collinear` package.

This section shows the basic usage of the package and offers a brief
explanation on the methods used within.

### Required libraries and example data

The libraries below are required to run the examples in this section.

``` r
library(collinear)
```

The package `collinear` is shipped with a data frame named `vi`, with
30.000 rows and 67 columns with response variables of different types
and a mixture of numeric and categorical predictors

``` r
dplyr::glimpse(head(vi))
```

The response variables are:

- `vi_numeric`: numeric continuous variable, values of a vegetation
  index.
- `vi_counts`: integer counts, derived from `vi_numeric`.
- `vi_binomial`: binomial variable of ones and zeros.
- `vi_category`: character variable with 5 categories from “very_low” to
  “very_high”.
- `vi_factor`: as `vi_category`, but coded as a factor.

The predictor variables are stored in the following objects:

- `vi_predictors`: 61 predictors of mixed types.
- `vi_predictors_numeric`: 49 predictors of classes numeric and integer.
- `vi_predictors_categorical`: 12 predictors of the classes character
  and factor.

``` r
vi_predictors
```

``` r
vi_predictors_numeric
```

``` r
vi_predictors_categorical
```

### `collinear()`

The function `collinear()` implements all tools required to facilitate
multicollinearity filtering.

It takes these inputs.

- **Input data**:
  - `df` (required): Data frame with numeric and/or categorical
    predictors, and optionally a response variable.
  - `response` (optional): Name of a response variable that can be
    categorical, numeric, binomial, or integer. If not provided, target
    encoding to transform categoricals into numeric and automated
    preference order are disabled.
  - `predictors` (optional): Names of predictors involved in the
    multicollinearity filtering. If not provided, all columns in `df`
    but `response` (if provided) are used as predictors.
- **Target encoding**:
  - `encoding_method` (optional, depends on `response`): Name of the
    method to transform categorical predictors into numeric. If NULL, or
    `response` is not numeric, target encoding is disabled.  
- **Preference order**:
  - `preference_order` (optional, depends on `response`): Ignored if
    argument `response` is not provided. Otherwise, valid values are:
    - Character vector: predictor names in a custom preference order.
    - Dataframe: result of `preference_order()`.
    - NULL: if `response` is provided, then preference order is computed
      automatically by calling to `preference_order()`.
  - `f` (optional, function name): Function used to compute preference
    order. Functions designed for this argument are named `f_...()`. If
    NULL, a default one based on the nature of the input data is
    selected by `f_default()`.
- **Multicollinearity filtering**:
  - `cor_method` (optional): pairwise correlation method.
  - `max_cor` (optional): Maximum pairwise correlation between
    predictors. If NULL, the pairwise correlation filtering is disabled.
  - `max_vif` (optional): Maximum Variance Inflation Factor of all
    predictors. If NULL, the VIF-based filtering is disabled.

``` r
selected_predictors <- collinear(
  df = vi,
  response = "vi_mean",
  predictors = vi_predictors,
  preference_order = NULL,
  max_cor = 0.75,
  max_vif = 5,
  encoding_method = "mean"
)

selected_predictors
```

The function has returned a list of predictors that should have a
correlation lower than 0.75 with each other, and a VIF lower than 5.
Let’s see if that’s true.

The function `cor_df()` returns a data frame with pairwise correlations,
arranged by the absolute value of the correlation.

``` r
selected_predictors_cor <- cor_df(
  df = vi,
  response = "vi_mean",
  predictors = selected_predictors
)
head(selected_predictors_cor)
```

The data frame above shows that the maximum correlation between two of
the selected predictors is below 0.75, so here `collinear()` worked as
expected.

The function `vif_df()` returns a data frame with the VIF scores of all
predictors.

``` r
selected_predictors_vif <- vif_df(
  df = vi,
  response = "vi_mean",
  predictors = selected_predictors
)
selected_predictors_vif
```

The output shows that the maximum VIF is 4.2, so here `collinear()` did
its work as expected.

#### Arguments `max_cor` and `max_vif`

The arguments `max_cor` and `max_vif` control the intensity of the
multicollinearity filtering.

``` r
#restrictive setup
selected_predictors_restrictive <- collinear(
  df = vi,
  response = "vi_mean",
  predictors = vi_predictors,
  max_cor = 0.5,
  max_vif = 2.5
)

#permissive setup
selected_predictors_permissive <- collinear(
  df = vi,
  response = "vi_mean",
  predictors = vi_predictors,
  max_cor = 0.9,
  max_vif = 10
)
```

These are the variables selected under a restrictive setup:

``` r
selected_predictors_restrictive
```

These are the variables selected under a more permissive setup:

``` r
selected_predictors_permissive
```

As expected, the restrictive setup resulted in a smaller set of selected
predictors. There are no hard rules for `max_cor` and `max_vif`, and
their selection will depend on the objective of the analysis and the
nature of the predictors.

#### The `response` argument

The response argument is used to encode categorical variables as
numeric. When omitted, the `collinear()` function ignores categorical
variables. However, the function `cor_select()` can help when there is
not a suitable `response` variable in a data frame. This option is
discussed at the end of this section.

``` r
selected_predictors_response <- collinear(
  df = vi,
  response = "vi_mean",
  predictors = vi_predictors
)

selected_predictors_no_response <- collinear(
  df = vi,
  predictors = vi_predictors
)
```

When the argument `response` is used, the output may contain categorical
predictors (tagged with `<chr>`, from “character” below).

``` r
dplyr::glimpse(vi[, selected_predictors_response])
```

However, when the argument `response` is ignored, all categorical
predictors are ignored.

``` r
dplyr::glimpse(vi[, selected_predictors_no_response])
```

If there are categorical variables in a data frame, but there is no
suitable `response` variable, then the function `cor_select()` can
handle the multicollinearity management via pairwise correlations, but
at a MUCH higher computational cost, and with different results, as
shown below.

``` r
tictoc::tic()
selected_predictors_response <- cor_select(
  df = vi,
  response = "vi_mean",
  predictors = vi_predictors
)
tictoc::toc()

tictoc::tic()
selected_predictors_no_response <- cor_select(
  df = vi,
  predictors = vi_predictors
)
tictoc::toc()
```

``` r
selected_predictors_response
```

``` r
selected_predictors_no_response
```

The variable selection results differ because the numeric
representations of the categorical variables are rather different
between the two options. When no `response` is provided, the function
`cor_select()` compares categorical predictors against numeric ones by
encoding each categorical after each numeric, and compares pairs of
categoricals using Cramer’s V, implemented in the function `cramer_v()`.
Additionally, Cramer’s V values are not directly comparable with Pearson
or Spearman correlation scores, and having them together in the same
analysis might induce bias during the variable selection. Not using the
`response` argument should always be the last option.

#### Preference order

The argument `preference_order` gives the user some control on what
predictors should be removed first and what predictors should be kept
during the multicollinearity filtering. This argument accepts a vector
of predictor names in the order of interest, or the result of the
function `preference_order()`, which allows to define preference order
following a quantitative criteria.

##### Manual preference order

Let’s start with the former option. Below, the argument
`preference_order` names several predictors that are of importance for a
hypothetical analysis. The `predictors` not in `preference_order` are
ranked by the absolute sum of their correlations with other predictors
during the pairwise correlation filtering, and by their VIF during the
VIF-based filtering.

``` r
selected_predictors <- cor_select(
  df = vi,
  response = "vi_mean",
  predictors = vi_predictors,
  preference_order = c(
    "soil_temperature_mean",
    "soil_temperature_max",
    "soil_type"
  )
)

selected_predictors
```

Notice that in the output, two of the variables in `preference_order`
are selected (“soil_temperature_mean” and “soil_type”), but one was
removed (“soil_temperature_max”). This happens because at some point in
the selection, the VIF of “soil_temperature_mean” and
“soil_temperature_max” was higher than `max_vif`, and the one with lower
preference was removed.

##### Quantitative preference order

The function `preference_order()` requires the `response` argument, and
takes a function `f` that returns a value of association between the
response and any predictor. This value is then located in the
“preference” column of the function’s output.

``` r
#parallelization setup
# future::plan(
#   future::multisession,
#   workers = 2 #set to parallelly::availableWorkers() - 1
# )

#progress bar
#progressr::handlers(global = TRUE)

#compute preference order
preference_rsquared <- preference_order(
  df = vi,
  response = "vi_mean",
  predictors = vi_predictors,
  f = f_rsquared,
)

preference_rsquared
```

The result of `preference_order()` can be plugged right away into the
`preference_order` argument of collinear.

``` r
selected_predictors <- collinear(
  df = vi,
  response = "vi_mean",
  predictors = vi_predictors,
  preference_order = preference_rsquared
)
selected_predictors
```

This variable selection satisfies three conditions at once: maximum
correlation between each predictor and the response, maximum pairwise
correlation, and maximum VIF.

The `f` argument used by default is the function `f_rsquared()`, that
returns the R-squared between the response and any predictor.

``` r
f_rsquared(
  x = "growing_season_length",
  y = "vi_mean",
  df = vi
)
```

There are several other `f` functions implemented:

- `f_gam_deviance()`: returns the explained deviance of a univariate GAM
  model between the response and each predictor, fitted with the
  function `mgcv::gam()`. Only if the R package `mgcv` is installed in
  the system.
- `f_rf_rsquared()` (also named `f_rf_deviance()`): returns the
  explained deviance of a univariate Random Forest model between the
  response and each predictor, fitted with the function
  `ranger::ranger()`. Only if the R package `ranger` is installed in the
  system.
- `f_logistic_auc_balanced()` and `f_logistic_auc_unbalanced()`: return
  the area under the ROC curve of univariate binomial GLM between a
  binary response of 1s and 0s and a numeric predictor. The former
  assumes the response is balanced, while the latter applies case
  weights to mitigate unbalances.
- `f_gam_auc_balanced()` and `f_gam_auc_unbalanced()`: return the area
  under the ROC curve of univariate binomial GAM between a binary
  response of 1s and 0s and a numeric predictor. The former assumes the
  response is balanced, while the latter applies case weights to
  mitigate unbalances.
- `f_rf_auc_balanced()` and `f_rf_auc_unbalanced()`: return the area
  under the ROC curve of univariate random forest models between a
  binary response of 1s and 0s and a numeric predictor. The former
  assumes the response is balanced, while the latter applies case
  weights to mitigate unbalances.

``` r
#example of preference order for a binary variable

#the binary variable
table(vi$vi_binary)

#computation of preference order with 
preference_auc <- preference_order(
  df = vi,
  response = "vi_binary",
  predictors = vi_predictors,
  f = f_gam_auc_unbalanced
)

preference_auc
```

Custom functions created by the user are also accepted as input, as long
as they have the `x`, `y`, and `df` arguments, and they return a single
numeric value.

### `cor_select()` and `vif_select()`

The functions `cor_select()` and `vif_select()`, called within
`collinear()`, perform the pairwise correlation filtering, and the
VIF-based filtering. The main difference between them is that
`cor_select()` can handle categorical predictors even when the
`response` is omitted, while `vif_select()` ignores them entirely in
such case.

``` r
selected_predictors_cor <- cor_select(
  df = vi,
  response = "vi_mean",
  predictors = vi_predictors,
  preference_order = preference_rsquared
)

selected_predictors_vif <- vif_select(
  df = vi,
  response = "vi_mean",
  predictors = vi_predictors,
  preference_order = preference_rsquared
)
```

``` r
selected_predictors_cor
```

``` r
selected_predictors_vif
```

### `target_encoding_lab()`

The function `target_encoding_lab()` is used within all other functions
in the package to encode categorical variables as numeric. It implements
four target encoding methods:

- “mean” (in `target_encoding_mean()`): replaces each category with the
  mean of the response across the category. White noise can be added to
  this option to increase data variability.
- “loo” (in `target_encoding_loo()`): replaces each value in a category
  with the mean of the response across all the other cases within the
  category. White noise can be added to this option to increase data
  variability.
- “rank” (in `target_encoding_rank()`): replaces each category with the
  rank of the mean of the response across the category.

The method “mean” is used as default throughout all functions in the
package, but can be changed via the argument `encoding_method`.

Below we use all methods to generate different numeric encodings for the
categorical variable “koppen_zone”.

``` r
df <- target_encoding_lab(
  df = vi,
  response = "vi_mean",
  predictors = "koppen_zone",
  encoding_methods = c(
    "mean",
    "loo",
    "rank"
  ),
  seed = 1,
  white_noise = c(0, 0.01, 0.1),
  verbose = TRUE
)
```

The relationship between these encoded versions of “koppen_zone” and the
response are shown below.

``` r
#get names of encoded variables
koppen_zone_encoded <- grep(
  pattern = "*__encoded*",
  x = colnames(df),
  value = TRUE
)

#record the user's graphical parameters
user.par <- par(no.readonly = TRUE)

#modify graphical parameters for the plot
par(mfrow = c(4, 3))

#plot target encoding
x <- lapply(
  X = koppen_zone_encoded,
  FUN = function(x) plot(
    x = df[[x]],
    y = df$vi_mean,
    xlab = x,
    ylab = "vi_mean",
    cex = 0.5
    )
)

#reset the user's graphical parameters
par(user.par)
```

The function implementing each method can be used directly as well. The
example below shows the “mean” method with the option `replace = FALSE`,
which replaces the categorical values with the numeric ones in the
output data frame.

``` r
head(vi[, c("vi_mean", "koppen_zone")], n = 10)
```

``` r
df <- target_encoding_mean(
  df = vi,
  response = "vi_mean",
  predictor = "koppen_zone",
  replace = TRUE
)

head(df[, c("vi_mean", "koppen_zone")], n = 10)
```

If you got here, thank you for your interest in `collinear`. I hope you
can find it useful!

Blas M. Benito, PhD
