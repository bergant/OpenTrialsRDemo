# Using OpenTrials API with R
Darko Bergant  
`r format(Sys.time(), "%Y-%m-%d")`  



# About OpenTrials

From [OpenTrials](http://opentrials.net) page: _OpenTrials is a collaboration between
Open Knowledge and Dr Ben Goldacre from the University of Oxford DataLab. It
aims to locate, match, and share all publicly accessible data and documents, on
all trials conducted, on all medicines and other treatments, globally._

In addition to [web search](http://explorer.opentrials.net/search), the 
OpenTrials database is also accessible via API, documented as Open API (Swagger)
format. 

This short tutorial shows how to use this API from R with 
[rapiclient](https://github.com/bergant/rapiclient).

# Issues

Report **rapiclient** bugs on [rapiclient issues](https://github.com/bergant/rapiclient/issues).
Source R Markdown for this page is [here](https://github.com/bergant/OpenTrialsRDemo).

Report errors with OpenTrials data (for example, if a trial has incorrect
treatments tagged to it) on [this
page](http://explorer.opentrials.net/flag-error).


# Preparation
Install rapiclient from Github:

```r
devtools::install_github("bergant/rapiclient")
```

Use `get_api` and `get_operations` to create R client functions:


```r
library(rapiclient)
open_trials_api <- get_api("http://api.opentrials.net/v1/swagger.yaml")
open_trials_schemas <- get_schemas(open_trials_api)
open_trials <- get_operations(open_trials_api, handle_response = content_or_stop)

names(open_trials)
```

```
##  [1] "searchTrials"    "autocomplete"    "getTrial"       
##  [4] "getPublication"  "getCondition"    "getOrganisation"
##  [7] "getRecords"      "getRecord"       "getPerson"      
## [10] "getIntervention" "list"
```


# Search Trials

`searchTrials` returns trials based on a search query. By default, it’ll search 
in all of a trial’s attributes. Parameter `q` is a 
[ElasticSearch](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/query-dsl-query-string-query.html#query-string-syntax)
query string. Parameters `page` (1 to 100) and `per_page` (10 to 100) are
optional.


```r
trials <- open_trials$searchTrials(q='public_title:(depressive OR depression)')

trials$total_count
```

```
## [1] 4601
```

```r
length(trials$items)
```

```
## [1] 20
```

There are 4601 trials matching this search query and
the first 20 already waiting in the `trials$items` list.

Take a look at the richness of the trial search result schema
before using the results:


```r
library(DiagrammeR)
grViz(
  rapiclient::get_schema_graphviz_dot(
    open_trials_api, 
    open_trials_api$definitions$TrialSearchResults
  ) 
)
```

<!--html_preserve--><div id="htmlwidget-f13fbb4c62d576ceb9e7" style="width:768px;height:864px;" class="grViz html-widget"></div>
<script type="application/json" data-for="htmlwidget-f13fbb4c62d576ceb9e7">{"x":{"diagram":"digraph schema {\nrankdir=\"LR\"\n\n\n\"TrialSearchResults\"[shape = \"Mrecord\", label=\"TrialSearchResults|total_count (integer)\\litems (array[Trial])\\l\"]\n\n\"Trial\"[shape = \"Mrecord\", label=\"Trial|id (string)\\lsource_id (string)\\lidentifiers (object)\\lurl (string)\\lpublic_title (string)\\lbrief_summary (string)\\ltarget_sample_size (integer)\\lgender (string)\\lhas_published_results (boolean)\\lregistration_date (string)\\lstatus (string)\\lrecruitment_status (string)\\llocations (array[TrialLocation])\\linterventions (array[Intervention])\\lconditions (array[Condition])\\lpersons (array[TrialPerson])\\lorganisations (array[TrialOrganisation])\\lrecords (array[RecordSummary])\\lpublications (array[PublicationSummary])\\ldiscrepancies (object)\\ldocuments (array[Document])\\lsources (object)\\lrisks_of_bias (array[RiskOfBias])\\l\"]\n\n\"TrialLocation\"[shape = \"Mrecord\", label=\"TrialLocation|role (string)\\l* (Location)\\l\"]\n\n\"Location\"[shape = \"Mrecord\", label=\"Location|id (string)\\lname (string)\\ltype (string)\\l\"]\n\n\"Intervention\"[shape = \"Mrecord\", label=\"Intervention|id (string)\\lname (string)\\lurl (string)\\ltype (string)\\l\"]\n\n\"Condition\"[shape = \"Mrecord\", label=\"Condition|id (string)\\lname (string)\\lurl (string)\\l\"]\n\n\"TrialPerson\"[shape = \"Mrecord\", label=\"TrialPerson|role (string)\\l* (Person)\\l\"]\n\n\"Person\"[shape = \"Mrecord\", label=\"Person|id (string)\\lname (string)\\lurl (string)\\l\"]\n\n\"TrialOrganisation\"[shape = \"Mrecord\", label=\"TrialOrganisation|role (string)\\l* (Organisation)\\l\"]\n\n\"Organisation\"[shape = \"Mrecord\", label=\"Organisation|id (string)\\lname (string)\\lurl (string)\\l\"]\n\n\"RecordSummary\"[shape = \"Mrecord\", label=\"RecordSummary|id (string)\\lurl (string)\\lsource_id (string)\\l\"]\n\n\"PublicationSummary\"[shape = \"Mrecord\", label=\"PublicationSummary|id (string)\\lurl (string)\\ltitle (string)\\lsource_id (string)\\lsource_url (string)\\l\"]\n\n\"Document\"[shape = \"Mrecord\", label=\"Document|name (string)\\ltype (string)\\lurl (string)\\ldocumentcloud_id (string)\\ltext (string)\\l\"]\n\n\"RiskOfBias\"[shape = \"Mrecord\", label=\"RiskOfBias|id (string)\\lsource_id (string)\\lsource_url (string)\\lstudy_id (string)\\lrisk_of_bias_criteria (array[RiskOfBiasCriteria])\\l\"]\n\n\"RiskOfBiasCriteria\"[shape = \"Mrecord\", label=\"RiskOfBiasCriteria|id (string)\\lname (string)\\lvalue (string)\\l\"]\n\n\n\"TrialSearchResults\"->\"Trial\"\n\"Trial\"->\"TrialLocation\"\n\"Trial\"->\"Intervention\"\n\"Trial\"->\"Condition\"\n\"Trial\"->\"TrialPerson\"\n\"Trial\"->\"TrialOrganisation\"\n\"Trial\"->\"RecordSummary\"\n\"Trial\"->\"PublicationSummary\"\n\"Trial\"->\"Document\"\n\"Trial\"->\"RiskOfBias\"\n\"TrialLocation\"->\"Location\"\n\"TrialPerson\"->\"Person\"\n\"TrialOrganisation\"->\"Organisation\"\n\"RiskOfBias\"->\"RiskOfBiasCriteria\"\n}","config":{"engine":"dot","options":null}},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

Print some of the trial attributes in a table:


```r
library(dplyr)
library(knitr)

lapply(trials$items, function(x) { data.frame(
    title = gsub("\n","",x$public_title), 
    source_id = x$source_id,
    sample_size = ifelse(length(x$target_sample_size)==0, NA, x$target_sample_size),
    status = x$status,
    registered = format(as.POSIXct(x$registration_date)),
    publications = length(x$publications),
    stringsAsFactors = FALSE
)}) %>% 
  bind_rows %>% 
  kable(caption = "Table: some data from trials search result")
```



Table: Table: some data from trials search result

title                                                                                                                                                                                                                    source_id    sample_size  status    registered    publications
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------  ----------  ------------  --------  -----------  -------------
Dallas 2K: A Natural History Study of Depression                                                                                                                                                                         nct                 2000  ongoing   2016-09-23               0
Magnetic Stimulation of the Brain in Schizophrenia or Depression                                                                                                                                                         nct                   80  ongoing   2016-09-14               0
A Study to Investigate the Safety, Tolerability, and Pharmacodynamics of JNJ-54175446 in Participants With Major Depressive Disorder                                                                                     nct                   64  ongoing   2016-09-13               0
A Study to Evaluate the Effects of a Single-Dose and Repeat-Administration of Intranasal Esketamine on On-Road Driving in Participants With Major Depressive Disorder                                                    nct                   24  ongoing   2016-09-01               0
Neural and Cognitive Mechanisms of Depression and Anxiety: a Multimodal MRI Study                                                                                                                                        nct                  150  ongoing   2016-08-30               0
Oral ketamine for treating depression    Orale ketamine als aanvullende behandeling bij patiënten met een therapieresistente depressie                                                                                   euctr                 NA  ongoing   2016-08-30               0
Oral ketamine for treating depression    Orale ketamine als aanvullende behandeling bij patiënten met een therapieresistente depressie                                                                                   euctr                 NA  ongoing   2016-08-30               0
Development of objective measures for depression, bipolar disorder and dementia by quantifying facial expression, body movement, and voice data during clinical interview and daily activity utilizing wearable device   ictrp                300  ongoing   2016-08-25               0
Augmenting Internet-Based Cognitive Behavioral Therapy for Major Depressive Disorder With Low-Level Light Therapy                                                                                                        nct                  200  ongoing   2016-08-24               0
CanDirect: Effectiveness of a Telephone-supported Depression Self-care Intervention for Cancer Survivors                                                                                                                 nct                  286  ongoing   2016-08-24               0
Brain Function in Depression and Insulin Resistance                                                                                                                                                                      ictrp                 60  ongoing   2016-08-23               0
Repetitive Transcranial Magnetic Stimulationfor 2010 criteria diagnosed Fibromyalgia with a comorbidity of depression:  Evidence from a pilot Randomized Sham-Controlled Study                                           ictrp                 40  ongoing   2016-08-22               0
Clinical and Biological Markers of Response to Cognitive Behavioural Therapy for Depression                                                                                                                              nct                   40  ongoing   2016-08-19               0
Feasibility study of group unified protocol psychotherapy for patients with depressive and anxiety disorders                                                                                                             ictrp                 24  ongoing   2016-08-17               0
The Clinical Research on the Relationship Between Depression and Gut Microbiota in TBI Patients                                                                                                                          nct                   50  ongoing   2016-08-17               0
A Study of Chinese Medicine Treating Depression                                                                                                                                                                          nct                 4600  ongoing   2016-08-16               0
Evaluation of the Impact of the Level of Mindfulness on the Management of Patients With Recurrent Depressive Disorders by the Mindfulness Based Cognitive Therapy ( MBCT ): an Exploratory Study                         nct                   66  ongoing   2016-08-16               0
Evaluation of Depression Based on Measurement of Brain Activity Induced by Emotional Stimuli: A Study using the Depression Detection System                                                                              ictrp                 40  ongoing   2016-08-15               0
Long-term, Open-label, Flexible-dose, Extension Study of Vortioxetine in Child and Adolescent Patients With Major Depressive Disorder (MDD) From 7 to 18 Years of Age                                                    nct                  850  ongoing   2016-08-15               0
Integrated Mental Health Care and Vocational Rehabilitation to Individuals on Sick Leave Due to Anxiety and Depression                                                                                                   nct                  768  ongoing   2016-08-15               0

# Tips & Tricks

Print a function to see its documentation. For example: 

```r
open_trials$getTrial
```

```
## getTrial 
##  
## Description:
##    Returns a trial's details and related entities (e.g. `conditions`). 
## 
## Parameters:
##   id (string)
##     ID of the trial
```

If parameter name is a reserved word in R, quote it with backticks:

```r
conditions <- 
  open_trials$autocomplete(`in` = "condition", q = "addiction", per_page = 100) %>% 
  getElement("items") %>% 
  bind_rows %>%  
  select(id, name)
```

Note that operations are not vectorised. You have to use the *apply functions to
get the data for arguments with length > 1. For example, count the trials for
each condition:


```r
conditions$trials <- 
  sapply(conditions$id, function(x) {
    open_trials$searchTrials(q = sprintf("conditions.id:(%s)", x))[["total_count"]] 
  })  
  
conditions %>% arrange(desc(trials)) %>% top_n(30, wt = trials) %>% knitr::kable(.)
```



id                                     name                                                      trials
-------------------------------------  -------------------------------------------------------  -------
12f8f74b-bff8-49ce-bba8-98268271f3b3   Opiate Addiction                                              52
f2a93773-ec26-4fa0-9f75-a5b920aac395   Drug Addiction                                                40
0eec2750-8c1e-11e6-be70-0242ac12000f   Addiction                                                     35
a1bbbf9d-e64e-40be-89b8-6fa3418e583d   Cocaine Addiction                                             29
0b2f9b78-8c1a-11e6-be70-0242ac12000f   Smoking addiction                                             25
faa9dce0-8c2e-11e6-be70-0242ac12000f   Nicotine addiction                                            25
b96c69de-8c3b-11e6-be70-0242ac12000f   Heroin addiction                                              16
3ff7db3d-07df-43eb-94d1-fef0a7bfe610   Opioid Addiction                                              14
ee16cd52-238e-42b7-8556-a1c3d29e6e8c   Alcohol Addiction                                             13
e84efb49-7b06-495a-8941-aba6a111ce3f   Nicotine Addiction with the desire to quit smoking            13
1ba56d32-8c21-11e6-be70-0242ac12000f   Tobacco Addiction                                             11
0ee556e6-8c1e-11e6-be70-0242ac12000f   Mental and Behavioural Disorders: Addiction                    8
a63ceada-adc5-41a2-adcf-135dd9efe7f4   Exercise Addiction                                             7
81741e64-c26c-40e9-8c31-91e45ed1ebd0   Substance Addiction                                            5
40461e86-e5ff-4ec4-8c13-2636bdbcdc98   Internet Addiction                                             5
2ec0ff7b-562d-4482-967b-3a053281fba7   opium addiction                                                4
0e329957-3a71-4dc9-ad84-c9a3c4f5c453   Methamphetamine Addiction                                      4
65396a20-e73b-4c33-a154-8bb510b723b1   Addictions                                                     3
5311bfed-a186-40fd-b50b-09407a195b65   Amphetamine Addiction                                          3
ba8aa58e-0e01-4fb8-9373-230b1f425f63   Opioid drug addiction                                          3
2cf7ee08-8d0b-11e6-988b-0242ac12000c   Food Addiction                                                 2
2ad535de-8cae-11e6-988b-0242ac12000c   Cigarette Addiction                                            2
614a0bac-f698-4f9c-a40f-0728f9a2806e   Smopking (nicotine addiction/Tobacco addiction)                2
7a23c9f0-2811-41ba-b43b-fa9dc9813353   pornographic addiction                                         2
8306af24-636a-4c82-aea3-c06738c09561   Narcotic Addiction                                             2
c9c2dd22-1f75-472d-974a-f632d092f9b8   treatment resistant heroin addiction                           2
94a37d12-5a73-47aa-ab1d-2c1ec3b03cf9   Nicotine addiction/smoking                                     2
36cf59ac-0b09-4d04-b5cc-76b71e44664b   Patients with opioid addiction                                 2
2cbed2c4-ae9a-498d-a278-94d717467fd8   Internet addiction disorder                                    2
d9c97b1c-c4b0-4a48-bc65-0b173130c94e   Addiction to illicit heroin                                    2
1983ca29-118f-4213-a552-78298006ea87   Opiate addiction (detoxification from illicit opiates)         2
3f04b9b8-bbdc-42b6-95d2-ee044be66a42   Subjects have a diagnosis of opiate addiction                  2
853f313c-83c8-4bb7-bc68-d2923862531c   treatment of dependence in alcohol addiction                   2

# OpenTrials Operations
From Swagger definition ("http://api.opentrials.net/v1/swagger.yaml") 

**searchTrials**

Description: Returns trials based on a search query. By default, it'll search in all of a trial's attributes.
- `q` is a [ElasticSearch query string](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/query-dsl-query-string-query.html#query-string-syntax) (e.g. `public_title:(depressive OR depression)`)
- `page` can take a value between `1` and `100`
- `per_page` can take a value between `10` and `100` 

**autocomplete**

Description: Autocomplete search feature for supported database entities (`condition`, `intervention`, `location`, `person`, `organisation`). It has the same options as a regular `search` operation, with an extra **required** `in` parameter indicating the entity type to search. 

**getTrial**

Description: Returns a trial's details and related entities (e.g. `conditions`). 

**getPublication**

Description: Returns publication details 

**getCondition**

Description: Returns condition details 

**getOrganisation**

Description: Returns organisation details 

**getRecords**

Description: Returns a trial's raw records from its sources 

**getRecord**

Description: Returns a trial's raw record from its sources 

**getPerson**

Description: Returns person details 

**getIntervention**

Description: Returns intervention details 

**list**

Description: Returns list of sources 

# List of Data Sources
OpenTrials data sources:

```r
sources <- open_trials$list()
knitr::kable(bind_rows(sources), caption = "Table of sources")
```



Table: Table of sources

id                       name                                url                                                    type       terms_and_conditions_url                                   
-----------------------  ----------------------------------  -----------------------------------------------------  ---------  -----------------------------------------------------------
cochrane_schizophrenia   Cochrane Schizophrenia Group        http://schizophrenia.cochrane.org/                     other      NA                                                         
hra                      Health Research Authority           http://www.hra.nhs.uk                                  other      http://www.hra.nhs.uk/terms-conditions/                    
fda                      U.S. Food and Drug Administration   http://www.fda.gov                                     other      NA                                                         
fdadl                    FDA Drug Labels                     https://open.fda.gov                                   other      https://open.fda.gov/terms/                                
icdcm                    ICD-CM                              https://www.cms.gov/Medicare/Coding/ICD10/index.html   other      NA                                                         
icdpcs                   ICD-PCS                             https://www.cms.gov/Medicare/Coding/ICD10/index.html   other      NA                                                         
pubmed                   PubMed                              http://www.ncbi.nlm.nih.gov/pubmed                     other      https://www.ncbi.nlm.nih.gov/home/about/policies.shtml     
euctr                    EU Clinical Trials Register         https://www.clinicaltrialsregister.eu                  register   https://www.clinicaltrialsregister.eu/disclaimer.html      
ictrp                    WHO ICTRP                           http://www.who.int/trialsearch/                        register   http://www.who.int/ictrp/search/download/en/               
nct                      ClinicalTrials.gov                  https://clinicaltrials.gov                             register   https://clinicaltrials.gov/ct2/about-site/terms-conditions 

# R Packages
**Attached packages:**

```r
sessionInfo()$otherPkgs %>%
  names %>% 
  lapply(citation) %>% 
  lapply(first) %>% 
  lapply(print, style = "html") %>% 
  invisible
```

<p>Xie Y (2016).
<em>knitr: A General-Purpose Package for Dynamic Report Generation in R</em>.
R package version 1.14, <a href="http://yihui.name/knitr/">http://yihui.name/knitr/</a>. 
</p>
<p>Wickham H and Francois R (2015).
<em>dplyr: A Grammar of Data Manipulation</em>.
R package version 0.4.3, <a href="https://CRAN.R-project.org/package=dplyr">https://CRAN.R-project.org/package=dplyr</a>. 
</p>
<p>Sveidqvist K, Bostock M, Pettitt C, Daines M, Kashcha A and Iannone R (2016).
<em>DiagrammeR: Create Graph Diagrams and Flowcharts Using R</em>.
R package version 0.8.3, <a href="https://github.com/rich-iannone/DiagrammeR">https://github.com/rich-iannone/DiagrammeR</a>. 
</p>
<p>Bergant D (2016).
<em>rapiclient: Dynamic Open API (Swagger) Client</em>.
R package version 0.1.0.9003, <a href="https://github.com/bergant/rapiclient">https://github.com/bergant/rapiclient</a>. 
</p>

**Other loaded packages in R session**


```r
sessionInfo()$loadedOnly %>%
  lapply(function(x) paste(x$Package, x$Version, x$URL)) %>% 
  paste(collapse = " - ") %>% 
  cat
```

Rcpp 0.12.7 http://www.rcpp.org, http://dirk.eddelbuettel.com/code/rcpp.html,
https://github.com/RcppCore/Rcpp - rstudioapi 0.5  - magrittr 1.5  - munsell 0.4.3  - colorspace 1.2-6  - R6 2.2.0 https://github.com/wch/R6/ - highr 0.6 https://github.com/yihui/highr - stringr 1.0.0  - httr 1.2.1 https://github.com/hadley/httr - plyr 1.8.3 http://had.co.nz/plyr, https://github.com/hadley/plyr - visNetwork 0.2.1 https://github.com/DataKnowledge/visNetwork - tools 3.2.5  - parallel 3.2.5  - rsvg 0.5 https://github.com/jeroenooms/rsvg
https://www.opencpu.org/posts/svg-release - DBI 0.5-1 http://rstats-db.github.io/DBI - htmltools 0.3.5 https://github.com/rstudio/htmltools - lazyeval 0.2.0  - yaml 2.1.13  - assertthat 0.1  - digest 0.6.10 http://dirk.eddelbuettel.com/code/digest.html - tibble 1.2 https://github.com/hadley/tibble - formatR 1.4 http://yihui.name/formatR - htmlwidgets 0.7 https://github.com/ramnathv/htmlwidgets - codetools 0.2-14  - curl 2.1 https://github.com/jeroenooms/curl#readme - evaluate 0.9 https://github.com/hadley/evaluate - rmarkdown 1.0.9015 http://rmarkdown.rstudio.com - stringi 1.0-1 http://stringi.rexamine.com/ http://site.icu-project.org/
http://www.unicode.org/ - scales 0.4.0 https://github.com/hadley/scales - jsonlite 1.1 http://arxiv.org/abs/1403.2805,
https://www.opencpu.org/posts/jsonlite-a-smarter-json-encoder



