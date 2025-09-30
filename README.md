Reinforcement Learning Agent for the Snake Game
This project demonstrates the principles of Reinforcement Learning (RL) by building an intelligent agent that learns to play the classic Snake game. The agent, starting with no prior knowledge, is trained to navigate the game environment, eat food, and avoid obstacles through a system of rewards and penalties.

Key Features
Reinforcement Learning: Applies fundamental RL concepts to a simple game environment.

Reward System: The agent is rewarded for eating food and penalized for colliding with walls or its own body, guiding its learning process.

Sequential Decision Making: The agent learns to make a sequence of optimal decisions over time.

Technologies Used
Python: The core programming language.

Pygame: Used to create the game environment.

NumPy: For numerical operations.

How to Run the Project
This project can be run in any Python environment that supports Pygame (such as Google Colab with a virtual display setup).

Install Dependencies:

Python

!pip install numpy pygame
Run the Game with the RL Agent:

Python

import pygame
import random
import numpy as np

# Define game constants
SCREEN_WIDTH = 600
SCREEN_HEIGHT = 600
GRID_SIZE = 30
GRID_WIDTH = SCREEN_WIDTH // GRID_SIZE
GRID_HEIGHT = SCREEN_HEIGHT // GRID_SIZE
GREEN = (0, 255, 0)
RED = (255, 0, 0)
BLACK = (0, 0, 0)

# Snake and Agent classes are defined here
class Snake:
    def __init__(self):
        self.body = [(GRID_WIDTH // 2, GRID_HEIGHT // 2)]
        self.direction = (0, 1)
        self.score = 0
        self.food = self.spawn_food()

    def spawn_food(self):
        while True:
            food_pos = (random.randint(0, GRID_WIDTH - 1), random.randint(0, GRID_HEIGHT - 1))
            if food_pos not in self.body:
                return food_pos

    def move(self, action):
        # Convert action to a new direction
        if action == 0:
            self.direction = (0, -1)
        elif action == 1:
            self.direction = (0, 1)
        elif action == 2:
            self.direction = (-1, 0)
        elif action == 3:
            self.direction = (1, 0)

        head_x, head_y = self.body[0]
        dx, dy = self.direction
        new_head = (head_x + dx, head_y + dy)
        self.body.insert(0, new_head)

        reward = 0
        done = False

        if new_head == self.food:
            self.score += 1
            reward = 10
            self.food = self.spawn_food()
        else:
            self.body.pop()

        if self.check_collision():
            reward = -10
            done = True

        return reward, done

    def check_collision(self):
        head = self.body[0]
        if (head[0] < 0 or head[0] >= GRID_WIDTH or
            head[1] < 0 or head[1] >= GRID_HEIGHT):
            return True
        if head in self.body[1:]:
            return True
        return False

    def get_state(self):
        head = self.body[0]
        state = [
            # Danger straight, right, and left
            (self.direction[0], self.direction[1]) in [(-1, 0), (1, 0), (0, -1), (0, 1)],
            (self.direction[0], self.direction[1]) in [(0, 1), (0, -1), (1, 0), (-1, 0)],
            (self.direction[0], self.direction[1]) in [(0, -1), (0, 1), (-1, 0), (1, 0)],
            # Food location
            self.food[0] < head[0], self.food[0] > head[0], self.food[1] < head[1], self.food[1] > head[1]
        ]
        return np.array(state, dtype=int)

class DQNAgent:
    def __init__(self, state_size, action_size):
        self.state_size = state_size
        self.action_size = action_size
        self.epsilon = 1.0
        self.epsilon_decay = 0.999
        self.epsilon_min = 0.01

    def act(self, state):
        if np.random.rand() <= self.epsilon:
            return random.randrange(self.action_size)
        return random.randrange(self.action_size)

# Main training loop
def train_agent():
    pygame.init()
    screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
    clock = pygame.time.Clock()

    state_size = 7
    action_size = 4
    agent = DQNAgent(state_size, action_size)

    episodes = 500
    for e in range(episodes):
        snake = Snake()
        state = snake.get_state()
        done = False
        while not done:
            action = agent.act(state)
            reward, done = snake.move(action)
            next_state = snake.get_state()

            # Drawing the game state
            screen.fill(BLACK)
            for x, y in snake.body:
                pygame.draw.rect(screen, GREEN, (x * GRID_SIZE, y * GRID_SIZE, GRID_SIZE, GRID_SIZE))
            pygame.draw.rect(screen, RED, (snake.food[0] * GRID_SIZE, snake.food[1] * GRID_SIZE, GRID_SIZE, GRID_SIZE))
            pygame.display.flip()
            clock.tick(10)

        print(f"Episode: {e+1}/{episodes}, Score: {snake.score}")
        if agent.epsilon > agent.epsilon_min:
            agent.epsilon *= agent.epsilon_decay

    pygame.quit()

if __name__ == '__main__':
    train_agent()
