# ALAD Spirometric Classification in R

This repository contains an R implementation of the 2025/2026 International Society for Heart and Lung Transplantation (ISHLT) spirometric definition of [Acute Lung Allograft Dysfunction (ALAD)](https://ucsfgreenlandlab.github.io/ALAD/).

The code was used to classify ALAD in UCSF and Lung Transplant Outcomes Group (LTOG) datasets and implements the consensus definitions evaluated in:

> Greenland JR, Shaver CM, Pandya K, et al. *Acute lung allograft dysfunction predicts lung allograft failure: Construct validity of the 2026 International Society for Heart and Lung Transplantation spirometric definition.* [Journal of Heart and Lung Transplantation](https://doi.org/10.1016/j.healun.2026.01.006). 2026.

## Overview

The script provides a function, `alad.fev.bl()`, that classifies a spirometry measurement according to ALAD criteria using longitudinal FEV1 measurements.

For a specified spirometry date, the algorithm:

1. Identifies the maximum FEV1 within a baseline window (default 180 days).
2. Determines whether the current FEV1 has declined by at least 10% from that baseline.
3. If ALAD criteria are met, evaluates subsequent spirometry measurements to classify the event as:

   * ALAD - Recovered
   * ALAD - Persistent
   * ALAD - Progressive
   * ALAD - Rapidly Progressive
   * ALAD - No follow up FEV1s

Default parameters follow the ISHLT consensus definition:

* Baseline window: 180 days
* Follow-up window: 90 days
* FEV1 decline threshold: 10%
* Rapid progression window: 30 days

## Function

### `alad.fev.bl()`

```r
alad.fev.bl(
  post.tx.days,
  fev1s,
  target.day,
  baseline.window = 180,
  f.u.window = 90,
  percent.drop = 10
)
```

### Arguments

| Argument          | Description                                                                     |
| ----------------- | ------------------------------------------------------------------------------- |
| `post.tx.days`    | Numeric vector of days post-transplant corresponding to spirometry measurements |
| `fev1s`           | Numeric vector of FEV1 measurements                                             |
| `target.day`      | Day post-transplant at which ALAD status is determined                          |
| `baseline.window` | Number of days before the target day used to identify the maximum baseline FEV1 |
| `f.u.window`      | Number of days after the target day used to classify recovery or progression    |
| `percent.drop`    | Percent decline in FEV1 required to meet ALAD criteria                          |

### Returns

One of:

* `"No prior FEV1"`
* `"Not ALAD"`
* `"ALAD - Recovered"`
* `"ALAD - Persistent"`
* `"ALAD - Progressive"`
* `"ALAD - Rapidly Progressive"`
* `"ALAD - No follow up FEV1s"`

## Example Dataset Structure

The script assumes a spirometry dataset containing one row per pulmonary function test.

Example columns:

| Column         | Description               |
| -------------- | ------------------------- |
| `ltog_id`      | Participant identifier    |
| `post.tx.days` | Days post-transplant      |
| `pft_fev1`     | FEV1 measurement (liters) |

## Example Usage

The example below applies ALAD classification to every spirometry measurement in a dataset using parallel processing.

```r
sapply(
  c("foreach", "dplyr", "magrittr", "doParallel", "tidyr"),
  require,
  character.only = TRUE
)

numCores <- detectCores()

cl <- makeCluster(numCores - 1)
registerDoParallel(cl)

ALAD.assignments <- foreach(
  i = 1:nrow(pfts),
  .combine = c,
  .packages = c("magrittr", "dplyr")
) %dopar% {

  ltog.wrapper(
    pfts$ltog_id[i],
    pfts$post.tx.days[i],
    baseline.window = 180
  )

}

stopCluster(cl)
```

The resulting vector, `ALAD.assignments`, contains the ALAD classification corresponding to each spirometry record in `pfts`.

## Research Use

This software is intended for research use only and is not intended for clinical decision making.

Users should independently validate results and ensure consistency with current ISHLT definitions and local clinical practice before any clinical application.

## License

Copyright © 2025 John Greenland.

Redistribution and use in source and binary forms, with or without modification, are permitted under the terms provided in the source file. See the LICENSE file for details.
