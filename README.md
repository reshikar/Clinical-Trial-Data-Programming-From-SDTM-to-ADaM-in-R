# Clinical Trial Data Programming: From SDTM to ADaM in R

**Author:** Reshika Rimal  
**Status:** In progress  
**Tools:** R, admiral, dplyr, lubridate, and xportr

## About this project

This beginner-friendly project explores how clinical trial data are transformed into analysis datasets using R and the pharmaverse ecosystem.

The goal is to build two ADaM datasets:

- **ADSL:** Subject-Level Analysis Dataset
- **ADAE:** Adverse Events Analysis Dataset

The project uses publicly available example data for educational purposes.

## Background

### What is SDTM?

The **Study Data Tabulation Model (SDTM)** provides a standard structure for organizing clinical trial data. Data are separated into datasets called domains.

This project starts with three domains:

| Domain | Description | What a record represents |
|---|---|---|
| DM | Demographics | A participant’s demographic information |
| EX | Exposure | A treatment exposure record, such as a dosing interval |
| AE | Adverse Events | A reported adverse event |

The variable `USUBJID` identifies each participant and connects their records across domains.

An adverse event is an unfavorable medical occurrence during a study. It does not necessarily mean that the study treatment caused the event.

### What is ADaM?

The **Analysis Data Model (ADaM)** provides a standard structure for datasets used in statistical analysis. It supports clear documentation and traceability between source data and analysis results.

Creating ADaM datasets involves deriving variables needed for analysis, such as treatment dates, relative study days, event durations, and analysis flags.

### What is admiral?

**admiral** stands for **ADaM in R Asset Library**. It is an open-source R package that provides reusable functions for developing ADaM datasets.

## Project objectives

### Build ADSL

Create one record per participant containing:

- Participant identifiers and demographic information
- Planned and actual treatment groups
- First and last treatment dates
- A documented safety-population flag

### Build ADAE

Create an adverse-event analysis dataset containing:

- Source adverse-event information
- Relevant subject-level variables from ADSL
- Analysis start and end dates
- Relative start and end days
- Event duration
- Treatment-emergent flags
- First-occurrence flags
- Analysis sequence numbers

### Check and summarize the data

- Verify record uniqueness and subject identifiers.
- Check date order and derivation logic.
- Review missing values and any date/time imputations.
- Summarize participants with treatment-emergent adverse events by treatment group.
- Prepare metadata and explore XPT export.

## Data source

Input datasets are available through the **pharmaversesdtm** R package.

- [Package documentation](https://pharmaverse.github.io/pharmaversesdtm/)
- [GitHub repository](https://github.com/pharmaverse/pharmaversesdtm)

The package contains test datasets sourced from the CDISC pilot project and datasets created for testing. Dataset-specific documentation provides their provenance.

This project uses:

```r
dm <- pharmaversesdtm::dm
ex <- pharmaversesdtm::ex
ae <- pharmaversesdtm::ae
```

## Getting started

### 1. Install the example-data package

Run the following in R or RStudio:

```r
install.packages("pharmaversesdtm")
```

### 2. Load the input datasets

```r
dm <- pharmaversesdtm::dm
ex <- pharmaversesdtm::ex
ae <- pharmaversesdtm::ae
```

### 3. Explore the data

```r
# Number of rows and columns
dim(dm)
dim(ex)
dim(ae)

# Variable names
names(dm)
names(ex)
names(ae)

# First six records
head(dm)
head(ex)
head(ae)

# Number of unique participants
length(unique(dm$USUBJID))
length(unique(ex$USUBJID))
length(unique(ae$USUBJID))
```

DM generally contains one record per participant. EX and AE may contain multiple records for the same participant.

## Planned workflow

1. Explore and document the input datasets.
2. Define derivation rules before programming.
3. Create ADSL from demographics and exposure data.
4. Create ADAE from adverse-event data and selected ADSL variables.
5. Validate the derived variables.
6. Produce an adverse-event summary.
7. Document results, limitations, and instructions for reproducing the analysis.

## Analysis decisions

The project will document decisions that affect results, including:

- Which exposure records qualify as valid treatment
- How missing date or time components are handled
- The treatment-emergent observation window
- How first occurrences are selected
- Which participants are included in summary denominators

These decisions will be specified for this educational project rather than assumed to apply to every clinical trial.

## Current progress

- [x] Define the project scope and identify example data
- [ ] Explore DM, EX, and AE
- [ ] Write derivation specifications
- [ ] Build ADSL
- [ ] Build ADAE
- [ ] Validate datasets
- [ ] Generate safety summaries
- [ ] Prepare metadata and XPT exports

## Limitations

This is an educational portfolio project. It is not a regulatory submission or a validated production workflow. Analysis outputs will be added after the programming and checks are completed.
