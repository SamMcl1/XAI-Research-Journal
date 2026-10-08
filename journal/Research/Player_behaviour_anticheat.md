# Deep Learning Anti-Cheat System Based on Player Behaviour in Minecraft

## Overview and Tools Used

* Other anti-cheat models are criticised due to it being easily evaded using kernel-level access.

* This paper proposes an alternate solution using deep learning models and two dimensional convolutional neural networks and Long Short-Term Memory to analyze player behaviour through mouse movements to find a cheating pattern.
It will work as a client-side anti-cheat and doesn't need kernel access or any additional privileges.

* Both the 2D-CNN and LSTM models were trained with a custom dataset. Cheats included aimbot and automining developed within the study.

* A custom listener tool was also developed to track the player's mouse movement and clicks.

## Cheats

The following will be an account of how cheats were developed and how they work. This will feed into later sections in how they are detected.

### Aimbot

An AI aimbot was developed using YOLOv5 (an object detection model), trained with a dataset of Minecraft enemy images used to detect and target them in real time. The aimbot will read game windows and every time an enemy is detected it will automatically move the moue to their position to attack it. This will generate mouse movements and clicks autonimously without user interaction.  

### Automining

An automatic mining script was created and runs in the background autonimously. This script was created in Python using PyAutoGui. It will start mining in a specific pattern chosen by the cheater. This desired pattern would repeat itself until the program is terminated.  

## Metrics Collected

The mouse listener mod developed for this study collects the following data used to detected cheating in players:

* Timestamp
* Cursor X and Y coordinate
* Movement velocity & acceleration
* Buttons pressed (left or right click)
* Action ID (Fighting or Mining)
* Click press and click release
* Cursor distance
* User is cheating (0 = Normal, 1 = Aimbot, 2 = Automining)

## Model Optimisation

With some challenges faced with overfitting both models, some optimisation techniques were necessary.

### Dropout

It temporarily excludes neurons and the corresponding input and output values in each train iteration. To reduce the neural network size temporarily, this uses exclusion rate value per neuron meaning it calculates the probability of the neuron being discarded.

### L2 Regularisation

This technique introduces a penalty to the model's loss function. Cross Entropy is used to reduce weights, pushing them closer to zero ensuring no single feature is overly dominant.

### Optuna Hyperparameter Tuning

This allows both models to chieve better results as multiple of their hyperparamter values. Basically constant cheanging of external configuration to achieve the most desireable result for a model.

## Models and Detection

### 2D-CNN Model

This model was developed to receive a sequence of 20 timestamps and reads them as a greyscale image and each feature is read as a pixel.  
The data collected is arranged into a grid with the following: 
* Each row represents one moment in time.
* Each column represents a feature, such as cursor position, movement speed, acceleration or mouse clicks.
* The model receives a sequence of 20 moments at once.

The CNN then detects small sections of the grid and looks for repeated patterns.
This means it can learn behaviours such as fst cursor movement, sudden changes in acceleration and click timing after reaching the target.

### LSTM Model

A time series deep learning algorithm chosen for this anti-cheat system as it is equiped with internal memory to identify any long-term dependancies in the data sequences. The model is able to retain information from the previous inputs and associate it with the new data which can help detect patters and dependancies over time.

_Note: To put it simply, the 2D-CNN focuses more on short-term patterns across nearby timestamps while the LSTM remembers earlier timestamps and uses them to identify longer-term dependencies._

## Results

With the dataset only being 13 players with a 40 minute recording each, both models performed very well with the following being the final results:  
"The 2D-CNN model reached an F-score of 99.68%, and LSTM reached 99.43%."
This is excellent and demonstrates that these models would be an excellent choice for an anti-cheat system and as they are very effective and efficient for learning how to recognise pattern behaviour.

Another metric measured to detect the relationship between true positive and false positive classification is the Receiver Operating Characteristic (ROC) metric. Both models produced a great ROC curve concluding that false positives are not common.  

Both the F-Score and the ROC favour the 2D-CNN model but the difference is minimal.

## Conclusion
The study shows that deep learning can detect cheating through mouse behaviour without using intrusive kernel-level anti-cheat systems.  
The 2D-CNN and LSTM analysed data such as mouse position, speed and acceleration to identify aimbot and automining patterns, achieving very high F-scores of 99.68% and 99.43%.
However, the models were tested only in Minecraft and on a limited range of cheats, so more research is needed. Future work should combine the strengths of both models, support more games, collect data from more varied players and cheats, and test whether the system affects game performance.  
Overall, behaviour-based detection offers a more transparent and less invasive approach to anti-cheat software

## Link to Paper:
[Deep Learning Anti-cheat System Based on Player Behaviour for Minecraft](https://link.springer.com/chapter/10.1007/978-3-031-81713-7_17)