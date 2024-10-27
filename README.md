
<!-- README.md is generated from README.Rmd. Please edit that file -->

# collinear: R Package for Seamless Multicollinearity Management

<!-- Development badges 
&#10;[![R-CMD-check](https://github.com/BlasBenito/collinear/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/BlasBenito/collinear/actions/workflows/R-CMD-check.yaml)
[![Devel-version](https://img.shields.io/badge/devel%20version-1.0.1-blue.svg)](https://github.com/blasbenito/collinear)
&#10;<!-- badges: start -->

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.10039489.svg)](https://doi.org/10.5281/zenodo.10039489)
[![CRAN
status](https://www.r-pkg.org/badges/version/collinear)](https://cran.r-project.org/package=collinear)
[![CRAN_Download_Badge](http://cranlogs.r-pkg.org/badges/grand-total/collinear)](https://CRAN.R-project.org/package=collinear)
[![R-CMD-check](https://github.com/BlasBenito/collinear/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/BlasBenito/collinear/actions/workflows/R-CMD-check.yaml)

<!-- badges: end -->

## Warning

Version 2.0.0 of `collinear` includes changes that may disrupt existing
workflows, and results from previous versions may not be reproducible
due to enhancements in the automated selection algorithms. Please refer
to the Changelog for details.

## Summary

The `collinear` package combines four methods for easy management of
multicollinearity in modelling data frames with numeric and categorical
variables:

- **Target Encoding**: Transforms categorical predictors to numeric
  using a numeric response as reference.
- **Preference Order**: Ranks predictors by their association with a
  response variable to preserve important ones in multicollinearity
  filtering.
- **Pairwise Correlation Filtering**: Automated multicollinearity
  filtering of numeric and categorical predictors based on pairwise
  correlations.
- **Variance Inflation Factor Filtering**: Automated multicollinearity
  filtering of numeric predictors based on Variance Inflation Factors.

These methods are combined in the function `collinear()`, and as
individual functions in `target_encoding_lab()`, `preference_order()`,
`cor_select()`, and `vif_select()`.

## Citation

If you find this package useful, please cite:

*Blas M. Benito (2024). collinear: R Package for Seamless
Multicollinearity Management. Version 1.2.0. doi:
10.5281/zenodo.10039489*

## Main Improvements in Version 2.0.0

1.  **Expanded Functionality**: Functions `collinear()` and
    `preference_order()` support both categorical and numeric responses
    and predictors, and can handle several responses at once.
2.  **Robust Selection Algorithms**: Enhanced selection in
    `vif_select()` and `cor_select()`.
3.  **Refined `preference_order()`**: New functions to compute
    association between response and predictors covering most use-cases,
    and automated function selection depending on data features.
4.  **Simplified Target Encoding**: Streamlined and parallelized for
    better efficiency, and new default is “loo” (leave-one-out).
5.  **Parallelization and Progress Bars**: Utilizes `future` and
    `progressr` for enhanced performance and user experience.

## Install

The package `collinear` can be installed from CRAN.

``` r
install.packages("collinear")
```

The development version can be installed from GitHub.

``` r
remotes::install_github(
  repo = "blasbenito/collinear", 
  ref = "development"
  )
```

Previous versions are in the “archive_xxx” branches of the GitHub
repository.

``` r
remotes::install_github(
  repo = "blasbenito/collinear", 
  ref = "archive_v1.1.1"
  )
```

## Required Packages

The packages required for this tutorial are:

## Parallelization Setup and Progress Bars

Most functions in the package now accept a parallelization setup via
`future::plan()` and progress bars via `progressr::handlers()`. However,
progress bars are ignored in this tutorial because they don’t work in
Rmarkdown.

``` r
future::plan(
  future::multisession,
  workers = parallelly::availableCores() - 1
  )

#progressr::handlers(global = TRUE)
```

## Example Data

The data frame `vi` consists of 30,000 rows and 67 columns, featuring
various response types and a mix of numeric and categorical predictors.
The code below makes it smaller to accelerate this tutorial.

``` r
set.seed(1)
df <- dplyr::slice_sample(
  vi, 
  n = 5000
  )
```

The response columns, all derived from the same data, have descriptive
names: `vi_numeric`, `vi_counts` (integers), `vi_binomial` (1s and 0s),
`vi_categorical` (five categories), and `vi_factor` (factor version of
the previous one).

Predictor names are grouped in character vectors:
`vi_predictors_numeric` (49 numeric and integer predictors),
`vi_predictors_categorical` (12 character and factor predictors), and
`vi_predictors` containing them all.

## Using `collinear()`

The function `collinear()` provides access to the key functionalities of
this package. The code below shows a call to `collinear()` with two
responses and mixed predictor types.

``` r
selection <- collinear::collinear(
  df = df,
  response = c(
    "vi_numeric",
    "vi_categorical"
    ),
  predictors = vi_predictors,
  quiet = FALSE
)
```

The function returns a named list of vectors with predictor names when
more than one response is provided, and a character vector otherwise.

``` r
selection
```

The predictor names are returned in the same order as the preference
order. If `response` is provided, `collinear()` returns non-collinear
variables ordered by their association with `response`. Otherwise, the
predictors are ordered from lower to higher multicollinearity.

### How It Works

The table below shows the functions called directly by `collinear()`,
and their functionality, data requirements, and settings required to
disable them.

| **Function**            | **Functionality**                           | **Requirements**                                      | **Disabled**                                                                       |
|-------------------------|---------------------------------------------|-------------------------------------------------------|------------------------------------------------------------------------------------|
| `target_encoding_lab()` | categorical predictors <br> to numeric      | \- numeric `response` <br> - categorical `predictors` | \- `response = NULL` <br> - categorical `response` <br> - `encoding_method = NULL` |
| `preference_order()`    | rank and preserve <br> important predictors | any `response`                                        | \- `response = NULL` <br> - `preference_order = NULL`                              |
| `cor_select()`          | reduce <br> pairwise correlation            | any `predictors`                                      | `cor_max = NULL`                                                                   |
| `vif_select()`          | reduce <br> variance inflation              | numeric `predictors`                                  | `vif_max = NULL`                                                                   |

The next sections follow the processing of the responses “vi_numeric”
and “vi_character” through all these functions.

#### `target_encoding_lab()`

- Requirements: numeric `response`.
- Arguments and default values:
  - `encoding_method`: controls

This function requires a numeric `response` to transform categorical
`predictors` to numeric. This transformation method allows applying the
same multicollinearity filtering methods to all predictors.

The categorical predictor “koppen_zone” has character values indicating
different climate zones.

``` r
head(unique(df$koppen_zone))
```

The code chunk below creates a little toy data frame with two of these
Koppen groups and the response “vi_numeric”.

``` r
df_toy <- df |> 
  dplyr::select(vi_numeric, koppen_zone) |> 
  dplyr::filter(koppen_zone %in% c("Af", "BSh")) |> 
  dplyr::group_by(koppen_zone) |> 
  dplyr::slice_head(n = 5) |> 
  dplyr::ungroup()
```

When introducing “koppen_zone” in `target_encoding_lab()` with the
method “loo” (leave-one-out), the data frame is grouped by the levels of
the predictor, and each value in a group is re-mapped as the average of
the `response` across all other group cases.

``` r
df_toy <- target_encoding_lab(
  df = df_toy,
  response = "vi_numeric",
  predictors = "koppen_zone",
  method = "loo",
  overwrite = TRUE,
  quiet = TRUE
)
```

The resulting data frame shows the variable “koppen_zone” re-mapped as
numeric, and ready for a multicollinearity analysis.

When this method is disabled or the response is not numeric,
`cor_select()` becomes much slower (because uses more complex methods to
achieve a solution), and `vif_select()` ignores all categorical
predictors.

#### `preference_order()`

This function ranks predictors by the strength of their association with
the response. The ranked predictors are considered by `cor_select()` and
`vif_select()` to help preserve as many relevant predictors as possible
during multicollinearity filtering.

#### `cor_select()`

#### `vif_select()`

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
“soil_temperature_max” was higher than `vif_max`, and the one with lower
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
  quiet = FALSE
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
