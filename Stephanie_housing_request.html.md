---
title: "Stephanie’s Housing Request"
subtitle: "A Data-Driven Approach to Housing Decisions"
author: "Felix Jena"
format:
  html:
    self-contained: true
    page-layout: full
    title-block-banner: true
    toc: true
    toc-depth: 3
    toc-location: body
    number-sections: false
    html-math-method: katex
    code-fold: true
    code-summary: "Show the code"
    code-overflow: wrap
    code-copy: hover
    code-tools:
        source: false
        toggle: true
        caption: "See code"
    keep-md: true 
execute: 
  warning: false
editor_options: 
  chunk_output_type: inline
---


::: {.cell}

```{.r .cell-code}
pacman::p_load(mosaic, tidyverse, pander, DT, ggplot2)
Rent <- read.csv("../R/Rent.csv")
```
:::



### Reasearch Question:
What housing options near BYU-Idaho campus meet Stephanie’s (a made up student) criteria of affordability, proximity, and social interaction, and how can we use statistical analysis to evaluate these options?





::: {.cell}

```{.r .cell-code}
Rent <- Rent %>%
  mutate(Monthly_Rent = paste0("$", round(AvgFloorPlanCost / 4, 2)), 
         TimeDistanceToMC = paste0(ceiling(CrowFlyMetersToMC / 80)),
         TimeDistanceToRicks = paste0(ceiling(CrowFlyMetersToRicks / 80)))
```
:::


Most school semesters last approximately four months. So, dividing the semester cost by four gives us a good estimate of the monthly rent Stephanie would need to pay. 



::: {.cell}

```{.r .cell-code}
filtered_apartments <- Rent %>%
  filter(Monthly_Rent <= 350,     
         CrowFlyMetersToMC <= 1500,
         Gender == "F",
         Residents >= 250)


filtered_apartments <- filtered_apartments %>%
  select(Name, Gender, Address, Phone, Residents, WebAddress, PublicDescription, Monthly_Rent, TimeDistanceToMC, TimeDistanceToRicks)
```
:::


A distance of 1500 meters was selected based on its approximately a 15-20 minute walk, which is generally considered a reasonable and manageable distance for students walking to and from campus. Apartments within this distance are typically still considered "close to campus," which aligns with Stephanie’s preference for proximity. 
The walking time to the campus (MC and Ricks building) was further estimated using a standard walking speed of about 80 meters per minute. This conversion allowed us to display walking times instead of raw distances, which is more intuitive for Stephanie.


### Monthly Rent vs Number of Residents



::: {.cell}

```{.r .cell-code}
# Converting Monthly_Rent to numeric for plotting
filtered_apartments <- filtered_apartments %>%
  mutate(Monthly_Rent_numeric = as.numeric(gsub("[$,]", "", Monthly_Rent)))

ggplot(filtered_apartments, aes(x = Monthly_Rent_numeric, y = Residents)) +
  geom_point(aes(color = Residents, size = Residents), alpha = 0.7) +
  geom_smooth(method = "lm", se = FALSE, color = "darkblue", linetype = "dashed") +
  scale_color_gradient(low = "lightblue", high = "darkblue") +
  labs(title = "Monthly Rent vs Number of Residents",
       x = "Monthly Rent ($)",
       y = "Number of Residents",
       color = "Residents") +
  theme_minimal() +
  theme(
    plot.title = element_text(hjust = 0.5, size = 14, face = "bold"),
    axis.title.x = element_text(size = 12),
    axis.title.y = element_text(size = 12)
  ) +
  scale_size(range = c(3, 8))
```

::: {.cell-output-display}
![](Stephanie_housing_request_files/figure-html/unnamed-chunk-4-1.png){width=672}
:::
:::


Apartments like Royal Crest, with the lowest rent ($248.75), house a relatively large number of residents (342), whereas apartments like Towers II, with a rent of $411.25, have fewer residents (248). This trend indicates that while rent may not drastically affect the number of residents, complexes with higher rents tend to have fewer residents, though the relationship is not very strong.

### Monthly Rent by Apartment



::: {.cell}

```{.r .cell-code}
# Bar plot with points representing the number of residents
ggplot(filtered_apartments, aes(x = reorder(Name, -as.numeric(gsub("[$,]", "", Monthly_Rent))), y = as.numeric(gsub("[$,]", "", Monthly_Rent)))) +
  geom_bar(stat = "identity", fill = "steelblue", color = "black") +
  geom_point(aes(size = Residents), color = "darkred", show.legend = TRUE) +
  labs(title = "Monthly Rent by Apartment", 
       x = "Apartment", 
       y = "Monthly Rent ($)", 
       size = "Number of Residents") +
  theme_minimal() +
  coord_flip() +
  theme(
    plot.title = element_text(hjust = 0.5, size = 16, face = "bold"),
    axis.title.x = element_text(size = 12, face = "bold"),
    axis.title.y = element_text(size = 12, face = "bold")
  )
```

::: {.cell-output-display}
![](Stephanie_housing_request_files/figure-html/unnamed-chunk-5-1.png){width=672}
:::
:::


Apartments such as Royal Crest and La Jolla stand out for offering relatively lower rent prices at $248.75 and $294.75, respectively, while Gates and Towers II are at the higher end of the spectrum, with rent prices around $438.12 and $411.25. Despite their higher rents, these apartments still offer a moderate number of residents (248–300), showing that there are both affordable and more expensive options available with varying social environments.

### Time to MC vs. Apartments



::: {.cell}

```{.r .cell-code}
ggplot(filtered_apartments, aes(x = reorder(Name, -as.numeric(gsub(" minutes", "", TimeDistanceToMC))), y = as.numeric(gsub(" minutes", "", TimeDistanceToMC)))) +
  geom_boxplot(fill = "lightblue", color = "darkblue") +
  labs(title = "Walking Time to MC Building", x = "Apartment", y = "Walking Time (minutes)") +
  theme_minimal() +
  coord_flip()
```

::: {.cell-output-display}
![](Stephanie_housing_request_files/figure-html/unnamed-chunk-6-1.png){width=672}
:::
:::


Looking at walking time to the MC, apartments like Carriage House and Heritage stand out with the shortest walking times (6 minutes), making them ideal for students like Stephanie who prefer being close to campus. La Jolla, on the other hand, is further away with a 15-minute walk. Note: Time was calculated by distance "as the crow flies".

### Time to Ricks vs. Apartments



::: {.cell}

```{.r .cell-code}
ggplot(filtered_apartments, aes(x = reorder(Name, -as.numeric(gsub(" minutes", "", TimeDistanceToRicks))), y = as.numeric(gsub(" minutes", "", TimeDistanceToRicks)))) +
  geom_boxplot(fill = "lightgreen", color = "darkgreen") +
  labs(title = "Walking Time to Ricks Building", x = "Apartment", y = "Walking Time (minutes)") +
  theme_minimal() +
  coord_flip()
```

::: {.cell-output-display}
![](Stephanie_housing_request_files/figure-html/unnamed-chunk-7-1.png){width=672}
:::
:::


With walking time to the Ricks Building, Legacy Ridge stands out as the closest apartment with just a 6-minute walk, while La Jolla and Sundance are further away, with walking times of 15 and 13 minutes, respectively.  Note: Time was calculated by distance "as the crow flies".


### Available Apartment Options Based on Stephanie's Criteria
The table is filtered according to Stephanie's preferences for rent, proximity, and social environment. Stephanie can click on any column header to sort the table in ascending or descending order based on her priorities.




::: {.cell}

```{.r .cell-code}
filtered_apartments <- filtered_apartments %>%
  select(Name, Monthly_Rent, Residents, TimeDistanceToMC, TimeDistanceToRicks)

datatable(filtered_apartments, options = list(lengthMenu = c(5, 10, 15)), extensions = "Responsive")
```

::: {.cell-output-display}


```{=html}
<div class="datatables html-widget html-fill-item" id="htmlwidget-0dba4dcb2b6f31e38393" style="width:100%;height:auto;"></div>
<script type="application/json" data-for="htmlwidget-0dba4dcb2b6f31e38393">{"x":{"filter":"none","vertical":false,"extensions":["Responsive"],"data":[["1","2","3","4","5","6","7","8","9","10"],["CEDARS, THE - WOMEN","CENTRE SQUARE - WOMEN","COVE, THE - WOMEN","GATES, THE - WOMEN","LEGACY RIDGE","LODGE, THE - WOMEN","NAUVOO HOUSE II","NORTHPOINT - WOMEN","ROYAL CREST","UNIVERSITY VIEW - WOMEN"],["$423.75","$362.25","$399.75","$438.12","$393.5","$418.92","$362.5","$399.75","$248.75","$337.5"],[444,546,342,300,288,550,338,548,342,450],["7","11","10","14","11","11","9","7","6","11"],["11","8","13","13","6","11","8","12","9","8"]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>Name<\/th>\n      <th>Monthly_Rent<\/th>\n      <th>Residents<\/th>\n      <th>TimeDistanceToMC<\/th>\n      <th>TimeDistanceToRicks<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"lengthMenu":[5,10,15],"columnDefs":[{"className":"dt-right","targets":3},{"orderable":false,"targets":0},{"name":" ","targets":0},{"name":"Name","targets":1},{"name":"Monthly_Rent","targets":2},{"name":"Residents","targets":3},{"name":"TimeDistanceToMC","targets":4},{"name":"TimeDistanceToRicks","targets":5}],"order":[],"autoWidth":false,"orderClasses":false,"responsive":true}},"evals":[],"jsHooks":[]}</script>
```


:::
:::


## Conclusion

The analysis shows that there are several housing options that meet Stephanie's criteria of affordability, proximity to campus, and social atmosphere. These apartments provide a solid starting point for Stephanie’s housing search, ensuring she finds a place that supports both her financial and social needs.



