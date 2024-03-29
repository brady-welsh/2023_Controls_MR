## Code for figure X

### Raphael Eisenhofer 25/7/2023

## Load packages and data in, set theme

``` r
library(tidyverse)
library(janitor)

df <- read_delim("../data/DataTable.csv", delim = ',') %>%
  clean_names() %>%
  mutate(year = str_extract(study_id, "[:digit:].*"),
         #replacing 2023 with 2022, as while the pub date is 2023, it was available online in 2022
         year = str_replace_all(year, "2023", "2022"),
         were_the_negative_controls_sequenced = replace_na(were_the_negative_controls_sequenced, "No"),
         were_the_negative_controls_sequenced = str_replace_all(were_the_negative_controls_sequenced, "Unknown", "No"))
```

## Plot of studies through time

``` r
#Get counts of publications per year
df %>%
  group_by(year) %>%
  summarise(publications = n()) %>%
#Line plot it
  ggplot(aes(x = year, y = publications, group = 1)) +
  geom_line() +
  geom_point() +
  theme_minimal()
```

![](Figure_5_files/figure-markdown_github/unnamed-chunk-2-1.png)

## Proportions of negative and positive controls through time

``` r
# Colour palette
colscheme <- c("Positive controls" = "#5B9BD5", "Negative controls" = "#EA3834")

# Counts of studies per year
n_studies <- df %>%
  group_by(year) %>%
  summarise(publications = n()) 

# Get counts of sequenced negative controls
seq_neg_controls <- df %>%
  summarise(count = n(), .by = c(year, were_the_negative_controls_sequenced)) %>%
  group_by(year) %>%
  mutate(proportion = count / sum(count)) %>%
  filter(were_the_negative_controls_sequenced == "Yes")

neg_prop <- df %>%
  summarise(count = n(), .by = c(year, were_there_negative_controls)) %>%
  group_by(year) %>%
  mutate(proportion = count / sum(count)) %>%
  filter(were_there_negative_controls == "Yes")

pos_prop <- df %>%
  summarise(count = n(), .by = c(year, were_there_positive_controls)) %>%
  group_by(year) %>%
  mutate(proportion = count / sum(count)) %>%
  filter(were_there_positive_controls == "Yes")

#plot it
neg_prop %>%
  ggplot(aes(x = year, y = proportion * 100, 
             group = were_there_negative_controls,
             colour = "Negative controls")) +
  geom_line(linewidth = 2) +
  geom_line(data = pos_prop, aes(x = year, y = proportion * 100,
            group = were_there_positive_controls,
            colour = "Positive controls"),
            linewidth = 2) +
  geom_point(data = pos_prop, aes(x = year, y = proportion * 100,
            group = were_there_positive_controls,
            colour = "Positive controls"),
            size = 4) +
  geom_line(data = n_studies, 
            aes(x = year, y = publications, group = "NA"), 
            color = "grey",
            linewidth = 1,
            linetype = "dashed") +
  scale_y_continuous(
    name = "Percentage of studies",
    sec.axis = sec_axis(~ ., name = "Number of Studies"),
    labels = scales::label_percent(scale = 1)
  ) +
  scale_fill_manual(colscheme) +
  theme_classic() +
  theme(
    legend.title = element_blank(),
    legend.text = element_text(size = 12),
    legend.position = c(0.5, 0.9),
    axis.title = element_text(size = 14),
    axis.text = element_text(size = 14),
    axis.line.y.right = element_line(color = "grey", linetype = "dashed", linewidth = 1),
    axis.ticks.y.right = element_line(color = "grey"),
    axis.text.y.right = element_text(color = "grey"),
    axis.title.y.right = element_text(color = "grey")
    ) +
  labs(x = "Year", y = "Percentage of studies")
```

![](Figure_5_files/figure-markdown_github/unnamed-chunk-3-1.png)

``` r
ggsave("Figure_5.png", height = 5, width = 7.5)
```
