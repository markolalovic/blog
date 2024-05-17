---
layout: post
author: Marko Lalovic
title: "Covid Calculator"
subtitle: "A visual tool to explore and analyze the impacts of Covid-19."
date: "2020-06-03"
nav_order: 4
keywords: covid, data visualization
refs: covid-calc
--- 
This blog post introduces ***Covid Calculator***. This tool allows for interactive data visualization to explore and analyze the potential effects of Covid-19. The tool has been translated into multiple languages and allows users to derive their own country-specific estimates based on current data. It is accessible at:

<div class="mono-italic">
<a href="https://markolalovic.github.io/covid-calc">https://markolalovic.github.io/covid-calc</a>
</div>

A screenshot of the Covid Calculator is shown in <a href="#figure1">Figure 1</a>. It shows the estimated number of deaths from Covid-19 based on the selected infection rate (30%) and infection fatality rates {% cite imperial --file {{ page.refs }} %}, and how they compare to other major causes of deaths in the US.

<div id="figure1" class="imgcap">
  <img src="/blog/assets/posts/covid-calc/covid-calc-deaths-estimates-for-us.png" width="90%" class="invertImg">
  <div class="thecap">
    Figure 1: A screenshot of the Covid Calculator.
  </div>
</div>

It began as a simple bar chart that displayed mortality by age for a selected country. Over time, by adding new features, it evolved into a more complex tool. This blog post is meant to describe technical details.

{: .note-title }
> Disclaimer
>
> The author is not a health expert or an epidemiologist and disclaims responsibility for any adverse effect resulting, directly or indirectly, from information contained in this website. For health, safety, and medical emergencies or updates on the novel coronavirus pandemic, you can get the latest information from <a href="https://www.who.int/emergencies/diseases/novel-coronavirus-2019">WHO</a> or search for official public health information for your country.


## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## Introduction
At the time of writing, the impacts of Covid-19 remain largely uncertain and depend on a whole range of possibilities. Organizing the overwhelming mass of the available information in the media and literature, coming up with a reasonable working estimates and comparing multiple scenarios can be challenging As an attempt to address this problem I used publicly available data and published information to create an online tool called *Covid Calculator* that allows users to derive their own country-specific estimates.

The tool is focused on simple presentation and pedagogical aspects and only offers crude estimates. It uses relatively simplistic methodology outlined below.

There are lots of improvements possible or more things to consider. One is to also include estimated fatality rates of Covid-19 by pre-existing health conditions. Having time to event data and applying survival analysis techniques would result in a more sensible estimates of expected years of life lost. Allowing parameters to evolve over time and comparing different time spans is another improvement.


## Age structure
We denote the number of something with $n(\cdot)$, for example 

$$
n(\text{people in the world}) \approx 7.59B.
$$

For selected location population we use data about age from {% cite pyramids --file {{ page.refs }} %} and divide it in 9 intervals or *age groups*:

$$
\begin{align*}
G = \{ \text{0-9}, \text{10-19}, \ldots, \text{70-79}, \text{80+} \}.
\end{align*}
$$

The size of population $N(g)$ by age group $g$ is estimated by counting how many people fall into each age group $g$:

$$
N(g) = n(\text{people in age group g})
$$

and total population size $N$ is

$$
N = \sum_{g \text{ in } G}
  n(\text{people in age group g}).
$$

For a more detailed analysis, age groups are divided into two sets

$$
\begin{align*}
G_{<60} &= \{ \text{0-9}, \text{10-19}, \ldots, \text{50-59} \} \\[.5em]
G_{60+} &= \{ \text{60-69}, \text{70-79}, \text{80+} \}.
\end{align*}
$$

The proportion of people over 60 in selected population is calculated as

$$ d_{60+} = \sum_{g \text{ in } G_{60+}} N(g) / N. $$

## Fatality rates
*Infection Fatality Rate (IFR)*, see {% cite imperial --file {{ page.refs }} %}, represents the proportion of deaths among all the infected individuals:

$$\text{IFR} = n(\text{deaths}) / n(\text{infected}).$$

*Case Fatality Rate (CFR)*, see {% cite cfr_wiki --file {{ page.refs }} %}, represents the proportion of confirmed deaths among all confirmed infected individuals:

$$ \text{CFR} = n( \text{confirmed cases of deaths} ) / n( \text{confirmed cases of infected} ). $$

By default, the estimates $\text{IFR}(g)$ by age groups are from {% cite imperial --file {{ page.refs }} %}. Users can also choose to use estimates of $\text{CFR}(g)$ by age groups based on data from different countries.

There is a big difference between the two measures. For example if in some particular time frame we have 5 confirmed cases of people infected and 2 confirmed deaths, then $CFR = 2/5 = 0.4$. However, if based on some other data and not only on confirmed cases, we know that there are actually more people infected, than our estimated IFR will be smaller than $\text{CFR}$.

Users can also adjust the fatality rate of each age group by input parameter $F$. It represents percent of increase or decrease.

*Fatality Rates* $\text{FR}(g)$ by age groups are calculated by multiplying selected estimates of fatality rate for each age group by $1 + F/100$ and used to estimate actual $\text{IFR}$:

$$ \text{FR}(g) = \text{*FR}(g) \cdot (1 + F/100). $$

Here, $\text{*FR}$ is estimated $\text{IFR}$ or user selected $\text{CFR}$ estimate for a particular country.

Note:

* Since "confirmed cases of infected" is a subset of "infected", wider testing can reduce $\text{CFR}$ estimates.
* When using $\text{CFR}$, the expected number of deaths in age group 0-9 is always 0 since no children under 10 appear to have died from Covid-19 when this data was aquired.
* Our proposed approach for later estimation assumes that the fatality rate by age in selected location has distribution similar to that estimated by {% cite imperial --file {{ page.refs }} %} or observed in the country of selected $\text{CFR}$.


## Proportion of infected
The selected *proportion of infected* $H$ is:

$$
H = n(\text{infected}) \cdot 100 / N.
$$

Users can adjust the proportion of infected people over 60 using $H_{60+}$. The overall $H$ can be decomposed as:

$$ 
\begin{equation}
H = (1 - d_{60+}) \cdot H_{<60} + d_{60+} \cdot H_{60+} 
\label{eq: decomposition}
\end{equation}
$$

where $H_{<60}$ is proportion of infected people below 60.

## Probability of eliminating Covid-19
Let $A$ be the event of achieving complete elimination of Covid-19 disease before it manages to infect proportion of population $H$. Let $I_{A}$ be the indicator variable for event $A$. Let

$$
\begin{align*}
E &= \text{Pr}(I_{A} = 1) \cdot 100 \\[.5em]
U &= n(\text{infected until elimination}) \cdot 100 / N.
\end{align*}
$$

The proportion of people over 60 infected until elimination $U_{60+}$ is calculated from 

$$
U_{60+} / U = H_{60+} / H
$$

and proportion of people below 60 infected until elimination $U_{<60}$ from decomposition (\ref{eq: decomposition}):

$$
U = (1 - d_{60+}) \cdot U_{<60} + d_{60+} \cdot U_{60+}.
$$

## Expected number of infected
The expected number of infected in age group $g$ is estimated as

$$
  \mathbb{E} \left[ n(\text{infected in age group g}) \right] =
    (1 - E/100) \cdot N(g) \cdot H_{\text{*}} + E/100 \cdot N(g) \cdot U_{\text{*}}.
$$

Here, `*` in $U_{\text{*}}$ is either `<60` or `60+`.

## Expected number of deaths
Expected number of deaths in age group $g$ in $G$ is estimated as

$$
  \mathbb{E} \left[ n(\text{deaths in age group g}) \right]  =
  \mathbb{E} \left[ n(\text{infected in age group g}) \right]  \cdot \text{FR}(g).
$$

Total expected numbers are simply sums over all age groups

$$
\begin{align*}
  \mathbb{E} \left[ n(\text{infected}) \right] &= \sum_{g \text{ in } G} 
  \mathbb{E} \left[  n(\text{infected in age group g}) \right]  \\
  \mathbb{E} \left[  n(\text{deaths}) \right] &= \sum_{g \text{ in } G} 
  \mathbb{E} \left[  n(\text{deaths in age group g}) \right].
\end{align*}
$$

## Expected years of life lost
For this, the life table for global population {% cite expectancies --file {{ page.refs }} %} with estimates about expected number of life years left for all ages in 2016 was used. For example, a person at the age of 60 had 20.5 expected number of life years left in 2016.

Next, by taking the average by gender and by age in a specific 10 year age group as an estimate of life expectancy of individual from a particular age group, calculate $\text{life exp}_g$. Finally, estimate *expected years of life lost (EYLL)* due to Covid-19 as:

$$
	\text{EYLL} = 
	\sum_{g \text{ in } G}
	\mathbb{E} \left[  n(\text{deaths in age group g}) \right] \cdot \text{life exp}_g.
$$

## Actual years of life lost
Years of life lost are estimated from confirmed deaths due to Covid-19 in the following way. First, calculate mean $\text{CFR}$ by age group from all the $\text{CFR}$ estimates by age group from countries as presented in <a href="#table1">Table 1</a>.

{% include posts/covid-calc/cfr-table.html %}

Next, let $D$ be the event that a person died from Covid-19 and let $A_g$ be the event that this persons age is in the age group $g$. If there is $n(\text{deaths})$ in selected country from Covid-19, estimate *years of life lost (YLL)* due to Covid-19 as:

$$
	\text{YLL} = n(\text{deaths}) \cdot \sum_{g \text{ in } G} Pr(A_g | D) \cdot \text{life exp}_g
$$

The probability of a person who died in a particular country being in age group $g$ is estimated as

$$
Pr(A_g | D) = \frac{Pr(D | A_g) Pr(A_g)}{P(D)}.
$$

Here,

$$
\begin{align*}
Pr(D | A_g) &= \text{estimated mean } \text{CFR}_g \\[.5em]
Pr(A_g) &= \text{proportion of people in age group } g \text{ in particular population} \\[.5em]
Pr(D) &= \sum_{g \text{ in } G} Pr(D | A_g) Pr(A_g).
\end{align*}
$$

#### Example
For example, in Italy there were 24114 confirmed deaths from Covid-19 until the end of May. Based on the demographics and life expectancies of Italy, we estimate 294369.66 years of life lost in Italy from Covid-19.

## Projected costs
A figure of \$129,000 represents what it would cost to give a person an additional *quality-of-life adjusted* year of life {% cite price --file {{ page.refs }} %}. This figure is multiplied with years of life lost to get estimated costs or money saved.

## Projected poverty
Take the difference between the IMF's GDP growth forecasts {% cite imf --file {{ page.refs }} %} for April 2020 and forecasts from October 2019. We take estimates of how many people lived in extreme poverty (income below \$1.9 per day) by country from 2015 from World Bank {% cite worldbank --file {{ page.refs }} %}. Then estimate the projected poverty increases by country due to Covid-19 by multiplying this number and subtract the difference in GDB growth forecasts.

#### Example
For example in Nigeria the difference between IMF's GDP growth forecasts is –5.9 percent and World Bank estimated that there were 85.15 million people living in extreme poverty. So we estimate there will be 

$$
5.15M (5.9 / 100) \approx 5.02M
$$

additional people living in extreme poverty in Nigeria due to Covid-19 pandemic.

Note: This are crude estimats of Covid-19 effect on poverty because not only Covid-19 pandemic has caused the IMF to alter its forecasts.


## References

{% bibliography --file {{ page.refs }} --cited %}
