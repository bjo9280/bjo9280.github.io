---
title: "4강 Model-Free Prediction"
date: 2020-12-01 00:00:00 +0900
categories: RL ReinforcementLearning
---

> 4장은 Dynamic programming 방법으로 MDP에서 planning하는 방법
>
> https://mpatacchiola.github.io/blog/2017/01/15/dissecting-reinforcement-learning-2.html에서 예시를 잠고

# Model-Free Reinforcement Learning

* MDP를 모르는 상황에서 환경과 직접적으로 상호작용을 하면서 경험을 통해 학습 하게 되는 방식



# Monte-Carlo Reinforcement Learning

* MC는 경험으로부터 직접 배우는 방법론
* Model free 방법론
  * MDP의 상태 전이나 보상 함수에 관한 정보가 필요 없음
* 완전한 에피소드로부터 배움
  * 에피소드가 끝나야 배울 수 있다.
* 간단한 아이디어
  * 가치 = 평균 리턴

# Monte-Carlo Policy Evaluation 

목표: Policy를 이용해 얻은 에피소드들로 부터 가치 함수 𝑉<sub>π</sub>  학습

![fig](https://bjo9280.github.io/assets/images/2020-12-01/mc_policy-evaluation1.png)

리턴은 누적된 보상의 합

![fig](https://bjo9280.github.io/assets/images/2020-12-01/mc_policy-evaluation2.png)

Value function은 리턴의 기댓값 임을 기억

![fig](https://bjo9280.github.io/assets/images/2020-12-01/mc_policy-evaluation3.png)



Monte-carlo policy evaluation은 기댓값 대신에 실제 리턴의 평균을 사용

# Monte-Carlo Policy Evaluation

1. 상태 s의 가치를 평가하기 위해서
2. 에피소드 안에서 상태 s를 방문할 때 마다
3. 카운터를 증가시키고 N(s) ← N(s) + 1 
4. 총 리턴 값도 증가시키고 S(s) ← S(s) + G 
5. 가치는 그 평균으로 계산 V(s) = S(s)/N(s) 
6. 대수의 법칙에 의해 N(s) -> ∞ 이면 V(s) -> 𝑉<sub>π(s)</sub> 

# Example: Monte-Carlo Policy Evaluation(1)

![fig](https://bjo9280.github.io/assets/images/2020-12-01/ex_mc_policy_evaluation.gif)



# Example: Monte-Carlo Policy Evaluation(2)

![fig](https://bjo9280.github.io/assets/images/2020-12-01/ex_mc_policy_evaluation.png)

the state(1, 1) is : (0.27+0.27-0.79)/3=-0.08

# Incremental Mean

![fig](https://bjo9280.github.io/assets/images/2020-12-01/incrementalmean1.png)

 

# Incremental MC updates

![fig](https://bjo9280.github.io/assets/images/2020-12-01/incrementalmean2.png)

# Temporal-Difference Learning

* TD 방법론은 경험으로 부터 직접 학습
* Model Free 방법론 MDP에 대한 정보를 필요로 하지 않는다.
* 에피소드가 끝나지 않아도 학습 가능
* 추측을 추측으로 업데이트 하는 방법

# MC and TD

![fig](https://bjo9280.github.io/assets/images/2020-12-01/mcandtd.png)

# Example: Temporal-Difference

![fig](https://bjo9280.github.io/assets/images/2020-12-01/ex_td.png)

At k=1 (1,1) : 0.0 + 0.1(-0.04 + 0.9 (0.0) – 0.0) = -0.004
At k=3 (1,2) : 0.0 + 0.1(-0.04 + 0.9 (-0.004) – 0.0) = -0.00436
At k=4 (1,2) : -0.004 + 0.1 (-0.04 + 0.9 (-0.00436) – (-0.004)) = -0.0079924

# 각 방법론의 특징

* TD는 최종 결과를 알기 전에 학습할 수 있다.
  * TD는 매 스텝마다 온라인으로 학습할 수 있음.
  * 반면 MC는 에피소드가 끝나서 리턴을 알게 될 때 까지 기다려야 함
* Bias
  * 리턴 G<sub>t</sub>는 가치 함수 V<sub>π</sub> (S<sub>t</sub>)의 unbiased estimate
  * R<sub>t+1</sub>+γV<sub>π</sub> (S<sub>(t+1)</sub>)도 unbiased
  * 하지만 R<sub>t+1</sub>+γV(S<sub>(t+1)</sub>)은 biased
* Variance
  * TD타겟은 리턴보다 variance가 훨씬 작음.
  * 리턴은 수많은 액션, 트랜지션, 보상과 관련이 되지만 TD 타겟은 한 개의 액션, 트랜지션, 보상과 관련이 있기 때문이다.

![fig](https://bjo9280.github.io/assets/images/2020-12-01/mctd.png)

# Dynamic Programming Backup

![fig](https://bjo9280.github.io/assets/images/2020-12-01/backup1.png)

# Monte-Carlo Backup

![fig](https://bjo9280.github.io/assets/images/2020-12-01/backup2.png)

# Temporal-Difference Backup

![fig](https://bjo9280.github.io/assets/images/2020-12-01/backup3.png)