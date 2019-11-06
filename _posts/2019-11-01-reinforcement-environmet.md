---
title: "OpenAI gym을 이용한 강화학습 환경 구축방법(틱택토예제)"
date: 2019-11-01 00:00:00 +0900
categories: Reinforcement
---

> 강화학습을 하려면 문제풀이기반의 환경을 구축해야됨
>
> OpenAI gym을 사용하여 구축된 tictactoe예제를 살펴봄

# 개요

<https://www.slideshare.net/ssuser163469/ss-78685946> 슬라이드에 tictactoe예제를 살펴봄

소스는 https://github.com/haje01/gym-tictactoe.git

적절한 상태와 보상의 설계가 강화학습 환경 제작의 핵심

O 승리의 보상 +1, X의 승리 보상 -1, 게임 진행 및 무승부 0으로 O는 보상을 최대화하려고 하며 X는 보상을 최소화 하려한다.



## Installation

```bash
pip install gym
```

## Environment

```python
class TicTacToeEnv(gym.Env):
    metadata = {'render.modes': ['human']}

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
    self.board = [0] * NUM_LOC
    self.mark = self.start_mark
    self.done = False
    return self._get_obs()

def _get_obs(self):
    return tuple(self.board), self.mark
```

## step

```python
def step(self, action):
    loc = action
    if self.done:
        return self._get_obs(), 0, True, None

    reward = NO_REWARD
    # place
    self.board[loc] = tocode(self.mark)
    status = check_game_status(self.board)
    logging.debug("check_game_status board {} mark '{}'"
                  " status {}".format(self.board, self.mark, status))
    if status >= 0:
        self.done = True
        if status in [1, 2]:
            # always called by self
            reward = O_REWARD if self.mark == 'O' else X_REWARD

    # switch turn
    self.mark = next_mark(self.mark)
    return self._get_obs(), reward, self.done, None
```

## render

```python
def render(self, mode='human', close=False):
    if close:
        return
    if mode == 'human':
        self._show_board(print)  # NOQA
        print('')
    else:
        self._show_board(logging.info)
        logging.info('')
        
def _show_board(self, showfn):
    """Draw tictactoe board."""
    for j in range(0, 9, 3):
        def mark(i):
            return tomark(self.board[i]) if not self.show_number or\
                self.board[i] != 0 else str(i+1)
        showfn(LEFT_PAD + '|'.join([mark(i) for i in range(j, j+3)]))
        if j < 6:
            showfn(LEFT_PAD + '-----')
```



