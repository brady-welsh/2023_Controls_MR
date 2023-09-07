## Code for Figure 4

### Brady Welsh 07/9/2023

## Load packages, import table and extract control data

``` r
library(ggplot2)
library(webr)
library(dplyr)
library(tidyr)
library(ggrepel)
library(plotly)
library(ggpubr)

data <- read.csv("Table.csv")
sb.data <- data %>% select(Sample.Blank, 
                          Extraction.Blank, 
                          Template.Blank..PCR.,
                          Mock.Communities, 
                          Spike.Ins)
```

## Plot prevalance of controls (4A)

``` r
# Count prevalence of positive, negative and positive and negative controls and add labels
prev.c.data <- data %>% count(Were.there.negative.controls., Were.there.positive.controls.)
prev.c.data$condition <- c("No Control", "Pos Only", "Neg Only", "Neg + Pos")

# Make the colour pallet
prev.colours <- c("#DCDADA", "#8D73BD", "#EA3834", "#5B9BD5")
pos.colours <- c("#BDD7EE", "#5B9BD5")
neg.colours <- c("#F6AAA8", "#EA3834", "#DCDADA")
#blue: #5B9BD5 red: #EA3834 purple: #8D73BD light red: #F6AAA8 light blue: #BDD7EE

#Plot it
full.pie <- ggplot(prev.c.data, aes(x = "", y = n, 
                        fill = factor(condition, 
                                      levels = (c("No Control", "Neg + Pos", 
                                                  "Neg Only", "Pos Only"))))) +
  geom_bar(width = 1, size = 1, stat = "identity", color = "white") +
  coord_polar("y") +
  geom_text(aes(x = 2, label = paste0(condition, "\n", round((n/sum(n)*100), 2), "%")), 
            position = position_stack(vjust = 0.5), 
            color = "black", size = 10) +
  scale_fill_manual(values = prev.colours) +
  theme_void()+
  theme(legend.position = "none")
```

## Plot sequencing occurances for each observed control (4C and 4D)

``` r
# First, positive controls (4C)

seq.pos.data <- data %>% count(Were.the.positive.controls.sequenced.)
seq.pos.data$sequenced <- c("No", "Yes", "NA")
seq.pos.data <- subset(seq.pos.data, sequenced!='NA')

# and negative controls (4D)

seq.neg.data <- data %>% count(Were.the.negative.controls.sequenced.)
seq.neg.data$sequenced <- c("No", "Unknown", "Yes", "NA")
seq.neg.data <- subset(seq.neg.data, sequenced!='NA')

# Set colours

#Plot positive
pos.pie <- ggplot(seq.pos.data, aes(x = "", y = n, 
                        fill = factor(sequenced, 
                                      levels = (c("No", "Yes"))))) +
  geom_bar(width = 1, size = 1, stat = "identity", color = "white") +
  coord_polar("y") +
  geom_text(aes(x = 2, label = paste0(sequenced, "\n", round((n/sum(n)*100), 2), "%")), 
            position = position_stack(vjust = 0.5), 
            color = "black", size = 10) +
  scale_fill_manual(values = pos.colours) +
  theme_void()+
  theme(legend.position = "none")

#Plot negative
neg.pie <- ggplot(seq.neg.data, aes(x = "", y = n, 
                        fill = factor(sequenced, 
                                      levels = (c("No", "Yes", 
                                                  "Unknown"))))) +
  geom_bar(width = 1, size = 1, stat = "identity", color = "white") +
  coord_polar("y") +
  geom_text(aes(x = 2, label = paste0(sequenced, "\n", round((n/sum(n)*100), 2), "%")), 
            position = position_stack(vjust = 0.5), 
            color = "black", size = 10) +
  scale_fill_manual(values = neg.colours) +
  theme_void()+
  theme(legend.position = "none")
  ```

  ## Arrange plots with space for sunburst plot (made with MSExcel) and export

  ``` r
  df <- data.frame()
blank <- ggplot(df) + geom_point() + xlim(0, 10) + ylim(0, 100) + theme_void()

ggarrange(full.pie, blank, pos.pie, neg.pie, nrow = 2, ncol = 2, common.legend = FALSE, 
          labels = list('A)', 'B)', 'C)', 'D)'), font.label = list(size = 30))

ggsave("Figure4_files/Figure4.svg", height = 18, width = 18)
ggsave("Figure4_files/Figure4.png", height = 18, width = 18, dpi = 600)
```

![](Figure4_files/Figure4.png)
