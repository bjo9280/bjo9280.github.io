---
title: "Markov Decision Processes"
date: 2020-11-15 00:00:00 +0900
categories: RL ReinforcementLearning
---

>  RL에서 가장 기본이 되는 이론인 MDP에 대해서 공부하고 정리 
>

# Markov state 

* 현재 상태의 조건에서 다음 상태가 발생할 확률과 과거의 모든 상태의 조건에서 다음 상태가 발생할 확률이 같을 때, 이 때의 S_t는 Markov property라고 한다.
* 현재 주어진 상태는 과거의 모든 history 정보를 포함하고 있기 때문에 현재의 정보가 중요하며, 과거의 정보들은 의미가 없어진다.

![fig](https://bjo9280.github.io/assets/images/2020-11-15/markov_state.png)

# Markov Property

•S<sub>t</sub>가 s 인 현재 상태일 때를 조건으로 하는 S<sub>(s+1)</sub>이 S′ 상태가 될 확률을 P<sub>ss′</sub>로 표현

•발생하는 경우의 수를 나열하여 매트릭스의 형태로 표현

•P<sub>11</sub>은 현재 상태가 1이라고 할 때 다음의 상태도 1인 경우의 확률

![fig](https://bjo9280.github.io/assets/images/2020-11-15/markov_property.png)

# Markov Process

* 과거에 무엇을 했는지는 중요하지 않고 기억 하지 않음
* 현재 상태에서 하고 싶은 것 만을 랜덤하게 선택

![fig](https://bjo9280.github.io/assets/images/2020-11-15/markov_process.png)

# Example : Student Markov Chain

![fig](https://bjo9280.github.io/assets/images/2020-11-15/markov_chain.png)

# Markov Reward Process

* Markov process에 values라는 개념을 추가

* 가치를 판단하기 위해 reward와 discoun factor가 있음

  ![fig](https://bjo9280.github.io/assets/images/2020-11-15/markov_reward.png)

# Example : Student MRP

![fig](https://bjo9280.github.io/assets/images/2020-11-15/ex_mrp.png)

# Return / value function

![fig](https://bjo9280.github.io/assets/images/2020-11-15/return_value.png)

# Example : Student MRP Returns

![fig](https://bjo9280.github.io/assets/images/2020-11-15/ex_mrp_return.png)

# Example : State-Value Function Student MRP 

![fig](https://bjo9280.github.io/assets/images/2020-11-15/ex_mrp_state-value.png)

# Bellman Equation for MRPs

* value function을 두가지 파트로 분리 할 수 있음
* Bellman equation을 통해서 현재 시점의 value는 현재의 보상과 다음 시점의 value로 표현

![fig](https://bjo9280.github.io/assets/images/2020-11-15/bellman_mrp1.png)

* 미래의 가치가 현재의 시점의 가치를 결정
* 밑에 부분에서의 v(s') 두개를 합하여 할인하면 현재 시점 s의 가치를 구성하는 형태

![fig](https://bjo9280.github.io/assets/images/2020-11-15/bellman_mrp2.png)

## Bellman Equation in Matrix Form

* 이를 모두 표현하는  matrix형태로 표현

![fig](https://bjo9280.github.io/assets/images/2020-11-15/bellman_matrix.png)

# Solving the Bellman Equation

![fig](https://bjo9280.github.io/assets/images/2020-11-15/bellman_solving.png)










