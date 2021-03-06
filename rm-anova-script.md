---
title: "Coding Club : 3 ways repeated mesure ANOVA "
author: "Emmy"
date: "05/11/2021"
output: 
  html_document: 
    keep_md: yes
---

# Rhizosphere analysis of the field experiment
## Loading library
```{r}
library(readxl)
library(tidyr)
library(dplyr)
library(data.table)
library(datasets)
library(ggpubr)
library(rstatix)
```


## Loading my data 
```{r}
ratio_rizo_champ_copie <- read_excel("~/Documents/project/project_fb/field/data/raw/rhizo/ratio_rizo_champ copie.xlsx", col_types = c("text", "text", "text","text","text", "numeric", "numeric", "numeric"))

```

### Rename my data table
```{r}
rhizo <- ratio_rizo_champ_copie

```

## Visualization
create some boxplot
```{r}
library("ggpubr")
bxp <- ggboxplot(
  rhizo, x = "Traitement", y = "Ratio",
  color = "temps", palette = "jco",
  facet.by = "MO", short.panel.labs = FALSE
  )
bxp
```

## Check assumptions 
### Normality assumption
We will check the residues normality. We need to do some Shapiro-Wilk test for each combinations of factor levels.
```{r}
options(dplyr.print_max = 100)
rhizo %>% 
group_by(MO, Traitement, Bloc) %>%
shapiro_test(Ratio) 

```
Many combinations isn't normal ( We need to transform the data)

### log Transformation 
```{r}
rhizo$log <- log(rhizo$Ratio)

```

### Check outliers
```{r}
identify_outliers(rhizo, log )

```
 We have no outlier.

### Normality assumption (AGAIN)
```{r}
options(dplyr.print_max = 100)
rhizo %>% 
group_by(MO, Traitement, Bloc) %>%
shapiro_test(log) 

```
All combination reach normality with the log transformation. We can continue our analysis.
We can also check the normality with this graph:
```{r}
ggqqplot(rhizo, "log", ggtheme = theme_bw()) +
  facet_grid(MO + Traitement ~ Bloc, labeller = "label_both")
```
From the plot above, as all the points fall approximately along the reference line, we can assume normality.

### Assumption of sphericity
```{r}
sphere <- data.frame("temps" = (rhizo$temps),
                     "log" = (rhizo$log),
                     "sample" = (rhizo$Échantillons))

res <- anova_test(data = sphere, dv = log, wid = sample, within = temps)
res
```
The Mauchlys test for sphericity have a p-value 0f 2,12, which means that the sphericity is reach

## three way repeated mesure ANOVA
```{r}
rhizo$Bloc <- as.factor(rhizo$Bloc)
rhizo$temps <- as.factor(rhizo$temps)
rhizo$Traitement <- as.factor(rhizo$Traitement)
rhizo$MO <- as.factor(rhizo$MO)


anova <- aov(log ~ Traitement*MO*temps+Bloc + Error(Échantillons), data=rhizo)
summary(anova)

```

## Post-hoc test
### Difference between each time point group by treatment
```{r}
options(dplyr.print_max = 200)
pwc <- rhizo %>%
  group_by(Traitement) %>%
  pairwise_t_test(log ~ temps, paired = TRUE, p.adjust.method = "bonferroni")  

pwc
```
### Difference between each treatment group by time point
```{r}
options(dplyr.print_max = 200)
pwc1 <- rhizo %>%
  group_by(temps) %>%
  pairwise_t_test(log ~ Traitement, paired = TRUE, p.adjust.method = "bonferroni")  

pwc1 
```
### Difference between each treatment group by time point AND organic matter
```{r}
options(dplyr.print_max = 200)
pwc2 <- rhizo %>%
  group_by(temps, MO) %>%
  pairwise_t_test(log ~ Traitement, paired = TRUE, p.adjust.method = "bonferroni")  

pwc2 
```













