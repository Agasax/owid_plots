README
================
Created by Lars Mølgaard Saxhaug

Last compiled on Wednesday 30 June, 2021

``` r
owid_covid_data <- import("https://raw.githubusercontent.com/owid/covid-19-data/master/public/data/owid-covid-data.csv")

df <- owid_covid_data %>%
  select(iso_code, continent, location, date, population, new_cases_smoothed, people_fully_vaccinated, new_deaths_smoothed, hosp_patients, icu_patients, new_deaths, positive_rate, tests_per_case, people_vaccinated, new_vaccinations_smoothed_per_million, new_vaccinations) %>%
  mutate(
    adjusted_deaths = (new_deaths_smoothed / (population - people_fully_vaccinated)),
    hosp_adjusted = hosp_patients / (population - people_fully_vaccinated),
    icu_adj = icu_patients / (population - people_fully_vaccinated),
    cases_adj = new_cases_smoothed / (population - people_fully_vaccinated)
  )
```

#### Countries of interest:

Bulgaria, Denmark, Romania

#### Date range of interest

From Monday 01 March, 2021 to Wednesday 30 June, 2021

#### Adjusted deaths

``` r
df %>%
  filter(location %in% countries) %>%
  filter(date >= from_date) %>%
  mutate(adjusted_deaths = ifelse(is.na(adjusted_deaths), new_deaths_smoothed / population, adjusted_deaths)) %>%
  drop_na(adjusted_deaths) %>%
  ggplot(aes(y = adjusted_deaths, x = date, colour = location)) +
  geom_line(size = .8) +
  scale_x_date(name = "Date") +
  scale_y_continuous(name = NULL, labels = scales::label_number(scale = 100000)) +
  ggthemes::scale_colour_fivethirtyeight(name = "Country") +
  ggthemes::theme_fivethirtyeight() +
  labs(
    subtitle = "Daily new COVID-19 deaths per 100.000 of population not fully vaccinated \nRolling 7 day average",
    caption = "@load_dependent \n Datasource: https://ourworldindata.org"
  )
```

![](README_files/figure-gfm/adjusted_deaths-1.png)<!-- --> \#\#\#\#
Vaccinations

``` r
p1 <- df %>%
  filter(location %in% countries) %>%
  mutate(location = factor(location, levels = countries)) %>%
  filter(date >= from_date) %>%
  drop_na(people_fully_vaccinated) %>%
  ggplot(aes(y = people_fully_vaccinated / population, x = date, colour = location)) +
  geom_line(size = 1) +
  scale_x_date(name = "Date", date_breaks = "2 months", date_labels = "%B") +
  ggthemes::scale_colour_fivethirtyeight(name = "Country") +
  scale_y_continuous(name = "null", labels = scales::percent_format()) +
  ggthemes::theme_fivethirtyeight() +
  labs(subtitle = "Proportion of population fully vaccinated")

p2 <- df %>%
  filter(location %in% countries) %>%
  mutate(location = factor(location, levels = countries)) %>%
  filter(date >= from_date) %>%
  drop_na(new_vaccinations) %>%
  ggplot(aes(y = new_vaccinations / population, x = date, colour = location)) +
  geom_ma(size = 1, n = 14, linetype = 1) +
  scale_x_date(name = "Date", date_breaks = "2 months", date_labels = "%B") +
  ggthemes::scale_colour_fivethirtyeight(name = "Country") +
  scale_y_continuous(name = "null", labels = scales::label_number(scale = 1000000)) +
  ggthemes::theme_fivethirtyeight() +
  labs(subtitle = "Daily new vaccinations per million, 14 day rolling average")

p1 / p2 + plot_annotation(title = "Vaccination", caption = "@load_dependent \n Datasource: https://ourworldindata.org") +
  plot_layout(guides = "collect") & theme(legend.position = "bottom")
```

![](README_files/figure-gfm/vaccinations-1.png)<!-- --> \#\#\#\# New
cases adjusted for vaccination coverage

``` r
df %>%
  filter(location %in% countries) %>%
  filter(date >= from_date) %>%
  drop_na(cases_adj) %>%
  ggplot(aes(y = cases_adj, x = date, colour = location)) +
  geom_line(size = 1) +
  # scale_y_log10(name=NULL)+
  scale_x_date(name = "Date") +
  ggthemes::scale_colour_fivethirtyeight(name = "Country") +
  scale_y_continuous(name = "null", labels = scales::label_number(scale = 100000)) +
  ggthemes::theme_fivethirtyeight() +
  labs(subtitle = "Rolling new cases\nAdjusted for unvaccinated share of population\nPer 100.000")
```

![](README_files/figure-gfm/new%20cases-1.png)<!-- -->

#### Cases, admissions and deaths

``` r
value_names <- c(
  "hosp_adjusted" = "Adjusted number of hospitalised",
  "adjusted_deaths" = "Adjusted number of deaths",
  "icu_adj" = "Adjusted number of ICU beds"
)

df %>%
  filter(location %in% c("United Kingdom")) %>%
  filter(date >= as.POSIXct("2021-02-01")) %>%
  drop_na(hosp_adjusted, adjusted_deaths, icu_adj) %>%
  pivot_longer(cols = c(hosp_adjusted, adjusted_deaths, icu_adj)) %>%
  ggplot(aes(y = value, x = date)) +
  geom_line(size = .8, colour = "brown") +
  scale_x_date(name = "Date") +
  scale_y_continuous(name = NULL, labels = scales::label_number(scale = 100000)) +
  facet_wrap(~name, scales = "free_y", labeller = as_labeller(value_names)) +
  ggthemes::scale_colour_fivethirtyeight(name = "Country") +
  ggthemes::theme_fivethirtyeight() +
  labs(
    subtitle = "COVID-19 deaths, hospitalisations and ICU beds \nAdjusted for share of population not fully vaccinated (per 100.000)",
    caption = "@load_dependent \n Datasource: https://ourworldindata.org"
  )
```

![](README_files/figure-gfm/cases_adm_deathc-1.png)<!-- -->

#### Test positivity rate and tests per case in Norway, Denmark, Sweden

``` r
p1 <- df %>%
  filter(location %in% test_countries) %>%
  filter(date >= as.POSIXct("2021-2-01")) %>%
  drop_na(positive_rate) %>%
  ggplot(aes(x = date, y = positive_rate, colour = location)) +
  geom_line(alpha = 0.4) +
  geom_ma(n = 14, size = 1) +
  scale_x_date(name = "Date") +
  scale_y_continuous(name = NULL, labels = scales::percent_format()) +
  ggthemes::scale_colour_fivethirtyeight(name = "Country") +
  ggthemes::theme_fivethirtyeight() +
  labs(subtitle = "Test positivity rate\n7 day rolling average")


p2 <- df %>%
  filter(location %in% c("Norway", "Denmark", "Sweden")) %>%
  filter(date >= as.POSIXct("2021-2-01")) %>%
  drop_na(tests_per_case) %>%
  ggplot(aes(x = date, y = tests_per_case, colour = location)) +
  geom_line(alpha = 0.4) +
  geom_ma(n = 14, size = 1) +
  scale_x_date(name = "Date") +
  scale_y_continuous(name = NULL) +
  ggthemes::scale_colour_fivethirtyeight(name = "Country") +
  ggthemes::theme_fivethirtyeight() +
  labs(subtitle = "Tests per case\n7 day rolling average")

p1 + p2 + plot_annotation(caption = "@load_dependent \n Datasource: https://ourworldindata.org") +
  plot_layout(guides = "collect") & theme(legend.position = "bottom")
```

![](README_files/figure-gfm/tests-1.png)<!-- -->
