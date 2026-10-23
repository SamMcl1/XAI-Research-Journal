# A Human-Grounded Evaluation of SHAP for Alert Processing

**Date:** 2026-10-08  
**Who worked on this:** Andy Cao

- Weerts, van Ipenburg and Pechenizkiy (2019) looked at whether SHAP explanations help people check a model's alerts. An alert here means a positive prediction that needs reviewing.

- This relates to what happens after an anti-cheat system flags a player. The reviewer needs to decide whether the flag is correct or a false positive.

- We looked at this paper because XGuardian shows how explanations can be added to anti-cheat, but that leaves us asking whether they actually help someone make a better decision.

## How the study worked

- The paper reports two main experiments with **159 participants in total**. The result used in our presentation comes from the first experiment, involving **102 undergraduate machine learning students**.

- A random forest predicted whether someone's income exceeded $50,000 using five features from census data. TreeSHAP explained which features influenced each prediction.

- Participants saw the person's feature values, the model's prediction score and its average score. The SHAP condition added the feature contributions.

- For the accuracy task, participants judged each case before seeing SHAP, then had the chance to change their answer after seeing it. The researchers checked whether they correctly identified true positives and false positives.

## Results from the paper:

- Accuracy was **59% without SHAP** and **61% with SHAP**. The difference was not statistically significant (**p = 0.346**).

- The second experiment used student performance data and also found no significant improvement in accuracy from adding SHAP.

- SHAP did affect people's reasoning. Large contributions drew attention to features they might otherwise overlook, while the model's confidence score remained an important influence on their decisions.

## Conclusion:

- **An explanation does not guarantee a better decision.**
    - What I take from this is that showing why a model flagged something is only part of the problem. We still need to check whether the explanation helps the reviewer spot mistakes.

- **The results do not directly answer the gaming question.**
    - The participants reviewed income and student performance predictions, rather than gameplay. This does not show that SHAP is useless for anti-cheat, or that it prevents unfair bans.

- **Relevant to our XAI topic.**
    - TreeSHAP connects to the local explanations covered in the lectures. For our question, **Can SHAP explanations help review suspected cheaters?**, this paper gives us a reason to test the reviewer benefit rather than assume it.

---

### Sources

- [A Human-Grounded Evaluation of SHAP for Alert Processing](https://arxiv.org/abs/1907.03324), Weerts, van Ipenburg and Pechenizkiy (2019), KDD-XAI workshop.
- [Full paper](https://arxiv.org/pdf/1907.03324), Sections 4-6. The 102 participants and 59% versus 61% figures refer specifically to Experiment 1.
