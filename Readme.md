# REGHDFE: Linear Regressions With Multiple Fixed Effects

## Recent Updates

* **version 4.1 28feb2017**: entirely rewriten in Mata
    - 3-10x faster thanks to [`ftools`](https://github.com/sergiocorreia/ftools/) package (use it if you have large datasets!)
    - Several minor bugs have been fixed, in particular some that did not allow complex factor variable expressions.
    - `reghdfe` is now written entirely as a Mata object. For an example of how to use it to write other programs, see [here](https://github.com/sergiocorreia/ivreg2_demo/blob/master/hdfe_example.do)
    - Additional estimation options are now supported, including [LSMR](http://web.stanford.edu/group/SOL/software/lsmr/) and [pruning of degree-1 vertices](https://arxiv.org/abs/1301.6628).
- **version 4.2 06apr2017**: fix numerical accuracy issues ([bugfixes](https://github.com/sergiocorreia/reghdfe/commit/79a8c0134ea089f93811be404a5405c38b5a596a))
- **version 4.3 07jun2017**: speed up fixed slopes (precompute `inv(xx)`)

####  Things to be aware of:

- `reghdfe` now depends on the `ftools` package (and `boottest` for Stata 12 and older)
- IV/GMM is not done directly with `reghdfe` but through `ivreg2`. See [this port](https://github.com/sergiocorreia/ivreg2_demo/), which adds an `absorb()` option to `ivreg2`.
- If you use commands that depend on reghdfe (`regife`, `poi2hdfe`, `ppml_panel_sg`, etc.), check that they have been updated before using the new version of reghdfe.
- Some options are not yet fully supported. They include `cache` and `groupvar`.
- The previous stable release (3.2.9 21feb2016) can be accessed with the `old` option

#### Future/possible updates

- Reduce memory usage. This is done by loading and processing the data by parts into Mata.
- Improve inference with more robust VCE options (`avar` package)
- Add back group3hdfe option

## Citation

`reghdfe` implements the estimator described in [Correia (2017)](http://scorreia.com/research/hdfe.pdf).
If you use it, please cite either the paper and/or the command's RePEc citation:

```bibtex
@TechReport {Correia2017:HDFE,
  Author = {Correia, Sergio},
  Title = {Linear Models with High-Dimensional Fixed Effects: An Efficient and Feasible Estimator},
  Note = {Working Paper},
  Year = {2016},
}
```

> Correia, Sergio. 2017. "Linear Models with High-Dimensional Fixed Effects: An Efficient and Feasible Estimator"
> Working Paper.
> http://scorreia.com/research/hdfe.pdf

> Sergio Correia, 2017. *reghdfe: Stata module for linear and instrumental-variable/GMM regression absorbing multiple levels of fixed effects.*
> https://ideas.repec.org/c/boc/bocode/s457874.html


## Install:

To find out which version you have installed, type `reghdfe, version`.

`reghdfe` 4.x is not yet in SSC. To quickly install it and all its dependencies, copy/paste these lines and run them:

```stata
cap ado uninstall moresyntax
cap ado uninstall ftools
net install ftools, from("https://github.com/sergiocorreia/ftools/raw/master/src/")

cap ado uninstall reghdfe
net install reghdfe, from("https://github.com/sergiocorreia/reghdfe/raw/master/src/")

if (c(version)<13) cap ado uninstall boottest
if (c(version)<13) ssc install boottest

cap ssc install moremata
```

To run IV/GMM regressions, run these lines:

```stata
cap ado uninstall ivreg2hdfe
cap ssc install ivreg2
net install ivreg2hdfe, from("https://github.com/sergiocorreia/ivreg2_demo/raw/master/")
```

To install the stable version from SSC (3.x):

```stata
cap ado uninstall reghdfe
ssc install reghdfe
```


------------------------------------------

`reghdfe` is a [Stata](http://www.stata.com/) package that estimates linear regressions with multiple levels of fixed effects. It works as a generalization of the built-in `areg`, `xtreg,fe` and `xtivreg,fe` regression commands. It's objectives are similar to the R package [lfe](http://cran.r-project.org/web/packages/lfe/index.html) by Simen Gaure and to the Julia package [FixedEffectModels](https://github.com/matthieugomez/FixedEffectModels.jl) by Matthieu Gomez (beta). It's features include:

- A novel and robust algorithm that efficiently absorbs multiple fixed effects. It improves on the work by [Abowd *et al*, 2002](https://ideas.repec.org/p/cen/tpaper/2002-06.html), [Guimaraes and Portugal, 2010](https://ideas.repec.org/a/tsj/stataj/v10y2010i4p628-649.html) and [Simen Gaure, 2013](http://www.sciencedirect.com/science/article/pii/S0167947313001266). This algorithm works particularly well on "hard cases" that converge very slowly (or fail to converge) with the existing algorithms.
- Extremely fast compared to similar Stata programs. 
  - With one fixed effect and clustered-standard errors, it is 3-4 times faster than `areg` and `xtreg,fe` (see [benchmarks](./misc/Benchmarks/areg_xtreg.log.txt)). Note: speed improvements in Stata 14 have reduced this gap. 
  - With multiple fixed effects, it is at least an order of magnitude faster that the alternatives (`reg2hdfe`, `a2reg`, `felsdvreg`, `res2fe`, etc.). Note: a recent paper by [Somaini and Wolak, 2015](http://web.stanford.edu/group/fwolak/cgi-bin/sites/default/files/jem-2014-0008.pdf) reported that `res2fe` was faster than `reghdfe` on some scenarios (namely, with only two fixed effects, where the second fixed effect was low-dimensional). This is no longer correct for the current version of `reghdfe`, which outperforms `res2fe` even on the authors' benchmark (with a low-dimensional second fixed effect; see the [benchmark results](./misc/Benchmarks/res2fe.log.txt) and the Stata [code](./misc/Benchmarks/res2fe.do)).
- Allows two- and multi-way clustering of standard errors, as described in [Cameron *et al* (2011)](http://amstat.tandfonline.com/doi/abs/10.1198/jbes.2010.07136)
- Allows an extensive list of robust variance estimators (thanks to the [avar](https://ideas.repec.org/c/boc/bocode/s457689.html) package by Kit Baum and Mark Schaffer).
- Works with instrumental-variable and GMM estimators (such as two-step-GMM, LIML, etc.) thanks to the [ivreg2](https://ideas.repec.org/c/boc/bocode/s425401.html) routine by Baum, Schaffer and Stillman.
- Allows multiple heterogeneous slopes (e.g. a separate slope coefficients for each individual).
- Supports all standard Stata features:
  - Frequency, probability, and analytic weights.
  - Time-series and factor variables.
  - Fixed effects and cluster variables can be expressed as factor interactions, for both convenience and speed (e.g. directly using `state#year` instead of previously using `egen group` to generate the state-year combination).
  - Postestimation commands such as `predict` and `test`.
- Allows precomputing results with the `cache()` option, so subsequent regressions are faster.
- If requested, saves the point estimates of the fixed effects (*caveat emptor*: these fixed effects may not be consistent nor identifiable; see the Abowd paper for an introduction to the topic).
- Calculates the degrees-of-freedom lost due to the fixed effects (beyond two levels of fixed effects this is still an open problem, but we provide a conservative upper bound).
- Avoids common pitfalls, by excluding singleton groups (see [notes](scorreia.com/software/reghdfe/nested_within_cluster.pdf)), computing correct within- adjusted-R-squares ([see initial discussion](http://www.statalist.org/forums/forum/general-stata-discussion/general/1290416-anyone-knows-what-is-an-adjusted-within-r2)), etc.

## Author

[Sergio Correia](http://scorreia.com)
<br>Board of Governors of the Federal Reserve
<br>Email: sergio.correia@gmail.com


## Acknowledgments

This package wouldn't have existed without the invaluable feedback and contributions of Paulo Guimaraes,
Amine Ouazad, Mark E. Schaffer, Kit Baum and Matthieu Gomez.
Also invaluable are the great bug-spotting abilities of many users.


## Contributing

Contributors and pull requests are more than welcome.
There are a number of extension possibilities, such as estimating standard errors for the fixed effects using bootstrapping,
exact computation of degrees-of-freedom for more than two HDFEs, and further improvements in the underlying algorithm.
