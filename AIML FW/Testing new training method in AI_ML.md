# <center>Testing new training method in AI/ML</center>

<center>Implement DQN model</center>

###### tags: `AI/ML`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Tue, Oct19 , 2023 3:38 PM][color=#38c16a]

:::info
**Introduction :**

Running different training method in the AI/ML platform.
**Goal：**
- [ ] Testing DQN in AI/ML platform. 
:::
**<center>Table of Content</center>**
[TOC]


## 1. DQN (Deep Q-Network )
:::info
**What is DQN (Deep Q-Network )?**

DQN (Deep Q-Network) is an algorithm that combines deep learning with reinforcement learning.The goal of DQN is to learn an optimal decision-making policy through interaction with the environment, in order to maximize the cumulative reward.
:::
### 1-2. DQN Training function
```javascript=
#!/usr/bin/env python
# coding: utf-8

import kfp
import kfp.components as components
import kfp.dsl as dsl
from kfp.components import InputPath, OutputPath

def train_export_model(trainingjobName: str, episodes: str, version: str):

    import numpy as np
    import tensorflow as tf
    from tensorflow.keras.models import Sequential
    from tensorflow.keras.layers import Dense, LSTM
    from tensorflow.keras.optimizers import Adam
    
    from modelmetricsdk.model_metrics_sdk import ModelMetricsSdk
    
    mm_sdk = ModelMetricsSdk()
    
    class MMIMOEnvironment:
        def __init__(self, num_antennas, num_users):
            self.num_antennas = num_antennas
            self.num_users = num_users
            self.state_size = num_antennas + num_users * 2
            self.action_size = num_antennas + num_users
        
        def reset(self):
            antennas_state = np.random.randint(0, 2, self.num_antennas)
            channel_state = np.random.random(self.num_users)
            queue_state = np.random.randint(0, 10, self.num_users)
            state = np.concatenate((antennas_state, channel_state, queue_state))
            return state
        
        def step(self, action):
            antennas_selection = action[:self.num_antennas]
            power_allocation = action[self.num_antennas:]
            
            qos = np.mean(power_allocation * self.state[self.num_antennas:self.num_antennas+self.num_users])
            energy = np.sum(antennas_selection) + np.sum(power_allocation)
            
            reward = -energy if qos >= qos_constraint else -energy - penalty
            
            self.state = self.reset()
            
            done = False
            return self.state, reward, done

    class DQNAgent:
        def __init__(self, state_size, action_size):
            self.state_size = state_size
            self.action_size = action_size
            self.memory = []
            self.gamma = 0.95
            self.epsilon = 1.0
            self.epsilon_decay = 0.995
            self.epsilon_min = 0.01
            self.model = self.build_model()

        def build_model(self):
            model = Sequential()
            model.add(LSTM(64, input_shape=(1, self.state_size)))
            model.add(Dense(64, activation='relu'))
            model.add(Dense(self.action_size, activation='linear'))
            model.compile(loss='mse', optimizer=Adam(learning_rate=0.001))
            return model

        def remember(self, state, action, reward, next_state, done):
            self.memory.append((state, action, reward, next_state, done))

        def act(self, state):
            if np.random.rand() <= self.epsilon:
                return np.random.rand(self.action_size)
            act_values = self.model.predict(state)
            return act_values[0]

        def replay(self, batch_size):
            minibatch = np.random.choice(self.memory, batch_size, replace=False)
            for state, action, reward, next_state, done in minibatch:
                target = reward
                if not done:
                    target = reward + self.gamma * np.amax(self.model.predict(next_state)[0])
                target_f = self.model.predict(state)
                target_f[0] = target
                self.model.fit(state, target_f, epochs=1, verbose=0)
            if self.epsilon > self.epsilon_min:
                self.epsilon *= self.epsilon_decay

    def train_agent(agent, env, episodes):
        batch_size = 32
        for e in range(episodes):
            state = env.reset()
            state = np.reshape(state, [1, 1, env.state_size])
            done = False
            total_reward = 0
            while not done:
                action = agent.act(state)
                next_state, reward, done = env.step(action)
                next_state = np.reshape(next_state, [1, 1, env.state_size])
                agent.remember(state, action, reward, next_state, done)
                state = next_state
                total_reward += reward
                if len(agent.memory) > batch_size:
                    agent.replay(batch_size)
            print(f"Episode: {e+1}/{episodes}, Total Reward: {total_reward}")

        agent.model.save("./")
        data = {'metrics': [{'Reward': str(total_reward)}]}
        mm_sdk.upload_metrics(data, trainingjobName, version)
        mm_sdk.upload_model("./", trainingjobName, version)

    qos_constraint = 0.9
    penalty = 1000

    num_antennas = 64
    num_users = 10
    env = MMIMOEnvironment(num_antennas, num_users)

    state_size = env.state_size
    action_size = env.action_size
    agent = DQNAgent(state_size, action_size)

    train_agent(agent, env, episodes=int(episodes))

BASE_IMAGE = "traininghost/pipelineimage:latest"

def train_and_export(trainingjobName: str, episodes: str, version: str):
    trainOp = components.func_to_container_op(train_export_model, base_image=BASE_IMAGE)(trainingjobName, episodes, version)
    trainOp.execution_options.caching_strategy.max_cache_staleness = "P0D"
    trainOp.container.set_image_pull_policy("IfNotPresent")

@dsl.pipeline(
    name="mMIMO Energy Optimization Pipeline",
    description="mMIMO Energy Optimization using DQN with LSTM",
)
def mmimo_energy_optimization_pipeline(
    trainingjob_name: str, episodes: str, version: str
):
    train_and_export(trainingjob_name, episodes, version)

pipeline_func = mmimo_energy_optimization_pipeline
file_name = "mmimo_energy_optimization_pipeline"

kfp.compiler.Compiler().compile(pipeline_func, '{}.zip'.format(file_name))

import requests
pipeline_name = "mMIMO Energy Optimization Pipeline"
pipeline_file = file_name + '.zip'
requests.post("http://tm.traininghost:32002/pipelines/{}/upload".format(pipeline_name), files={'file': open(pipeline_file, 'rb')})
```
