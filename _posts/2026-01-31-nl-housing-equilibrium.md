---
layout: post
title:  "4% rule in the Netherlands"
date:   2026-01-31 00:00:00
categories: finance
image: nl-housing.jpg
---
 
One of my favourite finance YouTubers, Ben Felix, made [multiple videos](https://www.youtube.com/watch?v=Uwl3-jBNEd4) on the "Rent vs Buy" topic, addressing different aspects of this question. One common conclusion is a 5% rule, which states: "If your yearly rent is less than 5% of the house price, renting may be more financially beneficial." Facing the same question in real life, I decided to do my own calculations with some additional considerations. Surprisingly, I arrived at roughly the same number.

![img1](https://eknm-hub-public.s3.eu-central-1.amazonaws.com/nl-housing-math/g1.png)

However, as I ran experiments with randomized settings, the deviation was huge. Sometimes it's a 3% rule, sometimes it's a 9% rule. Both of these numbers are unrealistic in practice, but they make it easy to test which parameters affect the equilibrium.

## Mortgage rate

Obviously, one of the most important inputs is the mortgage rate, and it affects the results significantly.

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

Current mortgage rates in the Netherlands hover around 4%, which reinforces the 5% rule.

## House appreciation / Rent increase

Yearly house value appreciation and yearly rent increases usually correlate. The former benefits the landlord; the latter makes renting more expensive for tenants.

| Yearly appreciation | Mean  | Median |
|--------------------:|:-----:|:------:|
| 1%                 | 7.34% | 7.0%   |
| 2%                 | 6.41% | 6.1%   |
| 3%                 | 5.48% | 5.2%   |
| 4%                 | 4.51% | 4.3%   |
| 5%                 | 3.8%  | 3.6%   |

In 2025, price growth was 7.8%. This rate is unsustainable in the long term, but given the housing shortage and lack of radical policy changes, I'd expect a long-term rate between 3% and 4%. Once again, the 5% rule is supported.

## Move-adjusted rent

This parameter turned out to be largely insignificant, but I include it here for two reasons:
1. To demonstrate that some numbers don't materially affect the result
2. I spent time researching and coding the adjustment

So, what is "move adjustment"? Renters change their home every 2.5 to 3 years on average. Until recently, 2-year contracts were the norm in the Netherlands. Homeowners move significantly less often, since the primary reason for moving is typically an upgrade. More details about the calculation can be found [here](https://nl-housing-math.streamlit.app/adjusted_rent_view), but TL;DR: moving increases real rent cost by 1–3%. Here's how it translates to the rent-buy equilibrium:

| Adjustment      | Mean  | Median |
|:----------------|:-----:|:------:|
| No adjustment   | 5.57% | 5.40%  |
| 1% increase     | 5.48% | 5.30%  |
| 2% increase     | 5.44% | 5.30%  |
| 3%              | 5.40% | 5.20%  |
| 4%              | 5.33% | 5.10%  |
| 5%              | 5.30% | 5.10%  |

## Tax discounts

Taxes are hard to account for because they depend heavily on personal circumstances. But if you can get 10% of your house payments back (on average), which is realistic in the Netherlands, the resulting distribution looks like this:

![img2](https://eknm-hub-public.s3.eu-central-1.amazonaws.com/nl-housing-math/g2.png)

## Reality check

According to Gemini in research mode, here's how rental yields looked in 2015 in major Dutch cities:

| City        | 1-Bed Yield | 2-Bed Yield | Avg Yield |
|:------------|:-----------:|:-----------:|:---------:|
| Amsterdam   | 6.77%       | 6.44%       | 5.35%     |
| The Hague   | 7.37%       | 6.28%       | 6.57%     |
| Rotterdam   | 6.56%       | 7.09%       | 6.91%     |

Obviously, these numbers don't account for many factors, but realistically it's highly unlikely to rent something for less than 5% of the purchase price, at least in 2025.
