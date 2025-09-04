# FTND Scoring Pipeline for Lifelines

A comprehensive R pipeline for processing and scoring the Fagerström Test for Nicotine Dependence (FTND) data from the Lifelines cohort study.

## Overview

This pipeline processes FTND questionnaire data from Lifelines Wave 3, handling the complex structure of tobacco-type-specific questions and threshold-based completion criteria. It produces standardized FTND scores with proper handling of missing data through weighted imputation.

## Key Features

- **Robust Missing Data Handling**: Weighted imputation for partially complete responses
- **Quality Control**: Comprehensive validation and consistency checks  
- **Flexible Output**: Both RDS and CSV export formats
- **Diagnostic Visualizations**: Distribution plots and summary statistics
- **Tobacco Type Support**: Handles cigarettes, e-cigarettes, cigars, cigarillos, and pipes

## Installation Requirements

```r
# Required R packages
install.packages(c("data.table", "dplyr", "tidyr", "ggplot2", "psych"))

# Load the pipeline
source("ftnd_pipeline.R")
```

## Quick Start

```r
# Run the complete pipeline
ftnd_data <- run_ftnd_pipeline(
  input_file = "/path/to/3a_q_2_results.csv",
  output_dir = "/path/to/output/",
  create_plots = TRUE
)

# Check results
summary(ftnd_data$ftnd_sum_score)
```

## Pipeline Steps

### 1. Data Loading and Preprocessing
- Loads raw Lifelines CSV data
- Selects FTND-related variables
- Handles missing value codes (`$7`, empty strings)
- Converts variables to appropriate numeric format

### 2. Variable Recoding
Transforms Lifelines response format to standard FTND scoring:

| Variable Type | Lifelines Format | FTND Format | Description |
|---------------|------------------|-------------|-------------|
| Binary items | 1=Yes, 2=No | 1=Yes, 0=No | Most FTND questions |
| CPD (frequency) | 1-4 scale | 0-3 scale | Cigarettes per day equivalent |
| Wake-up time | 1-4 scale | 3-0 scale | Reversed: sooner = higher score |

### 3. Threshold Assessment
Participants must answer "Yes" to at least one threshold question to complete the full FTND:
- `ftnd_lifetime_adu_q_01_a`: Regular cigarette use
- `ftnd_lifetime_adu_q_01_b`: Regular e-cigarette use  
- `ftnd_lifetime_adu_q_01_c`: Regular cigar use
- `ftnd_lifetime_adu_q_01_d`: Regular cigarillo use
- `ftnd_lifetime_adu_q_01_e`: Regular pipe use

### 4. Tobacco Type Assignment
For threshold participants, assigns primary tobacco type and corresponding cigarettes-per-day (CPD) score:
- **Cigarettes**: `ftnd_peak_adu_q_03_a`
- **E-cigarettes**: `ftnd_peak_adu_q_03_b`  
- **Cigars**: `ftnd_peak_adu_q_03_c`
- **Cigarillos**: `ftnd_peak_adu_q_03_d`
- **Pipes**: `ftnd_peak_adu_q_03_e`

### 5. FTND Score Calculation

#### FTND Components (6 items total):
1. **CPD** (tobacco-type-specific): 0-3 points (weight: 3)
2. **Time to first use**: 0-3 points (weight: 3)  
3. **Difficult to refrain**: 0-1 points (weight: 1)
4. **Hardest to give up**: 0-1 points (weight: 1)
5. **More in morning**: 0-1 points (weight: 1)
6. **Use when ill**: 0-1 points (weight: 1)

#### Scoring Rules:
- **0 missing items**: Raw sum score (0-10)
- **1-2 missing items**: Weighted imputation: `(observed_sum / personal_max) × 10`
- **3+ missing items**: `NA` (insufficient data)
- **Non-threshold participants**: `NA` (didn't meet completion criteria)

### 6. Quality Control and Validation
- Data consistency checks (smoking status vs. FTND completion)
- Score range validation (0-10)
- Missing data pattern analysis
- Calculation verification

### 7. Export and Visualization
- Saves processed data in RDS and CSV formats
- Generates distribution plots
- Provides summary statistics

## Output Variables

### Key Output Variables:
- `ftnd_sum_score`: Final FTND score (0-10, or NA)
- `tobacco_type`: Primary tobacco type used
- `ftnd_peak`: CPD-equivalent score for primary tobacco type

### Intermediate Variables (in full output):
- `smoking_lifetime_adu_q_1`: Ever smoked (0/1)
- `ftnd_lifetime_adu_q_01`: Primary tobacco during heaviest use (1-5)
- `ftnd_*_adu_q_*`: Individual FTND item responses (recoded)

## Data Quality Expectations

### Typical Lifelines Wave 3 Patterns:
- **Total participants**: ~57,000
- **Never smokers**: ~25,000 (45%)
- **Ever smokers**: ~31,000 (55%)
- **Threshold participants**: ~26,000 (85% of ever smokers)
- **Complete FTND scores**: ~25,000 (95% of threshold participants)

### Common Issues Handled:
- Never smokers who answered FTND questions (~100-200 cases)
- Ever smokers who skipped FTND questions (~50-100 cases)  
- Missing CPD data (~600-700 cases, mainly cigarette smokers)
- Partial FTND completion (~1,000 cases with 1-2 missing items)

## File Structure

```
ftnd_pipeline.R
├── Configuration section (variable names, weights)
├── Data loading functions
├── Preprocessing functions  
├── Threshold identification functions
├── Tobacco type assignment functions
├── Scoring calculation functions
├── Validation functions
├── Export functions
└── Main pipeline function
```

## Customization Options

### Modify Variable Names:
Update the `FTND_VARS` list for different datasets:
```r
FTND_VARS <- list(
  lifetime_smoking = "your_smoking_var",
  threshold_vars = c("your_threshold_vars"),
  # ... etc
)
```

### Adjust Missing Data Tolerance:
Change the threshold for imputation vs. exclusion:
```r
# Current: impute if ≤2 missing, exclude if ≥3 missing
# To change: modify the fifelse() conditions in calculate_ftnd_scores()
```

### Add New Tobacco Types:
Extend `TOBACCO_TYPES` and corresponding CPD variables:
```r
TOBACCO_TYPES <- c("cigarettes", "e_cigarettes", "cigars", 
                   "cigarillos", "pipes", "your_new_type")
```

## Validation and Quality Checks

The pipeline includes several validation steps:

### Automatic Checks:
-  Score range validation (0-10)
-  Missing data consistency
-  Calculation verification
-  Tobacco type assignment accuracy


## Citation  
If you use this code or build on this pipeline in your own work, please cite:
Luo, M., Trindade Pons, V., Pingault, JB., Gillespie, N. A., & van Loo, H. M. Genetic Nurture Effects on Nicotine Dependence and Alcohol Use Disorder.

### References:
- Fagerström, K. (2012). Determinants of tobacco use and renaming the FTND to the Fagerström Test for Cigarette Dependence. *Nicotine & Tobacco Research*, 14(1), 75-78.
- Lifelines Cohort Study: https://www.lifeline
