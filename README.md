# My-Python-Game
Python Game for Students
import pygame
import sys

# Initialize Pygame
pygame.init()

# Constants
WIDTH, HEIGHT = 800, 600
FPS = 60
GRAVITY = 1
JUMP_STRENGTH = 20
SCROLL_SPEED = 5

# Colors
WHITE = (255, 255, 255)
BLUE = (100, 150, 255)
GREEN = (0, 200, 50)
RED = (255, 50, 50)

# Setup
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Simple Mario Game")
clock = pygame.time.Clock()

# Player class
class Player(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = pygame.Surface((40, 50))
        self.image.fill(RED)
        self.rect = self.image.get_rect()
        self.rect.x = 100
        self.rect.y = HEIGHT - 150
        self.vel_y = 0
        self.on_ground = False

    def update(self, platforms):
        keys = pygame.key.get_pressed()
        dx = 0

        if keys[pygame.K_LEFT]:
            dx = -5
        if keys[pygame.K_RIGHT]:
            dx = 5
        if keys[pygame.K_SPACE] and self.on_ground:
            self.vel_y = -JUMP_STRENGTH
            self.on_ground = False

        # Gravity
        self.vel_y += GRAVITY
        dy = self.vel_y

        # Move horizontally
        self.rect.x += dx

        # Collision check (horizontal)
        for platform in platforms:
            if self.rect.colliderect(platform.rect):
                if dx > 0:
                    self.rect.right = platform.rect.left
                if dx < 0:
                    self.rect.left = platform.rect.right

        # Move vertically
        self.rect.y += dy

        # Collision check (vertical)
        self.on_ground = False
        for platform in platforms:
            if self.rect.colliderect(platform.rect):
                if self.vel_y > 0:
                    self.rect.bottom = platform.rect.top
                    self.vel_y = 0
                    self.on_ground = True
                elif self.vel_y < 0:
                    self.rect.top = platform.rect.bottom
                    self.vel_y = 0

        # Floor limit
        if self.rect.bottom > HEIGHT:
            self.rect.bottom = HEIGHT
            self.vel_y = 0
            self.on_ground = True

# Platform class
class Platform(pygame.sprite.Sprite):
    def __init__(self, x, y, w, h):
        super().__init__()
        self.image = pygame.Surface((w, h))
        self.image.fill(GREEN)
        self.rect = self.image.get_rect()
        self.rect.topleft = (x, y)

# Level setup
platforms = pygame.sprite.Group()
player = Player()

level_data = [
    (0, HEIGHT - 40, 2000, 40),
    (300, HEIGHT - 150, 150, 20),
    (600, HEIGHT - 250, 150, 20),
    (900, HEIGHT - 350, 150, 20),
    (1200, HEIGHT - 200, 150, 20),
]

for data in level_data:
    platform = Platform(*data)
    platforms.add(platform)

all_sprites = pygame.sprite.Group()
all_sprites.add(player)
all_sprites.add(platforms)

# Camera offset
scroll = 0

# Game Loop
running = True
while running:
    clock.tick(FPS)
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

    player.update(platforms)

    # Scroll logic
    if player.rect.x > WIDTH // 2:
        scroll = player.rect.x - WIDTH // 2
    else:
        scroll = 0

    # Draw
    screen.fill(BLUE)
    for platform in platforms:
        screen.blit(platform.image, (platform.rect.x - scroll, platform.rect.y))
    screen.blit(player.image, (player.rect.x - scroll, player.rect.y))

    pygame.display.flip()
