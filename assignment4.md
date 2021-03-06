Statistical assignment 4
================
Sunny Chou
01/03/2020

In this assignment you will need to reproduce 5 ggplot graphs. I supply
graphs as images; you need to write the ggplot2 code to reproduce them
and knit and submit a Markdown document with the reproduced graphs (as
well as your .Rmd file).

First we will need to open and recode the data. I supply the code for
this; you only need to change the file paths.

    ```r
    library(tidyverse)
    library(ggplot2)
    
    getwd()
    ```
    
    ```
    ## [1] "C:/Users/user/Documents/assignment-4-sunnychou19"
    ```
    
    ```r
    Data8 <- read_tsv("C:/Users/user/Desktop/data/UKDA-6614-tab/tab/ukhls_w8/h_indresp.tab")
    Data8 <- Data8 %>%
        select(pidp, h_age_dv, h_payn_dv, h_gor_dv)
    Stable <- read_tsv("C:/Users/user/Desktop/data/UKDA-6614-tab/tab/ukhls_wx/xwavedat.tab")
    Stable <- Stable %>%
        select(pidp, sex_dv, ukborn, plbornc)
    Data <- Data8 %>% left_join(Stable, "pidp")
    rm(Data8, Stable)
    Data <- Data %>%
        mutate(sex_dv = ifelse(sex_dv == 1, "male",
                           ifelse(sex_dv == 2, "female", NA))) %>%
        mutate(h_payn_dv = ifelse(h_payn_dv < 0, NA, h_payn_dv)) %>%
        mutate(h_gor_dv = recode(h_gor_dv,
                         `-9` = NA_character_,
                         `1` = "North East",
                         `2` = "North West",
                         `3` = "Yorkshire",
                         `4` = "East Midlands",
                         `5` = "West Midlands",
                         `6` = "East of England",
                         `7` = "London",
                         `8` = "South East",
                         `9` = "South West",
                         `10` = "Wales",
                         `11` = "Scotland",
                         `12` = "Northern Ireland")) %>%
        mutate(placeBorn = case_when(
                ukborn  == -9 ~ NA_character_,
                ukborn < 5 ~ "UK",
                plbornc == 5 ~ "Ireland",
                plbornc == 18 ~ "India",
                plbornc == 19 ~ "Pakistan",
                plbornc == 20 ~ "Bangladesh",
                plbornc == 10 ~ "Poland",
                plbornc == 27 ~ "Jamaica",
                plbornc == 24 ~ "Nigeria",
                TRUE ~ "other"))
    ```

Reproduce the following graphs as close as you can. For each graph,
write two sentences (not more\!) describing its main message.

1.  Univariate distribution (20 points).
    
    ``` r
    ggplot(Data, aes(x = h_payn_dv)) +
      geom_freqpoly() +
      xlab("Net Monthly Pay") + ylab('Number of Respondents')
    ```
    
    ![](assignment4_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

Most the the respondents, around 2500 of them, has the net monthly pay
between 1000 to 1500. The trend steeply rises to that point and steeply
declines until a surge of a small amount of respondents with income
around 5500.

2.  Line chart (20 points). The lines show the non-parametric
    association between age and monthly earnings for men and women.
    
    ``` r
    library(gcookbook)
    plot2<- ggplot(Data, aes(x = h_age_dv, y = h_payn_dv , linetype=sex_dv)) + geom_smooth()+ xlim(16,65) + xlab("Age") + ylab('Monthly Earnings')
    plot2
    ```
    
    ![](assignment4_files/figure-gfm/unnamed-chunk-3-1.png)<!-- --> Both
    male and female monthly earnings exhibit a similiar trend, steepy
    rising towards a more moderate rise, and falls gradually around 45
    to 50 respectively. However, female wages rise at a much slower rate
    after 25, and were capped at 1500.

3.  Faceted bar chart (20 points).
    
    ``` r
    Plot3data1 <- Data %>% group_by(placeBorn,sex_dv) 
    Plot3data2 <- Plot3data1 %>% summarise(MedianIncome = median(h_payn_dv, na.rm = TRUE))
    plot3data<-na.omit(Plot3data2)
    
    plot3<-ggplot(plot3data, aes(x= sex_dv, y = MedianIncome)) + geom_bar(stat = "identity") + ylim(0,2000)
    plot3<-plot3 + facet_wrap(.~placeBorn) + ylab('Median Monthly Net Pay') + xlab('Sex')
    plot3
    ```
    
    ![](assignment4_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

Median monthly net pay for male has been higher than female across all
countries of origin. More developed countries of Birth have shown
greater disparity between male and female pay, with male higher.

4.  Heat map (20 points).
    
    ``` r
     Plot4data <- Data %>% group_by(placeBorn,h_gor_dv) %>% summarise(MeanAge = mean(h_age_dv, na.rm = TRUE))
     Plot4data <- na.omit(Plot4data)
     plot4 <- ggplot(Plot4data, aes(x = h_gor_dv, y = placeBorn, fill = MeanAge))  + xlab('Region') + ylab('Countries of Birth')
     plot4 <- plot4 + geom_tile() + theme(axis.text.x = element_text(angle = 90, hjust = 1))
     plot4
    ```
    
    ![](assignment4_files/figure-gfm/unnamed-chunk-5-1.png)<!-- --> Mean
    age in the UK looks even dispersed across regions and countries of
    Birth, with Nigeria the youngest.

5.  Population pyramid (20 points).
    
    ``` r
     plot5data <- Data %>% group_by(h_age_dv, sex = sex_dv) %>% count(age = h_age_dv) 
    
     plot5data <- na.omit(plot5data)
    
     plot5 <- ggplot(data = plot5data, mapping = aes(x = h_age_dv, y = ifelse(test = sex == "male", yes = -n, no = n), 
                     fill = sex)) + geom_col() + coord_flip() + 
     scale_y_continuous(labels = abs, limits = max(plot5data$n) * c(-1,1))  + xlab('Age')+ ylab('n')
    
     plot5
    ```
    
    ![](assignment4_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

Female distribution greater than male between 25 to 50, also between
67.5 to 75. Otherwise similarly distributed.
