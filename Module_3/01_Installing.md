# Installing

Rodrigo Esteves de Lima Lopes\
_Universidade Estadual de Campinas_
[rll307@unicamp.br](mailto:rll307@unicamp.br)

## Introduction
BrPoliCorpus is a corpus consisting of Brazilian official Brazilian documents. It can be used both as a R package or a set of CSV files freely downloadable here. This manual is dedicated exclusively for the R package basic usage. 

## Objetvie

In this tutorial, we will discuss how to install and have access to the data it offers. 

## Instaling the package
These commands will install the package from [GitHub](https://github.com/rll307/BrPoliCorpus).

```r
library(devtools)
install_github("rll307/BrPoliCorpus")
```

## Data
BrPoliCorpus consists of over 440 billion words. Such large amount of data could not be distributed through GitHub or any other package hosting services. Each time a data function is called, the package will try to download the data from UNICAMP's servers into your R environment. 

# Commands
Here some important commands that will help you out through the corpus:

`download_index()`

Creates six data frames in your environment:

1. IndexProgrammes:
  - Indexes all government proposals available.
2. IndexInauguralSpeeches: 
  - Indexes all Inaugural Speeches available.
3. IndexFloorPlenaries
  - Indexes all Floor Plenaries available.
4. IndexCPI
  - Indexes all Congress Committees of Inquiry available.
5. IndexCommittees
  - Indexes all Congress Committees (_not Inquiry_) available.
6. GeneralIndex
  - Indexes all commands available.

For example, `download_Brasil_president_2014()` downloads the government proposals for the candidates of the Brazilian Presidency in 2014. A common usage would be:

```r
discursos.presidenciais <- download_InauguralSpeeches()
```

This creates a data frame with this part of the corpus. Please refer to `GeneralIndex` for the whole set of commands. The other data frames mentioned above will bring you necessary details, such as the date and the nature of each dataset. 
