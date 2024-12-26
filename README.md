# 复现

`./notebooks`：
*注意按照下文作对应修改。如果你的Julia版本较低可能就不需要修改（未经验证）。*
| 文件名                            | 跑通 | 备注                     |
|----------------------------------|--------------|------------------------|
| Figure2.ipynb        |      ✅       | `@time all_CI_Q_Δ = get_ratio_confidence_intervals` 耗时较长，约40min |
| Figure3.ipynb        |     ✅       |   |
| Figure4.ipynb      |      ❌        | 卡在`@time Threads.@threads for i in 1:nsamples`，仍在运行中       |
| Figure5.ipynb         |      ✅        |    |

# 部署

- ✅ Windows10+Julia1.10.4

1. download file form [Zenodo](https://doi.org/10.5281/zenodo.10467234) and put it in `./data/mESC`.
2. add packages.
    ```
    using Pkg
    Pkg.add([
        "IJulia",
        "Distributions",
        "StatsBase",
        "SpecialFunctions",
        "HypergeometricFunctions",
        "LogExpFunctions",
        "Optim",
        "BlackBoxOptim",
        "ForwardDiff",
        "Interpolations",
        "NonlinearSolve",
        "StatsFuns",
        "Optimization",
        "OptimizationOptimJL",
        "OptimizationMOI",
        "Ipopt",
        "NaNMath",
        "PyCall",
        "CairoMakie"
    ])
    #这里用你自己的安装路径，注意使用反斜杠
    ENV["PYTHON"] ="D:/Anaconda3/python.exe"
    Pkg.build("PyCall")
    ```
# 已作修改

1. 在 `Figure` 的构造中，`resolution` 参数已被废弃，应该使用 `size` 参数替代。
  
    例如将
    ```
    f = Figure(resolution = (size_pt[1]*0.55, size_pt[2]*0.95), figure_padding = 1)
    ```
    修改为
    ```
    f = Figure(size = (size_pt[1]*0.55, size_pt[2]*0.95), figure_padding = 1)
    ```
2. `lines!` 不支持 `markersize`属性，因此要其拆分为`lines!`和`scatter!`两部分。
  
    例如将
    ```
    lines!(ax, xs, ys, markersize=2, color=(c1, 0.8), linewidth=1.5, label="MLE fit")
    ```
    修改为
    ```
    lines!(ax, xs, ys, color=(c1, 0.8), linewidth=1.5, label="MLE fit")
    scatter!(ax, xs, ys; markersize=2, color=c1, label="")
    ```
    
3. 将 `rainclouds` 函数拆分为`Axis`对象并使用`rainclouds!`函数调用，来实现`xlabel`和`ylabel`两个参数的正确操作。

    例如将
    ```
    rainclouds(ga[2, 1], vcat(x1, x2), vcat(y1, y2), gap=-0.7,
                ylabel="", 
                xlabel="Burst size ratio", 
                orientation = :horizontal,
                color = vcat(fill(c1, length(y1)), fill(c2, length(y2))),
                cloud_width=0.9, show_median=false, violin_limits=(-Inf, Inf), clouds=violin,
                boxplot_width=0.1, boxplot_nudge=0.0, strokewidth = 0.7, whiskerwidth=0,
                jitter_width=0.2, markersize=1.3, side_nudge=0.12)
    ```
    修改为
    ```
    ax = Axis(ga[2, 1]; xlabel="Burst size ratio", ylabel="")
    rainclouds(ax, vcat(x1, x2), vcat(y1, y2), gap=-0.7,
                orientation = :horizontal,
                color = vcat(fill(c1, length(y1)), fill(c2, length(y2))),
                cloud_width=0.9, show_median=false, violin_limits=(-Inf, Inf), clouds=violin,
                boxplot_width=0.1, boxplot_nudge=0.0, strokewidth = 0.7, whiskerwidth=0,
                jitter_width=0.2, markersize=1.3, side_nudge=0.12)
    ```

## `./notebooks/Figure2.ipynb`

1. 匿名函数的定义应改为
    ```
    Δf = ((x, y) -> @. abs(x - y) / y)
    Δb = ((x, y) -> @. abs(x - y) / y)
    ```

## `./src/dists.jl`

1. 由于版本更新，`HypergeometricFunctions.norm2`应该为``HypergeometricFunctions.norm``

---
# CellCycle-RNAseq

This repository contains the code for the paper
&nbsp;&nbsp;&nbsp; A. Sukys and R. Grima, "Transcriptome-wide analysis of cell cycle-dependent bursty gene expression from single-cell RNA-seq data using mechanistic model-based inference" (2024).

The code is used to perform mechanistic model-based inference on scRNA-seq data for a population of mouse embryonic stem cells (mESCs), where the cell-cycle phase (G1, S or G2/M) and cell age $\theta$ (position along the cell cycle) are known for each cell. The processed dataset used throughout is uploaded on [Zenodo](https://doi.org/10.5281/zenodo.10467234), and is based on the original work by Riba et al. [[1]](#1).

### Structure

- `src` - main Julia code used to build quantitative models of gene expression, perform maximum likelihood estimation and model selection, and compute the confidence intervals using profile likelihood.
- `analysis` - scripts for the mESC dataset-specific analysis, considering cell age-dependent ($\theta$-dependent) and cell age-independent ($\theta$-independent) mechanistic models using both cell-cycle-phase-specific (G1 or G2/M) data and merged (G1 + G2/M) data. In each case, the scripts are used to perform inference, model selection and confidence interval estimation.
- `notebooks` - Jupyter notebooks (written in Julia) used to explore the results and to generate the figures in the paper.
- `data` - directory to save the generated files, such as model-specific fits. 

### References:

<a id="1">[1]</a> Riba, A., Oravecz, A., Durik, M. et al. Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning. Nat Comm 13, 2865 (2022). [https://doi.org/10.1038/s41467-022-30545-8](https://doi.org/10.1038/s41467-022-30545-8).
