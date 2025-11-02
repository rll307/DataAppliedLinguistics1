# Concordances and distribution

Rodrigo Esteves de Lima Lopes\
_Universidade Estadual de Campinas_
[rll307@unicamp.br](mailto:rll307@unicamp.br)

# Introduction
In this script we are going to study the more frequent words of our corpus, comparing their use. 

# Packages
```r
library(BrPoliCorpus)
download_index()
library(quanteda)
library(tidytext)
library(dplyr)
```
# Data
We will use the same data from last script, so:

```r
DiscursosPresidetes <- download_InauguralSpeeches()
Meu.corpus <- corpus(DiscursosPresidetes, docid_field = "doc_id", text_field = "text")
Minha.DFM <- dfm(Meus.Tokens) %>%
  dfm_remove(stopwords("portuguese"))
```

# More important words
A common procedure for the calculation of important words in a corpus is the TF-IDF approach:

- **TF (Term Frequency)** = the frequency of a word in a document
- **IDF (Inverse Document Frequency)** = how rare the word is across documents
- **TF-IDF** = highlights words that are frequent in a document but uncommon across the corpus
```r
Meu.tfidf <- dfm_tfidf(Minha.DFM)
Mais_frequentes_tfidf <- topfeatures(Meu.tfidf, n = 30)
print(Mais_frequentes_tfidf)
```

I can see the five more important in each doc:

```r
resultados <- lapply(1:ndoc(Meu.tfidf), function(i) {
  topfeatures(Meu.tfidf[i, ], n = 5)
})
print(resultados)
```

## More elegantly presented

We can see better if we use other commands with similar result:
```r
Minha.df.tidy <- tidy(Minha.DFM)
Minha.df.tidy <- Minha.df.tidy %>%
  bind_tf_idf(term, document, count)
```

Finally let us make a concordance list and see some results:
```r
kwic(Meus.Tokens, pattern = "fom*",window = 25,valuetype="glob") %>% View()
kwic(Meus.Tokens, pattern = "fome", window = 25, valuetype = "fixed") %>% View()
```

