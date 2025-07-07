# -Omani-Flappy-Bird-Pygame-Project
# 🇴🇲 Omani Flappy Bird – Pygame Project

import pygame
import requests
import io
import random
from sys import exit

#sounds
pygame.mixer.init()
pygame.mixer.music.load("Nizwa.ogg")
pygame.mixer.music.set_volume(0.4)
pygame.mixer.music.play(-1)  # Loop forever

jump_sound = pygame.mixer.Sound("jump.wav")
jump_sound.set_volume(1.0)
gameover_sound = pygame.mixer.Sound("gameover.wav")
gameover_sound.set_volume(1.0)







# Game settings
GAME_WIDTH = 660
GAME_HEIGHT = 640
bird_width = 43
bird_height = 34
pipe_width = 64
pipe_height = 512
bird_x = GAME_WIDTH / 8
bird_y = GAME_HEIGHT / 2
pipe_x = GAME_WIDTH
pipe_y = 0

# Initialize pygame
pygame.init()
window = pygame.display.set_mode((GAME_WIDTH, GAME_HEIGHT))
pygame.display.set_caption("Flappy Bird")
clock = pygame.time.Clock()

# Load image from GitHub raw URL
def load_image_from_url(url):
    response = requests.get(url)
    return pygame.image.load(io.BytesIO(response.content))

# Load images from GitHub
background_image = pygame.transform.scale(
    load_image_from_url("https://www.omandaily.om/uploads/imported_images/uploads/2019/06/1252468.png"),
    (GAME_WIDTH, GAME_HEIGHT)
)
bird_image = load_image_from_url("https://cdn.arabsstock.com/uploads/vectors/3528/vector-cartoon-of-an-omani-thumbnail-3528.webp")
bird_image = pygame.transform.scale(bird_image, (bird_width, bird_height))
top_pipe_image = load_image_from_url("https://www.i2clipart.com/cliparts/5/e/4/5/1282705e45d27c1fbe2311e6917555ff8753c3.png")
top_pipe_image = pygame.transform.scale(top_pipe_image, (pipe_width, pipe_height))
bottom_pipe_image = load_image_from_url("https://www.i2clipart.com/cliparts/5/e/4/5/1282705e45d27c1fbe2311e6917555ff8753c3.png")
bottom_pipe_image = pygame.transform.scale(bottom_pipe_image, (pipe_width, pipe_height))

# Bird class
class Bird(pygame.Rect):
    def __init__(self, img):
        pygame.Rect.__init__(self, bird_x, bird_y, bird_width, bird_height)
        self.img = img

# Pipe class
class Pipe(pygame.Rect):
    def __init__(self, img):
        pygame.Rect.__init__(self, pipe_x, pipe_y, pipe_width, pipe_height)
        self.img = img
        self.passed = False

# Game variables
bird = Bird(bird_image)
pipes = []
velocity_x = -2
velocity_y = 0
gravity = 0.4
score = 0
game_over = False

def draw():
    window.blit(background_image, (5, 5))
    window.blit(bird.img, bird)
    for pipe in pipes:
        window.blit(pipe.img, pipe)
    text_str = f"Game Over: {int(score)}" if game_over else str(int(score))
    font = pygame.font.SysFont("Comic Sans MS", 45)
    score_render = font.render(text_str, True, "white")
    window.blit(score_render, (10, 10))

def move():
    global velocity_y, score, game_over
    velocity_y += gravity
    bird.y += velocity_y
    bird.y = max(bird.y, 0)

    if bird.y > GAME_HEIGHT:
        game_over = True
        gameover_sound.play()

    for pipe in pipes:
        pipe.x += velocity_x
        if not pipe.passed and bird.x > pipe.x + pipe.width:
            score += 0.5
            pipe.passed = True
        if bird.colliderect(pipe):
            game_over = True
            gameover_sound.play()

    while pipes and pipes[0].x < -pipe_width:
        pipes.pop(0)

def create_pipes():
    random_y = pipe_y - pipe_height / 4 - random.random() * (pipe_height / 2)
    gap = GAME_HEIGHT / 4

    top_pipe = Pipe(top_pipe_image)
    top_pipe.y = random_y
    pipes.append(top_pipe)

    bottom_pipe = Pipe(bottom_pipe_image)
    bottom_pipe.y = top_pipe.y + top_pipe.height + gap
    pipes.append(bottom_pipe)

# Timer to create pipes every 1.5 seconds
CREATE_PIPE = pygame.USEREVENT
pygame.time.set_timer(CREATE_PIPE, 1500)

# Game loop
while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            exit()
        if event.type == CREATE_PIPE and not game_over:
            create_pipes()
        if event.type == pygame.KEYDOWN:
            if event.key in (pygame.K_SPACE, pygame.K_UP, pygame.K_x):
                velocity_y = -6
                jump_sound.play()

                if game_over:
                    bird.y = bird_y
                    pipes.clear()
                    score = 0
                    game_over = False

    if not game_over:
        move()

    draw()
    pygame.display.update()
    clock.tick(60)

