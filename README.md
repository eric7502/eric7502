import pygame
import random
from os import path

# Initialize Pygame
pygame.init()

# Set up directory paths
img_dir = path.join(path.dirname(__file__), 'img')
snd_dir = path.join(path.dirname(__file__), 'snd')

# Set up screen dimensions
WIDTH = 480
HEIGHT = 600
FPS = 60

# Define colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)
YELLOW = (255, 255, 0)

# Set up Pygame window
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Shmup!")
clock = pygame.time.Clock()

# Set up font
font_name = pygame.font.match_font('arial')
def draw_text(surf, text, size, x, y):
    font = pygame.font.Font(font_name, size)
    text_surface = font.render(text, True, WHITE)
    text_rect = text_surface.get_rect()
    text_rect.midtop = (x, y)
    surf.blit(text_surface, text_rect)

# Function to create a new meteor
def new_meteor():
    meteor = Meteor()
    all_sprites.add(meteor)
    meteors.add(meteor)

# Function to draw the shield bar
def draw_shield_bar(surf, x, y, pct):
    if pct < 0:
        pct = 0
    BAR_LENGTH = 100
    BAR_HEIGHT = 10
    fill = (pct / 100) * BAR_LENGTH
    outline_rect = pygame.Rect(x, y, BAR_LENGTH, BAR_HEIGHT)
    fill_rect = pygame.Rect(x, y, fill, BAR_HEIGHT)
    pygame.draw.rect(surf, GREEN, fill_rect)
    pygame.draw.rect(surf, WHITE, outline_rect, 2)

# Player class
class Player(pygame.sprite.Sprite):
    def __init__(self):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.transform.scale(player_img, (50, 38))
        self.image.set_colorkey(BLACK)
        self.rect = self.image.get_rect()
        self.radius = 20
        self.rect.centerx = WIDTH / 2
        self.rect.bottom = HEIGHT - 10
        self.speedx = 0
        self.shield = 100
        self.shoot_delay = 250
        self.last_shot = pygame.time.get_ticks()

    def update(self):
        self.speedx = 0
        keystate = pygame.key.get_pressed()
        if keystate[pygame.K_LEFT]:
            self.speedx = -8
        if keystate[pygame.K_RIGHT]:
            self.speedx = 8
        if keystate[pygame.K_SPACE]:
            self.shoot()
        self.rect.x += self.speedx
        if self.rect.right > WIDTH:
            self.rect.right = WIDTH
        if self.rect.left < 0:
            self.rect.left = 0

    def shoot(self):
        now = pygame.time.get_ticks()
        if now - self.last_shot > self.shoot_delay:
            self.last_shot = now
            bullet = Bullet(self.rect.centerx, self.rect.top)
            all_sprites.add(bullet)
            bullets.add(bullet)
            shoot_sound.play()

# Meteor class
class Meteor(pygame.sprite.Sprite):
    def __init__(self):
        pygame.sprite.Sprite.__init__(self)
        self.image = random.choice(meteor_images)
        self.image.set_colorkey(BLACK)
        self.rect = self.image.get_rect()
        self.radius = int(self.rect.width * .85 / 2)
        self.rect.x = random.randrange(WIDTH - self.rect.width)
        self.rect.bottom = random.randrange(-80, -20)
        self.speedy = random.uniform(1, 8)  # Use random.uniform for decimal speeds

    def update(self):
        self.rect.y += self.speedy
        if self.rect.top > HEIGHT + 10:
            self.rect.x = random.randrange(WIDTH - self.rect.width)
            self.rect.y = random.randrange(-100, -40)
            self.speedy = random.uniform(1, 8)

# Bullet class
class Bullet(pygame.sprite.Sprite):
    def __init__(self, x, y):
        pygame.sprite.Sprite.__init__(self)
        self.image = bullet_img
        self.image.set_colorkey(BLACK)
        self.rect = self.image.get_rect()
        self.rect.bottom = y
        self.rect.centerx = x
        self.speedy = -10

    def update(self):
        self.rect.y += self.speedy
        if self.rect.bottom < 0:
            self.kill()

# Load images
player_img = pygame.image.load("C:\\Users\\정윤석\\Desktop\\2023_7학기\\Visual Media Programming\\doomgeonkit\\DoomgeonKit\\Single-Characters\\Heroine-outline2.png").convert()
bullet_img = pygame.image.load("C:\\Users\\정윤석\\Desktop\\2023_7학기\\Visual Media Programming\\doomgeonkit\\DoomgeonKit\\Single-Items\\Sword1.png").convert()  
meteor_images = [pygame.image.load("C:\\Users\\정윤석\\Desktop\\2023_7학기\\Visual Media Programming\\doomgeonkit\\DoomgeonKit\\Single-Characters\\Skelet1.png").convert()]
# ...

# Load sound
pygame.mixer.music.load("C:\\Users\\정윤석\\Desktop\\2023_7학기\\Visual Media Programming\\space_echo.ogg")
shoot_sound = pygame.mixer.Sound("C:\\Users\\정윤석\\Desktop\\2023_7학기\\Visual Media Programming\\battle_sound_effects_0\\battle_sound_effects\\swish_4.wav")
zombie_sounds = [
    pygame.mixer.Sound("C:\\Users\\정윤석\\Desktop\\2023_7학기\\Visual Media Programming\\zombies\\zombies\\zombie-6.wav"),
    pygame.mixer.Sound("C:\\Users\\정윤석\\Desktop\\2023_7학기\\Visual Media Programming\\zombies\\zombies\\zombie-7.wav"),
    pygame.mixer.Sound("C:\\Users\\정윤석\\Desktop\\2023_7학기\\Visual Media Programming\\zombies\\zombies\\zombie-8.wav"),
    pygame.mixer.Sound("C:\\Users\\정윤석\\Desktop\\2023_7학기\\Visual Media Programming\\zombies\\zombies\\zombie-9.wav"),
    pygame.mixer.Sound("C:\\Users\\정윤석\\Desktop\\2023_7학기\\Visual Media Programming\\zombies\\zombies\\zombie-10.wav"),
    pygame.mixer.Sound("C:\\Users\\정윤석\\Desktop\\2023_7학기\\Visual Media Programming\\zombies\\zombies\\zombie-11.wav"),
    pygame.mixer.Sound("C:\\Users\\정윤석\\Desktop\\2023_7학기\\Visual Media Programming\\zombies\\zombies\\zombie-12.wav")
]

# Create sprite groups
all_sprites = pygame.sprite.Group()
meteors = pygame.sprite.Group()
bullets = pygame.sprite.Group()

# Create player
player = Player()
all_sprites.add(player)

# Create initial meteors
for _ in range(8):
    new_meteor()

# Play background music
pygame.mixer.music.play(loops=-1)

# Main game loop
running = True
while running:
    # Event handling
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    # Update
    all_sprites.update()

    # Check for collisions: Bullet vs. Meteor
    hits = pygame.sprite.groupcollide(meteors, bullets, True, True)
    for hit in hits:
        # Add sound effect when bullet hits meteor
        shoot_sound.play()
        random.choice(zombie_sounds).play()  # Play a random zombie sound
        new_meteor()  # Add a new meteor when one is destroyed

    # Draw / render
    screen.fill(BLACK)
    all_sprites.draw(screen)

    # Flip the display
    pygame.display.flip()

    # Cap the frame rate
    clock.tick(FPS)

# Quit the game
pygame.quit()
