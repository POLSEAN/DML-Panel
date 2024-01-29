# XTDML
The `XTDML` package implements double machine learning (DML) for static partially linear regression (PLR) models with fixed effects as in [Clarke and Polselli (2023)](https://arxiv.org/abs/2312.08174).
The package buids on 'DoubleML' by Chernozhukov et al. (2018).

The package allows for the choice of three approaches for handling unobserved individual heterogeneity:
1. Mundlak (1978)'s device or correlated random effects (CRE)
2. Approximation approach that requires to transform the variables (i.e, time-demeaning or first-differencing)
3. Hybrid approach that uses Mundlak's device to learn the nuisance parameters and than transforms the variables and nuisance predictions

The current version can be installed via devtools:
```
library(devtools)
install_github("POLSEAN/XTDML")
```
## Sample code
Below we report some illustrative examples of the use of the code with the three approaches. Simulation data is generated following DGP3 in [Clarke and Polselli (2023)](https://arxiv.org/abs/2312.08174) and is uploaded in the folder `/data`.

### Example for CRE
```
# load data
df = read.csv("https://raw.githubusercontent.com/POLSEAN/XTDML/main/data/dgp4_cre_short.csv")

# set up data
x_cols <- paste0("x", 1:30)
xbar_cols <- paste0("m_x", 1:30)

# set up data for DML procedure
obj_dml_data = dml_cre_data_from_data_frame(df,
                            x_cols = x_cols,  y_col = "y", d_cols = "d",
                            xbar_cols = xbar_cols, dbar_cols = "m_d",                                                 
                            cluster_cols = "id")

# lasso w/t dictionary
learner = lrn("regr.cv_glmnet", s="lambda.min")
ml_m = learner$clone()
ml_l = learner$clone()

ml_mbar = learner$clone()
ml_lbar = learner$clone()

# estimation with CRE with non-separable model
dml_obj = dml_cre_plr$new(obj_dml_data, ml_l = ml_l, ml_m = ml_m,
                          ml_lbar = ml_lbar, ml_mbar = ml_mbar,
                          score="orth-PO", model = "non-separable")
dml_obj$fit()
```

### Example for Approximation
```
obj_dml_data = dml_approx_data_from_data_frame(df,
                            x_cols = x_cols,  y_col = "y", d_cols = "d",
                            cluster_cols = "id")

# lasso w/t dictionary
learner = lrn("regr.cv_glmnet", s="lambda.min")
ml_m = learner$clone()
ml_l = learner$clone()

ml_mbar = learner$clone()
ml_lbar = learner$clone()

# estimation with within-group transformation
dml_obj = dml_approx_plr$new(obj_dml_data, ml_l = ml_l, ml_m = ml_m,
                          ml_lbar = ml_lbar, ml_mbar = ml_mbar,
                          score="orth-PO")
dml_obj$fit()
```

### Example for Hybrid
```
# load data
df = read.csv("https://raw.githubusercontent.com/POLSEAN/XTDML/main/data/dgp4_cre_short.csv")

# set up data
x_cols <- paste0("x", 1:30)
xbar_cols <- paste0("m_x", 1:30)

# set up data for DML procedure
obj_dml_data = dml_hybrid_data_from_data_frame(df,
                            x_cols = x_cols,  y_col = "y", d_cols = "d",
                            xbar_cols = xbar_cols, dbar_cols = "m_d",                                                 
                            cluster_cols = "id")

# lasso w/t dictionary
learner = lrn("regr.cv_glmnet", s="lambda.min")
ml_m = learner$clone()
ml_l = learner$clone()

ml_mbar = learner$clone()
ml_lbar = learner$clone()

# estimation with within-group transformation
dml_obj = dml_hybrid_plr$new(obj_dml_data, ml_l = ml_l, ml_m = ml_m,
                          ml_lbar = ml_lbar, ml_mbar = ml_mbar,
                          score="orth-PO", model = "wg")
dml_obj$fit()
```

## References
Chernozhukov, V., Chetverikov, D., Demirer, M., Duflo, E., Hansen, C., Newey, W., and Robins, J. (2018). Double/debiased machine learning for treatment and structural parameters. *The Econometrics Journal*, 21(1):C1–C68.

Clarke, P. and Polselli, A. (2023). Double machine learning for static panel models with fixed effects. *arXiv preprint arXiv:2312.08174*.

Mundlak, Y. (1978). On the pooling of time series and cross section data. *Econometrica*, pages 69–85.
