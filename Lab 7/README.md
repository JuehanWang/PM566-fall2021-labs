Lab 7
================
Juehan Wang
10/8/2021

## Learning goals

Use a real world API to make queries and process the data.

Use regular expressions to parse the information.

Practice your GitHub skills.

## Lab description

In this lab, we will be working with the NCBI API to make queries and
extract information using XML and regular expressions. For this lab, we
will be using the httr, xml2, and stringr R packages.

This markdown document should be rendered using github\_document
document.

### Question 1: How many sars-cov-2 papers?

Build an automatic counter of sars-cov-2 papers using PubMed. You will
need to apply XPath as we did during the lecture to extract the number
of results returned by PubMed in the following web address:

<https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2>

Complete the lines of code:

``` r
# Downloading the website
website <- xml2::read_html("https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2")

# Finding the counts
counts <- xml2::xml_find_first(website, "/html/body/main/div[9]/div[2]/div[2]/div[1]/span")

# Turning it into text
# or xml2::xml_text(counts)
counts <- as.character(counts)

# Extracting the data using regex
stringr::str_extract(counts, "[0-9,]+")
```

    ## [1] "114,592"

``` r
stringr::str_extract(counts, "[[:digit:],]+")
```

    ## [1] "114,592"

Question 2: Academic publications on COVID19 and Hawaii

You need to query the following The parameters passed to the query are
documented here.

Use the function httr::GET() to make the following query:

Baseline URL:
<https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi>

Query parameters:

db: pubmed

term: covid19 hawaii

retmax: 1000

``` r
library(httr)
query_ids <- GET(
  url    = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi",
  query  = list(db = "pubmed",
  term   = "covid19 hawaii",
  retmax = 1000)
)
# or
query_ids <- GET(
  url    = "https://eutils.ncbi.nlm.nih.gov/",
  path   = "entrez/eutils/esearch.fcgi",
  query  = list(db = "pubmed",
  term   = "covid19 hawaii",
  retmax = 1000)
)
# Extracting the content of the response of GET
ids <- httr::content(query_ids)
```

The query will return an XML object, we can turn it into a character
list to analyze the text directly with as.character(). Another way of
processing the data could be using lists with the function
xml2::as\_list(). We will skip the latter for now.

``` r
ids_list <- xml2::as_list(ids)
str(ids_list)
```

    ## List of 1
    ##  $ eSearchResult:List of 7
    ##   ..$ Count           :List of 1
    ##   .. ..$ : chr "150"
    ##   ..$ RetMax          :List of 1
    ##   .. ..$ : chr "150"
    ##   ..$ RetStart        :List of 1
    ##   .. ..$ : chr "0"
    ##   ..$ IdList          :List of 150
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34562997"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34559481"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34545941"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34536350"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34532685"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34529634"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34499878"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34491990"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34481278"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34473201"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34448649"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34417121"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34406840"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34391908"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34367726"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34355196"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34352507"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34334985"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34314211"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34308400"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34308322"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34291832"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34287651"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34287159"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34283939"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34254888"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34228774"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34226774"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34210370"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34195618"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34189029"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34183789"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34183411"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34183191"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34180390"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34140009"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34125658"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34108898"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34102878"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34091576"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "34062806"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33990619"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33982008"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33980567"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33973241"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33971389"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33966879"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33938253"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33929934"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33926498"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33900192"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33897904"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33894385"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33889849"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33889848"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33859192"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33856881"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33851191"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33826985"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33789080"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33781762"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33781585"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33775167"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33770003"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33769536"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33746047"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33728687"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33718878"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33717793"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33706209"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33661861"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33661727"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33657176"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33655229"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33607081"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33606666"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33606656"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33587873"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33495523"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33482708"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33471778"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33464637"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33442699"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33422679"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33422626"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33417334"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33407957"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33331197"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33316097"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33308888"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33301024"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33276110"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33270782"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33251328"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33244071"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33236896"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33229999"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33216726"
    ##   .. ..$ Id:List of 1
    ##   .. .. ..$ : chr "33193454"
    ##   .. .. [list output truncated]
    ##   ..$ TranslationSet  :List of 2
    ##   .. ..$ Translation:List of 2
    ##   .. .. ..$ From:List of 1
    ##   .. .. .. ..$ : chr "covid19"
    ##   .. .. ..$ To  :List of 1
    ##   .. .. .. ..$ : chr "\"covid-19\"[MeSH Terms] OR \"covid-19\"[All Fields] OR \"covid19\"[All Fields]"
    ##   .. ..$ Translation:List of 2
    ##   .. .. ..$ From:List of 1
    ##   .. .. .. ..$ : chr "hawaii"
    ##   .. .. ..$ To  :List of 1
    ##   .. .. .. ..$ : chr "\"hawaii\"[MeSH Terms] OR \"hawaii\"[All Fields]"
    ##   ..$ TranslationStack:List of 12
    ##   .. ..$ TermSet:List of 4
    ##   .. .. ..$ Term   :List of 1
    ##   .. .. .. ..$ : chr "\"covid-19\"[MeSH Terms]"
    ##   .. .. ..$ Field  :List of 1
    ##   .. .. .. ..$ : chr "MeSH Terms"
    ##   .. .. ..$ Count  :List of 1
    ##   .. .. .. ..$ : chr "110249"
    ##   .. .. ..$ Explode:List of 1
    ##   .. .. .. ..$ : chr "Y"
    ##   .. ..$ TermSet:List of 4
    ##   .. .. ..$ Term   :List of 1
    ##   .. .. .. ..$ : chr "\"covid-19\"[All Fields]"
    ##   .. .. ..$ Field  :List of 1
    ##   .. .. .. ..$ : chr "All Fields"
    ##   .. .. ..$ Count  :List of 1
    ##   .. .. .. ..$ : chr "175423"
    ##   .. .. ..$ Explode:List of 1
    ##   .. .. .. ..$ : chr "N"
    ##   .. ..$ OP     :List of 1
    ##   .. .. ..$ : chr "OR"
    ##   .. ..$ TermSet:List of 4
    ##   .. .. ..$ Term   :List of 1
    ##   .. .. .. ..$ : chr "\"covid19\"[All Fields]"
    ##   .. .. ..$ Field  :List of 1
    ##   .. .. .. ..$ : chr "All Fields"
    ##   .. .. ..$ Count  :List of 1
    ##   .. .. .. ..$ : chr "2283"
    ##   .. .. ..$ Explode:List of 1
    ##   .. .. .. ..$ : chr "N"
    ##   .. ..$ OP     :List of 1
    ##   .. .. ..$ : chr "OR"
    ##   .. ..$ OP     :List of 1
    ##   .. .. ..$ : chr "GROUP"
    ##   .. ..$ TermSet:List of 4
    ##   .. .. ..$ Term   :List of 1
    ##   .. .. .. ..$ : chr "\"hawaii\"[MeSH Terms]"
    ##   .. .. ..$ Field  :List of 1
    ##   .. .. .. ..$ : chr "MeSH Terms"
    ##   .. .. ..$ Count  :List of 1
    ##   .. .. .. ..$ : chr "8095"
    ##   .. .. ..$ Explode:List of 1
    ##   .. .. .. ..$ : chr "Y"
    ##   .. ..$ TermSet:List of 4
    ##   .. .. ..$ Term   :List of 1
    ##   .. .. .. ..$ : chr "\"hawaii\"[All Fields]"
    ##   .. .. ..$ Field  :List of 1
    ##   .. .. .. ..$ : chr "All Fields"
    ##   .. .. ..$ Count  :List of 1
    ##   .. .. .. ..$ : chr "29261"
    ##   .. .. ..$ Explode:List of 1
    ##   .. .. .. ..$ : chr "N"
    ##   .. ..$ OP     :List of 1
    ##   .. .. ..$ : chr "OR"
    ##   .. ..$ OP     :List of 1
    ##   .. .. ..$ : chr "GROUP"
    ##   .. ..$ OP     :List of 1
    ##   .. .. ..$ : chr "AND"
    ##   .. ..$ OP     :List of 1
    ##   .. .. ..$ : chr "GROUP"
    ##   ..$ QueryTranslation:List of 1
    ##   .. ..$ : chr "(\"covid-19\"[MeSH Terms] OR \"covid-19\"[All Fields] OR \"covid19\"[All Fields]) AND (\"hawaii\"[MeSH Terms] O"| __truncated__

Take a look at the data, and continue with the next question (don’t
forget to commit and push your results to your GitHub repo!).

### Question 3: Get details about the articles

The Ids are wrapped around text in the following way: <Id>… id number
…</Id>. we can use a regular expression that extract that information.
Fill out the following lines of code:

``` r
# Turn the result into a character vector
ids <- as.character(ids)

# Find all the ids 
ids <- stringr::str_extract_all(ids, "<Id>[[:digit:]]+</Id>")[[1]]

# Remove all the leading and trailing <Id> </Id>. Make use of "|"
ids <- stringr::str_remove_all(ids, "</?Id>")
ids <- stringr::str_remove_all(ids, "<Id>|</Id>")
```

With the ids in hand, we can now try to get the abstracts of the papers.
As before, we will need to coerce the contents (results) to a list
using:

Baseline url:
<https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi>

Query parameters:

db: pubmed

id: A character with all the ids separated by comma, e.g.,
“1232131,546464,13131”

retmax: 1000

rettype: abstract

Pro-tip: If you want GET() to take some element literal, wrap it around
I() (as you would do in a formula in R). For example, the text “123,456”
is replaced with “123%2C456”. If you don’t want that behavior, you would
need to do the following I(“123,456”).

``` r
publications <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/",
  path = "entrez/eutils/efetch.fcgi",
  query = list(
    db = "pubmed",
    id = I(paste(ids, collapse = ",")),
    retmax = 1000,
    rettype = "abstract"
    )
)
# Turning the output into character vector
publications <- httr::content(publications)
publications_txt <- as.character(publications)
```

With this in hand, we can now analyze the data.
