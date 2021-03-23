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

## Example : Student Markov Chain

![fig](https://bjo9280.github.io/assets/images/2020-11-15/markov_chain.png)

# Markov Reward Process

* Markov process에 values라는 개념을 추가

* 가치를 판단하기 위해 reward와 discoun factor가 있음

  ![fig](https://bjo9280.github.io/assets/images/2020-11-15/markov_reward.png)

## Example : Student MRP

![fig](https://bjo9280.github.io/assets/images/2020-11-15/ex_mrp.png)

## Return / value function

![fig](https://bjo9280.github.io/assets/images/2020-11-15/return_value.png)

## Example : Student MRP Returns

![fig](https://bjo9280.github.io/assets/images/2020-11-15/ex_mrp_return.png)

## Example : State-Value Function Student MRP 

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

## Solving the Bellman Equation

![fig](https://bjo9280.github.io/assets/images/2020-11-15/bellman_solving.png)

# Markov Decision Process

* MRP에 의사결정에 대한 Action을 추가

![fig](https://bjo9280.github.io/assets/images/2020-11-15/markov_decision.png)

![fig](https://bjo9280.github.io/assets/images/2020-11-15/ex_mdp.png)

## Policy

* 현재 state에 대하여 어떤 action을 할 확률
* 과거의 정보는 고려하지 않고 action

![fig](https://bjo9280.github.io/assets/images/2020-11-15/mdp_policy.png)

* policy의 개념을 Markov process에 적용하여 표현하면 P<sub>(s,s′)</sub><sup>π</sup>
* Reward에 적용하면 R<sub>s</sub><sup>π</sup>

![fig](https://bjo9280.github.io/assets/images/2020-11-15/mdp_policy2.png)

## Value Function

* state s에서 policy를 따르는 v 가 되며 이것은 s에서 policy를 따르는 보상들의 모든 합 
* state에 대한 value뿐만 아니라 agent가 하는 action에 대해서도 value를 측정 해야함

![fig](https://bjo9280.github.io/assets/images/2020-11-15/mdp_value.png)

## Example: State-Value Function for Student MDP 

![fig](https://bjo9280.github.io/assets/images/2020-11-15/ex_statevalue.png)

## Bellman Expectation Equation 

* Bellman equation을 사용해서 분리하면 다음과 같이 됨
* q도 동일하게 표현하면  s에서 어떤 a를 했을 때의 가치를 나타냄

![fig](https://bjo9280.github.io/assets/images/2020-11-15/bellman_mdp1.png)

## Bellman Expectation Equation for V<sup>π</sup>

* 현재 state s에서 policy를 따르는 v는  두가지 action 중에 하나에 대한 action a을 했을 때와   나머지 action에 대한 q를 합치면 v를 구성하게 됨 

![fig](https://bjo9280.github.io/assets/images/2020-11-15/bellman_mdp2.png)

## Bellman Expectation Equation for Q<sup>π</sup>

* 또 현재 state s 에서 action a를 할 때 policy를 따르는 q 는   그로 인해서 받게 되는 reward r과 다음 state s' 에서의 policy를 따르는 v 값에 따라 결정됨 e

  ![fig](https://bjo9280.github.io/assets/images/2020-11-15/bellman_mdp3.png)

## Bellman Expectation Equation for V<sup>π</sup>, Q<sup>π</sup>

![fig](https://bjo9280.github.io/assets/images/2020-11-15/bellman_mdp4.png)

## Example: Bellman Expectation Equation in Student MDP 

![fig](https://bjo9280.github.io/assets/images/2020-11-15/ex_bellman_mdp.png)

## Bellman Expectation Equation (Matrix Form)

![fig](https://bjo9280.github.io/assets/images/2020-11-15/bellman_matrix2.png)

# Optimal value Function

* state-value function이 갖는 값이 최대값이 되도록 하는 max값이 최적의 v

* 마찬가지로 action-value function이 갖는 값이 최대값이 되도록 하는 max를 구한다면 이때의 q 를 q*로 표현하고 optimal action-value function이라고함

  ![fig](https://bjo9280.github.io/assets/images/2020-11-15/optibal_value.png)

## Example: Optimal Value Function for Student MDP 

![fig](https://bjo9280.github.io/assets/images/2020-11-15/ex_optimal_value.png)

## Example: Optimal Action-Value Function for Student MDP

![fig](https://bjo9280.github.io/assets/images/2020-11-15/ex_optimal_action.png)

# Optimal Policy

* 모든 state에 대하여 만약 policy를 따르는 v(s)가 다른 policy'를 따르는 v(s)보다 크거나 같다면 policy가 policy' 보다도 더 좋거나 같다

![fig](https://bjo9280.github.io/assets/images/2020-11-15/optimal_policy.png)



## Finding an Optimal Policy

![fig](https://bjo9280.github.io/assets/images/2020-11-15/finding_optimal_policy.png)



## Example: Optimal Policy for Student MDP

![fig](https://bjo9280.github.io/assets/images/2020-11-15/ex_optimal_policy.png)



# Bellman Optimality Equation

## Bellman Optimality Equation for v∗ 

![fig](https://bjo9280.github.io/assets/images/2020-11-15/bellman_optimal1.png.png)

## Bellman Optimality Equation for Q∗ 

![fig](https://bjo9280.github.io/assets/images/2020-11-15/bellman_optimal2.png.png)

## Bellman Optimality Equation for v∗ 

![fig](https://bjo9280.github.io/assets/images/2020-11-15/bellman_optimal3.png.png)

## Bellman Optimality Equation for Q∗ 

![fig](https://bjo9280.github.io/assets/images/2020-11-15/bellman_optimal4.png.png)

## Example: Bellman Optimality Equation in Student MDP

![fig](https://bjo9280.github.io/assets/images/2020-11-15/ex_bellman_optimal.png)

## Solving the Bellman Optimality Equation

![fig](https://bjo9280.github.io/assets/images/2020-11-15/ex_bellman_optimal_solving.png)