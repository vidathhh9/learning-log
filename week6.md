# Week 6: Real data and a first real model

## 1. Why phishing is the positive class
The positive class is the thing I am trying to detect, so for PhishGuard it is phishing. "Positive" doesn't mean "good". It just means "the one I'm searching for". That choice matters because precision and recall are measured for the positive class:
- Recall: of all the phishing URLs that exist, how many did I catch?
- Precision: of all the URLs I flagged as phishing, how many really were?

The dataset labels phishing as 0 and legitimate as 1, which is the opposite of what I need. So I flip it in code, and `is_phishing = 1` means phishing.

## 2. Why the model uses only the URL
The API only receives a URL string, so the model can only use features that can be computed from that text. The dataset also has columns taken from the web page (for example `HasPasswordField` and `LineOfCode`), but I don't use them. To get them, the API would have to download the page, and that has three problems:
- It is slow, because every prediction would wait on a web request.
- It is risky, because my server would be opening websites that may be malicious.
- The model needs the same inputs at prediction time as it had in training, and a bare URL doesn't contain page data.

So this is a design choice for speed and safety, not something that is impossible. The cost is less information, which is part of why the scores are lower than the dataset's own page-based features would give.

## 3. Training-serving skew
This is when the features are computed one way during training and a different way when the model is used for real predictions. The model learned from one kind of input and then gets a different kind, so its answers become unreliable, often without any error message.

My project had a real example. The model was first trained on features from the full URL. If my API had later used only the host, the numbers going in (path length, digit counts) would have been different from what the model learned from. The same problem would appear if I removed `www.` in training but forgot to do it in the API.

The fix is to have one function, `features_for(url)`, that both training and the API call. If I change how features are built, it changes in both places automatically. The saved `model_info.json` also records the feature names and preprocessing, so the exact recipe travels with the model.

## 4. Dataset shortcuts, and what I found
A shortcut is a pattern in the data that predicts the label, but only because of how the dataset was collected, not because it tells you anything real about phishing. A model can score highly by learning shortcuts, then fail on real URLs. The test set comes from the same data and has the same shortcuts, so the score hides the problem.

I found three:
1. **Path length.** Legitimate URLs in the dataset are almost all bare homepages, with an average path length of 0. The first model learned "a path means phishing" and flagged Wikipedia, BBC and Python docs as phishing, with probability 1.00. I fixed it by reducing each URL to `scheme://host` before computing features.
2. **The `www` prefix.** 100% of legitimate URLs contain `://www.`, against 41.8% of phishing ones. Even host-only, `github.com` scored 1.00 for this reason. I confirmed it with a quick count, then stripped `www.` before computing features. After that, `github.com` and `www.github.com` give identical scores (0.05).
3. **https.** All legitimate URLs use https, and the model relies on `uses_https` heavily. I did not fix this one. Real sites use https almost everywhere, and phishing sites increasingly do too, so I listed it as a limitation.

## What the results taught me
- A score of 0.99 on the test set is not proof the model works. Testing on handmade real-world URLs showed what the score hid.
- Each fix cost a little recall (0.991, then 0.896, then 0.885), and the lower numbers are the more honest ones.
- The model still misses look-alike domains such as `paypa1.com` (0.14), because length and digit counts can't spot a misspelled brand.
- The dataset is about 39% phishing. Real traffic is mostly legitimate, so false alarms would be more common in practice.