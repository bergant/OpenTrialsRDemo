# Using OpenTrials API with R
Darko Bergant  
2016-10-14, updated: `r format(Sys.time(), "%Y-%m-%d")`  



# About OpenTrials

From [OpenTrials](http://opentrials.net) page: _OpenTrials is a collaboration 
between [Open Knowledge International](https://okfn.org/) and Dr Ben Goldacre 
from the University of Oxford [DataLab](https://ebmdatalab.net/). It aims to
locate, match, and share all publicly accessible data and documents, on all
trials conducted, on all medicines and other treatments, globally._

In addition to [web search](http://explorer.opentrials.net/search), the 
OpenTrials database is also accessible via
[API](https://api.opentrials.net/v1/docs/), documented as Open API (Swagger) 
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
##  [1] "searchTrials"        "autocomplete"        "searchFDADocuments" 
##  [4] "getTrial"            "getPublication"      "getCondition"       
##  [7] "getOrganisation"     "getRecords"          "getRecord"          
## [10] "getPerson"           "getIntervention"     "list"               
## [13] "listFDAApplications" "getFDAApplication"   "listDocuments"      
## [16] "getDocument"
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
## [1] 4050
```

```r
length(trials$items)
```

```
## [1] 20
```

There are 4050 trials matching this search query and
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

<!--html_preserve--><div id="htmlwidget-dfac77449f7e264987ba" style="width:864px;height:864px;" class="grViz html-widget"></div>
<script type="application/json" data-for="htmlwidget-dfac77449f7e264987ba">{"x":{"diagram":"digraph schema {\nrankdir=\"LR\"\n\n\n\"TrialSearchResults\"[shape = \"Mrecord\", label=\"TrialSearchResults|total_count (integer)\\litems (array[Trial])\\l\"]\n\n\"Trial\"[shape = \"Mrecord\", label=\"Trial|id (string)\\lsource_id (string)\\lidentifiers (object)\\lurl (string)\\lpublic_title (string)\\lbrief_summary (string)\\ltarget_sample_size (integer)\\lgender (string)\\lhas_published_results (boolean)\\lregistration_date (string)\\lcompletion_date (string)\\lresults_exemption_date (string)\\lstatus (string)\\lrecruitment_status (string)\\llocations (array[TrialLocation])\\linterventions (array[Intervention])\\lconditions (array[Condition])\\lpersons (array[TrialPerson])\\lorganisations (array[TrialOrganisation])\\lrecords (array[RecordSummary])\\lpublications (array[PublicationSummary])\\ldiscrepancies (object)\\ldocuments (array[DocumentSummary])\\lsources (object)\\lrisks_of_bias (array[RiskOfBias])\\l\"]\n\n\"TrialLocation\"[shape = \"Mrecord\", label=\"TrialLocation|id (string)\\lname (string)\\ltype (string)\\lrole (string)\\l\"]\n\n\"Intervention\"[shape = \"Mrecord\", label=\"Intervention|id (string)\\lname (string)\\lurl (string)\\ltype (string)\\l\"]\n\n\"Condition\"[shape = \"Mrecord\", label=\"Condition|id (string)\\lname (string)\\lurl (string)\\l\"]\n\n\"TrialPerson\"[shape = \"Mrecord\", label=\"TrialPerson|id (string)\\lname (string)\\lurl (string)\\lrole (string)\\l\"]\n\n\"TrialOrganisation\"[shape = \"Mrecord\", label=\"TrialOrganisation|id (string)\\lname (string)\\lurl (string)\\lrole (string)\\l\"]\n\n\"RecordSummary\"[shape = \"Mrecord\", label=\"RecordSummary|id (string)\\lurl (string)\\lsource_id (string)\\lis_primary (boolean)\\llast_verification_date (string)\\l\"]\n\n\"PublicationSummary\"[shape = \"Mrecord\", label=\"PublicationSummary|id (string)\\lurl (string)\\ltitle (string)\\lsource_id (string)\\lsource_url (string)\\l\"]\n\n\"DocumentSummary\"[shape = \"Mrecord\", label=\"DocumentSummary|id (string)\\lname (string)\\lurl (string)\\ltype (DocumentType)\\ltrials (array[TrialSummary])\\lfile (FileSummary)\\lfda_application (FDAApplication)\\lsource_id (string)\\lsource_url (string)\\l\"]\n\n\"TrialSummary\"[shape = \"Mrecord\", label=\"TrialSummary|id (string)\\lurl (string)\\lpublic_title (string)\\l\"]\n\n\"FileSummary\"[shape = \"Mrecord\", label=\"FileSummary|id (string)\\lsha1 (string)\\lsource_url (string)\\ldocumentcloud_id (string)\\l\"]\n\n\"FDAApplication\"[shape = \"Mrecord\", label=\"FDAApplication|id (string)\\ldrug_name (string)\\lactive_ingredients (string)\\lfda_approvals (array[FDAApproval])\\lorganisation (Organisation)\\ltype (string)\\lurl (string)\\l\"]\n\n\"FDAApproval\"[shape = \"Mrecord\", label=\"FDAApproval|id (string)\\lsupplement_number (integer)\\ltype (string)\\laction_date (string)\\lnotes (string)\\lfda_application (FDAApplication)\\l\"]\n\n\"Organisation\"[shape = \"Mrecord\", label=\"Organisation|id (string)\\lname (string)\\lurl (string)\\l\"]\n\n\"RiskOfBias\"[shape = \"Mrecord\", label=\"RiskOfBias|id (string)\\lsource_id (string)\\lsource_url (string)\\lstudy_id (string)\\lrisk_of_bias_criteria (array[RiskOfBiasCriteria])\\l\"]\n\n\"RiskOfBiasCriteria\"[shape = \"Mrecord\", label=\"RiskOfBiasCriteria|id (string)\\lname (string)\\lvalue (string)\\l\"]\n\n\n\"TrialSearchResults\"->\"Trial\"\n\"Trial\"->\"TrialLocation\"\n\"Trial\"->\"Intervention\"\n\"Trial\"->\"Condition\"\n\"Trial\"->\"TrialPerson\"\n\"Trial\"->\"TrialOrganisation\"\n\"Trial\"->\"RecordSummary\"\n\"Trial\"->\"PublicationSummary\"\n\"Trial\"->\"DocumentSummary\"\n\"Trial\"->\"RiskOfBias\"\n\"DocumentSummary\"->\"DocumentType\"\n\"DocumentSummary\"->\"TrialSummary\"\n\"DocumentSummary\"->\"FileSummary\"\n\"DocumentSummary\"->\"FDAApplication\"\n\"FDAApplication\"->\"FDAApproval\"\n\"FDAApplication\"->\"Organisation\"\n\"FDAApproval\"->\"FDAApplication\"\n\"RiskOfBias\"->\"RiskOfBiasCriteria\"\n}","config":{"engine":"dot","options":null}},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

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
  arrange(desc(sample_size)) %>% 
  kable(caption = "Table: some data from trials search result")
```



Table: Table: some data from trials search result

title                                                                                                         source_id    sample_size  status    registered    publications
------------------------------------------------------------------------------------------------------------  ----------  ------------  --------  -----------  -------------
Long-term Safety Study of Rapastinel as Adjunctive Therapy in Patients With Major Depressive Disorder         nct                  500  ongoing   2016-12-21               0
Mental Health in Adults and Children- Frugal Innovations (MAC-FI): Adult Depression Component                 nct                  256  ongoing   2016-12-20               0
Reducing Fetal Exposure to Maternal Depression to Improve Infant Risk Mechanisms                              nct                  240  ongoing   2017-01-04               0
Conventional Bilateral rTMS vs. Bilateral Theta Burst Stimulation for Late-Life Depression                    nct                  220  ongoing   2016-12-16               0
Preeclampsia Research on Vitamin D, Inflammation, & Depression                                                nct                  200  ongoing   2016-12-08               0
Study to Evaluate the Efficacy and Safety of Adjunctive Pimavanserin in Major Depressive Disorder (CLARITY)   nct                  188  ongoing   2017-01-10               0
Stepped Care for Depression in Heart Failure                                                                  nct                  180  ongoing   2016-12-15               0
Mantra Meditation in Major Depression                                                                         nct                  130  ongoing   2016-12-14               0
Interpersonal Counseling (IPC) for Treatment of Depression in Adolescents                                     nct                  120  ongoing   2016-12-20               0
Efficacy of H7-Coil DTMS Compared to H1-Coil DTMS in Subjects With Major Depression Disorder (MDD)            nct                  105  ongoing   2017-01-05               0
Dynamics of Inflammation and Its Blockade on Motivational Circuitry in Depression                             nct                   80  ongoing   2016-12-28               0
Brief CBT for the Treatment of Depression During Inpatient Hospitalization                                    nct                   75  ongoing   2017-01-04               0
A Study to Evaluate SAGE-217 in Subjects With Moderate to Severe Major Depressive Disorder                    nct                   62  ongoing   2016-12-14               0
Serum Cortisol Levels in Patients With Anxiety and Depression With Symptomatic Oral Lichen Planus             nct                   60  ongoing   2017-01-04               0
Are Bright Lights and Regulated Sleep Effective Treatment for Depression?                                     nct                   60  ongoing   2017-01-03               0
Effects of Online Cognitive Control Training on Rumination and Depressive Symptoms                            nct                   52  ongoing   2016-12-19               0
Emotional Awareness and SElf-regulation for Depression in Patients With Hypertension (EASE) Study             nct                   48  ongoing   2017-01-05               0
CSE v. Epidural for Postpartum Depression                                                                     nct                   46  ongoing   2017-01-11               0
Integrating HIV and Depression Self-Care to Improve Adherence in Perinatal Women                              nct                   40  ongoing   2017-01-06               0
The Experience of Older Adults Facing Depression for the First Time in Old Age                                nct                   15  ongoing   2016-12-23               0

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
  open_trials$autocomplete(`in` = "condition", q = "depression depressive", per_page = 100) %>% 
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



id                                     name                                                                                                                                                       trials
-------------------------------------  --------------------------------------------------------------------------------------------------------------------------------------------------------  -------
0188f4f0-10b1-44f2-b949-c95bee23b4e4   F32.9 - Depressive episode, unspecified                                                                                                                         4
004267e9-a20b-49d2-8d73-4313ce59db78   Topic: Mental Health Research Network, Primary Care Research Network for England; Subtopic: Depression, Not Assigned; Disease: Depression, All Diseases         4
016ae19a-8c44-11e6-be70-0242ac12000f   Mental Depression                                                                                                                                               2
01008ccb-c73d-4f15-8c10-2cb7ec1c65c9   Late Life Depression (LLD)                                                                                                                                      1
005a7e4e-b74d-4e96-a685-317923dab7c8   Depression Anxiety Sleep disturbances Circadian disturbances                                                                                                    1
00ae9a73-58f0-4871-b704-8399ffa79449   Major depressive disorder (episode or recurrent), moderately severe as principal DSM-IV diagnosis                                                               1
00a154f8-8c6f-11e6-be70-0242ac12000f   Moderate to severe, or mild persistent, episode of Major Depression (DSM-IV) who initiate a new antidepressant treatment episode                                1


# OpenTrials Operations
From Swagger definition ("http://api.opentrials.net/v1/swagger.yaml") 

**searchTrials**

Description: Returns trials based on a search query. By default, it'll search in all of a trial's attributes.
- `q` is a [ElasticSearch query string](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/query-dsl-query-string-query.html#query-string-syntax) (e.g. `public_title:(depressive OR depression)`)
- `page` can take a value between `1` and `100`
- `per_page` can take a value between `10` and `100` 

**autocomplete**

Description: Autocomplete search feature for supported database entities (`condition`, `intervention`, `location`, `person`, `organisation`). It has the same options as a regular `search` operation, with an extra **required** `in` parameter indicating the entity type to search. 

**searchFDADocuments**

Description: Search the FDA documents 

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

**listFDAApplications**

Description: Returns FDA applications 

**getFDAApplication**

Description: Returns an FDA application details 

**listDocuments**

Description: Returns documents 

**getDocument**

Description: Returns details of a document 

# List of Data Sources
OpenTrials data sources:

```r
sources <- open_trials$list()
knitr::kable(bind_rows(sources), caption = "Table of sources")
```



Table: Table of sources

id                       name                                source_url                                             type       terms_and_conditions_url                                   
-----------------------  ----------------------------------  -----------------------------------------------------  ---------  -----------------------------------------------------------
fda                      U.S. Food and Drug Administration   http://www.fda.gov                                     other      NA                                                         
cochrane_schizophrenia   Cochrane Schizophrenia Group        http://schizophrenia.cochrane.org/                     other      NA                                                         
euctr                    EU Clinical Trials Register         https://www.clinicaltrialsregister.eu                  register   https://www.clinicaltrialsregister.eu/disclaimer.html      
nct                      ClinicalTrials.gov                  https://clinicaltrials.gov                             register   https://clinicaltrials.gov/ct2/about-site/terms-conditions 
hra                      Health Research Authority           http://www.hra.nhs.uk                                  other      http://www.hra.nhs.uk/terms-conditions/                    
ictrp                    WHO ICTRP                           http://www.who.int/trialsearch/                        register   http://www.who.int/ictrp/search/download/en/               
fdadl                    FDA Drug Labels                     https://open.fda.gov                                   other      https://open.fda.gov/terms/                                
icdcm                    ICD-CM                              https://www.cms.gov/Medicare/Coding/ICD10/index.html   other      NA                                                         
icdpcs                   ICD-PCS                             https://www.cms.gov/Medicare/Coding/ICD10/index.html   other      NA                                                         
pubmed                   PubMed                              http://www.ncbi.nlm.nih.gov/pubmed                     other      https://www.ncbi.nlm.nih.gov/home/about/policies.shtml     

# Other R Packages
**Attached packages:**
<p>Xie Y (2016).
<em>knitr: A General-Purpose Package for Dynamic Report Generation in R</em>.
R package version 1.15.1, <a href="http://yihui.name/knitr/">http://yihui.name/knitr/</a>. 
</p>
<p>Wickham H and Francois R (2016).
<em>dplyr: A Grammar of Data Manipulation</em>.
R package version 0.5.0, <a href="https://CRAN.R-project.org/package=dplyr">https://CRAN.R-project.org/package=dplyr</a>. 
</p>
<p>Sveidqvist K, Bostock M, Pettitt C, Daines M, Kashcha A and Iannone R (2017).
<em>DiagrammeR: Create Graph Diagrams and Flowcharts Using R</em>.
R package version 0.9.0, <a href="https://CRAN.R-project.org/package=DiagrammeR">https://CRAN.R-project.org/package=DiagrammeR</a>. 
</p>

**Other loaded packages in R session**

Rcpp 0.12.9 http://www.rcpp.org, http://dirk.eddelbuettel.com/code/rcpp.html,
https://github.com/RcppCore/Rcpp - highr 0.6 https://github.com/yihui/highr - RColorBrewer 1.1-2  - influenceR 0.1.0 https://github.com/rcc-uchicago/influenceR - plyr 1.8.4 http://had.co.nz/plyr, https://github.com/hadley/plyr - viridis 0.3.4 https://github.com/sjmgarnier/viridis - tools 3.3.2  - digest 0.6.11 http://dirk.eddelbuettel.com/code/digest.html - jsonlite 1.2 https://arxiv.org/abs/1403.2805,
https://www.opencpu.org/posts/jsonlite-a-smarter-json-encoder - evaluate 0.10 https://github.com/hadley/evaluate - tibble 1.2 https://github.com/hadley/tibble - gtable 0.2.0  - rgexf 0.15.3 http://bitbucket.org/gvegayon/rgexf, http://www.ggvega.com - igraph 1.0.1 http://igraph.org - rstudioapi 0.6  - DBI 0.5-1 http://rstats-db.github.io/DBI - curl 2.3 https://github.com/jeroenooms/curl#readme - yaml 2.1.14  - gridExtra 2.2.1 https://github.com/baptiste/gridextra - httr 1.2.1 https://github.com/hadley/httr - stringr 1.1.0 https://github.com/hadley/stringr - htmlwidgets 0.8 https://github.com/ramnathv/htmlwidgets - rprojroot 1.1 https://github.com/krlmlr/rprojroot,
https://krlmlr.github.io/rprojroot - grid 3.3.2  - R6 2.2.0 https://github.com/wch/R6/ - Rook 1.1-1  - XML 3.98-1.5 http://www.omegahat.net/RSXML - rmarkdown 1.3 http://rmarkdown.rstudio.com - ggplot2 2.2.1 http://ggplot2.tidyverse.org, https://github.com/tidyverse/ggplot2 - magrittr 1.5  - backports 1.0.4 https://github.com/mllg/backports - scales 0.4.1 https://github.com/hadley/scales - codetools 0.2-15  - htmltools 0.3.5 https://github.com/rstudio/htmltools - assertthat 0.1  - colorspace 1.3-2 https://hclwizard.org/ - brew 1.0-6  - stringi 1.1.2 http://www.gagolewski.com/software/stringi/
http://site.icu-project.org/ http://www.unicode.org/ - visNetwork 1.0.2 https://github.com/datastorm-open/visNetwork - lazyeval 0.2.0  - munsell 0.4.3 



