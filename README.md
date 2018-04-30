# roadoi - Use Unpaywall with R




[![Build Status](https://travis-ci.org/ropensci/roadoi.svg?branch=master)](https://travis-ci.org/ropensci/roadoi)
[![AppVeyor Build Status](https://ci.appveyor.com/api/projects/status/github/ropensci/roadoi?branch=master&svg=true)](https://ci.appveyor.com/project/ropensci/roadoi)
[![codecov.io](https://codecov.io/github/ropensci/roadoi/coverage.svg?branch=master)](https://codecov.io/github/ropensci/roadoi?branch=master)
[![cran version](http://www.r-pkg.org/badges/version/roadoi)](https://cran.r-project.org/package=roadoi)
[![rstudio mirror downloads](http://cranlogs.r-pkg.org/badges/roadoi)](https://github.com/metacran/cranlogs.app)
[![review](https://badges.ropensci.org/115_status.svg)](https://github.com/ropensci/onboarding/issues/115)



roadoi interacts with the [Unpaywall API](http://unpaywall.org/products/api), 
a simple web-interface which links DOIs and open access versions of scholarly works. 
The API powers [Unpaywall](http://unpaywall.org/).

This client supports the most recent API Version 2.

API Documentation: <http://unpaywall.org/products/api>

## How do I use it? 

Use the `oadoi_fetch()` function in this package to get open access status
information and full-text links from Unpaywall.



```r
roadoi::oadoi_fetch(dois = c("10.1038/ng.3260", "10.1093/nar/gkr1047"), 
                    email = "najko.jahn@gmail.com")
#> # A tibble: 2 x 13
#>   doi      best_oa_location oa_locations data_standard is_oa journal_is_oa
#>   <chr>    <list>           <list>               <int> <lgl> <lgl>        
#> 1 10.1038… <tibble [1 × 9]> <tibble [1 …             2 T     F            
#> 2 10.1093… <tibble [1 × 9]> <tibble [5 …             2 T     T            
#> # ... with 7 more variables: journal_issns <chr>, journal_name <chr>,
#> #   publisher <chr>, title <chr>, year <chr>, updated <chr>,
#> #   non_compliant <list>
```

There are no API restrictions. However, providing an email address is required and a rate limit of 100k is suggested. If you need to access more data, use the [data dump](https://unpaywall.org/products/snapshot) instead.

### RStudio Addin

This package also has a RStudio Addin for easily finding free full-texts in RStudio.

![](inst/img/oadoi_addin.gif)

## How do I get it? 

Install and load from [CRAN](https://cran.r-project.org/package=roadoi):


```r
install.packages("roadoi")
library(roadoi)
```

To install the development version, use the [devtools package](https://cran.r-project.org/package=devtools)


```r
devtools::install_github("ropensci/roadoi")
library(roadoi)
```

## Long-Form Documentation including use-case



Open access copies of scholarly publications are sometimes hard to find. Some are published in open access journals. Others are made freely available as preprints before publication, and others are deposited in institutional repositories, digital archives maintained by universities and research institutions. This document guides you to roadoi, a R client that makes it easy to search for these open access copies by interfacing the [Unpaywall](https://unpaywall.org/) service where DOIs are matched with freely available full-texts available from open access journals and archives.

### About Unpaywall

[Unpaywall](https://unpaywall.org/), developed and maintained by the [team of Impactstory](https://profiles.impactstory.org/about), is a non-profit service that finds open access copies of scholarly literature simply by looking up a DOI (Digital Object Identifier). It not only returns open access full-text links, but also helpful metadata about the open access status of a publication such as licensing or provenance information.

Unpaywall uses different data sources to find open access full-texts including:

- [Crossref](http://www.crossref.org/): a DOI registration agency serving major scholarly publishers.
- [Directory of Open Access Journals (DOAJ)](https://doaj.org/): a registry of open access journals
- Various OAI-PMH metadata sources. OAI-PMH is a protocol often used by open access journals and repositories such as arXiv and PubMed Central.

See Piwowar et al. (2017) for a comprehensive overview of Unpaywall.[^1]

### Basic usage

There is one major function to talk with Unpaywall, `oadoi_fetch()`, taking a character vector of DOIs and your email address as required arguments.


```r
library(roadoi)
roadoi::oadoi_fetch(dois = c("10.1186/s12864-016-2566-9",
                             "10.1103/physreve.88.012814"), 
                    email = "najko.jahn@gmail.com")
#> # A tibble: 2 x 13
#>   doi      best_oa_location oa_locations data_standard is_oa journal_is_oa
#>   <chr>    <list>           <list>               <int> <lgl> <lgl>        
#> 1 10.1186… <tibble [1 × 9]> <tibble [5 …             2 T     T            
#> 2 10.1103… <tibble [1 × 9]> <tibble [1 …             2 T     F            
#> # ... with 7 more variables: journal_issns <chr>, journal_name <chr>,
#> #   publisher <chr>, title <chr>, year <chr>, updated <chr>,
#> #   non_compliant <list>
```

#### What's returned?

The client supports API version 2. According to the [Unpaywall Data Format](http://unpaywall.org/data-format), the following variables with the following definitions are returned:

**Column**|**Description**
|:------------|:----------------------------------------------
`doi`|DOI (always in lowercase)
`best_oa_location`|list-column describing the best OA location. Algorithm prioritizes publisher hosted content (e.g. Hybrid or Gold)
`oa_locations`|list-column of all the OA locations. 
`data_standard`|Indicates the data collection approaches used for this resource. `1` mostly uses Crossref for hybrid detection. `2` uses more comprehensive hybrid detection methods. 
`is_oa`|Is there an OA copy (logical)? 
`journal_is_oa`|Is the article published in a fully OA journal? Uses the Directory of Open Access Journals (DOAJ) as source. 
`journal_issns`|ISSNs
`journal_name`|Journal title
`publisher`|Publisher
`title`|Publication title. 
`year`|Year published. 
`updated`|Time when the data for this resource was last updated. 
`non_compliant`|Lists other full-text resources that are not hosted by either publishers or repositories. 

The columns  `best_oa_location` and  `oa_locations` are list-columns
that contain useful metadata about the OA sources found by oaDOI. These are

**Column**|**Description**
|:------------|:----------------------------------------------
`evidence`|How the OA location was found and is characterized by oaDOI?
`host_type`|OA full-text provided by `publisher` or `repository`. 
`license`|The license under which this copy is published
`url`|The URL where you can find this OA copy.
`versions`|The content version accessible at this location following the DRIVER 2.0 Guidelines  (<https://wiki.surfnet.nl/display/DRIVERguidelines/DRIVER-VERSION+Mappings>)

There at least [two ways to simplify these list-columns](http://r4ds.had.co.nz/many-models.html#simplifying-list-columns).

To get the full-text links from the list-column `best_oa_location`, you may want to use `purrr::map_chr()`.


```r
library(dplyr)
roadoi::oadoi_fetch(dois = c("10.1186/s12864-016-2566-9",
                             "10.1103/physreve.88.012814"), 
                    email = "najko.jahn@gmail.com") %>%
  dplyr::mutate(
    urls = purrr::map(best_oa_location, "url") %>% 
                  purrr::map_if(purrr::is_empty, ~ NA_character_) %>% 
                  purrr::flatten_chr()
                ) %>%
  .$urls
#> [1] "https://bmcgenomics.biomedcentral.com/track/pdf/10.1186/s12864-016-2566-9?site=bmcgenomics.biomedcentral.com"
#> [2] "http://arxiv.org/pdf/1304.0473"
```

If you want to gather all full-text links and to explore where these links are hosted, simplify the list-column `oa_locations` with `tidyr::unnest()`:


```r
library(dplyr)
roadoi::oadoi_fetch(dois = c("10.1186/s12864-016-2566-9",
                             "10.1103/physreve.88.012814"), 
                    email = "najko.jahn@gmail.com") %>%
  tidyr::unnest(oa_locations) %>% 
  dplyr::mutate(
    hostname = purrr::map(url, httr::parse_url) %>% 
                  purrr::map_chr(., "hostname", .null = NA_integer_)
                ) %>% 
  dplyr::mutate(hostname = gsub("www.", "", hostname)) %>%
  dplyr::group_by(hostname) %>%
  dplyr::summarize(hosts = n())
#> # A tibble: 6 x 2
#>   hostname                      hosts
#>   <chr>                         <int>
#> 1 arxiv.org                         1
#> 2 bmcgenomics.biomedcentral.com     1
#> 3 doi.org                           1
#> 4 europepmc.org                     1
#> 5 ncbi.nlm.nih.gov                  1
#> 6 pub.uni-bielefeld.de              1
```


Note that fields to be returned might change according to the [Unpaywall API specs](http://unpaywall.org/products/api)

#### Any API restrictions?

There are no API restrictions. However, providing your email address when using this client is required by Unpaywall. Set email address in your `.Rprofile` file with the option `roadoi_email` when you are too tired to type in your email address every time you want to call Unpaywall.

```r
options(roadoi_email = "najko.jahn@gmail.com")
```

#### Keeping track of crawling

To follow your API call, and to estimate the time until completion, use the `.progress` parameter inherited from `plyr` to display a progress bar.


```r
roadoi::oadoi_fetch(dois = c("10.1186/s12864-016-2566-9",
                             "10.1103/physreve.88.012814"), 
                    email = "najko.jahn@gmail.com", 
                    .progress = "text")
#>   |                                                                         |                                                                 |   0%  |                                                                         |================================                                 |  50%  |                                                                         |=================================================================| 100%
#> # A tibble: 2 x 13
#>   doi      best_oa_location oa_locations data_standard is_oa journal_is_oa
#>   <chr>    <list>           <list>               <int> <lgl> <lgl>        
#> 1 10.1186… <tibble [1 × 9]> <tibble [5 …             2 T     T            
#> 2 10.1103… <tibble [1 × 9]> <tibble [1 …             2 T     F            
#> # ... with 7 more variables: journal_issns <chr>, journal_name <chr>,
#> #   publisher <chr>, title <chr>, year <chr>, updated <chr>,
#> #   non_compliant <list>
```

#### Catching errors

oaDOI is a reliable API. However, this client follows [Hadley Wickham's Best practices for writing an API package](https://CRAN.R-project.org/package=httr/vignettes/api-packages.html) and throws an error when the API does not return valid JSON or is not available. To catch these errors, you may want to use [purrr's `safely()`](https://purrr.tidyverse.org/reference/safely.html) function


```r
random_dois <-  c("ldld", "10.1038/ng.3260", "§dldl  ")
my_data <- purrr::map(random_dois, 
              .f = purrr::safely(function(x) roadoi::oadoi_fetch(x, email ="najko.jahn@gmail.com")))
# return results as data.frame
purrr::map_df(my_data, "result")
#> # A tibble: 1 x 13
#>   doi     best_oa_location oa_locations  data_standard is_oa journal_is_oa
#>   <chr>   <list>           <list>                <int> <lgl> <lgl>        
#> 1 10.103… <tibble [1 × 9]> <tibble [1 ×…             2 T     F            
#> # ... with 7 more variables: journal_issns <chr>, journal_name <chr>,
#> #   publisher <chr>, title <chr>, year <chr>, updated <chr>,
#> #   non_compliant <list>
#show errors
purrr::map(my_data, "error")
#> [[1]]
#> NULL
#> 
#> [[2]]
#> NULL
#> 
#> [[3]]
#> <simpleError: Oops, API did not return json after calling '§dldl  ':
#>         check your query - or api.oadoi.org may experience problems>
```

### Use Case: Studying the compliance with open access policies

An increasing number of universities, research organisations and funders have launched open access policies in recent years. Using roadoi together with other R-packages makes it easy to examine how and to what extent researchers comply with these policies in a reproducible and transparent manner. In particular, the [rcrossref package](https://github.com/ropensci/rcrossref), maintained by rOpenSci, provides many helpful functions for this task.

#### Gathering DOIs representing scholarly publications

DOIs have become essential for referencing scholarly publications, and thus many digital libraries and institutional databases keep track of these persistent identifiers. For the sake of this vignette, instead of starting with a pre-defined set of publications originating from these sources, we simply generate a random sample of 100 DOIs registered with Crossref by using the [rcrossref package](https://github.com/ropensci/rcrossref).


```r
library(dplyr)
library(rcrossref)
# get a random sample of DOIs and metadata describing these works
random_dois <- rcrossref::cr_r(sample = 100) %>%
  rcrossref::cr_works() %>%
  .$data
random_dois
#> # A tibble: 100 x 34
#>    container.title    created  deposited DOI   indexed ISBN  issued member
#>    <chr>              <chr>    <chr>     <chr> <chr>   <chr> <chr>  <chr> 
#>  1 2010 Second IITA … 2010-10… 2017-03-… 10.1… 2017-1… 9781… 2010-… 263   
#>  2 2011 IEEE PES CON… 2011-11… 2017-03-… 10.1… 2018-0… 9781… 2011-… 263   
#>  3 Paediatrics and C… 2014-12… 2015-03-… 10.1… 2017-1… <NA>  2015-… 78    
#>  4 Journal Français … 2009-01… 2017-06-… 10.1… 2018-0… <NA>  2008-… 78    
#>  5 Books Abroad       2010-11… 2010-11-… 10.2… 2017-1… <NA>  1941   1121  
#>  6 Acta Psychiatrica… 2007-08… 2017-04-… 10.1… 2017-1… <NA>  1960-… 311   
#>  7 Cancer Biology & … 2016-06… 2016-12-… 10.1… 2018-0… <NA>  2016-… 301   
#>  8 Solar Energy       2003-11… 2015-06-… 10.1… 2018-0… <NA>  1993-… 78    
#>  9 International Jou… 2014-11… 2014-11-… 10.1… 2017-1… <NA>  2014-… 7168  
#> 10 Cytokine           2011-09… 2016-01-… 10.1… 2017-1… <NA>  2011-… 78    
#> # ... with 90 more rows, and 26 more variables: prefix <chr>,
#> #   publisher <chr>, reference.count <chr>, score <chr>, source <chr>,
#> #   title <chr>, type <chr>, URL <chr>, author <list>, link <list>,
#> #   alternative.id <chr>, ISSN <chr>, issue <chr>, license_date <chr>,
#> #   license_URL <chr>, license_delay.in.days <chr>,
#> #   license_content.version <chr>, page <chr>, subject <chr>,
#> #   volume <chr>, update.policy <chr>, assertion <list>, funder <list>,
#> #   abstract <chr>, subtitle <chr>, archive <chr>
```

Let's see when these random publications were published


```r
random_dois %>%
  # convert to years
  mutate(issued, issued = lubridate::parse_date_time(issued, c('y', 'ymd', 'ym'))) %>%
  mutate(issued, issued = lubridate::year(issued)) %>%
  group_by(issued) %>%
  summarize(pubs = n()) %>%
  arrange(desc(pubs))
#> # A tibble: 45 x 2
#>    issued  pubs
#>     <dbl> <int>
#>  1     NA    10
#>  2   2017     8
#>  3   2011     6
#>  4   2013     5
#>  5   1995     4
#>  6   1996     4
#>  7   1997     4
#>  8   2014     4
#>  9   2015     4
#> 10   1993     3
#> # ... with 35 more rows
```

and of what type they are


```r
random_dois %>%
  group_by(type) %>%
  summarize(pubs = n()) %>%
  arrange(desc(pubs))
#> # A tibble: 8 x 2
#>   type                 pubs
#>   <chr>               <int>
#> 1 journal-article        73
#> 2 book-chapter           12
#> 3 proceedings-article     7
#> 4 component               3
#> 5 book                    2
#> 6 dataset                 1
#> 7 journal-issue           1
#> 8 standard                1
```

#### Calling Unpaywall

Now let's call Unpaywall. We are capturing possible errors.


```r
oa_df <- purrr::map(random_dois$DOI, .f = purrr::safely(
  function(x) roadoi::oadoi_fetch(x, email = "najko.jahn@gmail.com")
  )) %>%
  purrr::map_df("result")
```

and merge the resulting information about open access full-text links with parts of our Crossref metadata-set


```r
my_df <- random_dois %>%
  select(DOI, type) %>% 
  left_join(oa_df, by = c("DOI" = "doi"))
```

#### Reporting

After gathering the data, reporting with R is very straightforward. You can even generate dynamic reports using [R Markdown](http://rmarkdown.rstudio.com/) and related packages, thus making your study reproducible and transparent for others.

To display how many full-text links were found and which sources were used in a nicely formatted markdown-table using the [`knitr`](https://yihui.name/knitr/)-package:


```r
my_df %>%
  group_by(is_oa) %>%
  summarise(Articles = n()) %>%
  mutate(Proportion = Articles / sum(Articles)) %>%
  arrange(desc(Articles)) %>%
  knitr::kable()
```



|is_oa | Articles| Proportion|
|:-----|--------:|----------:|
|FALSE |       85|       0.85|
|TRUE  |       15|       0.15|

How did oaDOI find those Open Access full-texts, which were characterized as best matches, and how are these OA types distributed over publication types?


```r
my_df %>%
  filter(is_oa == TRUE) %>%
  tidyr::unnest(best_oa_location) %>% 
  group_by(evidence, type) %>%
  summarise(Articles = n()) %>%
  arrange(desc(Articles)) %>%
  knitr::kable()
```



|evidence                                                 |type            | Articles|
|:--------------------------------------------------------|:---------------|--------:|
|open (via free pdf)                                      |journal-article |        7|
|oa journal (via publisher name)                          |component       |        2|
|oa repository (via OAI-PMH doi match)                    |journal-article |        2|
|oa repository (via OAI-PMH title and first author match) |book-chapter    |        1|
|open (via crossref license)                              |journal-article |        1|
|open (via page says license)                             |journal-article |        1|
|open (via page says Open Access)                         |journal-article |        1|

#### More examples

For more  examples, see Piwowar et al. 2018.[^1] Together with the article, they shared their analysis of oaDOI-data as [R Markdown supplement](https://github.com/Impactstory/oadoi-paper1/).

[^1]: Piwowar, H., Priem, J., Larivière, V., Alperin, J. P., Matthias, L., Norlander, B., … Haustein, S. (2018). The state of OA: a large-scale analysis of the prevalence and impact of Open Access articles. PeerJ, 6, e4375. <https://doi.org/10.7717/peerj.4375>

## Meta

Please note that this project is released with a [Contributor Code of Conduct](CONDUCT.md). By participating in this project you agree to abide by its terms.

License: MIT

Please use the [issue tracker](https://github.com/ropensci/roadoi/issues) for bug reporting and feature requests.

[![ropensci_footer](https://ropensci.org/public_images/ropensci_footer.png)](https://ropensci.org)
