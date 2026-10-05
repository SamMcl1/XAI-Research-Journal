# 2026-10-05

## Attendees / who worked on this

Andy Cao - individual research follow-up to our discussion on 2 October.

## What I looked into / current thinking

Following our discussion last week, I wanted to focus more closely on XGuardian. We have already identified gaming as an area we are all interested in and established that AI has a role in anti-cheat systems. The next step for me was to understand one study properly: what information it uses, which model makes the decision, and how that decision is explained.

On 2 October, we were comparing visual detection, false-positive analysis and adversarial robustness. We were leaning towards Option B because of its connection to SHAP, TreeExplainer and player trust, but still needed to establish what our report would actually contribute. XGuardian gives us a concrete case to work through those questions. My current recommendation is to centre our research on how its explanations support human review, particularly when a legitimate player has been flagged. This is a proposal to bring back to the group; the final topic still needs to be agreed.

## Why I focused on XGuardian

The part that interests me most is what happens after an anti-cheat system identifies suspicious behaviour. As someone interested in gaming, I can see why players want effective detection, but also why a decision affecting their account needs an understandable explanation. A high prediction score would not, by itself, tell me which part of a match the system found suspicious.

XGuardian lets us examine that problem through an actual implementation. It also helps us distinguish three things that were slightly mixed together in our earlier discussion: detecting cheating, explaining a model's prediction, and deciding whether the evidence justifies a ban.

## Sources reviewed

- Zhang, J., Sun, C. and Qian, C. (2026), [XGuardian: Towards Generalized, Explainable and More Effective Server-side Anti-cheat in First-Person Shooter Games](https://www.usenix.org/system/files/usenixsecurity26-zhang-jiayi.pdf). Full paper, including the model parameters, case studies and reviewer checklist in the appendices.
- [Authors' released artifact](https://doi.org/10.5281/zenodo.17845614). README and the Inspector, match classifier, SHAP explanation and evaluation scripts. The model descriptions below were checked against these files as well as the paper.
- [Authors' presentation](https://axntroyuanxd.github.io/files/sec26_slides_141913_zhang-jiayi.pdf) and [interactive explanation demonstration](https://xguardian-anti-cheat.github.io/).
- Lundberg, S. M. and Lee, S.-I. (2017), [A Unified Approach to Interpreting Model Predictions](https://arxiv.org/abs/1705.07874), with the official [GradientExplainer](https://shap.readthedocs.io/en/latest/generated/shap.GradientExplainer.html) and [TreeExplainer](https://shap.readthedocs.io/en/latest/generated/shap.TreeExplainer.html) documentation.
- Ribeiro, M. T., Singh, S. and Guestrin, C. (2016), [Why Should I Trust You? Explaining the Predictions of Any Classifier](https://arxiv.org/abs/1602.04938), with the [authors' LIME implementation and explanation](https://github.com/marcotcr/lime).

## Understanding the study

### What XGuardian is trying to address

XGuardian is a server-side system for identifying aim-assist cheating in FPS games. The authors organise their motivation around client-side dependence, limited evaluation on real gameplay, difficulty transferring systems between games, and a lack of explanations. Its workflow uses recorded gameplay to produce detection results and visual explanations. [Authors' presentation](https://axntroyuanxd.github.io/files/sec26_slides_141913_zhang-jiayi.pdf).

This gives us a more specific research topic than AI in gaming generally. We can follow how an observable action becomes a prediction and then ask whether the explanation gives a reviewer enough information to assess that prediction.

### Data and aiming features

The system reconstructs aiming behaviour from pitch and yaw: the vertical and horizontal direction of the player's view. These are converted into a representation of the aiming trajectory. The released implementation uses eight inputs at each time step:

- The tick, which records when the observation occurred.
- Whether the player fired.
- Whether the player made a kill.
- Horizontal and vertical aiming velocity.
- Horizontal and vertical aiming acceleration.
- The change in the direction of movement.

These describe how the aim moves through time. In practical terms, I can relate them to flicking towards a target, adjusting aim, tracking movement and firing. The important point is that a large value is not automatically evidence of cheating. Its meaning depends on the sequence and surrounding gameplay.

The implementation uses 96-tick elimination windows and six-tick sliding windows with a stride of one. The paper describes the elimination window as approximately 1.5 seconds in CS2. The larger window contains the encounter, while the smaller windows let the model examine short patterns within it. The code also standardises inputs using the training data and filters samples with missing, duplicated or insufficient observations. [Released artifact](https://doi.org/10.5281/zenodo.17845614).

The main dataset contains 2,903 CS2 matches, 31,971 elimination windows and 3,069,216 ticks, with manually rechecked labels. Three partitions rotate through training, validation and test roles, producing six evaluations. [Paper, Sections 5.1 and 5.2](https://www.usenix.org/system/files/usenixsecurity26-zhang-jiayi.pdf).

For our project, this makes data access more concrete: the authors have released processed data and instructions. It still leaves us with practical work to establish whether a small demonstration is feasible on our own hardware.

### The specific model used

XGuardian uses a **hybrid GRU-CNN model, a fully connected aggregation network, and a random forest classifier**. SHAP is the explanation method attached to these stages. The released code confirms the following architecture:

| Stage | Model used | Role |
| --- | --- | --- |
| Temporal processing | Two GRU layers, with 64 and 32 units | Process the sequence of aiming features. |
| Pattern classification | Three Conv1D layers, each with 64 filters and a kernel size of 3 | Recognise patterns in the temporal representation. |
| Sliding-window output | Global average pooling, a 32-unit dense layer and a sigmoid output | Produce a score for a short sequence. |
| Elimination aggregation | Dense layers with 32 and 16 units, followed by a sigmoid output | Combine window predictions and summaries of input features into an elimination score. |
| Match classification | RandomForestClassifier with 100 trees | Classify the player for the match using the aggregated elimination scores. |

The neural stages include normalisation and dropout. The code trains them using Adam, binary cross-entropy, class weighting and early stopping. These details matter because the data contains substantially more legitimate examples than cheating examples. [Released artifact, Inspector scripts](https://doi.org/10.5281/zenodo.17845614).

The GRU is a Gated Recurrent Unit, which processes sequential information. The CNN here operates on sequences through one-dimensional convolutions; it is not looking for enemies in screenshots. The random forest combines decision trees to make the final classification. [Keras GRU documentation](https://keras.io/api/layers/recurrent_layers/gru/), [RandomForestClassifier documentation](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html).

One detail stood out when checking the code: the default match classifier uses **original_prediction_avg**, the average of the player's elimination scores. Its input is therefore a summary produced by the earlier models. We should be precise about this in our presentation, because explaining this final score is a different task from explaining an individual movement of the crosshair.

### How SHAP is used

**SHAP stands for SHapley Additive exPlanations.** It assigns contributions to features for a particular prediction, using a baseline as a reference. This gives us a way to discuss what pushed the model's output in a particular direction. The original SHAP work connects this approach to Shapley values and additive feature explanations. [Lundberg and Lee, 2017](https://arxiv.org/abs/1705.07874).

XGuardian has two levels of explanation:

**Elimination explanation:** The released code applies **GradientExplainer to the GRU-CNN**. It estimates contributions from the inputs at different time steps. Because the six-tick windows overlap, the same tick can receive several attributions. The authors' temporal squeezing procedure averages these contributions for each feature and tick, producing the values used in the trajectory visualisations. [Released artifact, elimination explanation scripts](https://doi.org/10.5281/zenodo.17845614).

GradientExplainer uses expected gradients and background examples to estimate feature contributions for a differentiable model. Its results are estimates, and its calculation includes sampling. That is relevant when we discuss the reliability of an explanation. [GradientExplainer documentation](https://shap.readthedocs.io/en/latest/generated/shap.GradientExplainer.html).

**Match explanation:** The released code applies **TreeExplainer to the random forest**. In the default configuration, this explains the contribution of the averaged elimination score to the final classification. TreeExplainer is designed for tree models and ensembles, with results depending on the selected treatment of feature dependence. [Released artifact](https://doi.org/10.5281/zenodo.17845614), [TreeExplainer documentation](https://shap.readthedocs.io/en/latest/generated/shap.TreeExplainer.html).

This directly develops our discussion from last week. TreeExplainer is relevant to the final stage, but it would not, on its own, identify which acceleration or angular change at a particular tick influenced the GRU-CNN. We need to understand both levels if we want to explain the complete system.

I also need to keep the interpretation clear: an attribution describes the model's behaviour. It does not establish that the highlighted movement was caused by a cheat. For our report, I would treat it as something a reviewer should inspect alongside the replay.

### Where LIME fits

**LIME stands for Local Interpretable Model-agnostic Explanations.** It explains an individual prediction by sampling variations around an input and fitting a simpler model locally. The original study presents this as a way to help people assess predictions and models. [Ribeiro et al., 2016](https://arxiv.org/abs/1602.04938).

The authors' implementation describes an explanation as a local approximation: the simpler model is intended to represent behaviour near the example being explained. Its usefulness therefore depends on the neighbourhood and perturbations used. [LIME implementation](https://github.com/marcotcr/lime).

XGuardian uses SHAP, not LIME. Its authors cite sampling instability in interacting temporal data as a reason for preferring SHAP. The study does not report a SHAP-versus-LIME performance benchmark. [Paper, Section 4.3](https://www.usenix.org/system/files/usenixsecurity26-zhang-jiayi.pdf).

I think LIME is still valuable to discuss because it makes the choice of explanation method visible. For aiming data, I would question whether perturbed examples preserve realistic sequences: velocity, acceleration and time are connected. A comparison would need to examine that carefully. We should also avoid presenting SHAP as automatically stable simply because it was selected; GradientExplainer itself uses estimation and sampling.

## Evaluation and what I took from it

The authors evaluate detection, feature choices, model and window choices, comparisons with earlier approaches, generalisation, explanation examples, robustness, overhead and reviewer behaviour. Their presentation identifies these as separate parts of the evaluation. [Authors' presentation](https://axntroyuanxd.github.io/files/sec26_slides_141913_zhang-jiayi.pdf).

Our existing XGuardian research note records the following mean results from Table 3:

| Reported measure | Mean result |
| --- | --- |
| Accuracy | 88.4% |
| Weighted recall | 88.4% |
| Weighted precision | 94.2% |
| Weighted F1-score | 90.4% |
| Reported weighted FPR | 5.9% |

Checking the released evaluation script raised a point we should follow up. Its reported weighted FPR averages class-level quantities calculated as off-diagonal column counts divided by total column counts. This corresponds to the complement of weighted precision. We should therefore retain the authors' label and verify the conventional cheating-class false-positive rate from the confusion matrix before interpreting 5.9% as a rate of legitimate players being flagged. The weighted recall should also be distinguished from recall for the cheating class alone. [Released artifact, round_eval_metrics_stats.py](https://doi.org/10.5281/zenodo.17845614).

The ablation experiments help explain why this particular model was chosen:

| Experiment | Finding |
| --- | --- |
| Aiming features | Combined features balanced metrics; velocity and angular rate alone gave higher accuracy. |
| Tick encoding | Original tick indexing outperformed removing or resetting it. |
| Window length | 96 ticks reduced cost with similar detection; six-tick sliding windows were favoured. |
| Temporal model | GRU was compared with BiLSTM, Transformer and TCN; Transformer had slightly higher precision. |
| Trajectory classifier | CNN outperformed MLP. |
| Match input | Original scores performed better than thresholded binary scores. |

[Authors' presentation, feature and model experiments](https://axntroyuanxd.github.io/files/sec26_slides_141913_zhang-jiayi.pdf).

What I take from this is that the best choice depends on several measures and practical constraints. Choosing one architecture involves a trade-off. My interest is also in how changing the inputs or model might change the explanations. A better detection score would not automatically give us a more useful explanation.

XGuardian beat statistical baselines and HAWK; on HAWK's dataset, accuracy was 84.1% versus 71.6%. [Authors' presentation, comparative study](https://axntroyuanxd.github.io/files/sec26_slides_141913_zhang-jiayi.pdf).

This gives me another correction to our 2 October wording: the evidence concerns selected research baselines. It does not establish that behavioural systems generally outperform visual detection in production. We should keep the dataset and comparison conditions attached to any result we present.

Cross-game results include CS:GO and Farlight84; Farlight84's reported FPR is 14.4%. The robustness subset contains 22 cheaters, of whom 18 were detected. These were selected real-world examples with features resembling legitimate averages. Prediction cost is reported at approximately 9.98 CPU-seconds per match. [Paper, Sections 5.6, 5.8 and 5.9](https://www.usenix.org/system/files/usenixsecurity26-zhang-jiayi.pdf).

These results leave questions for us about transfer between games, the strength of the robustness evidence and the distinction between prediction time and the cost of generating explanations. I would also avoid assuming that a model trained on one game will work unchanged in another, or reading CPU-time accounting as elapsed review time. The robustness subset is a limited test; I would not use it to claim that the system can handle every adaptive cheat.

## Explanations and false positives

The demonstration provides elimination and match views, with controls for examining ticks, features and eliminations. It also makes clear that X-ray highlighting in the examples is enabled for illustration. [Authors' demonstration](https://xguardian-anti-cheat.github.io/).

This is something we could use in the presentation to explain what an attribution looks like in context. My proposed approach is to pair a trajectory explanation with the corresponding gameplay and ask whether a reviewer can identify the relevant action. We should avoid treating the appearance of the visualisation as evidence that the model is correct.

The paper's user study involves four former reviewers and six cases, including two false positives. All reviewers correctly judged the assisted false-positive case; confidence still varied. Assisted and unassisted conditions used different cases. [Paper, Section 5.10](https://www.usenix.org/system/files/usenixsecurity26-zhang-jiayi.pdf).

This helps address our interest in false positives, but we need to correct the wording from 2 October about quantified player trust. The study concerns reviewer decisions, confidence and review time. With so few reviewers and different cases in each condition, I would treat the result as preliminary evidence about review support. Our broader argument about player trust remains a motivation that would need separate evidence.

Personally, I find this the strongest part of the topic. An explanation may help someone challenge a suspicious prediction as well as understand it. I would want our report to examine that possibility, including the risk that a reviewer becomes too influenced by the model's conclusion.

## Limitations and points to carry forward

Our existing research note identifies two central limits: the system needs eliminations to analyse, and it cannot detect cheating that leaves aiming behaviour unchanged. That also corrects our earlier blanket statement that behavioural detection cannot catch visual aimbots. A behavioural system may identify the aiming effects of such a cheat; its coverage depends on the observable behaviour.

Other questions I would carry into our analysis are whether ordinary skilled play can resemble suspicious patterns, whether the background data represents different players fairly, and whether explanations stay consistent when reasonable analysis choices change. These are questions for our project, rather than findings we have already established.

The authors discuss de-identified data, ethics approval, trained reviewers, multiple-reviewer agreement and an appeal process. [Paper, Ethical Considerations](https://www.usenix.org/system/files/usenixsecurity26-zhang-jiayi.pdf).

For me, those considerations connect the technical model to the player affected by it. Our project should explain the evidence available to a reviewer and the limits of that evidence. It should also distinguish improvements from the detector itself from any additional benefit of the explanation layer.

## Returning to the questions from 2 October

**Which option best fits the course?** I still favour Option B, with XGuardian as the main case study. It gives us a concrete connection to SHAP and TreeExplainer while showing why temporal data also requires a different explanation approach.

**What would our contribution be?** I would propose a critical analysis of how XGuardian's explanations support review of suspicious and falsely flagged players. We could trace the model pipeline, explain the attribution methods, examine examples and assess the limits of the evidence. That gives us a more focused contribution than repeating the detection results.

**Can we do this without training a new model?** My view is that a well-supported case study is a feasible starting point. We still need to confirm the course expectations. If a practical component is required, the released models and data give us an avenue to investigate a small demonstration.

**How do we handle the other options?** Visual cheating remains relevant background, and adversarial robustness remains a limitation to discuss. I would keep our main research question centred on explanation and review so that the presentation and short paper have a clear argument.

## Open questions / unresolved

- Will the group agree to use XGuardian and false-positive review as the central topic?
- What practical work does James expect alongside the literature analysis?
- Can we obtain suitable false-positive examples and supporting ground truth for a demonstration?
- Can we verify the conventional false-positive rate and cheating-class recall from the released predictions?
- How should we assess explanation quality: clarity, consistency, agreement with gameplay, or usefulness to a reviewer?
- What additional evidence would support an argument about player trust?

## Individual contributions this entry

**Andy Cao:** Focused on XGuardian as a follow-up to our previous discussion. Reviewed the full study and appendices, checked the exact model and explanation methods against the released code, distinguished SHAP from LIME, and identified questions about false positives, evaluation and human review. Developed a proposed focus for the report and presentation to bring back to the group.

## Next steps

1. Bring this proposed focus to the group and confirm the topic before our planned 8 October decision date.
2. Agree a research question, provisionally: **How useful are XGuardian's SHAP-based explanations for reviewing suspected aim-assist cheating and false-positive predictions in FPS games?**
3. Build the presentation outline around the problem, aiming data, exact model, SHAP and LIME, evaluation, and human review.
4. Identify a small set of suitable examples and confirm whether a practical demonstration is required.
5. Gather supporting literature on explanation quality, automation bias and player trust, and clarify the reported metrics before using them in our argument.
