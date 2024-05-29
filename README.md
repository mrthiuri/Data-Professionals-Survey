# Data-Professionals-Survey
This repository contains an R markdown file whose content lies in the range of data cleaning to data visualisation of the survey conducted on various data careers in Kenya.

## Overview
This document outlines the steps and methodologies used in analyzing the Phoenix Compensation Survey data using R. The goal of this analysis is to extract meaningful insights and patterns from the responses to inform decision-making or provide actionable recommendations.

### Table of Contents
1.Introduction
2.Data Import, Cleaning, and Wrangling
3.Exploratory Data Analysis (EDA)
4.Statistical Analysis and Visualization
5.Insights and Conclusions
6.Tools and Libraries Used
7.Contact Information

### 1. Introduction
#### Project Details
Title: Phoenix Compensation Survey
Date: 2024-05-28
Analyst: Morgan Thiuri, Data Analyst and Aspiring Data Scientist
Key Findings
The survey consisted of 173 respondents.
Gender split: 49% male and 50% female.
33 different job titles from data professionals in Kenya.
Most common role: Data Analyst.
Majority hold Mid-Level roles.
Typical experience: 3-5 years.
Common work setup: Full office.
Average salary: Ksh 109,362.54.
Tools Used
R packages: tidyverse, janitor, skimr, readxl, stringr, ggplot2, wordcloud2.
### 2. Data Import, Cleaning, and Wrangling
Data Import
r
# importing packages
library(tidyverse)
library(janitor)
library(skimr)
library(readxl)
library(stringr)
library(ggplot2)
library(wordcloud2)

# importing the dataset
file_path <- "C:/Users/user pc/Desktop/My Projects/Compensation Analysis/compensation data (Responses).xlsx"
comp_v1 <- read_excel(file_path)
View(comp_v1)

# obtaining a brief overview of the dataset
skim_without_charts(comp_v1)
Data Cleaning
Initial Steps:
Created a working copy of the dataset.
Dropped the timestamp column.
Renamed columns for clarity.
r
comp_v2 <- comp_v1
comp_v2 <- comp_v2 %>%
  select(-1) %>%
  rename(
    "Current_Role" = 2,
    "Other_roles" = 3,
    "Level" = 4,
    "Experience" = 5,
    "Industry" = 6,
    "Gender" = 7,
    "Main_tech_stack" = 8,
    "Other_relavant_tools" = 9,
    "Monthly_gross_salary" = 10,
    "Other_benefits" = 11,
    "Work_setup" = 12,
    "Employer_type" = 13
  )
View(comp_v2)
Addressing Inconsistencies:
Combined Main_tech_stack and Other_relavant_tools into a single column Tech_stack.
Handled missing values and standardized salary data.
Converted currencies to Kenyan Shillings.
r
# Data transformation
comp_v3 <- comp_v2 %>%
  mutate(Main_tech_stack = ifelse(str_detect(Main_tech_stack, ", Other$"), str_replace(Main_tech_stack, ", Other$", ""), Main_tech_stack)) %>%
  mutate(Main_tech_stack = ifelse(Main_tech_stack == 'Other', Other_relavant_tools, Main_tech_stack)) %>%
  rename("Tech_stack" = Main_tech_stack) %>%
  mutate(Tech_stack = ifelse(is.na(Other_relavant_tools), Tech_stack, paste(Tech_stack, Other_relavant_tools, sep = ","))) %>%
  mutate(Tech_stack = ifelse(Tech_stack == "Microsoft office,Microsoft office", "Microsoft office", Tech_stack)) %>%
  select(-Other_relavant_tools)

# Handling salary inconsistencies
exchange_rate_eur = 144.6
exchange_rate_dollar = 133

comp_v3 <- comp_v3 %>%
  mutate(Monthly_gross_salary = case_when(
    Monthly_gross_salary == "920 EUROS" ~ 920 * exchange_rate_eur,
    Monthly_gross_salary == "500$" ~ 500 * exchange_rate_dollar,
    Monthly_gross_salary == "15,000/-" ~ 15000,
    Monthly_gross_salary == "112,000KES" ~ 112000,
    Monthly_gross_salary == "Unemployed" ~ 0,
    Monthly_gross_salary %in% c("Nill", "Nan", "n/a") ~ 0,
    TRUE ~ as.numeric(Monthly_gross_salary)
  )) %>%
  mutate(Monthly_gross_salary = ifelse(is.na(Monthly_gross_salary), mean(Monthly_gross_salary, na.rm = TRUE), Monthly_gross_salary))

# Final cleanup
clean_names(comp_v3)
### 3. Exploratory Data Analysis (EDA)
Overview of Data:
Checked distribution of job roles, levels, and industries.
Examined gender distribution and employer types.
Assessed work setups and experience levels.
Data Distribution
r

# Distribution of data professionals by roles
ggplot(data=comp_v3) +
  geom_bar(mapping=aes(x=Current_Role, fill=Current_Role)) +
  geom_text(stat = 'count', aes(x = Current_Role, label = ..count..), vjust = -0.1) +
  labs(x="Data Professional", y="Number of professionals from survey", title="Distribution of data professionals across different roles", subtitle="Data from the Phoenix Ke Analytics Survey", caption="Data Analysis done by Morgan Thiuri")

# Distribution by gender
ggplot(data=comp_v3) +
  geom_bar(mapping=aes(x=Gender, fill=Current_Role)) +
  geom_text(stat = 'count', aes(x = Gender, label = ..count..), vjust = -0.1) +
  labs(x="Gender", y="Number of professionals from survey", title="Distribution of data professionals by Gender and roles", subtitle="Data from the Phoenix Ke Analytics Survey", caption="Data Analysis done by Morgan Thiuri")
### 4. Statistical Analysis and Visualization
Gender and Role Distribution:
Observed gender parity in data roles with a notable exception in Data Engineering.
Employer Type:
Visualized the distribution across different employer types.
Work Setup and Experience
r
Copy code
# Work setup visualization
ggplot(data=comp_v3) +
  geom_bar(mapping=aes(x=Work_setup, fill=Work_setup)) +
  geom_text(stat = 'count', aes(x = Work_setup, label = ..count..), vjust = -0.1) +
  labs(x="Employer", y="Number of professionals from survey", title="Distribution of data professionals across different work setups", subtitle="Data from the Phoenix Ke Analytics Survey", caption="Data Analysis done by Morgan Thiuri") +
  theme(axis.text.x = element_blank())

# Experience levels
ggplot(data=comp_v3) +
  geom_bar(mapping=aes(x=Experience, fill=Experience)) +
  geom_text(stat = 'count', aes(x = Experience, label = ..count..), vjust = -0.1) +
  labs(x="Years of Experience", y="Number of professionals from survey", title="Distribution of data professionals across different experience levels", subtitle="Data from the Phoenix Ke Analytics Survey", caption="Data Analysis done by Morgan Thiuri")
### 5. Insights and Conclusions
Key Trends and Patterns
Data professionals in Kenya are predominantly in mid-level roles.
There is a balanced gender representation, although fewer female Data Engineers.
Majority of professionals prefer a full office setup.
Tool Usage
Most commonly used tools: Microsoft Office, R, Power BI.
Recommendations
Encourage diversity in Data Engineering roles.
Promote flexible work setups to attract diverse talent.
Limitations
Open-ended responses led to data inconsistencies.
Currency variations required assumptions for conversions.
### 6. Tools and Libraries Used
tidyverse: Data manipulation and visualization.
janitor: Data cleaning.
skimr: Data summarization.
readxl: Reading Excel files.
stringr: String operations.
ggplot2: Data visualization.
wordcloud2: Creating word clouds.
7. Contact Information
Name: Morgan Thiuri
LinkedIn: Connect with Morgan
Conclusion
This README provides a comprehensive overview of the Phoenix Compensation Survey analysis process, ensuring transparency and reproducibility. For any further questions or clarifications, please contact Morgan Thiuri.

This document should be updated as necessary to reflect any changes or additional steps taken during the analysis process.





User
follow the same format as before for the readme file, just follow the data I just provided
ChatGPT
README: Phoenix Compensation Survey Analysis
Overview
This document outlines the steps and methodologies used in analyzing the Phoenix Compensation Survey data using R. The goal of this analysis is to extract meaningful insights and patterns from the responses to inform decision-making or provide actionable recommendations.

Table of Contents
Introduction
Data Import, Cleaning, and Wrangling
Exploratory Data Analysis (EDA)
Statistical Analysis and Visualization
Insights and Conclusions
Tools and Libraries Used
Contact Information
1. Introduction
Project Details
Title: Phoenix Compensation Survey
Date: 2024-05-28
Analyst: Morgan Thiuri, Data Analyst and Aspiring Data Scientist
Key Findings
The survey consisted of 173 respondents.
Gender split: 49% male and 50% female.
33 different job titles from data professionals in Kenya.
Most common role: Data Analyst.
Majority hold Mid-Level roles.
Typical experience: 3-5 years.
Common work setup: Full office.
Average salary: Ksh 109,362.54.
Tools Used
R packages: tidyverse, janitor, skimr, readxl, stringr, ggplot2, wordcloud2.
2. Data Import, Cleaning, and Wrangling
Data Import
r
Copy code
# importing packages
library(tidyverse)
library(janitor)
library(skimr
library(readxl)
library(stringr)
library(ggplot2)
library(wordcloud2)

# importing the dataset
file_path <- "C:/Users/user pc/Desktop/My Projects/Compensation Analysis/compensation data (Responses).xlsx"
comp_v1 <- read_excel(file_path)
View(comp_v1)

# obtaining a brief overview of the dataset
skim_without_charts(comp_v1)
Data Cleaning
Initial Steps:
Created a working copy of the dataset.
Dropped the timestamp column.
Renamed columns for clarity.
r
Copy code
comp_v2 <- comp_v1
comp_v2 <- comp_v2 %>%
  select(-1) %>%
  rename(
    "Current_Role" = 2,
    "Other_roles" = 3,
    "Level" = 4,
    "Experience" = 5,
    "Industry" = 6,
    "Gender" = 7,
    "Main_tech_stack" = 8,
    "Other_relavant_tools" = 9,
    "Monthly_gross_salary" = 10,
    "Other_benefits" = 11,
    "Work_setup" = 12,
    "Employer_type" = 13
  )
View(comp_v2)
Addressing Inconsistencies:
Combined Main_tech_stack and Other_relavant_tools into a single column Tech_stack.
Handled missing values and standardized salary data.
Converted currencies to Kenyan Shillings.
r
Copy code
# Data transformation
comp_v3 <- comp_v2 %>%
  mutate(Main_tech_stack = ifelse(str_detect(Main_tech_stack, ", Other$"), str_replace(Main_tech_stack, ", Other$", ""), Main_tech_stack)) %>%
  mutate(Main_tech_stack = ifelse(Main_tech_stack == 'Other', Other_relavant_tools, Main_tech_stack)) %>%
  rename("Tech_stack" = Main_tech_stack) %>%
  mutate(Tech_stack = ifelse(is.na(Other_relavant_tools), Tech_stack, paste(Tech_stack, Other_relavant_tools, sep = ","))) %>%
  mutate(Tech_stack = ifelse(Tech_stack == "Microsoft office,Microsoft office", "Microsoft office", Tech_stack)) %>%
  select(-Other_relavant_tools)

# Handling salary inconsistencies
exchange_rate_eur = 144.6
exchange_rate_dollar = 133

comp_v3 <- comp_v3 %>%
  mutate(Monthly_gross_salary = case_when(
    Monthly_gross_salary == "920 EUROS" ~ 920 * exchange_rate_eur,
    Monthly_gross_salary == "500$" ~ 500 * exchange_rate_dollar,
    Monthly_gross_salary == "15,000/-" ~ 15000,
    Monthly_gross_salary == "112,000KES" ~ 112000,
    Monthly_gross_salary == "Unemployed" ~ 0,
    Monthly_gross_salary %in% c("Nill", "Nan", "n/a") ~ 0,
    TRUE ~ as.numeric(Monthly_gross_salary)
  )) %>%
  mutate(Monthly_gross_salary = ifelse(is.na(Monthly_gross_salary), mean(Monthly_gross_salary, na.rm = TRUE), Monthly_gross_salary))

# Final cleanup
clean_names(comp_v3)
3. Exploratory Data Analysis (EDA)
Overview of Data:
Checked distribution of job roles, levels, and industries.
Examined gender distribution and employer types.
Assessed work setups and experience levels.
Data Distribution
r
Copy code
# Distribution of data professionals by roles
ggplot(data=comp_v3) +
  geom_bar(mapping=aes(x=Current_Role, fill=Current_Role)) +
  geom_text(stat = 'count', aes(x = Current_Role, label = ..count..), vjust = -0.1) +
  labs(x="Data Professional", y="Number of professionals from survey", title="Distribution of data professionals across different roles", subtitle="Data from the Phoenix Ke Analytics Survey", caption="Data Analysis done by Morgan Thiuri")

# Distribution by gender
ggplot(data=comp_v3) +
  geom_bar(mapping=aes(x=Gender, fill=Current_Role)) +
  geom_text(stat = 'count', aes(x = Gender, label = ..count..), vjust = -0.1) +
  labs(x="Gender", y="Number of professionals from survey", title="Distribution of data professionals by Gender and roles", subtitle="Data from the Phoenix Ke Analytics Survey", caption="Data Analysis done by Morgan Thiuri")
4. Statistical Analysis and Visualization
Gender and Role Distribution:
Observed gender parity in data roles with a notable exception in Data Engineering.
Employer Type:
Visualized the distribution across different employer types.
Work Setup and Experience
r
Copy code
# Work setup visualization
ggplot(data=comp_v3) +
  geom_bar(mapping=aes(x=Work_setup, fill=Work_setup)) +
  geom_text(stat = 'count', aes(x = Work_setup, label = ..count..), vjust = -0.1) +
  labs(x="Employer", y="Number of professionals from survey", title="Distribution of data professionals across different work setups", subtitle="Data from the Phoenix Ke Analytics Survey", caption="Data Analysis done by Morgan Thiuri") +
  theme(axis.text.x = element_blank())

# Experience levels
ggplot(data=comp_v3) +
  geom_bar(mapping=aes(x=Experience, fill=Experience)) +
  geom_text(stat = 'count', aes(x = Experience, label = ..count..), vjust = -0.1) +
  labs(x="Years of Experience", y="Number of professionals from survey", title="Distribution of data professionals across different experience levels", subtitle="Data from the Phoenix Ke Analytics Survey", caption="Data Analysis done by Morgan Thiuri")
5. Insights and Conclusions
Key Trends and Patterns
Data professionals in Kenya are predominantly in mid-level roles.
There is a balanced gender representation, although fewer female Data Engineers.
Majority of professionals prefer a full office setup.
Salary Insights
Highest salary: Ksh 920 * exchange_rate_eur.
Lowest salary: Ksh 0.
Average salary: Ksh 109,362.54.
Tool Usage
Most commonly used tools: Microsoft Office, R, Power BI.
Recommendations
Encourage diversity in Data Engineering roles.
Promote flexible work setups to attract diverse talent.
Limitations
Open-ended responses led to data inconsistencies.
Currency variations required assumptions for conversions.
6. Tools and Libraries Used
tidyverse: Data manipulation and visualization.
janitor: Data cleaning.
skimr: Data summarization.
readxl: Reading Excel files.
stringr: String operations.
ggplot2: Data visualization.
wordcloud2: Creating word clouds.
7. Contact Information
Name: Morgan Thiuri
LinkedIn: Connect with Morgan
Conclusion
This README provides a comprehensive overview of the Phoenix Compensation Survey analysis process, ensuring transparency and reproducibility. For any further questions or clarifications, please contact Morgan Thiuri.
