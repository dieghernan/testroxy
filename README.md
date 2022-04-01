
<!-- README.md is generated from README.Rmd. Please edit that file -->

# testroxy

<!-- badges: start -->
<!-- badges: end -->

This is an example package for testing **roxygen2** changes.

## Analisis of urls on Description field of `DESCRIPTION` files

Run the following:

``` r
library(stringr)
library(dplyr, warn.conflicts = FALSE)
library(tidyr, warn.conflicts = FALSE)

# Get db
cran <- tools::CRAN_package_db() %>%
  mutate(date_pack = as.Date(str_split_fixed(Packaged, " ", 2)[, 1])) %>%
  select(Package, date_pack, Description)

# Search for patterns
pattern <- "<(DOI|doi|http|https|arXiv|arXiv):(.*?)>"

extract_urls <- str_extract_all(cran$Description,
  regex(pattern),
  simplify = TRUE
) %>%
  as.data.frame() %>%
  bind_cols(cran, .) %>%
  filter(V1 != "") %>%
  select(-Description)

# Analyse patterns
allurls <- extract_urls %>%
  pivot_longer(
    cols = -c(Package, date_pack),
    names_to = "dropfield",
    values_to = "url"
  ) %>%
  # Remove blanks. etc
  filter(url != "" & !is.na(url)) %>%
  mutate(domain = str_split(url, "<|:", simplify = TRUE)[, 2]) %>%
  select(-dropfield)

allurls
#> # A tibble: 13,696 x 4
#>    Package     date_pack  url                                             domain
#>    <chr>       <date>     <chr>                                           <chr> 
#>  1 aaSEA       2019-11-09 <doi:10.4172/2379-1764.1000250>                 doi   
#>  2 abbyyR      2019-06-25 <http://ocrsdk.com/>                            http  
#>  3 abcADM      2019-11-08 <doi:10.1080/00401706.2018.1512900>             doi   
#>  4 ABCanalysis 2017-03-13 <DOI:10.1371/journal.pone.0129767>              DOI   
#>  5 abclass     2022-03-03 <doi:10.1093/biomet/asu017>                     doi   
#>  6 ABCoptim    2017-11-06 <http://mf.erciyes.edu.tr/abc/pub/tr06_2005.pd~ http  
#>  7 abcrf       2019-10-31 <doi:10.1093/bioinformatics/btv684>             doi   
#>  8 abcrf       2019-10-31 <http://journal-sfds.fr/article/view/709>       http  
#>  9 abcrf       2019-10-31 <doi:10.1093/bioinformatics/bty867>             doi   
#> 10 abcrlda     2020-05-27 <doi:10.1109/LSP.2019.2918485>                  doi   
#> # ... with 13,686 more rows


unique(allurls$domain)
#> [1] "doi"   "http"  "DOI"   "arXiv" "https"

# With mask symbols
masked <- allurls %>%
  slice(grep("%", allurls$url))

masked
#> # A tibble: 52 x 4
#>    Package        date_pack  url                                          domain
#>    <chr>          <date>     <chr>                                        <chr> 
#>  1 BivRegBLS      2019-10-10 <https://dial.uclouvain.be/pr/boreal/object~ https 
#>  2 blindrecalc    2021-07-06 <doi:10.1002/(SICI)1097-0258(20000415)19:7%~ doi   
#>  3 clust.bin.pair 2018-02-15 <doi:10.1002/(SICI)1097-0258(19980715)17:13~ doi   
#>  4 clusterSim     2021-01-04 <doi:10.1007%2FBF01908075>                   doi   
#>  5 colorBlindness 2021-04-17 <doi:10.1002/(SICI)1520-6378(199908)24:4%3C~ doi   
#>  6 cvcrand        2020-04-13 <doi:10.1002/1097-0258(20010215)20:3%3C351:~ doi   
#>  7 cvcrand        2020-04-13 <doi:10.1002/(SICI)1097-0258(19960615)15:11~ doi   
#>  8 cvequality     2019-01-07 <DOI:10.1002/(SICI)1097-0258(19960330)15:6%~ DOI   
#>  9 edfReader      2019-03-21 <http://www.teuniz.net/edfbrowser/bdfplus%2~ http  
#> 10 electoral      2022-03-31 <http://cne.gob.ec/documents/Estadisticas/A~ http  
#> # ... with 42 more rows
```
