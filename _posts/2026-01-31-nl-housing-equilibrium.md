---
layout: post
title:  "4% rule in the Netherlands"
date:   2026-01-31 00:00:00
categories: finance
image: nl-housing.jpg
---

One of my favourite finance youtubers, Ben Felix, made [multiple videos](https://www.youtube.com/watch?v=Uwl3-jBNEd4) on "Rent vs Buy" topic, addressing different aspects of this question. One common conclustion is a 5% rule, which sounds like "If your yearly rent is less than 5% of the house price, renting may be more financially beneficial". Facing the same question in real life, I decided to do my own calculations with some additional considerations. Surpisingly, I arrived at roughly the same number.

![img1](https://eknm-hub-public.s3.eu-central-1.amazonaws.com/nl-housing-math/g1.png)

However, as I was running experiments with randomized settings, the deviation was huge. Sometimes it's a 3% rule, sometimes it's a 9%. Both of these numbers are very unrealistic in the real life, but at least it's easy to test what affects them in what way.

## Mortgage rate

Obviously, one of the most important numbers is the mortgage rate, and, naturally if affects the results significantly. 

| Mortgage Rate | Mean  | Median |
|--------------:|:-----:|:------:|
| 2%           | 4.2%  | 3.9%   |
| 2.5%         | 4.4%  | 4.1%   |
| 3%           | 4.64% | 4.4%   |
| 3.5%         | 4.9%  | 4.6%   |
| 4%           | 5.15% | 4.9%   |
| 4.5%         | 5.41% | 5.2%   |
| 5%           | 5.7%  | 5.5%   |
| 5.5%         | 5.99% | 5.8%   |
| 6%           | 6.29% | 6.1%   |

Current mortagage rates in Netherlands hover around 4%, so this reinforces the 5% result number.

## House appreciation / Rent increase

Yearly house value appreciating and yearly rent increase usually correlate with each other. At the same time, first positively affects the landlord and second negatively affects the renter. 

| Yearly appreciation | Mean  | Median |
|--------------------:|:-----:|:------:|
| 1%                 | 7.34% | 7.0%   |
| 2%                 | 6.41% | 6.1%   |
| 3%                 | 5.48% | 5.2%   |
| 4%                 | 4.51% | 4.3%   |
| 5%                 | 3.8%  | 3.6%   |

In 2025, the price growth was 7.8%. Obviously, this number is unsustainable in the long term, but given the housing shortage and lack of radical decisions to resolve it, I'd expect long-term number to be between 3% and 4%. Once again, 5% rule is confirmed.

## Move-adjusted rent

This parameter turned out to be utterly insignificant, but I decided to include it here because of two reasons:
1. To demostrate that some number don't really matter
2. I spent some time to research and code it

So, what is "move adjustment"? Renters change their home every 2.5 to 3 years in average. Until recently, 2-year contracts were the norm in the Netherlands. At the same time, homeowners move significantly less often, as primary moving reason is an upgrade. More details about calculation can be found [here](https://nl-housing-math.streamlit.app/adjusted_rent_view), but TLDR is that moving increases real rent cost by 1-3%. Here's how it translates to rent-buy equilibrium:

| Adjustment      | Mean  | Median |
|:----------------|:-----:|:------:|
| No adjustment   | 5.57% | 5.40%  |
| 1% increase     | 5.48% | 5.30%  |
| 2% increase     | 5.44% | 5.30%  |
| 3%              | 5.40% | 5.20%  |
| 4%              | 5.33% | 5.10%  |
| 5%              | 5.30% | 5.10%  |

## Tax discounts

Taxes are very hard to account for, as they significantly depend on personal circumstances. But if you can get 10% of your house payments back (in average), which is absolutely realistic in the Netherlands, the resulting distribution will look like this:

![img2](https://eknm-hub-public.s3.eu-central-1.amazonaws.com/nl-housing-math/g2.png)

## Reality check

According to Gemini in reasearch mode, here's how rental yields look like in 2015 in major Dutch cities:

| City        | 1-Bed Yield | 2-Bed Yield | Avg Yield |
|:------------|:-----------:|:-----------:|:---------:|
| Amsterdam   | 6.77%       | 6.44%       | 5.35%     |
| The Hague   | 7.37%       | 6.28%       | 6.57%     |
| Rotterdam   | 6.56%       | 7.09%       | 6.91%     |

Obviously, these number don't account for a lot of things, but realistically, it's highly unlikely to rent something for less than 5% of the buy price, at least in 2025. That's basically it.