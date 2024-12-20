# Scenario Modeling Hub - Process Data

This folder contains example files about data processing examples done in SMH.
The "output-processed" files are generated using the "model-output/" files for
each round. 

## Workflow process

### Calculating Quantiles, Peak Targets and Cumulative Value

If the submissions files have `sample` output type specified via the 
`stochastic_run` and `run_grouping` columns, the `output_type_id` columns for 
this output type will be re-generated by:

- concatenating the value into the two into one: 
`[run_grouping]-[stochastic_run]`
- transform the new value as a factor to assign a unique numeric to each 
unique value 
- transform the factor into numeric to only keep numeric information of the 
factor
- remove the `stochastic_run` and `run_grouping` columns

##### For the weekly cumulative/incident target

Cumulative samples is calculated by each task id (except horizon) and output 
type id group (for example: by scenario, target, location, age group, 
output type_id.) by using samples for the associated incident targets.
Use the function `“calc_sum(.SD)”` from R package `data.table` to calculate 
the cumulative value by imputed group

The quantiles is also calculated by each task id group (for example: by 
scenario, target, location, age group)

##### For the peak target:

- Calculate peak timing and size by target, location, scenario, age group on 
  the complete time series. (take the maximum peak `value` and `horizon` 
  information)
    - If a projection has multiple peaks with the same value, only the first
    one will be selected
    - If a projection projects the same value for the complete time series, the
    projection will be excluded from the calculation.
2. Calculate the distributions across all individual trajectories by target,
location, scenario, team-model and complete time series
3. Calculate the Average probability of peak across the time frame:

For example:
if we have: 
|        |Week 1|Week 2|Week 3|
|:------:|:----:|:----:|:----:|
|Sample 1 |  0   |   1  |  0   |
|Sample 2 |  0   |   1  |  0   |
|Sample 3 |  1   |   0  |  0   |
|Sample 4 |  0   |   1  |  0   |
|Average  |1/4   | 3/4  |  0/4 |
|Cum. Avrg|1/4   | 4/4  |  4/4 |


### Calculating the ensembles:     
 
- Calculate the "Ensemble": The ensemble is the weighted median of each 
quantile by quantiles and task id (scenario, location, target, horizon, 
age group)
  
- Calculating the "Ensemble_LOP" and "Ensemble_LOP_untrimmed": The LOP ensemble 
projection is calculated by averaging cumulative probabilities of a given value 
across submissions. 
  - LOP: For each value, the highest and lowest probabilities are given zero 
  weight and the remaining are weighted equally.  
  - LOP untrimmed: all value are included
From the resulting distribution, medians and uncertainty intervals are derived. 
Ensemble projection includes only those submissions that reported quantiles for 
their targets. The trimmed linear opinion pool ensemble (Ensemble LOP) 
is estimated by 
1) simulating N forecasts from each of the component models, where N is 
selected proportionate to the weight of the model, 
2) linearly pooling those forecasts into a multi-modal distribution, and 
3) truncating the pooled distribution at the lower and upper bounds, by an 
amount equivalent to 1 over the sum of the model weights. 

By default a weight of 1 is assigned for each submission, 0.5 if a team makes 
two submissions
	
