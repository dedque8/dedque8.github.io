---
layout: post
title:  "TruffleHog vs probabilistic methods"
date:   2024-10-29 00:00:00
categories: sec
image: bomb_vs_baby.jpg
---

It's a typical situation: you are in a hurry, leave your keys somewhere, then try to find them and fail miserably. Kitchen table? Coat hangers? Random json file in git repo? Maybe. That's why tools like TruffleHog exist. They may be good, but are they perfect? Let's test it on an example of AWS IAM secret key, which usually looks like `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEx4mPL3kY1`. This is already a potential leak, but let's place two more truffles: one in an otherwise empty file with nonambiguous name `secret.key` and another one in a python snippet written to connect to AWS. Let's see if TruffleHog finds it:

```
🐷🔑🐷  TruffleHog. Unearth your secrets. 🐷🔑🐷

2024-10-29T08:52:07+01:00	info-0	trufflehog	running source	{"source_manager_worker_id": "TFlbD", "with_units": true}
2024-10-29T08:52:07+01:00	info-0	trufflehog	finished scanning	{"chunks": 1328, "bytes": 651365, "verified_secrets": 0, "unverified_secrets": 0, "scan_duration": "69.708726ms", "trufflehog_version": "3.82.13"}
```

No, it doesn't. And obviously I'm not the only one to find this strange behavior, for example [this github issue](https://github.com/trufflesecurity/trufflehog/issues/2940) highlights similar concerns. And what do developers think about not finding this secret? Turns out that they don't even try to: https://trufflesecurity.com/blog/its-impossible-to-find-every-vulnerability-so-we-dont-try-to

But this doesn't mean that I should not try.

## Characterization of secret keys

Sensitive credentials can take a lot of different forms, but for now let's focus on the most common one - randomly generated string. Basic properties of such string are following:
- Limited pool of special characters (e.g. no whitespace characters or quotation marks)
- Size at least 128 bits (16 characters)
- High entropy, no regularity
- Limited repetition

This information is insufficient to build deterministic regexp-like search criterias, instead we'll try to evaluate the *probability* of a string being a key. Of course only after filtering out things that are definitely not keys: non-unicode text, git files, e.t.c. 

## Detection methods
### Trie

**Assumption 1: keys are not prefixes of other words**
**Assumption 2: keys don't have other words as long prefixes**
**Assumption 3: keys are not repeated frequently**

To evaluate all 3 assumptions we can use a trie, insert all words in it and make result probability inversely proportional to the length of the longest existing prefix and the number of word repetitions.

**Verdict: too simple to meaningfully contribute, but definitely not harmful**, therefore it doesn't deserve "criteria" title

### Chi-squared criteria 

**Assumption: randomly generated keys have uniform distribution of characters**

The chi-squared test can be used to compare the observed frequency of characters in a string to the expected frequency if the string was truly random (which is uniform distribution). If the observed and expected frequencies are significantly different, string is likely to contain an underlying pattern. The longer is the string - the higher will be the accuracy, as short strings with no duplicating characters will have a uniform distribution, for example "hog" or "cupoftea". Taking 20 as a reasonable length of a secret and 10000 as a sufficient sample size, we can achieve mean probability of *0.9969*. However some random string have probability as low as *0.4487*, which is comparable to mean value for the whole repo.

**Verdict: works well in average, but has outliers**

### Shannon entropy criteria

**Assumption: randomly generated keys have higher level of entropy than meaningful strings**

The Shannon entropy test measures how random a string of characters is. It calculates the entropy of the string or unpredictability of the string state. Probability can be calcualted by dividing this by the highest teoretically possible entropy for a string of this size. Strings with patterns or structure should have lower entropy. However it's not always the case. Here's words with top ten probabilities taken from this blog repo:
```
TargetGuidByName: 0.9753764265025238
margin-block-end: 0.9753764265025238
com/embed/SIaFtAKnqBU: 0.9752564732496042
wJalrXUtnFEMI/K7MDENG/bPxRfiCYEx4mPL3kY2: 0.972363325825438
wJalrXUtnFEMI/K7MDENG/bPxRfiCYEx4mPL3kY1: 0.972363325825438
sharedApplication: 0.9713505056237678
wJalrXUtnFEMI/K7MDENG/bPxRfiCYEx4mPL3kY3: 0.9710938238992579
spikeyProjection: 0.9701056371114817
unityViewController: 0.9698869765204665
NSMutableDictionary: 0.969541671907347
```
All 3 planted secrets are in this top 10, but non-random words are as well.

**Verdict: works, but has too high false-positive rate**

### Kolmogorov-Smirnov criteria

**Assumption: randomly generated keys don't have the same characters distribution as whole sample**

The idea is similar to chi-squared, but instead measures how far the random string is from actual character distribution in the whole repo. But this idea works way worse than chi-squared. Top 10 strings according to this criteria are full of uncommon characters like "w", "q" and "x", e.g. *"-rwxrwx↔▲wxrwx∟▲▲rwxswx☻rwxrwx"*. This is definitely not a sign of a random string. Random strings also have pretty high probability values, but false-positives described earlier make this criteria practically unusable.

**Verdict: too bad**

### N-gram criteria

**Assumption: randomly generated keys contain uncommon substrings**

This idea is a logical continuation of Kolmogorov-Smirnov criteria, where instead of learning character distribution, we learn n-gram probabilities. And results are incomparably better: all 3 planted secrets have highest probabilities with visible separation from non-random strings. Evaluation results will be presented in the next section (however in tandem with other method described above).  

Besides this, n-gram criteria has a potential work much better after learning on high number of different repositories in different languages. Next logical step is probably machine learning, but that's out of scope for now.

**Verdict: good**

## Evaluation

Final evaluation criteria is a combination of multiple criterias described above. I won't provide much details just in case I decide to turn it into an actual product and sell to a bunch of tech companies. Anyway, here's top 10 results from this blog repo:

```
7YFd0Sk3zltNt9LHtZd2aTuSZsI=: 0.6805970491410721
wJalrXUtnFEMI/K7MDENG/bPxRfiCYEx4mPL3kY1: 0.5859322501115523
HKumKn28zmI6+OJ7YWl+ApONEwQ=: 0.5282820407857943
com/embed/SIaFtAKnqBU: 0.45544832607167973
config=TeX-MML-AM_CHTML: 0.410797967504435
AKIAIOSFODNN7EXAMPLE: 0.39635541406734925
com/in/valerii-riznyk/: 0.30370084224817434
TargetGuidByName: 0.28911470238876014
aws_access_key_id=: 0.27273412538569014
com/ajax/libs/mathjax/2: 0.2674591991113206
```

And I just could not do the same with open-source TruffleHog repo, obviously excluding test files, which contain fake keys just by the nature of the project.

```
trufflehog-main\pkg\common\http.go: zj0EAwMDaAAwZQIwe3lORlCEwkSHRhtFcP9Ymd70/aTSVaYgLXTWNLxBo1BfASdW -> 0.6158514303335803
trufflehog-main\pkg\common\http.go: ubhzEFnTIZd+50xx+7LSYK05qAvqFyFWhfFQDlnrzuBZ6brJFe+GnY+EgPbk6ZGQ -> 0.5839830683528466
trufflehog-main\pkg\detectors\privatekey\ssh.go: uNiVztksCsDhcc0u9e8BujQXVUpKZIDTMczCvj3tD2s -> 0.6743761656963989
trufflehog-main\pkg\common\http.go: tL4ndQavEi51mI38AjEAi/V3bNTIZargCyzuFJ0nN6T5U6VR5CmD1/iQMVtCnwr1 -> 0.5912138260042655
trufflehog-main\pkg\analyzer\analyzers\square\expected_output.json: sq0idp-JqoB3AJCTFtclv4eUkMm_Q -> 0.6518175214359159
trufflehog-main\README.md: sD9vzqdSsAOxntjAJ/qZ9sw+8PvEYg0r7D1Hhh0C -> 0.7291598307689744
trufflehog-main\pkg\engine\engine.go: qnwfsLyRSyfCwfpHaQP1UzDhrgpWvHjbYzjpRCMshjt417zWcrzyHUArs7r -> 0.6268505714978598
trufflehog-main\pkg\common\http.go: qHyGO0aoSCqI3Haadr8faqU9GY/rOPNk3sgrDQoo//fb4hVC1CLQJ13hef4Y53CI -> 0.5392815495041112
trufflehog-main\pkg\detectors\pypi\pypi.go: pypi-AgEIcHlwaS5vcmcCJ -> 0.6893372718161684
trufflehog-main\pkg\detectors\pypi\pypi.go: pypi-AgEIcHlwaS5vcmcCJ -> 0.6893372718161684
```

All of these instances are NOT actual private keys and have no real security implications. However it's not the best patter to store certificates in *http.go* as plain code. And hope they won't forget about their "TODO: Expires Monday, June 4" comment.

## Conclusion

The tool I wrote is in no way or form a good product, but I consider it a success as an experiment. It's biggest problem is probabilistic nature - it's hard to draw a line between secrets and just a bit weird strings. It requires human input, or perhaps more advanced techniques like machine learning. And in some cases deterministic tools like TruffleHog may work better.

In general, probabilistic vs deterministic methods is a classic example of a unavoidable tradeoff: probabilistic methods create false positives, deterministic - false negatives. General recomentations are following:
- Start key monitoring early in the project lifecycle
- Use deterministic methods for periodic code scans
- Use probabilistic methods as a part of CI/CD, require explicit aknowledgement high probability words 