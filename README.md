# A Beginner’s Guide to Clinical Trial Data: SDTM, ADaM, and Analysis in R

**Author:** Reshika Rimal  
**Project status:** In progress  
**Current tools:** R and pharmaversesdtm  
**Planned tools:** admiral and xportr

## 1. What is this project about?

Clinical trials collect information about participants, treatments, health measurements, and medical events.

Before statisticians can analyze these data, programmers organize them, check their quality, and derive variables needed for analysis.

This project demonstrates part of that process using publicly available example data. It starts with standardized clinical trial datasets and creates two working analysis datasets:

- **ADSL:** Information about each participant
- **ADAE:** Information about each participant’s adverse events

The initial programming uses base R so that each calculation is visible and understandable.

## 2. What is a clinical trial?

A clinical trial studies a health intervention, such as a medicine.

For example, researchers may compare:

- A placebo
- A lower dose of a medicine
- A higher dose of the same medicine

A **placebo** resembles the study treatment but does not contain its active ingredient.

Researchers collect information to evaluate outcomes and safety. Examples include age, treatment exposure, laboratory measurements, and adverse events.

An **adverse event** is an unfavorable medical occurrence. A headache during a trial may be an adverse event, but its occurrence alone does not show that the medicine caused it.

## 3. What is CDISC?

**CDISC** stands for **Clinical Data Interchange Standards Consortium**.

CDISC develops standards that help organizations structure and exchange clinical research data consistently.

Two standards relevant to this project are:

| Standard | Full name | Main purpose |
|---|---|---|
| SDTM | Study Data Tabulation Model | Organize collected clinical trial data |
| ADaM | Analysis Data Model | Organize data for statistical analysis and traceability |

## 4. Where do SDTM and ADaM fit?

A simplified clinical programming workflow is:

1. Collect information through study forms and other sources.
2. Organize collected information into SDTM datasets.
3. Derive analysis variables and create ADaM datasets.
4. Produce statistical analyses, tables, listings, and figures.

**This project begins at Step 2.** It uses existing SDTM example datasets rather than collecting data or building SDTM from raw records.

## 5. What is SDTM?

SDTM provides a standard structure for clinical trial data.

Information is organized into datasets called **domains**. Each domain focuses on a particular type of information.

| Domain | Meaning | Examples of information |
|---|---|---|
| DM | Demographics | Age, sex, participant identifiers, treatment groups |
| EX | Exposure | Treatment, dose, exposure start and end dates |
| AE | Adverse Events | Event term, severity, seriousness, start and end dates |
| LB | Laboratory Test Results | Laboratory tests and results |
| VS | Vital Signs | Blood pressure, pulse, weight |
| CM | Concomitant Medications | Other medications used by participants |
| MH | Medical History | Medical conditions recorded as history |
| DS | Disposition | Study completion, discontinuation, and related events |

This project uses **DM, EX, and AE**.

### What does “one row” mean?

The meaning of a row depends on the domain.

- In **DM**, one row represents a participant.
- In **EX**, one row can represent a treatment exposure interval.
- In **AE**, one row represents an adverse-event record.

A participant can therefore have one DM record, several EX records, and several AE records.

## 6. How are participants connected across datasets?

Two important identifiers are:

| Variable | Meaning |
|---|---|
| STUDYID | Study identifier |
| USUBJID | Unique subject identifier |

For this project, joins use both `STUDYID` and `USUBJID`.

These identifiers allow us to connect a participant’s demographic information, treatment exposure, and adverse events.

### What are sequence numbers?

A participant can have multiple records in a domain. Sequence numbers distinguish those records.

| Variable | Meaning |
|---|---|
| EXSEQ | Exposure record sequence number |
| AESEQ | Adverse-event record sequence number |
| ASEQ | Analysis record sequence number |

Source identifiers are retained so that analysis records can be traced back to the original data.

## 7. Understanding the three input datasets

### DM: Demographics

DM contains participant-level information.

Important variables include:

| Variable | Meaning |
|---|---|
| AGE | Age |
| SEX | Sex |
| RACE | Race |
| COUNTRY | Country |
| ARM | Planned study arm |
| ACTARM | Actual study arm |

Our DM dataset contains **306 records for 306 participants**.

### EX: Exposure

EX describes exposure to study treatment.

| Variable | Meaning |
|---|---|
| EXTRT | Treatment name |
| EXDOSE | Dose amount |
| EXDOSU | Dose unit |
| EXSTDTC | Exposure start date/time as character text |
| EXENDTC | Exposure end date/time as character text |

Our EX dataset contains **591 records for 254 participants**.

In these example data, placebo exposure is recorded as **0 mg**. Those records still represent study-treatment exposure and must be retained when defining our safety population.

### AE: Adverse Events

AE contains reported adverse-event records.

| Variable | Meaning |
|---|---|
| AETERM | Reported event term |
| AEDECOD | Dictionary-derived preferred term |
| AEBODSYS | Body system or system organ class |
| AESEV | Event severity |
| AESER | Whether the event is serious |
| AEREL | Recorded relationship to treatment |
| AEOUT | Event outcome |
| AESTDTC | Event start date/time as character text |
| AEENDTC | Event end date/time as character text |

Our AE dataset contains **1,191 records for 225 participants**.

### Severity and seriousness are different

**Severity** describes intensity, such as mild, moderate, or severe.

**Seriousness** concerns criteria such as hospitalization, a life-threatening event, or death.

A severe event is not automatically a serious event.

## 8. Why create ADaM datasets?

SDTM provides standardized source information, but statistical analysis often needs additional variables.

For example:

- When did the participant first receive treatment?
- Did an event start before or after treatment?
- What study day did it begin?
- How long did it last?
- How many participants had at least one qualifying event?

ADaM provides structures and principles for analysis datasets that support these questions.

Creating ADaM involves more than renaming columns. It requires documented derivations, appropriate record structures, metadata, and traceability.

## 9. What is traceability?

**Traceability** means being able to explain where an analysis value came from and how it was derived.

For example:

1. `EXSTDTC` contains an exposure start date as text.
2. The text is converted into an R Date.
3. The earliest qualifying exposure date becomes `TRTSDT`.
4. `TRTSDT` is used when calculating an adverse event’s relative study day.

Keeping source identifiers and documenting these steps helps another programmer reproduce and review the result.

## 10. Common ADaM structures

ADaM includes different structures for different analytical needs.

| Structure | Main purpose | Examples |
|---|---|---|
| ADSL | Participant-level analysis information | Demographics, treatment groups, population flags |
| BDS: Basic Data Structure | Parameter-based analysis records | Laboratory or vital-sign analyses |
| OCCDS: Occurrence Data Structure | Analysis of occurrences and subject counts | Adverse events or concomitant medications |

Dataset names alone do not establish conformance. The structure, variables, metadata, and analysis purpose must also be appropriate.

This project focuses on **ADSL and an OCCDS-style ADAE**.

## 11. What is ADSL?

**ADSL** means **Subject-Level Analysis Dataset**.

It contains one record per participant and supplies information used across analyses.

Our initial ADSL contains:

| Variable | Meaning | Project derivation |
|---|---|---|
| TRT01P | Planned treatment for period 1 | Copied from ARM |
| TRT01A | Actual treatment for period 1 | Copied from ACTARM |
| TRTSDT | First treatment date | Earliest qualifying exposure start |
| TRTEDT | Last treatment date | Latest exposure end when all qualifying end dates are available |
| SAFFL | Safety-population flag | Y when qualifying exposure is present |

### What is an analysis population?

An analysis population defines which participants are included in a particular analysis.

For this educational project, the safety population includes participants with qualifying study-treatment exposure, including placebo.

| Group | Participants | Safety population |
|---|---:|---:|
| Placebo | 86 | 86 |
| Xanomeline High Dose | 72 | 72 |
| Xanomeline Low Dose | 96 | 96 |
| Screen Failure | 52 | 0 |
| **Total** | **306** | **254** |

The 52 screen failures remain in ADSL with `SAFFL = "N"`.

## 12. What is ADAE?

**ADAE** means **Adverse Events Analysis Dataset**.

It combines adverse-event information with relevant participant-level information and derived analysis variables.

Unlike ADSL, ADAE can contain many records per participant.

Our preliminary ADAE retains all **1,191 source AE records**.

| Variable | Meaning |
|---|---|
| ASTDT | Analysis start date |
| AENDT | Analysis end date |
| ASTDY | Analysis start day relative to treatment |
| AENDY | Analysis end day relative to treatment |
| ADURN | Analysis duration |
| ADURU | Duration unit |
| TRTEMFL | Preliminary treatment-emergent flag under our project rule |
| ASEQ | Analysis sequence number |

## 13. How are dates handled?

SDTM date/time values are commonly stored as character strings.

Examples include:

| Source value | Information available |
|---|---|
| 2014-03-15 | Complete calendar date |
| 2014-03 | Year and month |
| 2014 | Year only |
| NA | Missing value in R |

A partial date is not an exact calendar date. We must not silently assume its missing components.

### Our project approach

- Complete dates are converted into R Date objects.
- Partial start dates retain missing analysis dates.
- Missing end dates remain missing.
- No exact dates are imputed.

The input AE data contain:

- **26 partial start dates**
- **473 missing end dates**

All events with missing end dates have an outcome of `NOT RECOVERED/NOT RESOLVED`.

## 14. What are relative study days?

Relative study days express when an event occurred compared with first treatment.

Our rule is:

- Treatment start date is **Day 1**.
- The preceding date is **Day −1**.
- There is **no Day 0**.

For a participant starting treatment on January 10:

| Event date | Relative day |
|---|---:|
| January 9 | −1 |
| January 10 | 1 |
| January 11 | 2 |
| January 12 | 3 |

Relative days remain missing when a required analysis date is unavailable.

## 15. How is event duration calculated?

For events with complete start and end dates, this project uses **inclusive calendar-day duration**:

```r
ADURN <- as.integer(AENDT - ASTDT) + 1L
```

An event starting January 9 and ending January 11 has a duration of **3 days**.

This counts calendar days, not elapsed hours.

In our results:

| Duration availability | Events |
|---|---:|
| Calculable | 714 |
| Missing because a required analysis date is unavailable | 477 |

## 16. What does treatment-emergent mean?

A treatment-emergent definition identifies events according to their timing and, where specified, worsening after treatment begins.

The exact definition depends on the study’s analysis rules.

Our first implementation uses a simpler **onset-based rule**:

> An event qualifies if it starts on or after first treatment and on or before 30 days after last treatment.

This implementation does not assess worsening of pre-existing events.

A qualifying event is not necessarily caused by treatment.

### Why use a separate review status?

A missing flag could otherwise hide the difference between:

- An event that does not meet the rule
- An event that cannot yet be classified

Our project-specific `TE_STATUS` records:

| Value | Meaning |
|---|---|
| YES | Meets the onset rule |
| NO | Does not meet the onset rule |
| REVIEW | Available information does not establish the answer |
| NOT ELIGIBLE | Participant is outside the project safety population |

`TRTEMFL` is set to `"Y"` for qualifying events. Other values are missing, so `TE_STATUS` must be consulted to distinguish their meaning.

## 17. Can a partial date still support classification?

Sometimes we can classify timing without knowing the exact date.

For example, `"2003"` represents a possible start between January 1 and December 31, 2003. If treatment started in 2014, the entire possible interval precedes treatment.

Our project creates lower and upper bounds for partial start dates. An event is classified only when every possible start gives the same answer under the rule.

These bounds do not replace the missing analysis start date.

This approach classified 26 initially unresolved events:

- 20 did not meet the onset rule.
- 6 met the onset rule.
- 5 other events remain unresolved.

## 18. Event counts and participant counts are different

One participant can experience several adverse events.

Therefore:

- **Event counts** count records.
- **Participant counts** count people.

A participant with five qualifying events contributes five to an event count but only one to the count of participants with at least one qualifying event.

### Event-level results

| Treatment group | Meets rule | Does not meet rule | Unresolved |
|---|---:|---:|---:|
| Placebo | 281 | 20 | 0 |
| Xanomeline High Dose | 414 | 18 | 4 |
| Xanomeline Low Dose | 426 | 27 | 1 |
| **Total** | **1,121** | **65** | **5** |

### Participant-level results

Percentages use all safety-population participants in each group.

| Treatment group | Safety participants | At least one qualifying event | Unresolved | No qualifying event recorded |
|---|---:|---:|---:|---:|
| Placebo | 86 | 65 (75.6%) | 0 (0.0%) | 21 (24.4%) |
| Xanomeline High Dose | 72 | 68 (94.4%) | 1 (1.4%) | 3 (4.2%) |
| Xanomeline Low Dose | 96 | 83 (86.5%) | 1 (1.0%) | 12 (12.5%) |
| **Total** | **254** | **216 (85.0%)** | **2 (0.8%)** | **36 (14.2%)** |

Participants with a confirmed qualifying event remain in the first category even if another event is unresolved.

These descriptive example-data results do not establish comparative treatment safety.

## 19. What are metadata and XPT files?

### Metadata

Metadata describe the datasets and their variables.

Examples include:

- Variable name and label
- Data type
- Units and allowed values
- Source and derivation rule
- Dataset record structure and keys

Metadata help reviewers understand and interpret the data correctly.

### XPT files

A SAS transport file, commonly using the `.xpt` extension, is a format used to exchange datasets in clinical programming.

The planned xportr extension will explore metadata preparation and XPT export.

Exporting an XPT file does not by itself establish regulatory readiness or standards compliance.

## 20. What is pharmaverse?

**Pharmaverse** is an ecosystem of open-source tools for clinical reporting and related workflows.

Packages relevant to this project include:

| Package | Role |
|---|---|
| pharmaversesdtm | Supplies example SDTM data |
| admiral | Provides reusable ADaM derivation functions |
| dplyr | Supports data manipulation |
| lubridate | Supports date and time handling |
| xportr | Supports metadata application and XPT export |

The current implementation uses **base R and pharmaversesdtm**. Other packages are planned additions.

## 21. Data source and first steps in R

Data are available from:

- [pharmaversesdtm documentation](https://pharmaverse.github.io/pharmaversesdtm/)
- [pharmaversesdtm GitHub repository](https://github.com/pharmaverse/pharmaversesdtm)

The package includes CDISC pilot-project data and datasets developed for testing. See dataset-specific documentation for provenance.

Install the package once:

```r
install.packages("pharmaversesdtm")
```

Load the input data:

```r
dm <- pharmaversesdtm::dm
ex <- pharmaversesdtm::ex
ae <- pharmaversesdtm::ae
```

Explore their structure:

```r
dim(dm)
dim(ex)
dim(ae)

names(dm)
names(ex)
names(ae)

head(dm)
head(ex)
head(ae)
```

## 22. Quality checks performed

Checks passed during interactive development for:

- Participant and event identifier uniqueness
- Matching participants across source domains
- Preservation of AE records after joining ADSL
- Conversion of complete dates
- Date-order consistency
- Treatment start availability in the safety population
- Absence of Day 0 in relative-day variables
- Consistent partial-date bounds
- Unique analysis sequence numbers
- Participant summaries accounting for all 254 safety participants

Running the complete saved workflow from a clean R session remains pending.

## 23. Current progress

- [x] Explore DM, EX, and AE
- [x] Build initial ADSL
- [x] Build preliminary ADAE
- [x] Derive relative days and durations
- [x] Add analysis sequence numbers
- [x] Classify partial-date timing where possible
- [x] Generate participant-level summaries
- [ ] Verify the saved scripts from a clean R session
- [ ] Add an admiral implementation and compare results
- [ ] Add documented first-occurrence flags
- [ ] Prepare metadata and XPT exports
- [ ] Complete final standards review

## 24. Limitations

This is an educational portfolio project using example data. It is not a regulatory submission or a validated production workflow.

Five event classifications remain unresolved. Treatment end dates remain missing for six participants under the project’s chosen rule.

The analysis does not evaluate worsening of pre-existing events, infer causality, or test treatment-group differences. Full ADaM conformance has not been established.
