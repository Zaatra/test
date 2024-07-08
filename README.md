import pygame
import random

pygame.init()

# ---- Constants ----
WIDTH, HEIGHT = 400, 600
FPS = 60
BIRD_X = 50
PIPE_GAP = 200  # Space between pipes
PIPE_SPEED = 3
WHITE = (255, 255, 255)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)

# ---- Load assets ----
bird_img = pygame.image.load("bird.png")  # Replace 'bird.png'
bird_img = pygame.transform.scale(bird_img, (40, 30))
pipe_img = pygame.image.load("pipe.png")  # Replace 'pipe.png'

# ---- Classes ----
class Bird:
    def __init__(self):
        self.x = BIRD_X
        self.y = HEIGHT // 2
        self.velocity = 0
        self.gravity = 0.5

    def update(self):
        self.velocity += self.gravity
        self.y += self.velocity

        # Keep bird within screen bounds (top and bottom)
        if self.y < 0: 
            self.y = 0
            self.velocity = 0
        if self.y > HEIGHT:
            self.y = HEIGHT
            self.velocity = 0

    def jump(self):
        self.velocity = -8  # Upward velocity boost

    def draw(self, screen):
        screen.blit(bird_img, (self.x, self.y))

class Pipe:
    def __init__(self):
        self.x = WIDTH
        self.top_pipe_height = random.randint(100, 400)
        self.bottom_pipe_y = self.top_pipe_height + PIPE_GAP

    def update(self):
        self.x -= PIPE_SPEED
        if self.x < -pipe_img.get_width():
            self.x = WIDTH
            self.top_pipe_height = random.randint(100, 400)
            self.bottom_pipe_y = self.top_pipe_height + PIPE_GAP

    def draw(self, screen):
        screen.blit(pipe_img, (self.x, self.top_pipe_height - pipe_img.get_height()))
        screen.blit(pipe_img, (self.x, self.bottom_pipe_y))

# ---- Game setup ----
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Flappy Bird")
clock = pygame.time.Clock()
bird = Bird()
pipes = [Pipe()] 

# ---- Game loop ----
running = True
while running:
    # ---- Event handling ----
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                bird.jump()

    # ---- Game logic ----
    bird.update()
    for pipe in pipes:
        pipe.update()
        # Check for collision
        if (bird.x + bird_img.get_width() > pipe.x and 
            bird.x < pipe.x + pipe_img.get_width() and 
            (bird.y < pipe.top_pipe_height or 
             bird.y + bird_img.get_height() > pipe.bottom_pipe_y)):
            running = False  # Game Over

        # Create new pipe when old one is halfway
        if pipe.x < WIDTH // 2 and len(pipes) == 1:
            pipes.append(Pipe())

        # Remove pipe when it goes off-screen 
        if pipe.x < -pipe_img.get_width():
            pipes.pop(0)

    # ---- Drawing ----
    screen.fill(BLUE)  # Background
    for pipe in pipes:
        pipe.draw(screen)
    bird.draw(screen)
    pygame.display.flip()  

    clock.tick(FPS)

pygame.quit()
