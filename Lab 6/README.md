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





Notes:

  Add README files -- git add "Lab 6/README*"
  
  Remove cache files before committing -- git rm --cache "Lab 6/README_cache*"
  
  Then commit -- git commit -a -m "Lab 6 ..."
  
  Finally push -- git push



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

![](README_files/figure-html/dist1-1.png)<!-- -->

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


```r
ggplot(specialties, aes(x = n, y = fct_reorder(medical_specialty,n))) +
  geom_col()
```

![](README_files/figure-html/dist2-1.png)<!-- -->

These are not evenly (uniformly) distributed.

### Question 2

Tokenize the the words in the transcription column

Count the number of times each token appears

Visualize the top 20 most frequent words

Explain what we see from this result. Does it makes sense? What insights (if any) do we get?


```r
mtsamples %>%
  unnest_tokens(output = word, input = transcription) %>%
  count(word, sort = TRUE) %>%
  top_n(20) %>%
  ggplot(aes(x = n, y = fct_reorder(word,n))) +
    geom_col()
```

```
## Selecting by n
```

![](README_files/figure-html/token-trans-1.png)<!-- -->

The word "status" seems to be important (duh!), but we observe a lot of stopwords.

### Question 3

What do we see know that we have removed stop words? Does it give us a better idea of what the text is about?


```r
# Redo visualization but remove stopwords before
mtsamples %>%
  unnest_tokens(output = word, input = transcription) %>%
  count(word, sort = TRUE) %>%
  anti_join(stop_words, by = "word") %>%
  # using regular expressions to remove numbers
  filter(!grepl(pattern = "^[0-9]+$", x = word)) %>%
  top_n(20) %>%
  ggplot(aes(x = n, y = fct_reorder(word,n))) +
    geom_col()
```

```
## Selecting by n
```

![](README_files/figure-html/token-trans-wo-stop-1.png)<!-- -->

### Question 4

repeat question 2, but this time tokenize into bi-grams. how does the result change if you look at tri-grams?


```r
mtsamples %>%
  unnest_ngrams(output = bigram, input = transcription, n = 2) %>%
  count(bigram, sort = TRUE) %>%
  top_n(20) %>%
  ggplot(aes(x = n, y = fct_reorder(bigram,n))) +
    geom_col()
```

```
## Selecting by n
```

![](README_files/figure-html/bigram-trans-1.png)<!-- -->

Using bi-grams is not very informative, let's try tri-grams.


```r
mtsamples %>%
  unnest_ngrams(output = trigram, input = transcription, n = 3) %>%
  count(trigram, sort = TRUE) %>%
  top_n(20) %>%
  ggplot(aes(x = n, y = fct_reorder(trigram,n))) +
    geom_col()
```

```
## Selecting by n
```

![](README_files/figure-html/trigram-trans-1.png)<!-- -->

Now some phrases start to show up, e.g. "tolerated the procedure", "prepped and draped".

### Question 5

Using the results you got from questions 4. Pick a word and count the words that appears after and before it.


```r
bigrams <- mtsamples %>%
  unnest_ngrams(output = bigram, input = transcription, n = 3) %>%
  separate(bigram, into = c("w1", "w2"), sep = " ")

# before
bigrams %>%
  filter(w1 == "status") %>%
  select(w1,w2) %>%
  count(w2, sort = TRUE)
```

```
## # A tibble: 98 × 2
##    w2              n
##    <chr>       <int>
##  1 post          783
##  2 intact         53
##  3 and            43
##  4 examination    43
##  5 exam           39
##  6 is             34
##  7 was            27
##  8 the            25
##  9 at             17
## 10 changes        17
## # … with 88 more rows
```

```r
# after
bigrams %>%
  filter(w2 == "status") %>%
  select(w1,w2) %>%
  count(w1, sort = TRUE)
```

```
## # A tibble: 210 × 2
##    w1              n
##    <chr>       <int>
##  1 mental        212
##  2 is             88
##  3 vascular       67
##  4 disease        65
##  5 2              40
##  6 3              33
##  7 performance    27
##  8 1              24
##  9 cancer         24
## 10 diagnosis      20
## # … with 200 more rows
```

Since we are looking at single words again, it is a good idea to treat these as single tokens. So let's rename the stopwords and the numbers.


```r
bigrams %>%
  filter(w1 == "status") %>%
  filter(!(w2 %in% stop_words$word) & !grepl(pattern = "^[0-9]+$", w2)) %>%
  count(w2, sort = TRUE) %>%
  top_n(10) %>%
  knitr::kable(caption = "Words AFTER 'status'")
```

```
## Selecting by n
```



Table: Words AFTER 'status'

|w2          |   n|
|:-----------|---:|
|post        | 783|
|intact      |  53|
|examination |  43|
|exam        |  39|
|change      |   9|
|rbans       |   9|
|epilepticus |   8|
|married     |   8|
|assessed    |   7|
|history     |   7|

```r
bigrams %>%
  filter(w2 == "status") %>%
  filter(!(w1 %in% stop_words$word) & !grepl(pattern = "^[0-9]+$", w1)) %>%
  count(w1, sort = TRUE) %>%
  top_n(10) %>%
  knitr::kable(caption = "Words BEFORE 'status'")
```

```
## Selecting by n
```



Table: Words BEFORE 'status'

|w1          |   n|
|:-----------|---:|
|mental      | 212|
|vascular    |  67|
|disease     |  65|
|performance |  27|
|cancer      |  24|
|diagnosis   |  20|
|pain        |  18|
|respiratory |  16|
|cholesterol |  14|
|marital     |  14|

### Question 6

Which words are most used in each of the specialties. you can use group_by() and top_n() from dplyr to have the calculations be done within each specialty. Remember to remove stopwords. How about the most 5 used words?


```r
mtsamples %>%
  unnest_tokens(word, input = transcription) %>%
  group_by(medical_specialty) %>%
  count(word, sort = TRUE) %>%
  filter(!(word %in% stop_words$word) & !grepl("^[0-9]+$", word)) %>%
  top_n(5) %>%
  arrange(medical_specialty, desc(n)) %>%
  knitr::kable()
```

```
## Selecting by n
```



|medical_specialty             |word         |    n|
|:-----------------------------|:------------|----:|
|Allergy / Immunology          |history      |   38|
|Allergy / Immunology          |noted        |   23|
|Allergy / Immunology          |patient      |   22|
|Allergy / Immunology          |allergies    |   21|
|Allergy / Immunology          |nasal        |   13|
|Allergy / Immunology          |past         |   13|
|Autopsy                       |left         |   83|
|Autopsy                       |inch         |   59|
|Autopsy                       |neck         |   55|
|Autopsy                       |anterior     |   47|
|Autopsy                       |body         |   40|
|Bariatrics                    |patient      |   62|
|Bariatrics                    |history      |   50|
|Bariatrics                    |weight       |   36|
|Bariatrics                    |surgery      |   34|
|Bariatrics                    |gastric      |   30|
|Cardiovascular / Pulmonary    |left         | 1550|
|Cardiovascular / Pulmonary    |patient      | 1516|
|Cardiovascular / Pulmonary    |artery       | 1085|
|Cardiovascular / Pulmonary    |coronary     |  681|
|Cardiovascular / Pulmonary    |history      |  654|
|Chiropractic                  |pain         |  187|
|Chiropractic                  |patient      |   85|
|Chiropractic                  |dr           |   66|
|Chiropractic                  |history      |   56|
|Chiropractic                  |left         |   54|
|Consult - History and Phy.    |patient      | 3046|
|Consult - History and Phy.    |history      | 2820|
|Consult - History and Phy.    |normal       | 1368|
|Consult - History and Phy.    |pain         | 1153|
|Consult - History and Phy.    |mg           |  908|
|Cosmetic / Plastic Surgery    |patient      |  116|
|Cosmetic / Plastic Surgery    |procedure    |   98|
|Cosmetic / Plastic Surgery    |breast       |   95|
|Cosmetic / Plastic Surgery    |skin         |   88|
|Cosmetic / Plastic Surgery    |incision     |   67|
|Dentistry                     |patient      |  195|
|Dentistry                     |tooth        |  108|
|Dentistry                     |teeth        |  104|
|Dentistry                     |left         |   94|
|Dentistry                     |procedure    |   82|
|Dermatology                   |patient      |  101|
|Dermatology                   |skin         |  101|
|Dermatology                   |cm           |   77|
|Dermatology                   |left         |   58|
|Dermatology                   |procedure    |   44|
|Diets and Nutritions          |patient      |   43|
|Diets and Nutritions          |weight       |   40|
|Diets and Nutritions          |carbohydrate |   37|
|Diets and Nutritions          |day          |   28|
|Diets and Nutritions          |food         |   27|
|Diets and Nutritions          |plan         |   27|
|Discharge Summary             |patient      |  672|
|Discharge Summary             |discharge    |  358|
|Discharge Summary             |mg           |  301|
|Discharge Summary             |history      |  208|
|Discharge Summary             |hospital     |  183|
|Emergency Room Reports        |patient      |  685|
|Emergency Room Reports        |history      |  356|
|Emergency Room Reports        |pain         |  273|
|Emergency Room Reports        |normal       |  255|
|Emergency Room Reports        |denies       |  149|
|Endocrinology                 |thyroid      |  129|
|Endocrinology                 |patient      |  121|
|Endocrinology                 |left         |   63|
|Endocrinology                 |history      |   57|
|Endocrinology                 |dissection   |   45|
|Endocrinology                 |gland        |   45|
|Endocrinology                 |nerve        |   45|
|ENT - Otolaryngology          |patient      |  415|
|ENT - Otolaryngology          |nasal        |  281|
|ENT - Otolaryngology          |left         |  219|
|ENT - Otolaryngology          |ear          |  182|
|ENT - Otolaryngology          |procedure    |  181|
|Gastroenterology              |patient      |  872|
|Gastroenterology              |procedure    |  470|
|Gastroenterology              |history      |  341|
|Gastroenterology              |normal       |  328|
|Gastroenterology              |colon        |  240|
|General Medicine              |patient      | 1356|
|General Medicine              |history      | 1027|
|General Medicine              |normal       |  717|
|General Medicine              |pain         |  567|
|General Medicine              |mg           |  503|
|Hematology - Oncology         |patient      |  316|
|Hematology - Oncology         |history      |  290|
|Hematology - Oncology         |left         |  187|
|Hematology - Oncology         |mg           |  107|
|Hematology - Oncology         |mass         |   97|
|Hospice - Palliative Care     |patient      |   43|
|Hospice - Palliative Care     |mg           |   28|
|Hospice - Palliative Care     |history      |   27|
|Hospice - Palliative Care     |daughter     |   22|
|Hospice - Palliative Care     |family       |   19|
|Hospice - Palliative Care     |pain         |   19|
|IME-QME-Work Comp etc.        |pain         |  152|
|IME-QME-Work Comp etc.        |patient      |  106|
|IME-QME-Work Comp etc.        |dr           |   82|
|IME-QME-Work Comp etc.        |injury       |   81|
|IME-QME-Work Comp etc.        |left         |   70|
|Lab Medicine - Pathology      |cm           |   35|
|Lab Medicine - Pathology      |tumor        |   35|
|Lab Medicine - Pathology      |lymph        |   30|
|Lab Medicine - Pathology      |lobe         |   29|
|Lab Medicine - Pathology      |upper        |   20|
|Letters                       |pain         |   80|
|Letters                       |abc          |   71|
|Letters                       |patient      |   65|
|Letters                       |normal       |   53|
|Letters                       |dr           |   46|
|Nephrology                    |patient      |  348|
|Nephrology                    |renal        |  257|
|Nephrology                    |history      |  160|
|Nephrology                    |kidney       |  144|
|Nephrology                    |left         |  132|
|Neurology                     |left         |  672|
|Neurology                     |patient      |  648|
|Neurology                     |normal       |  485|
|Neurology                     |history      |  429|
|Neurology                     |time         |  278|
|Neurosurgery                  |patient      |  374|
|Neurosurgery                  |c5           |  289|
|Neurosurgery                  |c6           |  266|
|Neurosurgery                  |procedure    |  247|
|Neurosurgery                  |left         |  222|
|Obstetrics / Gynecology       |patient      |  628|
|Obstetrics / Gynecology       |uterus       |  317|
|Obstetrics / Gynecology       |procedure    |  301|
|Obstetrics / Gynecology       |incision     |  293|
|Obstetrics / Gynecology       |normal       |  276|
|Office Notes                  |normal       |  230|
|Office Notes                  |negative     |  193|
|Office Notes                  |patient      |   94|
|Office Notes                  |history      |   76|
|Office Notes                  |noted        |   60|
|Ophthalmology                 |eye          |  456|
|Ophthalmology                 |patient      |  258|
|Ophthalmology                 |procedure    |  176|
|Ophthalmology                 |anterior     |  150|
|Ophthalmology                 |chamber      |  149|
|Orthopedic                    |patient      | 1711|
|Orthopedic                    |left         |  998|
|Orthopedic                    |pain         |  763|
|Orthopedic                    |procedure    |  669|
|Orthopedic                    |lateral      |  472|
|Pain Management               |patient      |  236|
|Pain Management               |procedure    |  197|
|Pain Management               |needle       |  156|
|Pain Management               |injected     |   76|
|Pain Management               |pain         |   76|
|Pediatrics - Neonatal         |patient      |  247|
|Pediatrics - Neonatal         |history      |  235|
|Pediatrics - Neonatal         |normal       |  155|
|Pediatrics - Neonatal         |child        |   82|
|Pediatrics - Neonatal         |mom          |   82|
|Physical Medicine - Rehab     |patient      |  220|
|Physical Medicine - Rehab     |left         |  104|
|Physical Medicine - Rehab     |pain         |   95|
|Physical Medicine - Rehab     |motor        |   62|
|Physical Medicine - Rehab     |history      |   54|
|Podiatry                      |foot         |  232|
|Podiatry                      |patient      |  231|
|Podiatry                      |left         |  137|
|Podiatry                      |tendon       |   98|
|Podiatry                      |incision     |   96|
|Psychiatry / Psychology       |patient      |  532|
|Psychiatry / Psychology       |history      |  344|
|Psychiatry / Psychology       |mg           |  183|
|Psychiatry / Psychology       |mother       |  164|
|Psychiatry / Psychology       |reported     |  141|
|Radiology                     |left         |  701|
|Radiology                     |normal       |  644|
|Radiology                     |patient      |  304|
|Radiology                     |exam         |  302|
|Radiology                     |mild         |  242|
|Rheumatology                  |history      |   50|
|Rheumatology                  |patient      |   34|
|Rheumatology                  |mg           |   26|
|Rheumatology                  |pain         |   23|
|Rheumatology                  |day          |   22|
|Rheumatology                  |examination  |   22|
|Rheumatology                  |joints       |   22|
|Sleep Medicine                |sleep        |  143|
|Sleep Medicine                |patient      |   69|
|Sleep Medicine                |apnea        |   35|
|Sleep Medicine                |activity     |   31|
|Sleep Medicine                |stage        |   29|
|SOAP / Chart / Progress Notes |patient      |  537|
|SOAP / Chart / Progress Notes |mg           |  302|
|SOAP / Chart / Progress Notes |history      |  254|
|SOAP / Chart / Progress Notes |pain         |  239|
|SOAP / Chart / Progress Notes |blood        |  194|
|Speech - Language             |patient      |  105|
|Speech - Language             |therapy      |   41|
|Speech - Language             |speech       |   35|
|Speech - Language             |patient's    |   28|
|Speech - Language             |evaluation   |   17|
|Speech - Language             |goals        |   17|
|Speech - Language             |term         |   17|
|Speech - Language             |time         |   17|
|Surgery                       |patient      | 4855|
|Surgery                       |left         | 3263|
|Surgery                       |procedure    | 3243|
|Surgery                       |anesthesia   | 1687|
|Surgery                       |incision     | 1641|
|Urology                       |patient      |  776|
|Urology                       |bladder      |  357|
|Urology                       |procedure    |  306|
|Urology                       |left         |  288|
|Urology                       |history      |  196|
