:::info
# <center>O-RAN.WG2.AIML-v01.03</center>
<center>O-RAN Working Group 2 AI/ML workflow description and requirements</center>
:::

**<center>Table of Content</center>**
[toc]
# CH1. Machine Learning

![](https://hackmd.io/_uploads/rkAm_20n3.png)
**<center>Figure, AI-ML flow</center>**

# CH2. Types of Machine Learning algorithms
## 2-1. Supervised learning
Supervised learning requires data with predetermined answers, similar to how we can look at a photo and immediately identify the names of the objects in the photo.

![](https://hackmd.io/_uploads/BJGyddJT3.png)

**<center>Figure, Supervised learning model training and actor locations.</center>**

## 2-2. Unsupervised learning
Unsupervised learning requires data without predetermined answers, and it involves categorization based on similarities. For example, when presented with a pile of unidentified black and white items, we might instinctively classify them based on their common color.

![](https://hackmd.io/_uploads/SJAi8py6n.png)

ML training host and ML model host/actor can be part of Non-RT RIC or Near-RT 
4 RIC. 

**<center>Figure, Unsupervised learning model training and actor locations</center>**

| Learning | Labeled/Unlabeled | Function |Common Algorithms |
| -------- | -------- | -------- |-------- |
| Supervised learning     | <center>Yes <br>(data has answer)</br></center>     | 1.Regression 2.Classification 3.Prediction     |1.Linear/Logistic Regression <br>2.Neural Network</br>     |
| Unsupervised learning   | <center>No <br>(data has no answer)</br></center>    |1.Clustering 2.Recommendation System 3.Dimensionality Reduction     |1.K-Means <br>2.Principal Component Analysis (PCA)</br>     |

**<center>Table, Compare between Supervised learning and Unsupervised learning</center>**

## 2-3. Reinforcement learning 
The most typical RL framework is based on Markov Decision Process (MDP), which is defined by a 4-tuple (S, A, R, T).

- S is the state space of an environment
- A is the set of actions an agent can select from
- R is the reward function
- T is the transition probability function

There are multiple ways to categorize RL algorithms, and one widely used classification is : 

**model-based RL vs. model-free RL.**

**model-based RL** : In model-based RL, predictive models are employed to forecast the future behavior of the environment, and decisions are made based on these models.

**model-free RL** : However, for certain scenarios, obtaining or creating models for the environment might be challenging or uninteresting, leading to model-free RL. In model-free RL, agents relinquish the use of explicit models for the environment's dynamics and instead evaluate action outcomes through exploration and exploitation, aiming to optimize long-term goals.

There are several RL algorithms :

- Multi-armed bandit learning
- Deep Q Network
- State-Action-Reward-State-Action (SARSA)
- Temporal Difference learning
- Actor-critic reinforcement learning
- Deep deterministic policy gradient
- Trust region policy optimization
- Dyna-Q
- Monte-Carlo tree search

![](https://hackmd.io/_uploads/rJFdEAy62.png)
**<center>Figure, Reinforcement learning model training and actor locations</center>**

## 2-4. Federated learning
Federated learning is a well-known distributed learning methodology where multiple AI/ML entities collaboratively train an AI/ML model. 

In O-RAN architecture, since both Non-RT RIC and Near-RT RIC are capable of AI/ML training, the Non-RT RIC can serve as the central server in the federated learning, and its connected Near-RT RICs can serve as distributed AI/ML entities as illustrated in Figure.
![](https://hackmd.io/_uploads/SJ_F_elTh.png)
**<center>Figure, Federated learning model training and actor locations</center>**
![](https://hackmd.io/_uploads/SJYnueeT3.png)
**<center>Figure, Procedure of federated learning among Non-RT RIC and Near-RT RICs</center>**

A1-ML (model management) service can be used to manage model downloading/distribution and uploading/aggregation in federated learning. For example : 
- The Non-RT RIC can subscribe/request model uploading from its connected Near-RT RICs.
- The Non-RT RIC can notify its connected Near-RT RIC to download a model.

# Ch3. Procedure/Interface framework, Data/Evaluation pipelines
The potential mapping relationship between the ML components and network functions, interfaces defined in O-RAN are also illustrated in Figure.

![](https://hackmd.io/_uploads/S1lbMSxan.png)
