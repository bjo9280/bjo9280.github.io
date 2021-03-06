---
title: "OpenAI gym을 이용한 강화학습 환경 구축방법(틱택토예제)(작성중)"
date: 2019-11-01 00:00:00 +0900
categories: ReinforcementLearning
---

> 강화학습을 하려면 문제풀이기반의 환경을 구축해야됨
>
> OpenAI gym을 사용하여 구축된 tictactoe예제를 살펴봄
>
> <https://www.slideshare.net/ssuser163469/ss-78685946> 슬라이드 참고하여 작성

# **틱택토**(tic-tac-toe)

두 명이 번갈아가며 **O**와 **X**를 3×3 판에 써서 같은 글자를 가로, 세로, 혹은 대각선 상에 놓이도록 하는 놀이이다. [m,n,k-게임](https://ko.wikipedia.org/wiki/M,n,k-%EA%B2%8C%EC%9E%84)으로, (3,3,3)-게임이다. 

![fig1](https://bjo9280.github.io/assets/images/2019-11-01/fig1.png)

# 개요

소스는 <https://github.com/haje01/gym-tictactoe.git> 

적절한 상태와 보상의 설계가 강화학습 환경 제작의 핵심

O 승리의 보상 +1, X의 승리 보상 -1, 게임 진행 및 무승부 0으로 O는 보상을 최대화하려고 하며 X는 보상을 최소화 하려한다.



## Installation

```bash
pip install gym
```

## Environment

```python
class TicTacToeEnv(gym.Env): # gym.Env를 상속
    metadata = {'render.modes': ['human']} # 렌더링 모드

    def step(self, action):
        ...
    def reset(self):
        ...
    def render(self, mode='human', close=False):
        ...
```

## reset

```python
def reset(self):
    # 환경 초기화
    
    self.board = [0] * 9
    self.mark = 'O'
    self.done = False
    return self._get_obs()

def _get_obs(self):
    # 관측한 상태 (보드의 상태 + 다음 차례 마크) 반환
    return tuple(self.board), self.mark
```

## step

```python
def step(self, action):
    # 주어진 동작을 위치로 마크를 배치
    loc = action
    reward = NO_REWARD
    self.board[loc] = tocode(self.mark)
    
    # 승/패가 결정되면 마크에 맞는 리워드를 반환
    status = check_game_status(self.board)
    if status >= 0:
        self.done = True
        if status in [1, 2]:
            reward = O_REWARD if self.mark == 'O' else X_REWARD

    # 다음 플레이어로 교체
    self.mark = next_mark(self.mark)
    # 새 상태, 리워드, 에피소드 종료 여부 반환
    return self._get_obs(), reward, self.done, None
```

## render

```python
def render(self, mode='human', close=False):
    # human 모드에서만 보드를 표시
    if mode == 'human':
        self._show_board(print)  
        
def _show_board(self, showfn):
    # 보드 그리기
    for j in range(0, 9, 3):
        # 마크 코드를 문자로 0 -> ' ', 1 -> 'O' 2 -> 'X'
        print('|'.join([tomark(self.board[i])
        for i in range(j, j+3)]))
        if j < 6:
            print('--------')
```



## play

```python
def play(show_number):
    env = TicTacToeEnv(show_number=show_number)
    agents = [HumanAgent('O'),
              HumanAgent('X')]
    episode = 0
    while True:
        state = env.reset()
        _, mark = state
        done = False
        env.render()
        while not done:
            agent = agent_by_mark(agents, next_mark(mark))
            env.show_turn(True, mark)
            ava_actions = env.available_actions()
            action = agent.act(ava_actions)
            if action is None:
                sys.exit()

            state, reward, done, info = env.step(action)

            print('')
            env.render()
            if done:
                env.show_result(True, mark, reward)
                break
            else:
                _, mark = state
        episode += 1
```

