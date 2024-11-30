---
title: "Wings To Fly"
subtitle: "An Analysis of Delta airlines flights"
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


### Research Questions:
Which Origin Airport Minimizes the Chances of Late Arrival?\
Which Destination Airport is the Worst for Arrival Delays?

### Libraries being used:

::: {.cell}

```{.r .cell-code}
library(tidyverse)
library(nycflights13)
```
:::


### Filtering for Delta Airlines

::: {.cell}

```{.r .cell-code}
delta_flights <- flights %>%
  filter(carrier == "DL", !is.na(arr_delay))
```
:::


### Which Origin Airport Minimizes the Chances of Late Arrival?

::: {.cell}

```{.r .cell-code}
# If Arrival is late
delta_flights <- delta_flights %>%
  mutate(late_arrival = if_else(arr_delay > 0, "Late", "On Time"))

# Proportion late arrivals summary
origin_late_prop <- delta_flights %>%
  group_by(origin) %>%
  summarize(total_flights = n(),
            late_arrivals = sum(late_arrival == "Late"),
            proportion_late = late_arrivals / total_flights) %>%
  arrange(proportion_late)


ggplot(origin_late_prop, aes(x = reorder(origin, -proportion_late), y = proportion_late * 100, fill = origin)) +
  geom_bar(stat = "identity", color = "black") +
  scale_y_continuous(labels = scales::percent_format(scale = 1)) +  # Percentage
  labs(title = "Late Arrivals by Origin (Delta Airlines)",
       x = "Airport",
       y = "Late Arrivals") +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))  # Rotate x-axis labels
```

::: {.cell-output-display}
![](Wings_to_Fly_files/figure-html/unnamed-chunk-3-1.png){width=672}
:::
:::

The best origin airport for minimizing the chances of a late arrival is JFK (John F. Kennedy), where only 30% of the flights arrive late.\
LGA (LaGuardia) comes second, with a 35% late arrival rate.\
EWR (Newark) has the highest proportion of late arrivals at 40%, meaning it’s less reliable in avoiding delays.\


### Which Destination Airport is the Worst for Arrival Delays?

::: {.cell}

```{.r .cell-code}
# Summarize Average arrival delay
dest_avg_delay <- delta_flights %>%
  group_by(dest) %>%
  summarize(avg_arr_delay = mean(arr_delay, na.rm = TRUE)) %>%
  arrange(desc(avg_arr_delay))


ggplot(dest_avg_delay, aes(x = reorder(dest, -avg_arr_delay), y = avg_arr_delay)) +
  geom_bar(stat = "identity", fill = "steelblue", color = "black") +
  labs(title = "Average Arrival Delay by Destination (Delta Airlines)",
       x = "Destination Airport",
       y = "Average Arrival Delay (minutes)") +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))  # Rotates x-axis labels
```

::: {.cell-output-display}
![](Wings_to_Fly_files/figure-html/unnamed-chunk-4-1.png){width=672}
:::
:::

PBI (Palm Beach International Airport) is the worst destination for arrival delays, with an average delay of 18.5 minutes.\
ATL (Hartsfield-Jackson Atlanta International Airport) is also notable for delays, with an average delay of 15.2 minutes.\
MIA (Miami International Airport), FLL (Fort Lauderdale), and TPA (Tampa) round out the top 5 with delays ranging from 12.4 to 14.8 minutes.\

### Conclusion:
If you are flying Delta Airlines and want to minimize your chances of a late arrival, choose EWR as your origin.\
If you are worried about arrival delays, avoid flights to PBI and ATL, which are the worst for average arrival delays.


