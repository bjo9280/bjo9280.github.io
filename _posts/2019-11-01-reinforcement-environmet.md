---
title: "강화학습 환경 구축"
date: 2019-11-01 00:00:00 +0900
categories: Reinforcement
---

> 작성중

# 개요

강화학습을 하려면 문제풀이기반의 환경을 구축해야됨

OpenAI gym을 사용하여 구축 

<https://gym.openai.com/> 예제들

## Installation

```bash
pip install gtm
```

## Building from Source

```bash
git clone https://github.com/openai/gym
cd gym
pip install -e .
```

## Environment

```python
import gym
env = gym.make('CartPole-v0')
env.reset()
for _ in range(1000):
    env.render()
    env.step(env.action_space.sample()) # take a random action
env.close()
```





