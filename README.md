# Multi-Omics Factor Analysis v2 (MOFA+)

MOFA is a factor analysis model that provides a **general framework for the integration of multi-omic data sets** in an unsupervised fashion.  
Intuitively, MOFA can be viewed as a versatile and statistically rigorous generalization of principal component analysis (PCA) to multi-omics data. Given several data matrices with measurements of multiple ‘omics data types on the same or on overlapping sets of samples, MOFA infers an **interpretable low-dimensional data representation in terms of (hidden) factors**. These learnt factors represent the driving sources of variation across data modalities, thus facilitating the identification of cellular states or disease subgroups.  

In MOFA v2 (MOFA+) we added the following improvements:
* Fast Stochastic variational inference framework amenable to GPU computations: enables inference with very large data sets
* Multi-group functionality: intuitively, this breaks the assumption of independent samples and allows inference across multiple groups, where groups are predefined sets of samples (i.e. different conditions, batches, cohorts, etc.).


For more details you can read our papers: 
- MOFA v1: http://msb.embopress.org/cgi/doi/10.15252/msb.20178124
- MOFA v2: XXX

<p align="center"> 
<img src="images/logo.png" style="width: 50%; height: 50%"/>​
</p>


## Installation

The core of MOFA is implemented in Python. However, the whole procedure can be run with R and we provide the downstream analysis functions only in R.

### Python dependencies 

Python dependencies can be installed using pip (from the Unix terminal)

```r
pip install mofapy2
```

Alternatively, they can be installed from R itself using the reticulate package:

```r
library(reticulate)
py_install("mofapy2", envname = "r-reticulate", method="auto")
```

### MOFA2 R package

Can be installed using R:

```r
devtools::install_github("bioFAM/MOFA2", build_opts = c("--no-resave-data"))
```

--------------

### Using Docker image

You can build an image with `mofa2py` python library and `MOFA2` R package using the provided Dockerfile:

```
docker build -t mofa2 .
```

You will then be able to use R or Python from the container. 

```
docker run -ti --rm -v $DATA_DIRECTORY:/data mofa2 R
#                   ^
#                   |
#                    use `-v` to map a folder on your machine to a container directory
```

The command above will launch R with MOFA2 and its dependencies installed while mounting `$DATA_DIRECTORY` to the container.

## Usage

### Step 1: Prepare the data

- Data processing
- Filtering
- Regressing out technical variation


If you work with single-cell data, MOFA+ comes with interfaces to build and train a model directly from objects commonly used for scRNA-seq data analysis, namely [AnnData](https://github.com/theislab/anndata) ([scanpy](https://github.com/theislab/scanpy)) in Python and [Seurat](https://github.com/satijalab/seurat) in R.

See the vignette XXXX and the documentation for details


### Step 2: Fitting the model

### Step 3: Downstream analysis
- Disentangling variance explained across views and groups
- Visualisation of factors
- Visualisation of loadings
- Transfer learning and imputation
	

Downstream analysis: disentangle the variability between omics

## Tutorials/Vignettes
We currently provide the following vignettes:

* **Data processing and creation of MOFA object**: bazzz
* **Integration of heterogeneous scRNA-seq data**: foo.
* **Integration of heterogeneous DNA methylation data**: bar.
* **Integration of single-cell multi-modal data:**: baz.
* **Transfer learning and imputation:**: baz.
* **Model selection and robustness with simulated data**: bazz



## Frequently asked questions on the data processing

**(Q) How do I normalise the data?**  
Proper normalisation of the data is critical for the model to work. First, one needs to remove library size effects. For count-based data such as RNA-seq or ATAC-seq we recommend size factor normalisation + variance stabilisation. For microarray DNA methylation data, make sure that samples have no differences in the average intensity. If this is not done correctly, the model will learn a very strong Factor 1 that will capture this variability, and more subtle sources of variation will be harder to identify.  

**(Q) Should I remove batch effects?**  
Yes
We have implemented a function called `regress_covariates` that allows the user to regress out a covariate using linear models. See the documentation and the CLL vignette for examples.

**(Q) How many samples do I need?**  
Factor Analysis models are only useful with large sample sizes, let's say more than 15. With few samples you will detect very few relevant factors 

**(Q) Should I remove undesired sources of variability (i.e. batch effects) before fitting the model?**  
Yes, if you have clear technical factors, we strongly encourage to regress it out a priori using a simple linear model. The reason for this is that the model will "focus" on the huge variability driven by the technical factors, and smaller sources of variability could be missed.
You can regress out known covaraites using the function `regressCovariates`. See the corresponding documentation and the CLL vignette for details.

**(Q) Should I do any filtering to the input data?**  
You must remove features with zero variance and ideally also features with low variance, as they can cause numerical issues in the model. In practice we generally select the top N most variable features per assay

**(Q) My data sets have different dimensionalities, does this matter?**  
Yes, this is important. Bigger data modalities will tend to be overrepresent in the MOFA model. It is good practice to filter features (based for example on variance, as lowly variable features provide little information) in order to have the different dimensionalities within the same order of magnitudes. If this is unavoidable, take into account that the model has the risk of missing (small) sources of variation unique to the small data set.

**(Q) What input formats are allowed?**  
XXXX

**(Q) Does MOFA handle missing values?**  
Yes! and there is no hidden imputation step, it simply ignores them. Matrix factorisation models are known to be very robust to the presence of missing values!

**(Q) How do I define groups?**  
XXXX

**(Q) How do I define views?**  
XXXX


## Frequently asked questions on the software

**(Q) I get one of the following errors when running MOFA:**  
```
AttributeError: 'module' object has no attribute 'core.entry_point

Error in py_module_import(module, convert = convert) :
 ModuleNotFoundError: No module named 'mofapy'
```
First thing: restart R and try again. If the error still holds, this means that either:  
(1) you did not install the mofa Python package (see instructions above).
(2) you have multiple python installations and R is not detecting the correct one where mofa is installed. You need to find out the right Python interpreter, which usually will be the one you get when running `which python` in the terminal. You can test if the mofa packaged is installed by running INSIDE python: `import mofapy`.  
Once everything is figured out, specify the following at the beginning of your R script:
```
library(reticulate)
use_python("YOUR_PYTHON_PATH", required=TRUE)
```
You can also use `use_conda` instead of `use_python` if you work with conda environments. Read more about the [reticulate](https://rstudio.github.io/reticulate/) package and [how it integrates Python and R](https://rstudio.github.io/reticulate/articles/versions.html)

**(Q) I get the following error when installing the R package:**  
```
ERROR: dependencies 'XXX', 'YYYA' are not available for package 'MOFA'
```
You probably tried to install them using `install.packages()`. These packages should be installed from Bioconductor.

**(Q) I hate R, can I do MOFA only with Python?**  
You can use Python to train the model, see [this template script](https://github.com/bioFAM/MOFA2/blob/master/template_run.py). However, we currently do not provide downstream analysis functions in Python. We strongly recommend that you use our MOFA2 R package for this.


## Frequently asked questions on the model options

**(Q) How many factors should I learn?**  
Similar to other latent variable models, this is a hard question to answer. It depends on the data set and the aim of the analysis. If you want to get an overview on the major sources of variability then use a small number of factors (K<=10). If you want to capture small sources of variability, for example to do imputation or eQTL mapping, then go for a large number of factors (K>25).

**(Q) Can MOFA automatically learn the number of factors?**  
Yes, but the user needs to specify a minimum value of % variance explained. Then, MOFA will actively remove factors (during training) that explain less than the specified amount of variance.
If you have no idea on what to expect, it is better to start with a fixed number of factors and set the % variance threshold to 0.

**(Q) Can I put known covariates in the model?**  
We extensively tested this functionality and it was not yielding good results. The reason is that covariates are usually discrete labels that do not reflect the underlying molecular biology. For example, if you introduce age as a covariate, but the actual age is different from the “molecular age”, the model will simply learn a new factor that corresponds to this “latent” molecular age, and it will drop the covariate from the model.  
We recommend that you learn the factors in a completely unsupervised manner and then relate them to the biological covariates (see vignettes). If your covariate of interest is an important driver of variability, do not worry, MOFA will find it! 

**(Q) The weights have different values between runs. Is this expected?**  
This is normal and it happens because of two reasons. The first one is that the model does not always converge to the same exact solution (see below in the FAQ), although different model instances should be pretty similar. The second reason is that factor analysis models are rotation invariant. This means that you can rotate your factors and your weights and still find the same solution. This implies that the signs of the weight or the factors can NOT be compared across trials, only within a trial.

**(Q) What data modalities can MOFA cope with?**  
* Continuous data: modelled using a gaussian likelihood
* Binary data: modelled using a bernoulli likelihood
* Count data: using a poisson likelihood.
Importantly, the use of non-gaussian likelihoods require further approximations and are not as accurate as the gaussian likelihood. Hence, if your data can be safely transformed to match the gaussian likelihood assumptions, this is ALWAYS recommended. For example RNA-seq data is expected to be normalised and modelled with a gaussian distribution, do not input the counts directly.

**(Q) The model does not converge smoothly, and it oscillates between positive and negative deltaELBO values**  
First, check that you are using the right likelihood model (see above). Second, make sure that you have no features or samples that are full of missing values. Third, check that you have no features with zero (or very little) variance. If the problem does not disappear, please contact us via mail or the Slack group.


**(Q) Does MOFA always converge to the same solutions?**  
No, as occurs in most complex Bayesian models, they are not guaranteed to always converge to the same (optimal) solution.
In practice, however, we observed that the solutions are highly consistent, particularly for strong factors. However, one should always assess the robustness and do a proper model selection. For this we recommend to train the model multiple times and check the robustness of the factors across the different solutions. For downstream analysis a single model can be chosen based on the best value of the Evidence Lower Bound (ELBO). We provide functions for these two steps, which are explained in the vignette *Integration of simulated data* (`vignette("MOFA_example_simulated")`).


## Frequently asked questions on the downstream analysis

**(Q) How do I interpret the weights?**
XXX

**(Q) How do I interpret the factors?**
XXX

**(Q) How can I do Gene Set Enrichment Analysis?**  
First, you need to create your binary gene set matrix where rows are feature sets and columns are features (genes). We have manually processed some of Reactome and MSigDB gene sets for mouse and human. Contact us if you would like to use the data.  
Then, you will have to choose a local statistic per feature (the loading, by default), a global statistic per pathway (average loading, by default), and a statistical test. The most trustworthy one is a permutation test with a long number of iterations, but this is slow and a fast parametric tests is also available. However, note that it tends to inflate the p-values due to the correlation structure between related genes (see for example [Gatti2010](https://bmcgenomics.biomedcentral.com/articles/10.1186/1471-2164-11-574)).

## Contact
The package is maintained by Ricard Argelaguet (ricard@ebi.ac.uk) and Danila Bredikhin (danila.bredikhin@embl.de). Please, reach us for problems, comments or suggestions. You can also contact us via a Slack group where we provide quick and personalised help, [this is the link](https://join.slack.com/t/mofahelp/shared_invite/enQtMjcxNzM3OTE3NjcxLWNhZmM1MDRlMTZjZWRmYWJjMGFmMDkzNDBmMDhjYmJmMzdlYzU4Y2EzYTI1OGExNzM2MmUwMzJkZmVjNDkxNGI).

