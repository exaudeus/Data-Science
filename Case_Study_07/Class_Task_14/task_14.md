---
title: 'Task 14: Counting Words and Occurrences'
author: "Etienne Beaulac"
date: "2/21/2018"
output: 
  html_document:
    keep_md: true
---

## Counting, graphing, coloring...

When I first read about the task I thought it was going to be a challenge. However, after a little bit of experimentation, I was able to accurately get the information that was required for all 3 questions.

----------

#### What is the average verse length (number of words) in the New Testament compared to the Book of Mormon?

Hint: the Book of Mormon wins. For these I had to learn how to use sapply, which is a very useful function when you want to do individual calculations for each value in a list. There are also other variations of this function that work with different things.


```r
dat_ <- read_csv("lds-scriptures.csv")

dat <- dat_ %>%
  mutate(lower_verse = strsplit(tolower(scripture_text), " "), verse_length = sapply(lower_verse, length))

dat %>%
  filter(volume_id == 2 | volume_id == 3) %>%
  group_by(volume_title) %>%
  summarise(average_verse_length = mean(verse_length))
```

```
## # A tibble: 2 x 2
##   volume_title   average_verse_length
##   <chr>                         <dbl>
## 1 Book of Mormon                 40.4
## 2 New Testament                  22.7
```

---------

#### How often is the word Jesus in the New Testament compared to the Book of Mormon?

I was really suprised about this data, and I thought that I had done something wrong. Then I remembered that Jesus has a lot of names and that "Jesus" was very popular in the New Testament times.

```r
dat %>%
  filter(volume_id == 2 | volume_id == 3) %>%
  mutate(jesus = sapply(sapply(lower_verse, grep, pattern = "jesus"), length)) %>%
  group_by(volume_title) %>%
  summarize(jesus_count = sum(jesus))
```

```
## # A tibble: 2 x 2
##   volume_title   jesus_count
##   <chr>                <int>
## 1 Book of Mormon         184
## 2 New Testament          984
```

----------

#### How does the word count distribution by verse look for each book in the Book of Mormon?

I had a hard time representing this one properly. I couldn't really find anything that perfectly answered the question, so I created two graphs. The one on the left is nice because it shows everything on one facet, but it's a little difficult to compare. 

![](distribution.png)


```r
new_dat <- dat %>% 
  filter(volume_id == 3)

book_f <- new_dat$book_title %>% factor() %>% fct_inorder()
plot1 <- new_dat %>% 
  ggplot(aes(x = verse_length, y = book_f, fill = book_f)) +
  geom_density_ridges(rel_min_height = 0.01) +
  labs(x = "Word count per verse") +
  guides(fill = FALSE) +
  theme(axis.title.y = element_blank())

plot2 <- new_dat %>% 
  ggplot(aes(x = verse_number, y = verse_length, col = book_title)) +
  geom_point(alpha = 0.5, position = "jitter") +
  facet_wrap(~book_title, ncol = 3) +
  guides(col = FALSE) +
  labs(x = "Verse number", y = "Word count per verse")

ggarrange(plot1, plot2, ncol = 2) %>% 
  annotate_figure(top = text_grob("How does the word count distribution by verse look for each book in the Book of Mormon?", color = "red", face = "bold", size = 14)) +
    ggsave("Case_Study_07/Class_Task_14/distribution.png", width = 12)
```