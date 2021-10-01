---
title: "Lab 5"
author: "Juehan Wang"
date: "9/24/2021"
output:
    html_document:
      toc: yes 
      toc_float: yes 
      keep_md: yes
    github_document:
      keep_html: true
      html_preview: false
always_allow_html: true
---





First, download the data.


```r
fn <- "mtsamples.csv"
if (!file.exists(fn))
  download.file("https://raw.githubusercontent.com/USCbiostats/data-science-data/master/00_mtsamples/mtsamples.csv", destfile = fn)

mtsamples<-read.csv(fn)
mtsamples<-as_tibble(mtsamples)
```

### Question 1: What specialties do we have?

We can use count() from dplyr to figure out how many different categories do we have? Are these categories related? overlapping? evenly distributed?


```r
specialties <- mtsamples %>%
  count(medical_specialty, sort = TRUE)
```

There are 40 specialties. Let's take a look at the distributions.


```r
ggplot(mtsamples, aes(x = medical_specialty)) +
  geom_histogram(stat = "count") +
  coord_flip()
```

```
## Warning: Ignoring unknown parameters: binwidth, bins, pad
```

![](README_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r
specialties %>%
  arrange(desc(n)) %>%
  top_n(n, 15) %>%
  knitr::kable()
```

```
## Warning in if (n > 0) {: the condition has length > 1 and only the first element
## will be used
```



|medical_specialty             |    n|
|:-----------------------------|----:|
|Surgery                       | 1103|
|Consult - History and Phy.    |  516|
|Cardiovascular / Pulmonary    |  372|
|Orthopedic                    |  355|
|Radiology                     |  273|
|General Medicine              |  259|
|Gastroenterology              |  230|
|Neurology                     |  223|
|SOAP / Chart / Progress Notes |  166|
|Obstetrics / Gynecology       |  160|
|Urology                       |  158|
|Discharge Summary             |  108|
|ENT - Otolaryngology          |   98|
|Neurosurgery                  |   94|
|Hematology - Oncology         |   90|
|Ophthalmology                 |   83|
|Nephrology                    |   81|
|Emergency Room Reports        |   75|
|Pediatrics - Neonatal         |   70|
|Pain Management               |   62|
|Psychiatry / Psychology       |   53|
|Office Notes                  |   51|
|Podiatry                      |   47|
|Dermatology                   |   29|
|Cosmetic / Plastic Surgery    |   27|
|Dentistry                     |   27|
|Letters                       |   23|
|Physical Medicine - Rehab     |   21|
|Sleep Medicine                |   20|
|Endocrinology                 |   19|
|Bariatrics                    |   18|
|IME-QME-Work Comp etc.        |   16|
|Chiropractic                  |   14|
|Diets and Nutritions          |   10|
|Rheumatology                  |   10|
|Speech - Language             |    9|
|Autopsy                       |    8|
|Lab Medicine - Pathology      |    8|
|Allergy / Immunology          |    7|
|Hospice - Palliative Care     |    6|









