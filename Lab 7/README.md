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

Question 2: Academic publications on COVID19 and Hawaii You need to
query the following The parameters passed to the query are documented
here.

Use the function httr::GET() to make the following query:

Baseline URL:
<https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi>

Query parameters:

db: pubmed term: covid19 hawaii retmax: 1000
