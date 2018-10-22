Analysis of BLAST Results
================
Leila Esmaeili
October 21, 2018

Introduction
============

We this new understanding that bacterial communites on human skin are personalized, scientists have been curious to find out if this fact can be manipulated for forensic applications. In the assigned artilce, Freier et. al have tried to answer the question whether a person can be idntified based on microbial colonies on the the objects he/she has touched.

Methods
=======

First, The authors compared bacteria on individual keys of three computer keyboards to those found on the fingers of the keyboard owners. Second, we examined the similarity between skin-associated bacterial communities on objects stored at −20 °C(a standard method for storing samples before DNA extraction) versus those objects stored under typical indoor environmental con-ditions for up to 14 days. Finally, we linked objects to specific individuals by comparing the bacteria on their computer mice against a database containing bacterial community information for more than 250 hand surfaces, including the hand of the owner.

Their method of doing this was very simple. They just swabbed bacteria on the surface of keyboards and computer mice and the fingers of people touching them. They then grew these bacteria in petri dishes and isolated communites. And, then they extracted DNA and sequences the DNA and finall compared their sequences to a natiaonl data base.

Sample origin and sequencing
----------------------------

DNA extraction was done using the MO BIO PowerSoil DNA Isolation kit. PCR was used for DNA amplification. Pysrosequencing was carried out on 454 Life Sciences Genome Sequencer FLX (Roche) at University of South Carolina.

Computational
-------------

With our R studio codes, we have created a large table where all the information from above analysis are tabulated. Our codes allow for creatingtidy data, generating reports and creating plots to answer our questions. For example, I wanted to know how different are the bacterial colonies of 32 yr old vs those of 37. The histograms clearly show that percent identiy match graphs are different between these two age groups. In other words, we can conculde that we maybe able to determine a person's age based on their bacterial community on their skin. And another paragraph or two here.

Results
=======

The results of the histograms indicate there are distinct differences between age groups. In addition, each gener has its own kind of bacterial colonies distinct from the other gender.

``` r
# Be sure to install these packages before running this script
# They can be installed either with the install.packages() function
# or with the 'Packages' pane in RStudio

# load packages
library("dplyr")  # This is for tables
library("tidyr") # creates tidy data
library("knitr") # generates reports
library("ggplot2") # creates plot
```

``` r
# Output format from BLAST is as detailed on:
# https://www.ncbi.nlm.nih.gov/books/NBK279675/
# In this case, we used: '10 sscinames std'
# 10 means csv format
# sscinames means unique Subject Scientific Name(s), separated by a ';'
# std means the standard set of result columns, which are:
# 'qseqid sseqid pident length mismatch
# gapopen qstart qend sstart send evalue bitscore',


# this function takes as input a quoted path to a BLAST result file
# and produces as output a dataframe with proper column headers
# and the 'qseqid' column split into sample and seq number
read_blast_output <- function(filename) {
  data_in <- read.csv(filename,
                      header = FALSE, # files don't have column names in them
                      col.names = c("sscinames", # unique Subject Sci Name(s)
                                    "qseqid",    # Query Seq-id
                                    "sseqid",    # Subject Seq-id
                                    "pident",    # Percntge of identical matches
                                    "length",    # Alignment length
                                    "mismatch",  # Number of mismatches
                                    "gapopen",   # Number of gap openings
                                    "qstart",    # Start of alignment in query
                                    "qend",      # End of alignment in query
                                    "sstart",    # Start of alignment in subj
                                    "send",      # End of alignment in subject
                                    "evalue",    # Expect value
                                    "bitscore"))  # Bit score

  # Next we want to split the query sequence ID into
  # Sample and Number components so we can group by sample
  # They originally look like "ERR1942280.1"
  # and we want to split that into two columns: "ERR1942280" and "1"
  # we can use the separate() function from the tidyr library to do this
  # Note that we have to double escape the period for this to work
  # the syntax is
  # separate(column_to_separate,
  # c("New_column_name_1", "New_column_name_2"),
  # "seperator")
  data_in <- data_in %>%
    separate(qseqid, c("sample_name", "sample_number"), "\\.")
}
```

``` r
# this makes a vector of all the BLAST output file names, including
# the name(s) of the directories they are in
files_to_read_in <- list.files(path = "output/blast",
                               full.names = TRUE)

# We need to create an empty matrix with the right number of columns
# so that we can rbind() each dataset on to it
joined_blast_data <- matrix(nrow = 0,
                            ncol = 14)

# now we loop over each of the files in the list and append them
# to the bottom of the 'joined_blast_data' object
# we do this with the rbind() function and the function we
# made earlier to read in the files, read_blast_output()
for (filename in files_to_read_in) {
  joined_blast_data <- rbind(joined_blast_data,
                             read_blast_output(filename))
}
```

``` r
# Next we want to read in the metadata file so we can add that in too
# This is not a csv file, so we have to use a slightly different syntax
# here the `sep = "\t"` tells the function that the data are tab-delimited
# and the `stringsAsFactors = FALSE` tells it not to assume that things are
# categorical variables
metadata_in <- read.table(paste0("data/metadata/",
                                 "fierer_forensic_hand_mouse_SraRunTable.txt"),
                          sep = "\t",
                          header = TRUE,
                          stringsAsFactors = FALSE)

# Finally we use the left_join() function from dplyr to merge or 'join' the
# combined data and metadata into one big table, so it's easier to work with
# in R the `by = c("Run_s" = "sample_name")` syntax tells R which columns
# to match up when joining the datasets together
joined_blast_data_metadata <- metadata_in %>%
  left_join(joined_blast_data,
            by = c("Run_s" = "sample_name"))
```

``` r
joined_blast_data_metadata <- joined_blast_data_metadata %>%
  mutate(host_subject_id_sex = substr(host_subject_id_s, 1, 1))
```

``` r
unique(as.character(joined_blast_data_metadata$host_subject_id_s))
```

    ##  [1] "F2" "F5" "F6" "F7" "F8" "M1" "M2" "M7" "M8" "M9"

``` r
# Here we're using the dplyr piping syntax to select a subset of rows matching a
# criteria we specify (using the filter) function, and then pull out a column
# from the data to make a histogram.

joined_blast_data_metadata %>%
  filter(age_s == 32) %>%  # filtering based on specfic column
  ggplot(aes(x = pident)) +  # this is for x axis
    geom_histogram() +
    ggtitle("Percent Identiy Match age32") + # This is our title
    xlab("Percent") # this is our x axis label
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](Analysis_of_BLAST_Results_files/figure-markdown_github/histograms32-1.png)

Don't forget to report what your figures show in words, here in the Results section.

``` r
# Here we're using the dplyr piping syntax to select a subset of rows matching 
# criteria we specify (using the filter) function, and then pull out a column
# from the data to make a histogram.
joined_blast_data_metadata %>%
  filter(age_s == 25) %>%
  ggplot(aes(x = pident)) +
    geom_histogram() +
    ggtitle("Percent Identity Match age25") +
    xlab("Percent")
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](Analysis_of_BLAST_Results_files/figure-markdown_github/histograms25-1.png)

``` r
# Here we're using the dplyr piping syntax to select a subset of rows matching 
# criteria we specify (using the filter) function, and then pull out a column
# from the data to make a histogram.
joined_blast_data_metadata %>%
  filter(age_s == 37) %>%
  ggplot(aes(x = pident)) +
    geom_histogram() +
    ggtitle("Percent Identity Match age37") +
    xlab("Percent")
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](Analysis_of_BLAST_Results_files/figure-markdown_github/histograms37-1.png)

    colnames (joined_blast_data_metadata)
    joined_blast_data_metadata["env_materila_s"]
    unique(joined_blast_data_metadata["env_material_s"])

``` r
# Here we're using the dplyr piping syntax to select a subset of rows matching 
# criteria we specify (using the filter) function, and then pull out a column
# from the data to make a histogram.
joined_blast_data_metadata %>%
  filter(env_material_s == "dust") %>%
  ggplot(aes(x = pident)) +
    geom_histogram() +
    ggtitle("Percent Identity Match dust") +
    xlab("Percent")
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](Analysis_of_BLAST_Results_files/figure-markdown_github/histogramsdust-1.png)

``` r
# Here we're using the dplyr piping syntax to select a subset of rows matching 
# criteria we specify (using the filter) function, and then pull out a column
# from the data to make a histogram.
joined_blast_data_metadata %>%
  filter(env_material_s == "sebum") %>%
  ggplot(aes(x = pident)) +
    geom_histogram() +
    ggtitle("Percent Identity Match sebum") +
    xlab("Percent")
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](Analysis_of_BLAST_Results_files/figure-markdown_github/histogramssebum-1.png)

    {r summary-table}
    # Finally, we'd like to be able to make a summary table of the counts of
    # sequences for each subject for both sample types. To do that we can use the
    # table() function. We add the kable() function as well(from the tidyr package)
    # in order to format the table nicely when the document is knitted
    kable(table(joined_blast_data_metadata$host_subject_id_s,
                joined_blast_data_metadata$sample_type_s))

Discussion
==========

Add 2-3 paragraphs here interpreting your results and considering future directions one might take in analyzing these data. The histograms allow us to compare distrubition of bacteria btween age 32 and 37 and 25. We can see that histograms between the two age groups can easily be distinguished. In other words, we can say looking a the histogram if the person who has touched the mouse is 32 or 37 or 25.

Dust vs Sebum histograms are basically comparing mouse vs finger bacterial colonies.