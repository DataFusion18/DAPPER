
     ___   _   ___ ___ ___ ___ 
    |   \ /_\ | _ \ _ \ __| _ \
    | |) / _ \|  _/  _/ _||   /
    |___/_/ \_\_| |_| |___|_|_\

(Data Assimilation Platform in Python for Experimental Research)

![EnKF - Lorenz'63](./data/figs/anims/Lor63_ens_anim_2.gif)

* DAPPER lets you benchmark data assimilation (DA) methods
* The typical set-up is a "twin experiment", where you
  * specify a
      * dynamic model 
      * observational model 
  * use these to generate a synthetic
      * "truth"
      * and observations thereof
  * assess how different DA methods
      perform in estimating the truth
* Licence: See licence.txt

<!---
* Developed at the Nansen centre. Contributors:
  * Patrick N. Raanes
  * Maxime Tondeur
-->

Installation
------------------------------------------------
Prerequisite: python3.5 with scipy.

Then, download DAPPER, and run:

    > python -i benchmarks.py

Methods
------------

Method name                     | Literature RMSE results reproduced
------------------------------- | ---------------------------------------
EnKF (Stoch., ETKF, DEnKF)      | sakov'2008 ("deterministic")
EnKF-N                          | bocquet'2012 ("combining"), bocquet'2015 ("expanding")
EnKS, EnRTS                     | raanes'2015 ("EnRTS and EnKS")
Iterative versions of the above | sakov'2012 ("an iterative"), TODO: bocquet'2014
Extended KF                     | raanes'2016 thesis
Particle filter (bootstrap)     | "
3D-Var                          | "
Climatology                     | "
TODO: Sqrt model noise methods  | raanes'2014 ("sqrt model noise")

#### How to add a new method
Just add it to `da_algos.py`, using the others in there as templates.
(TODO: split `da_algos.py` into multiple files.)

Models
------------

Model name    | Linear? | Phys.dim. | State len. | # of + Lyap    | Thanks to
-----------   | ------- | --------- | ---------- | -------------- | ----------
Linear Advect | Yes     | 1D        |  1000      |  51            | Evensen
Lorenz63      | No      | 0D        |  3         |  2+            | Lorenz/Sakov
Lorenz95      | No      | 1D        |  40        |  13+           | "
LorenzXY      | No      | 2x 1D     |  256 + 8   |  ca 13         | Lorenz/Raanes
MAOOAM        | No      | 2x 1D     |  36        |  ?             | Tondeur / Vannitsen
Shallow Water | No      | 2D        |  xx        |  ?             | Gharamti


#### How to add a new model
* Make a new dir: DAPPER/mods/**your_mod**
* See other examples, e.g. DAPPER/mods/Lorenz63/sak12.py
* Make sure that your model (and obs operator) support
    * **ensemble input**
      (hence forecast parallelization is in users's hands)
    * should not modify in-place.
    * the same applies for the observation operator/model
* To begin with, test whether the model works
    * on 1 realization
    * on several realizations (simultaneously)
* Thereafter, try assimilating using
    * a big ensemble
    * a safe (e.g. 1.2) inflation value
    * small initial perturbations
      (big/sharp noises might cause model blow up)
    * very large observation noise (free run)
    * or very small observation noise (perfectly observed system)

<!---
* Nice read: "Perfect Model Experiment Overview" section of
    http://www.image.ucar.edu/DAReS/DART/DART_Starting.php

-->


Additional features
------------------------------------------------
Many
* visualizations 
* diagnostics


Also:
* Highly modular.
* Balance between efficiency and readability.
* Consistency checks (e.g. time).

<!---
E.g. Lorenz-96 uses native vectorization (i.e. fast numpy),
  but no parallelization.
-->

<!---
For -N stuff, compared to Boc's code, DAPPER
* uses more matrix decompositions (efficiency),
* allows for non-diag R.
-->

####Sugar:
* Progressbar
* Confidence interval on times series (e.g. rmse) with
	* automatic correction for autocorrelation 
	* significant digits printing
* X-platform random number generator
* Chronology/Ticker
* CovMat class (input flexibility, / overloading, lazy eval)
* Live plotting with on/off toggle
* Intelligent defaults (e.g. plot duration estimated from acf,
    axis limits esitmated from percentiles)


What it can't do
------------------------------------------------
* Store full ensembles (could write to file)
* Run different DA methods concurrently (i.e. step-by-step)
     allowing for online (visual or console) comparison
* Time-dependent noises and length changes in state/obs
     (but it does support autonomous f and h)
* Non-uniform time sequences


Implementation choices
------------------------------------------------
* Uses python version >= 3.5
* On-line vs off-line stats and diagnostics
* NEW: Use N-by-m ndarrays. Pros:
    * Python default
        * speed of (row-by-row) access, especially for models
        * ordering of random numbers
    * numpy sometimes returns ndarrays even when input is matrix
    * works well with ens space formulea,
        * e.g. 
        * yields beneficial operator precedence without (). E.g. dy@Ri@Y.T@Pw
    * Bocquet's choice
    * Broadcasting
    * Avoids reshape's and asmatrix
    * Fewer indices: [k,:] becomes [k]
* OLD: Use m-by-N matrix class. Pros:
    * Litterature uses m-by-N
    * Matrix class allowss desired broadcasting
    * Deprecated: syntax (* vs @)


Alternatives
------------------------------------------------
##### Big
* DART        (NCAR)
* SANGOMA     (Liege/CNRS/Nersc/Reading/Delft)
* PDAF        (Nerger)
* ERT         (Statoil)
* OpenDA      (TU Delft)
* PyOSSE      (Edinburgh)
* ?           (DHI)

##### Medium
* DAPPER      (Raanes)
* FilterPy    (R. Labbe)
* PyIT        (CIPR)
    
##### Small
* Datum       (Raanes)
* EnKF-Matlab (Sakov)
* IEnKS code  (Bocquet)
* pyda        (Hickman)


TODO
------------------------------------------------
* Localization
* add_noise()
* 1D model preserving some quantity (aside from L95)
* 2D model
* KdVB model? (Zupanski 2006)
* Doc models

* Should (direct) observations return copy? e.g. x[:,obsInds].copy()
* Take advantage of pass-by-ref
* Decide on conflicts np vs math vs sp

* Truncate SVD at 95 or 99% (evensen)
* unify matrix vs array (e.g. randn)
* vs 1d array (e.g. xx[:,0] in L3.dxdt)
* prevent CovMat from being updated


"Outreach"
---------------
* http://stackoverflow.com/a/38191145/38281
* http://stackoverflow.com/a/37861878/38281
