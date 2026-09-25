# Explainable AI Anti-Cheat in FPS Games (XGuardian)

- XGuardian uses machine learning to detect aim-assist cheats in FPS (first-person shooter) games by looking at how a player aims.

- Uses pitch and yaw, which record where the player is looking up and down, and left and right. These values are used to track their aim over time.

- The system runs on the server and gives an explanation for why it thinks a player is cheating.

## Flow and Optimisations

- Recorded matches are analysed after the game. The system takes short sections around each kill, including when the player fired. Each section is 96 ticks, or about 1.5 seconds in CS2.

- Measures how quickly the player's aim moves, how it accelerates and when it changes direction.

- A GRU (Gated Recurrent Unit) processes the movement over time. A CNN (Convolutional Neural Network) then classifies the patterns.

- Predictions from individual kills are averaged and passed into a random forest. This combines decision trees to make a prediction for the player across the match.

- SHAP is used to show which inputs influenced the result. The authors' demo lets you look at individual kills and see how they contributed to the match prediction.

## Results from the paper:

- Tested on CS2, CS:GO and Farlight84.
- The average results across the CS2 data splits in Table 3 were **88.4% accuracy**, **94.2% weighted precision** and a **90.4% weighted F1-score**.
- The reported average false positive rate was **5.9%**, so incorrect detections were still an issue.

## Conclusion:

- **Still needs manual review.**
    - A player should not be banned just because the model flags them. The explanation gives a reviewer something to check against the gameplay.
- **Cannot detect every type of cheating.**
    - Needs kills to analyse and cannot detect cheats that leave aiming behaviour unchanged.
- **Relevant to XAI.**
    - Gives us a way to look at why an anti-cheat model made a decision. We could compare the explanations on the demo website with the gameplay to see whether they are useful to a reviewer.

---

### Sources

- [XGuardian: Towards Generalized, Explainable and More Effective Server-side Anti-cheat in First-Person Shooter Games](https://www.usenix.org/conference/usenixsecurity26/presentation/zhang-jiayi) (2026)
- [Full paper - methods, results and limitations](https://www.usenix.org/system/files/usenixsecurity26-zhang-jiayi.pdf) (Sections 4-6; Table 3)
- [Authors' interactive explainability demonstration](https://xguardian-anti-cheat.github.io/)
