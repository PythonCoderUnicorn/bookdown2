# Text Analysis 

## Text Mining

This section is from **RLadies Freiburg** Kyla McConnell & Julia Müller. The source of the material is *Text Mining with R* by Julia Silge & David Robson. 


The libraries needed for this tutorial

```r
library(tidyverse)  # for various data manipulation tasks
#> ── Attaching packages ─────────────────── tidyverse 1.3.1 ──
#> ✓ ggplot2 3.3.5     ✓ purrr   0.3.4
#> ✓ tibble  3.1.6     ✓ dplyr   1.0.7
#> ✓ tidyr   1.1.4     ✓ stringr 1.4.0
#> ✓ readr   2.1.1     ✓ forcats 0.5.1
#> ── Conflicts ────────────────────── tidyverse_conflicts() ──
#> x dplyr::filter() masks stats::filter()
#> x dplyr::lag()    masks stats::lag()
library(tidytext)   # for text mining specifically, main package in book
library(stringr)    # for various text operations
library(gutenbergr) # to access full-text books that are in the public domain
library(scales)     # for visualizing percentages
#> 
#> Attaching package: 'scales'
#> The following object is masked from 'package:purrr':
#> 
#>     discard
#> The following object is masked from 'package:readr':
#> 
#>     col_factor
library(readtext)   # for reading in txt files
library(wordcloud)  # for creating wordclouds
#> Loading required package: RColorBrewer
```

### Reading in texts

Here's how you can read in one .txt file that is saved in the same location as this script (i.e. in the same folder on your computer):
```{}
hf <- readtext("Adventures of Huckleberry Finn.txt")
```

If you want to read all files from a sub-folder, type the name of the folder followed by / and * to ask R to read in all files in that folder:
```{}
shakes <- readtext("Shakespeare txts/*")
shakes$doc_id <- sub(".txt", "", shakes$doc_id) # this gets rid of .txt in the play titles
```

### Book data from Project Gutenberg
* Project Gutenberg: free downloads of books in the public domain (i.e. lots of classic literature)
* Currently in a legal battle in Germany -* impossible to download via the website
* Still accessible via the R package gutenbergr by ID
* Top 100 books for inspiration (changes daily based on demand): https://www.gutenberg.org/browse/scores/top
* Catalog: https://www.gutenberg.org/catalog/ 

To find the id of a book (some have multiple copies):
```{}
gutenberg_metadata %>%
  filter(title %in% c("Alice's Adventures in Wonderland", "Grimms' Fairy Tales", "Andersen's Fairy Tales"))
```


Can also search by author name:
```{}
gutenberg_works(author == "Carroll, Lewis")
gutenberg_works(str_detect(author, "Carroll")) #if you only have a partial name
```


For more Gutenberg search options: https://ropensci.org/tutorials/gutenbergr_tutorial/

Once you found your books, download with gutenberg_download:
```{}
fairytales_raw <- gutenberg_download(c(11, 2591, 1597))
fairytales_raw
```

11: Alice's Adventures in Wonderland
2591: Grimm's Fairytales
1597: Hans Christian Anderson's Fairytales

### Preparing data
- convert Gutenberg ID to a factor and replacing the ID numbers with more descriptive labels 
```{}
fairytales_raw$gutenberg_id <- as.factor(fairytales_raw$gutenberg_id)
fairytales_raw$gutenberg_id <- plyr::revalue(fairytales_raw$gutenberg_id,
                                              c("11" = "Alice's Adventures in Wonderland",
                                                "2591" = "Grimm's Fairytales",
                                                "1597" = "Hans Christian Anderson's Fairytales"))
```


## Tidy text
* One word per row, facilitates analysis
* Token: "a meaningful unit of text, most often a word, that we are interested in using for further analysis"

### the unnest_tokens function
* Easy to convert from full text to token per row with unnest_tokens()
Syntax: unnest_tokens(df, newcol, oldcol)
* unnest_tokens() automatically removes punctuation and converts to lowercase (unless you set to_lower = FALSE)
* by default, tokens are set to words, but you can also use token = "characters", "ngrams", "sentences", "lines", "regex", "paragraphs", and even "tweets" (which will retain usernames, hashtags, and URLs)

```{}
fairytales_tidy <- fairytales_raw %>% 
  unnest_tokens(word, text)
# this keeps the information on which sentence the words came from
fairytales_raw %>% 
  unnest_tokens(sentence, text, token = "sentences") %>% 
  mutate(sent_nr = row_number()) %>% 
  unnest_tokens(word, sentence)
fairytales_tidy
shakes_unnest <- shakes %>% 
  unnest_tokens(word, text)
```

### Removing non-alphanumeric characters
* Project Gutenberg data sometimes contains underscores to indicate italics
* str_extract is used to get rid of non-alphanumeric characters (because we don't want to count _word_ separately from word)
```{}
fairytales_tidy <- fairytales_tidy %>% 
  mutate(word = str_extract(word, "[a-z']+"))
shakes_unnest <- shakes_unnest %>% 
  mutate(word = str_extract(word, "[a-z']+"))
``` 

### Stop words
* Stop words: very common, "meaningless" function words like "the", "of" and "to" -- not usually important in an analysis (i.e. to find out that the most common word in two books you are comparing is "the")
* tidytext has a built-in df called stop_words for English 
* remove these from your dataset with anti_join

We can take a look:
```{}
stop_words
```


```{}
fairytales_tidy <- fairytales_tidy %>% 
  anti_join(stop_words)
fairytales_tidy
```

Define other stop words:
```{}
meaningless_words <- tibble(word = c("von", "der", "thy", "thee", "thou"))
fairytales_tidy <- fairytales_tidy %>% 
  anti_join(meaningless_words)
```
This could also be used to remove character names, for example.


The stopwords package also contains lists of stopwords for other languages, so to get a list of German stopwords, you could use:
```{}
library(stopwords)
stop_german <- data.frame(word = stopwords::stopwords("de"), stringsAsFactors = FALSE)
```
More info: https://cran.r-project.org/web/packages/stopwords/readme/README.html

Break: Prepare your data with the steps above. 1) Unnest tokens, 2) Remove alpha-numeric characters, 3) Remove stopwords 

## Analysing frequencies

### Find most frequent words
* Easily find frequent words using count() 
* Data must be in tidy format (one token per line)
* sort = TRUE to show the most frequent words first

tidy_books %>%
  count(word, sort = TRUE) 

```{}
fairytales_freq <- fairytales_tidy %>% 
  group_by(gutenberg_id) %>% #including this ensures that the counts are by book and the id column is retained
  count(word, sort=TRUE)
fairytales_freq
shakes_freq <- shakes_unnest %>% 
  group_by(doc_id) %>% 
  count(word, sort = TRUE)
```


Reminder: filter can be used to look at subsets of the data, i.e. one book, all words with freq above 100, etc. (Note here that I don't save this output)
```{}
fairytales_tidy %>% 
  group_by(gutenberg_id) %>% 
  count(word, sort=TRUE) %>% 
  filter(gutenberg_id == "Grimm's Fairytales")
```



#### Plotting word frequencies - bar graphs

Bar graph of top words in Grimm's fairytales.

Basic graph:
```{}
fairytales_freq %>% 
  filter(n>90 & gutenberg_id == "Grimm's Fairytales") %>% 
  ggplot(aes(x=word, y=n)) +
  geom_col()
```



Readable labels:
```{}
fairytales_freq %>% 
  filter(n>90 & gutenberg_id == "Grimm's Fairytales") %>% 
  ggplot(aes(x=word, y=n)) +
  geom_col() +
  theme(axis.text.x = element_text(angle = 45))
```

Descending order:
```{}
fairytales_freq %>% 
  filter(n>90 & gutenberg_id == "Grimm's Fairytales") %>% 
  ggplot(aes(x=reorder(word, -n), y=n)) +
  geom_col() +
  theme(axis.text.x = element_text(angle = 45))
```


Axis names and colors:
```{}
fairytales_freq %>% 
  filter(n>90 & gutenberg_id == "Grimm's Fairytales") %>% 
  ggplot(aes(x=reorder(word, -n), y=n, fill=n)) +
  geom_col(show.legend=FALSE) +
  theme(axis.text.x = element_text(angle = 45)) +
  xlab("Word") +
  ylab("Frequency") +
  ggtitle("Most frequent words in Grimm's Fairytales")
```



Or: flip coordinate system to make more space for words
```{}
fairytales_freq %>% 
  filter(n>90 & gutenberg_id == "Grimm's Fairytales") %>% 
  ggplot(aes(x=reorder(word, n), y=n, fill=n)) +
  geom_col(show.legend=FALSE) +
  xlab("Word") +
  ylab("Frequency") +
  ggtitle("Most frequent words in Grimm's Fairytales") +
  coord_flip()
```



### Normalised frequency
* when comparing the frequencies of words from different texts, they are commonly normalised
* convention in corpus linguistics: report the frequency per 1 million words
* for shorter texts: per 10,000 or per 100,000 words
* calculation: raw frequency * 1,000,000 / total numbers in text
```{}
# see the total number of words per play (doc_id)
shakes_freq %>% 
  group_by(doc_id) %>% 
  mutate(sum(n)) %>% 
  distinct(doc_id, sum(n))
shakes_freq <- shakes_freq %>% 
  na.omit() %>% 
  group_by(doc_id) %>% 
  mutate(pmw = n*1000000/sum(n)) %>% # creates a new column called pmw
  ungroup() %>% 
  anti_join(stop_words) # removing stopwords afterwards
shakes_freq %>% select(word, pmw)
```


#### Plotting normalised frequency
Now we can plot, for example, the 20 most frequent words (by pmw).
```{}
shakes_freq %>% 
  filter(doc_id == "Othello") %>% 
  top_n(20, pmw) %>% 
  ggplot(aes(x=reorder(word, -pmw), y=pmw, fill=pmw)) +
  geom_col(show.legend=FALSE) +
  theme(axis.text.x = element_text(angle = 45)) +
  xlab("Word") +
  ylab("Frequency per 1 million words") +
  ggtitle("Most frequent words in Othello")
```


Break: Gather frequency counts for your text, normalize them, and create a bar chart of most the normalized frequencies.


### Word clouds

Let's visualise the most frequent words in a word cloud. Here, the size indicates the frequency, with words that occur more often being displayed in a larger font size, but this can also be used to visualise e.g. normalised frequency (pmw) or length or anything else you pass to the freq = part of the command.
```{}
wordcloud(words = shakes_freq$word, freq = shakes_freq$n, 
          min.freq = 150, max.words=200, random.order=FALSE, rot.per=0.35, 
          colors=brewer.pal(8, "Dark2"))
```


## Comparing the vocabulary of texts

Next, we'll create two graphs to compare the vocabulary of our texts. First, we focus on Alice's Adventures and Anderson's Fairytales. The newly created comp_2 data frame contains only the words and their frequencies in the two texts in two separate columns.

### Comparing two texts
```{}
comp_2 <- fairytales_freq %>% 
  filter(gutenberg_id == "Alice's Adventures in Wonderland"|gutenberg_id == "Hans Christian Anderson's Fairytales") %>% 
  group_by(gutenberg_id) %>% 
  mutate(proportion = n / sum(n)) %>% #creates proportion column (word frequency divided by overall frequency per author)
  select(-n) %>%
  spread(gutenberg_id, proportion)
head(comp_2)
```

Now, we can plot the words. Their placement depends on the word frequencies. Additionally, colour coding shows how different the frequencies are - darker items are more similar in terms of their frequencies, lighter-coloured ones more frequent in one text compared to the other. We'll discuss the interpretation in more detail once we've created the threeway comparison.
```{}
ggplot(comp_2, 
       aes(x = `Alice's Adventures in Wonderland`, y = `Hans Christian Anderson's Fairytales`, 
           color = abs(`Alice's Adventures in Wonderland` - `Hans Christian Anderson's Fairytales`))) +
  geom_abline(color = "gray40", lty = 2) +
  geom_jitter(alpha = 0.1, size = 2.5, width = 0.3, height = 0.3) +
  geom_text(aes(label = word), check_overlap = TRUE, vjust = 1.5) +
  scale_x_log10(labels = percent_format()) +
  scale_y_log10(labels = percent_format()) +
  theme_light() +
  theme(legend.position="none") +
  labs(y = "Hans Christian Anderson's Fairytales", x = "Alice's Adventures in Wonderland")
```


### Comparing three texts
In order to compare three texts, we need to add one step to the data preparation: Grimm's Fairytales will be the text that the other two will be compared to, so its word frequencies will be contained in the Grimm's Fairytales column. The gutenberg_id column contains both Alice's Adventures and Anderson's Fairytales so we can pass this column to the facet_wrap command and create two graphs.
```{}
comp_3 <- fairytales_freq %>% 
  group_by(gutenberg_id) %>% 
  mutate(proportion = n / sum(n)) %>% #creates proportion column (word frequency divided by overall frequency per author)
  select(-n) %>% 
  spread(gutenberg_id, proportion) %>% 
  gather(gutenberg_id, proportion, "Alice's Adventures in Wonderland":"Hans Christian Anderson's Fairytales") # only done for plotting
head(comp_3); unique(comp_3$gutenberg_id)
```


The ggplot command is very similar to the one used above but facet_wrap is added to create two comparisons - Grimm's Fairytales compared to Alice's Adventures (left graph) and Grimm's compared to Anderson's fairytales (right graph):
```{}
ggplot(comp_3, 
       aes(x = proportion, y = `Grimm's Fairytales`, 
           color = abs(`Grimm's Fairytales` - proportion))) +
  geom_abline(color = "gray40", lty = 2) +
  geom_jitter(alpha = 0.1, size = 2.5, width = 0.3, height = 0.3) +
  geom_text(aes(label = word), check_overlap = TRUE) +
  scale_x_log10(labels = percent_format()) +
  scale_y_log10(labels = percent_format()) +
  theme_light() +
  facet_wrap(~ gutenberg_id, ncol = 2) +
  theme(legend.position="none") +
  labs(y = "Grimm's Fairytales", x = NULL)
```


### Interpretation
Words that are close to the diagonal line have similar frequencies in both texts (e.g. king, cook, calling in the left graph). Words that are far from the line are more frequent in one of the two texts (e.g. wife or son are more frequent for Grimm compared to Alice, while turtle and hare are more frequent in Alice than in Grimm). Again, this is also indicated by the colour.  
Generally, if the words are closer to the line and there's a smaller gap for low frequencies, the vocabulary of the texts overall is more similar. In our case, the two fairytale sources contain more of the same words than Grimm compared to Alice.  
An additional step in an analysis of word frequencies and something we won't cover today is to calculate correlations of word frequencies to quantify how similar the vocabularies of texts are.  




## Word and document frequencies

### tf-idf
How can we quantify what a text/document is about? We could analyse the term frequency (tf) - how often does a term occur in a text/document. However, common words are the same in most texts, e.g. grammatical words like articles. A solution would be to instead analyse the inverse document frequency (idf) which lowers the importance of frequent words and raises the importance of rare words in documents. So it's a measure of how important a word is to a text compared to how important it is in the collection of texts.

#### The bind_tf_idf-function
* input: format needs to be one row per token (term), per document
* one column (here: word) contains the terms/tokens
* one column (here: gutenberg_id) contains the documents

```{}
fairytales_idf <- fairytales_freq %>% 
  bind_tf_idf(word, gutenberg_id, n)
fairytales_idf %>%
  select(gutenberg_id, word, tf_idf) %>% 
  arrange(desc(tf_idf))
```

interpretation:

* low tf_idf if words appear in many books, high if they occur in few books  
* characteristic words for documents  
* so unsurprisingly, in our data, the first hits with the highest tf_idf are character names  

#### Characteristic words per book
visualisation of the top 20 tf-idf words per book:
```{}
fairytales_idf$word <- as.factor(fairytales_idf$word)
fairytales_idf %>%
  group_by(gutenberg_id) %>% 
  arrange(desc(tf_idf)) %>% 
  top_n(20, tf_idf) %>% 
  ggplot(aes(x = reorder(word, tf_idf), y = tf_idf, fill = gutenberg_id)) +
  geom_col(show.legend = FALSE) +
  labs(x = NULL, y = "tf-idf") +
  facet_wrap(~gutenberg_id, scales = "free") +
  coord_flip()
```


#### Characteristic words per chapter
tf_idf can also be used to find characteristic words per chapter, or any other text unit. We'll only use "Alice in Wonderland" as an example since it consists of several chapters.  
We first need to create a column that contains the chapter the word came from. The code below:

* finds "chapter" followed by a Roman numeral by using a regular expression regex("^chapter [\\divclx]" in the str_detect() command  
* extracts the chapter number by counting how often this regex is found with cumsum(). This is basically a counter that starts at 0 if the regex isn't matched, then counts up by one every time chapter + Roman numeral is found in the text  
* write it to a new column called "chapter"  
* also preserves the original line numbers (optional)  
We then remove the gutenberg_id column, words from chapter 0, i.e. the title and information on the edition, unnest tokens, and remove stopwords.  
```{}
alice <- fairytales_raw %>% 
  filter(gutenberg_id == "Alice's Adventures in Wonderland") %>% 
  mutate(linenumber = row_number(),
         chapter = cumsum(str_detect(text, regex("^chapter [\\divclx]",
                                                 ignore_case = TRUE)))) %>%
  select(-gutenberg_id) %>% 
  filter(chapter != 0) %>% 
  mutate(chapter = as_factor(chapter)) %>% 
  unnest_tokens(word, text) %>% 
  anti_join(stop_words)
head(alice); summary(alice$chapter)
```


Now let's calculate the tf-idf per chapter:

* first step is to calculate the frequency for each word per chapter
* then apply bind_tf_idf function
* show words with the highest tf_idf

```{}
alice <- alice %>% 
  group_by(chapter) %>% 
  count(word, sort = TRUE)
alice_idf <- alice %>% 
  bind_tf_idf(word, chapter, n)
alice_idf %>%
  select(chapter, word, tf_idf) %>% 
  arrange(desc(tf_idf))
```



Again, we can visualise the most characteristic words, this time per chapter:
```{}
alice_idf %>%
  group_by(chapter) %>% 
  arrange(desc(tf_idf)) %>% 
  top_n(5, tf_idf) %>% 
  ggplot(aes(reorder(word, tf_idf), tf_idf, fill = chapter)) +
  geom_col(show.legend = FALSE) +
  theme_light() +
  labs(x = NULL, y = "tf-idf") +
  facet_wrap(~ chapter, scales = "free") +
  coord_flip()
```











