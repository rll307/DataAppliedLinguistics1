# Textual Analysis of Brazilian Presidential Speeches (2022)
## Using `quanteda`, `rainette`, and `BrPoliCorpus` in R

Rodrigo Esteves de Lima Lopes\
_Universidade Estadual de Campinas_
[rll307@unicamp.br](mailto:rll307@unicamp.br)

# Introduction
This walhthrough shows how to download, preprocess, and analyze Brazilian presidential speeches from 2022 using R.   It applies tokenization, type-token ratio (TTR) computation, stopword filtering, document-feature matrix (DFM) creation, and segment-level clustering via `rainette`.

### Packages
If the packages are not installed, this command will do it for you:
```r
install.packages(c("quanteda", "quanteda.textstats", "dplyr", "ggplot2", "rainette"))
devtools::install_github("rll307/BrPoliCorpus")
```

| Package | Purpose |
|----------|----------|
| **quanteda** | Text processing (tokenization, corpus, dfm) |
| **quanteda.textstats** | Lexical statistics and diversity measures |
| **rainette** | Segment-based clustering for textual data |
| **dplyr** | Data wrangling and manipulation |
| **BrPoliCorpus** | Download political speech corpora from Brazil |


##  Data Download

The `BrPoliCorpus` package provides direct access to speech datasets.

```r
library(BrPoliCorpus)

# Download the index of available corpora
download_index()

# Download presidential data from 2022
Brasil_president_2022 <- download_Brasil_president_2022()
```

>  `download_index()` shows all available datasets. Use `View(IndexFunctions)` interactively to explore options.

## Data Preparation

Remove unwanted entries and create a document ID:

```r
Brasil_president_2022 <- Brasil_president_2022[-9, ]  # remove row 9
Brasil_president_2022 <- Brasil_president_2022 %>%
  mutate(ID = paste(Partido, Número, sep = "_"))
```

##  Corpus Creation

```r
library(quanteda)

presidents <- corpus(
  Brasil_president_2022,
  text_field = "text",
  docid_field = "ID"
)
```

Creates a `quanteda` corpus, using `text` as the content field and `ID` as document identifier.

## Lexical Richness (TTR)

Compute token and type counts:

```r
Presidents.TTR <- corpus(Brasil_president_2022, text_field = "text", docid_field = "ID")

ttr <- data.frame(
  Tokens = ntoken(Presidents.TTR),
  Types = ntype(Presidents.TTR)
)

summary_stats <- apply(ttr, 2, function(x) {
  c(
    min = min(x, na.rm = TRUE),
    max = max(x, na.rm = TRUE),
    mean = mean(x, na.rm = TRUE),
    sd = sd(x, na.rm = TRUE)
  )
}) %>%
  as.data.frame() %>%
  mutate_if(is.numeric, ~ round(., 2))

round(sum(ttr$Types) / sum(ttr$Tokens), 2)

por_partido <- data.frame(
  tokens = ntoken(Presidents.TTR),
  types = ntype(Presidents.TTR)
)
```

**Explanation:**  

```r
apply(ttr, 2, function(x) { ... })
```

- The `apply()` function runs a custom function over the **rows or columns** of a matrix or data frame.
- The argument `2` means it applies the function **to each column**.
- So, for every column `x` in the data frame `ttr`, the code inside the function is executed.

For each column `x`, the code calculates:

```r
c(
  min = min(x, na.rm = TRUE),
  max = max(x, na.rm = TRUE),
  mean = mean(x, na.rm = TRUE),
  sd = sd(x, na.rm = TRUE)
)
```

This produces a vector with:
- `min`: the minimum value, ignoring missing values (`NA`s)
- `max`: the maximum value
- `mean`: the average (arithmetic mean)
- `sd`: the standard deviation

The `apply()` call returns a **matrix**, where:
- Each column corresponds to a variable from `ttr`.
- Each row corresponds to a summary statistic (`min`, `max`, `mean`, `sd`).

```r
as.data.frame()
```

Converts the matrix into a **data frame**, making it easier to handle with `dplyr` and other tidyverse tools.

```r
mutate_if(is.numeric, ~ round(., 2))
```
- This command looks for all numeric columns.
- It rounds all their values to **two decimal places** using `round()`.

- `ntoken()` counts total tokens per document.  
- `ntype()` counts unique word forms.  
- The ratio between total types and tokens gives a measure of lexical diversity.

## Segmenting the Corpus

```r
My.corpus <- split_segments(presidents, segment_size = 30)
```

Splits long speeches into smaller segments (≈30 tokens).  
This allows clustering algorithms like `rainette()` to identify local topic structures.

---

## Stopword Handling

```r
MySWL <- stopwords("pt") |> as.data.frame()
MySWL <- MySWL[-c(25, 28, 29, 31, 35, 42, 54, 58, 76, 79.80, 81:88, 100, 103:121, 147, 148, 149:203, 12), ]
MySWL <- c(MySWL, "s", "i", "p", "https", "m", "c", "n", "t", "z", "í", "2022", "2023", "2026", "$", "67", "nº", "ptb", "fi", "r", "fi", "ﬁ", "eﬁ", "ﬁ", "ﬂ", "proﬁ")
```
## Tokenization

```r
My.tok <- tokens(
  My.corpus,
  remove_punct = TRUE,
  remove_symbols = TRUE,
  remove_numbers = TRUE,
  verbose = TRUE
) %>% tokens_tolower()

My.tok <- tokens_remove(My.tok, MySWL, verbose = TRUE)
```

**Explanation:**  
- Removes punctuation, symbols, and numbers.  
- Converts everything to lowercase.  
- Removes Portuguese stopwords and extra undesired tokens.

## DFM Construction and Cleaning

```r
dfm.presidents <- dfm(My.tok)
dfm.presidents <- dfm_select(dfm.presidents, pattern = "fi", selection = "remove")
My.dfm <- dfm_trim(dfm.presidents, min_docfreq = 5)
```

Creates a **Document-Feature Matrix (DFM)** and trims rare terms (appearing in fewer than 5 segments).

---

##  Segment Clustering with `rainette`

```r
res <- rainette(My.dfm, k = 4, min_segment_size = 15)

rainette_plot(
  res, My.dfm,
  k = 4, n_terms = 20,
  measure = "lr",
  show_negative = FALSE,
  text_size = 12
)

My.corpus$cluster <- cutree(res, k = 4)
groups <- cutree_rainette(res, k = 4)

Explore_groups <- rainette_stats(
  groups, My.dfm,
  measure = "lr",
  n_terms = 30,
  show_negative = TRUE,
  max_p = 0.05
)
```

**Explanation:**
- `rainette()` performs hierarchical clustering of segments.
- `rainette_plot()` visualizes the most frequent or discriminant terms in each cluster.
- `rainette_stats()` provides per-cluster statistics.

##  Plotting

- `rainette_plot()` produces a faceted plot of the top words per cluster.
- `Explore_groups` is a data frame summarizing distinctive words for each group.

Exploring corpus with detail:

```r
rainette_explor(res, My.dfm, My.corpus)
```
