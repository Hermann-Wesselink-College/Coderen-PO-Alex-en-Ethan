import pygame
import math
import random # If my memory does not disappoint me, this is only used like, five times. and only in enemy code.
              # Update: We're proud to announce it's used one more time, in boss code.

# Important things for getting pygame working.
pygame.init()
pygame.mixer.init()
clock = pygame.time.Clock()

#sounds
death_sound = pygame.mixer.Sound("sounds/death.wav")
graze_sound = pygame.mixer.Sound("sounds/graze.wav")
hit_sound = pygame.mixer.Sound("sounds/hit.wav")
warning_sound = pygame.mixer.Sound("sounds/warning.wav")


death_sound.set_volume(0.4)

hit_sound.set_volume(0.5)

# Screen borders and FPS (frames per second)
WIDTH = 1000
HEIGHT = 800
FPS = 60

 # Funky pygame things. Looked this thing up.
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Wonderland Project") 

# yes
running = True

# Game starts at the menu
game_state = "menu"
menu_timer = 0 # (makes one piece of text blink)
quirk = random.choice(["sprites", "BPM", "Erin", "genre", "tape", "Q", "enemies"])

stage_timer = 0

# Prevents people from skipping the victory screen, because that's rude.
clear_guard_timer = 0

# Colo(u)rs for sprites (fill in if appropiate)
WHITE = (255, 255, 255)
GREY = (128, 128, 128)
BLACK = (0, 0, 0)
RED = (255, 60, 60)
CYAN = (80, 255, 255)
GREEN = (100, 255, 100)
ORANGE = (255, 165, 0)
PINK = (100, 30, 60)
PURPLE = (60, 10, 70)

# Font, and bigger font.
font = pygame.font.SysFont("consolas", 28)
fontbig = pygame.font.SysFont("consolas", 56)
fonttiny = pygame.font.SysFont("consolas", 18)
title_font = pygame.font.SysFont("trebuchetms", 80)
logo_font = pygame.font.SysFont("trebuchetms", 40)


# =====================
# PLAYER
# =====================
# Aka you. We're roping you into this we're not sorry.
class Player:
    def __init__(self):
        self.x = 350   # These 2 are starting positions
        self.y = 600
        

        self.visible_radius = 8  # Self-explanatory
        self.hitbox_radius = 3 # Your actual hitbox, visible by holding Shift.

        self.shot_timer = 0 # Changes every frame, dependent on shot_delay                   
        self.shot_delay = 5 # Value that controls how often an object can shoot

        self.invincible_timer = 0
        self.invincible_length = 120
        #Imagine how horrifying bullet hells are with no player invincibility

        self.lives = 3 # duh
        self.score = 0 # if this is 5000 something went wrong.
        
        self.clear = 0 # Marks the game clear state. What is this doing here.

    # (keys is used because you're pretty reliant on your keyboard for controls)
    def update(self, keys):
        
        # No player self.speed value because of the following line of code.
        speed = 2 if keys[pygame.K_LSHIFT] else 5

        dx = 0
        dy = 0

        if keys[pygame.K_LEFT] or keys[pygame.K_a]:
            dx -= 1
        if keys[pygame.K_RIGHT] or keys[pygame.K_d]:
            dx += 1
        if keys[pygame.K_UP] or keys[pygame.K_w]:
            dy -= 1
        if keys[pygame.K_DOWN] or keys[pygame.K_s]:
            dy += 1
            
       


        # If both x and y have a difference greater than 0 in any given frame,(fancy name for diagonol movement.
        # this code kicks in so you don't move at twice the speed.
        if dx and dy:
            dx *= 0.707
            dy *= 0.707

        self.x += dx * speed
        self.y += dy * speed
        
        # So you don't get lost outside the screen borders
        self.x = max(self.visible_radius, min(700 - self.visible_radius, self.x))
        self.y = max(self.visible_radius, min(HEIGHT - self.visible_radius, self.y))
        
        # Your shoot cooldown. For some reason it's in the gameplay loop.
        if self.shot_timer > 0:
            self.shot_timer -= 1
        
        # So you can't get hit immediately. Also in the gameplay loop for some reason.
        # What was I cooking.
        if self.invincible_timer > 0:
            self.invincible_timer -= 1

    # Draw function... draws.
    def draw(self, screen):
        
        
        # keys is much easier to say than keys = pygame.key.get_pressed()
        # Try it out loud.
        keys = pygame.key.get_pressed()


        if self.invincible_timer % 10 < 5:
            pygame.draw.circle(screen, WHITE, (int(self.x), int(self.y)), self.visible_radius)
            
            # Graze radius
            if keys[pygame.K_LSHIFT]:
                pygame.draw.circle(screen,RED,(int(self.x), int(self.y)),self.hitbox_radius)
    
    #Shows your hitbox when holding shift
        if keys[pygame.K_LSHIFT]:
            pygame.draw.circle(screen,CYAN,(int(self.x), int(self.y)),self.visible_radius + 12,1)
# =====================
# BULLETS
# =====================

class PlayerBullet:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.speed = 10
        self.radius = 4

    def update(self):
        self.y -= self.speed # Bullet moves up

    def draw(self, screen):
        pygame.draw.circle(screen, CYAN, (int(self.x), int(self.y)), self.radius)


class EnemyBullet:
    def __init__(self, x, y, dx, dy, radius=6, color = RED):

        self.x = x
        self.y = y

        self.dx = dx
        self.dy = dy

        self.radius = radius

        self.grazed = False # Used to see if an object has already been grazed by the player.

    def update(self):
        self.x += self.dx
        self.y += self.dy

    def draw(self, screen):
        pygame.draw.circle(screen, RED, (int(self.x), int(self.y)), self.radius)
        pygame.draw.circle(screen, WHITE, (int(self.x), int(self.y)), self.radius*0.9)
        


# Enemy (incl. boss) code ends at about line 1300. This isn't funny.
# =====================
# ENEMIES
# =====================

#Now we're beyond testing, Dummy doesn't exist anymore. Poor Dummy.
class EnemyDummy:
    def __init__(self):
        self.x = 350
        self.y = 200
        self.speed = 2
        self.radius = 15
        self.hp = 1000
        self.score_value = 2000

    def update(self, enemy_bullets):
        self.x += self.speed
        if self.x < 50 or self.x > 650:
            self.speed *= -1
            
    def on_death(self):
        pass
        
    def draw(self, screen):
        pygame.draw.circle(screen, RED, (int(self.x), int(self.y)), self.radius)
        
#Wait this isn't an enemy.
class EnemyRecovery:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.speed = 1.5
        self.radius = 10
        self.hp = 5
        if player.score < 5000: # To make the boss less annoying for the inexperienced.
            self.score_value = 5000 - player.score
        else:
            self.score_value = 0
            
    # No bullets.
    def update(self, enemy_bullet):
        self.x += self.speed
        
    def on_death(self):
        player.lives += 1   # This enemy spawns before the boss, so this is nice.
            
    def draw(self, screen):
        pygame.draw.circle(screen, PINK, (int(self.x), int(self.y)), self.radius)
            
    
            

#Literally the first 4 enemies in the game.
class EnemyBaby:
    def __init__(self, x, y):
        # x and y are being assigned to ambigious values, so that we can choose where they spawn in the gameplay loop.
        self.x = x  
        self.y = y
        self.speed = 3.5
        self.radius = 10
        self.hp = 10 # How tough enemies are.
        self.score_value = 50 # How much score you gain
        
        #Tracks how long an enemy attacks (60 FPS)
        self.timer = 0

        self.shot_timer = 0
        self.shot_delay = 120

    def update(self, enemy_bullets):
        
        # Enemy states, that change after some time.
        if self.y <= 300:
            self.state = "enter"
        elif self.y >= 300 and self.timer < 900:
            self.state = "attack"
        elif self.y >= 300 and self.timer > 900:
            self.state = "leave"
            
        if self.state == "enter":
            self.y += self.speed
            self.shoot = False  # This prevents enemies from shooting while entering.
        elif self.state == "attack":
            self.speed = 0
            self.shoot = True
            self.timer += 1
        elif self.state == "leave":
            self.speed = 3
            if self.x <= 350:
                self.x -= self.speed
            elif self.x >= 350:
                self.x += self.speed
            self.shoot = True
            
            
        if self.shoot == True:
            if self.shot_timer > 0:
                self.shot_timer -= 1
            else:
                self.radius = 11 # I think this was made to make it look like the enemies were animated.
                
                #Checks for the player's position on the frame the bullet spawns, and calculates the distance.
                dx = player.x - self.x
                dy = player.y - self.y
                
                # Pythagoras' theorem.
                distance = math.hypot(dx, dy)

                bullet_speed = 4
                
                # Distance decreases every frame.
                dx = (dx / distance) * bullet_speed
                dy = (dy / distance) * bullet_speed
                
                # line of code that actually creates the bullets.
                enemy_bullets.append(
                    EnemyBullet(self.x, self.y, dx, dy)
                )

                self.shot_timer = self.shot_delay # Shot timer gets reset.
                
    def on_death(self):
        pass

    def draw(self, screen):
        pygame.draw.circle(screen, GREEN, (int(self.x), int(self.y)), self.radius)
        
class EnemyStandard:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.speed = 2.5
        self.radius = 10
        self.hp = 15
        self.score_value = 100
        
        self.timer = 0

        self.shot_timer = 0
        self.shot_delay = 120

    def update(self, enemy_bullets):
        if self.y <= 300:
            self.state = "enter"
        elif self.y >= 300 and self.timer < 600:
            self.state = "attack"
        elif self.y >= 300 and self.timer > 600:
            self.state = "leave"
            
        if self.state == "enter":
            self.y += self.speed
            self.shoot = False
        elif self.state == "attack":
            self.speed = 0
            self.shoot = True
            self.timer += 1
        elif self.state == "leave":
            self.speed = 3
            if self.x <= 350:
                self.x -= self.speed
            elif self.x >= 350:
                self.x += self.speed
            self.shoot = True
            
            
        if self.shoot == True:
            if self.shot_timer > 0:
                self.shot_timer -= 1
                self.radius = 10
            else:
                self.radius = 11
                dx = player.x - self.x
                dy = player.y - self.y

                distance = math.hypot(dx, dy)

                bullet_speed = 4

                dx = (dx / distance) * bullet_speed
                dy = (dy / distance) * bullet_speed

                enemy_bullets.append(
                    EnemyBullet(self.x, self.y, dx, dy)
                )

                self.shot_timer = self.shot_delay
    
    def on_death(self):
        pass

    def draw(self, screen):
        pygame.draw.circle(screen, GREEN, (int(self.x), int(self.y)), self.radius)
        
class EnemyStandardBack:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.speed = 4
        self.radius = 10
        self.hp = 15
        self.score_value = 100
        
        #Tracks how long an enemy attacks (60 FPS)
        self.timer = 0

        self.shot_timer = 0
        self.shot_delay = 120

    def update(self, enemy_bullets):
        if self.y <= 250:
            self.state = "enter"
        elif self.y >= 250 and self.timer < 600:
            self.state = "attack"
        elif self.y >= 250 and self.timer > 600:
            self.state = "leave"
            
        if self.state == "enter":
            self.y += self.speed
            self.shoot = False
        elif self.state == "attack":
            self.speed = 0
            self.shoot = True
            self.timer += 1
        elif self.state == "leave":
            self.speed = 3
            if self.x <= 350:
                self.x -= self.speed
            elif self.x >= 350:
                self.x += self.speed
            self.shoot = True
            
            
        if self.shoot == True:
            if self.shot_timer > 0:
                self.shot_timer -= 1
            else:
                self.radius = 11
                dx = player.x - self.x
                dy = player.y - self.y

                distance = math.hypot(dx, dy)

                bullet_speed = 4

                dx = (dx / distance) * bullet_speed
                dy = (dy / distance) * bullet_speed

                enemy_bullets.append(
                    EnemyBullet(self.x, self.y, dx, dy)
                )

                self.shot_timer = self.shot_delay
    
    def on_death(self):
        pass

    def draw(self, screen):
        pygame.draw.circle(screen, GREEN, (int(self.x), int(self.y)), self.radius)
        
# 3 is better than 1.      
class EnemyBurst:
    def __init__(self, x, y):

        self.x = x
        self.y = y

        self.speed = 2
        self.radius = 12
        
        self.timer = 0

        self.hp = 25
        self.score_value = 250

        self.shot_timer = 0

        self.burst_delay = 12
        self.attack_delay = 90

        self.burst_shots_left = 3
        
    def update(self, enemy_bullets):
        if self.y <= 300 and self.timer <1:
            self.state = "enter"
        elif self.y >= 300 and self.timer <= 450:
            self.state = "attack"
        elif self.y >= 300 and self.timer > 450:
            self.state = "leave"
            
        if self.state == "enter":
            self.y += self.speed
            self.shoot = False
        elif self.state == "attack":
            self.speed = 0
            self.shoot = True
            self.timer += 1
        elif self.state == "leave":
            self.speed = 2
            if self.x < 350:
                self.x -= self.speed
            elif self.x > 350:
                self.x += self.speed
                
            elif self.x == 350:
                self.leave_direction = random.choice(["left", "right"])

                if self.leave_direction == "left":
                    self.x -= self.speed

                elif self.leave_direction == "right":
                    self.x += self.speed
            self.shoot = True
            
            
        if self.shoot == True:

            if self.shot_timer > 0:
                self.shot_timer -= 1

            else:

                dx = player.x - self.x
                dy = player.y - self.y

                distance = math.hypot(dx, dy)

                bullet_speed = 4

                dx = (dx / distance) * bullet_speed
                dy = (dy / distance) * bullet_speed

                enemy_bullets.append(
                    EnemyBullet(self.x, self.y, dx, dy)
                )

                self.burst_shots_left -= 1

                if self.burst_shots_left > 0:
                    self.shot_timer = self.burst_delay

                else:
                    self.shot_timer = self.attack_delay
                    self.burst_shots_left = 3
        
    def on_death(self):
        pass
    
    def draw(self, screen):
        pygame.draw.circle(screen, CYAN, (int(self.x), int(self.y)), self.radius)
        

class EnemyBurstBack:
    def __init__(self, x, y):

        self.x = x
        self.y = y

        self.speed = 2
        self.radius = 12
        
        self.timer = 0

        self.hp = 25
        self.score_value = 250

        self.shot_timer = 0

        self.burst_delay = 5
        self.attack_delay = 90

        self.burst_shots_left = 3
        
    def update(self, enemy_bullets):
        if self.y <= 300 and self.timer <1:
            self.state = "enter"
        elif self.y >= 300 and self.timer <= 450:
            self.state = "attack"
        elif self.y >= 300 and self.timer > 450:
            self.state = "leave"
            
        if self.state == "enter":
            self.y += self.speed
            self.shoot = False
        elif self.state == "attack":
            self.speed = 0
            self.shoot = True
            self.timer += 1
        elif self.state == "leave":
            self.speed = 2
            if self.x < 350:
                self.x -= self.speed
            elif self.x > 350:
                self.x += self.speed
                
            elif self.x == 350:
                self.leave_direction = random.choice(["left", "right"])

                if self.leave_direction == "left":
                    self.x -= self.speed

                elif self.leave_direction == "right":
                    self.x += self.speed
            self.shoot = True
            
            
        if self.shoot == True:
            
            if self.shot_timer > 0:
                self.shot_timer -= 1

        else:

            dx = player.x - self.x
            dy = player.y - self.y

            distance = math.hypot(dx, dy)

            bullet_speed = 4

            dx = (dx / distance) * bullet_speed
            dy = (dy / distance) * bullet_speed

            enemy_bullets.append(
                EnemyBullet(self.x, self.y, dx, dy)
            )

            self.burst_shots_left -= 1

            if self.burst_shots_left > 0:
                self.shot_timer = self.burst_delay

            else:
                self.shot_timer = self.attack_delay
                self.burst_shots_left = 3
                
    def on_death(self):
        pass
    
    def draw(self, screen):
        pygame.draw.circle(screen, CYAN, (int(self.x), int(self.y)), self.radius)
        
# 10 is better than 1 as well.    
class EnemyBurstHard:
    def __init__(self, x, y):

        self.x = x
        self.y = y

        self.speed = 2
        self.radius = 12
        
        self.timer = 0

        self.hp = 30
        self.score_value = 400

        self.shot_timer = 0

        self.burst_delay = 3
        self.attack_delay = 90

        self.burst_shots_left = 10
        
        
    def update(self, enemy_bullets):
        if self.y <= 300 and self.timer < 1:
            self.state = "enter"
        elif self.y >= 300 and self.timer <= 600:
            self.state = "attack"
        elif self.y >= 300 and self.timer > 600:
            self.state = "leave"
            
        if self.state == "enter":
            self.y += self.speed
            self.shoot = False
        elif self.state == "attack":
            self.speed = 0
            self.shoot = True
            self.timer += 1
        elif self.state == "leave":
            self.speed = 2
            if self.x < 350:
                self.x -= self.speed
            elif self.x > 350:
                self.x += self.speed
                
            if self.x == 350:
                self.leave_direction = random.choice(["left", "right"])

                if self.leave_direction == "left":
                    self.x -= self.speed

                elif self.leave_direction == "right":
                    self.x += self.speed
            self.shoot = True
            
            
        if self.shoot == True:

            if self.shot_timer > 0:
                self.shot_timer -= 1

            else:

                dx = player.x - self.x
                dy = player.y - self.y

                distance = math.hypot(dx, dy)

                bullet_speed = 4

                dx = (dx / distance) * bullet_speed
                dy = (dy / distance) * bullet_speed

                enemy_bullets.append(
                    EnemyBullet(self.x, self.y, dx, dy)
                )

                self.burst_shots_left -= 1

                if self.burst_shots_left > 0:
                    self.shot_timer = self.burst_delay

                else:
                    self.shot_timer = self.attack_delay
                    self.burst_shots_left = 10
    
    def on_death(self):
        pass
    
    def draw(self, screen):
        pygame.draw.circle(screen, CYAN, (int(self.x), int(self.y)), self.radius)
        
class EnemyBurstHardCorner:

    def __init__(self, x, y):

        self.x = x
        self.y = y

        self.radius = 12
        self.hp = 30
        if player.score < 5000:
            self.score_value = 5000 - player.score
        else:
            self.score = 300

        self.timer = 0

        self.shot_timer = 0
        self.burst_delay = 6
        self.attack_delay = 120
        self.burst_shots_left = 10

        if self.x < 350:
            self.speed = 1
        else:
            self.speed = -1

        self.state = "enter"

    def update(self, enemy_bullets):

        self.timer += 1

        if self.state == "enter":

            self.x += 2 * self.speed

            if self.speed == 1 and self.x >= 20:
                self.state = "attack"

            elif self.speed == -1 and self.x <= 680:
                self.state = "attack"


        elif self.state == "attack":
            self.timer += 1

            if self.timer > 1200:
                self.state = "leave"

            # Burst shooting
            if self.shot_timer > 0:
                self.shot_timer -= 1

            else:

                dx = player.x - self.x
                dy = player.y - self.y

                distance = math.hypot(dx, dy)

                bullet_speed = 4

                dx = (dx / distance) * bullet_speed
                dy = (dy / distance) * bullet_speed

                enemy_bullets.append(
                    EnemyBullet(self.x, self.y, dx, dy)
                )

                self.burst_shots_left -= 1

                if self.burst_shots_left > 0:
                    self.shot_timer = self.burst_delay

                else:
                    self.shot_timer = self.attack_delay
                    self.burst_shots_left = 10


        elif self.state == "leave":

            self.y += 5
    
    def on_death(self):
        pass

    def draw(self, screen):

        pygame.draw.circle(
            screen,
            CYAN,
            (int(self.x), int(self.y)),
            self.radius
        )
        

# Sandwich spread, because you're toast.
class EnemySpreadDLeft:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.speed = 2
        self.radius = 14
        self.hp = 20
        self.score_value = 500

        self.shot_timer = 0
        self.shot_delay = 180
        
    def update(self, enemy_bullets):    
        self.x -= self.speed
            
        if self.shot_timer > 0:
            self.shot_timer -= 1

        else:

            directions = [
                (0, -4),    # up
                (0, 4),     # down
                (-4, 0),    # left
                (4, 0),     # right

                (-3, -3),   # up-left
                (3, -3),    # up-right
                (-3, 3),    # down-left
                (3, 3)      # down-right
            ]

            for dx, dy in directions:

                enemy_bullets.append(
                    EnemyBullet(self.x, self.y, dx, dy, 8)
                )

            self.shot_timer = self.shot_delay

    def on_death(self):
        pass
        
        
    def draw(self, screen):
        pygame.draw.circle(screen, ORANGE, (int(self.x), int(self.y)), self.radius)
        
class EnemySpreadDRight:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.speed = 2
        self.radius = 14
        self.hp = 20
        self.score_value = 500

        self.shot_timer = 0
        self.shot_delay = 180
        
    def update(self, enemy_bullets):    
        self.x += self.speed
            
        if self.shot_timer > 0:
            self.shot_timer -= 1

        else:

            directions = [
                (0, -4),    # up
                (0, 4),     # down
                (-4, 0),    # left
                (4, 0),     # right

                (-3, -3),   # up-left
                (3, -3),    # up-right
                (-3, 3),    # down-left
                (3, 3)      # down-right
            ]

            for dx, dy in directions:

                enemy_bullets.append(
                    EnemyBullet(self.x, self.y, dx, dy, 8)
                )

            self.shot_timer = self.shot_delay

    def on_death(self):
        pass  
        
    def draw(self, screen):
        pygame.draw.circle(screen, ORANGE, (int(self.x), int(self.y)), self.radius)
        
        
class EnemySpreadDDown:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.speed = 1.5
        self.radius = 14
        self.hp = 30
        self.score_value = 500
        self.timer = 0

        self.shot_timer = 0
        self.shot_delay = 150
        
    def update(self, enemy_bullets):    
        if self.y <= 300:
            self.state = "enter"
        elif self.y >= 300 and self.timer < 600:
            self.state = "attack"
        elif self.y >= 300 and self.timer > 600:
            self.state = "leave"
            
        if self.state == "enter":
            self.y += self.speed
            self.shoot = False
        elif self.state == "attack":
            self.speed = 0
            self.shoot = True
            self.timer += 1
        elif self.state == "leave":
            self.speed = 2
            if self.x < 350:
                self.x -= self.speed
            elif self.x > 350:
                self.x += self.speed
                
            elif self.x == 350:
                self.leave_direction = random.choice(["left", "right"])

                if self.leave_direction == "left":
                    self.x -= self.speed

                elif self.leave_direction == "right":
                    self.x += self.speed
            self.shoot = True
            
            
        if self.shoot == True:
            
            if self.shot_timer > 0:
                self.shot_timer -= 1

            else:

                directions = [
                    (0, -4),    # up
                    (0, 4),     # down
                    (-4, 0),    # left
                    (4, 0),     # right

                    (-3, -3),   # up-left
                    (3, -3),    # up-right
                    (-3, 3),    # down-left
                    (3, 3)      # down-right
                ]

                for dx, dy in directions:

                    enemy_bullets.append(
                        EnemyBullet(self.x, self.y, dx, dy, 8)
                    )

                self.shot_timer = self.shot_delay

    def on_death(self):
        pass
    
    def draw(self, screen):
        pygame.draw.circle(screen, ORANGE, (int(self.x), int(self.y)), self.radius)

# =====================
# BOSS
# =====================
class Boss:

    def __init__(self, x, y):
        Pain = False
        self.x = x
        self.y = y
        self.speed = 1
        self.speed_enter = 2
        self.radius = 45
        self.shot_timer = 0
        self.shot_delay = 60
        self.burst_timer = 0
        self.burst_delay = 10
        self.burst_attack_delay = 150
        self.burst_left = 5
        
        self.circle_timer = 0
        self.circle_delay = 10
        self.circle_angle = 0
        self.onslaught_timer = 0

        self.phase = "enter"
        self.phase_timer = 0

        self.hp = 1200
        self.max_hp = 1200
        
        self.invulnerable = False
        
        self.final_timer = 0 # Used to handle post-death timings.
        self.transition_timer = 0
        
        self.score_value = 100000
        
    
    def update(self, enemy_bullet):           
        
        if self.phase == "enter":
            self.y += self.speed_enter
        
        # Phase transitions! If a certain HP threshold is reached, the boss phase (and "behaviour") changes!
        if self.y >= 100 and self.hp > self.max_hp*0.75:
            self.phase = "attack1"
            
        if self.hp <= self.max_hp*0.75 and self.hp > self. max_hp*0.5 and self.phase == "attack1":
            self.phase = "attack2"
            self.burst_attack_delay = 180
            self.burst_delay = 16
            self.burst_left = 10
            self.shot_delay = 120
                
        elif self.hp <= self.max_hp*0.5 and self.hp > self.max_hp*0.1 and self.phase == "attack2":
            self.phase = "onslaught_enter"
                
        elif self.hp <= self.max_hp*0.1 and self.hp > 10 and self.phase == "onslaught":
            self.phase = "desperation"
            
        elif self.hp <= 10 and self.phase != "final":
            if self.transition_timer < 1:
                warning_sound.play()
            self.transition_timer += 1
            self.invulnerable = True
            if self.transition_timer == 60:
                death_sound.play()
                self.phase = "final"
                self.final_timer = 0
            
        if self.phase == "attack1":
            if self.x < 200 or self.x > 500:
                 self.speed *= -1
            self.x += self.speed
                 
            if self.shot_timer > 0:
                self.shot_timer -= 1
        
            else:
                dx = player.x - self.x
                dy = player.y - self.y

                distance = math.hypot(dx, dy)

                bullet_speed = 4

                dx = (dx / distance) * bullet_speed
                dy = (dy / distance) * bullet_speed

                enemy_bullets.append(
                    EnemyBullet(self.x, self.y, dx, dy)
                )

                self.shot_timer = self.shot_delay
                
            if self.burst_timer > 0:
                self.burst_timer -= 1

            else:

                dx = player.x - self.x
                dy = player.y - self.y

                distance = math.hypot(dx, dy)

                bullet_speed = 5

                dx = (dx / distance) * bullet_speed
                dy = (dy / distance) * bullet_speed

                enemy_bullets.append(
                    EnemyBullet(self.x, self.y, dx, dy, 6)
                )

                self.burst_left -= 1

                if self.burst_left > 0:
                    self.burst_timer = self.burst_delay

                else:
                    self.burst_timer = self.burst_attack_delay
                    self.burst_left = 5
            
            
        if self.phase == "attack2":
            self.x += self.speed
            if self.x < 200 or self.x > 500:
                 self.speed *= -1
                    
            if self.shot_timer > 0:
                self.shot_timer -= 1

            else:
                #These bullets appear from both upper corners
                dx1 = player.x - (-5)
                dy1 = player.y - (-5)

                dist1 = math.hypot(dx1, dy1)
                speed = 3
                dx1 = (dx1 / dist1) * speed
                dy1 = (dy1 / dist1) * speed

                enemy_bullets.append(EnemyBullet(-5, -5, dx1, dy1))

                dx2 = player.x - 705
                dy2 = player.y - (-5)

                dist2 = math.hypot(dx2, dy2)
                dx2 = (dx2 / dist2) * speed
                dy2 = (dy2 / dist2) * speed

                enemy_bullets.append(EnemyBullet(705, -5, dx2, dy2))

                self.shot_timer = self.shot_delay // 2
            
            if self.burst_timer > 0:
                self.burst_timer -= 1
                
            else:

                dx = player.x - self.x
                dy = player.y - self.y

                distance = math.hypot(dx, dy)

                bullet_speed = 6

                dx = (dx / distance) * bullet_speed
                dy = (dy / distance) * bullet_speed

                enemy_bullets.append(
                    EnemyBullet(self.x, self.y, dx, dy)
                )

                self.burst_left -= 1

                if self.burst_left > 0:
                    self.burst_timer = self.burst_delay

                else:
                    self.burst_timer = self.burst_attack_delay
                    self.burst_left = 10
                    
                    
        
            
        if self.phase == "onslaught_enter":
            self.speed = 4
            if self.x < 350:
                self.x += self.speed
            if self.x > 350:
                self.x -= self.speed
            if abs(self.x - 350) < 5:
                self.x = 350
                self.speed = 0
                self.phase = "onslaught"
            
        if self.phase == "onslaught": # Fancy name for big boss attack
            Pain = True # Seems funny if you see this when debugging the game. No actual use.
            
            self.shot_delay = 60
            
            self.onslaught_timer += 1

            if self.circle_timer > 0:
                self.circle_timer -= 1

            else:
                self.circle_timer = 20  
                
                # As time goes on, more bullets get added to the ring.
                
                if self.onslaught_timer <= 40:
                    bullet_count = 8
                elif self.onslaught_timer > 40 and self.onslaught_timer <= 100:
                    bullet_count = 12
                elif self.onslaught_timer > 100 and self.onslaught_timer <= 160:
                    bullet_count = 16
                elif self.onslaught_timer > 160:
                    bullet_count = 22
                    
                # We wanted to remind the player what sub-genre this game belongs to. (Bullet hell of course!)
                
                # How do I explain this if I suck hard at math.
                
                # The range for i is the bullet_count (see above)
                # The desired angle is the circle angle (every frame we add + 20 to the angle) + whatever i is,
                # times 360 divided by the amount of bullets, so you get evenly-spaced gaps.
                
                # You then use awful sine and cosine functions to calculate the chance in direction.
                # And then we insert it in the usual bullet movement function.

                for i in range(bullet_count):

                    angle = self.circle_angle + i * (360 / bullet_count)
                    radians = math.radians(angle)

                    dx = math.cos(radians) * 3
                    dy = math.sin(radians) * 3

                    enemy_bullets.append(
                        EnemyBullet(self.x, self.y, dx, dy)
                    )

                self.circle_angle += 20
             
            
            
            if self.shot_timer > 0:
                self.shot_timer -= 1
        
            else:
                dx = player.x - self.x
                dy = player.y - self.y

                distance = math.hypot(dx, dy)
                
                # 1/4 chance for regular bullets (not the pretty ring) to be shot at twice the speed
                self.bullet_fast = random.choice(["yes", "no", "no", "no"])
                
                if self.bullet_fast == "yes":
                    bullet_speed = 6
                elif self.bullet_fast == "no":
                    bullet_speed = 3

                dx = (dx / distance) * bullet_speed
                dy = (dy / distance) * bullet_speed
                

                enemy_bullets.append(
                    EnemyBullet(self.x, self.y, dx, dy, 12)
                )

                self.shot_timer = self.shot_delay
                
        if self.phase == "desperation":
            
            self.shot_delay = 120
            
            # These few lines of code make the boss shake uncontrollably.
            # As if it's almost dead (it is)
            self.x = 350 + random.randint(-3, 3)
            self.y = 100 + random.randint(-3, 3)
            
            if self.hp < self.max_hp *0.06 and self.hp > self.max_hp* 0.01:
                self.x = 350 + random.randint(-8, 8)
                self.y = 100 + random.randint(-8, 8)
                
            if self.hp < self.max_hp *0.03:
                self.x = 350 + random.randint(-15, 15)
                self.y = 100 + random.randint(-15, 15)
                
            if self.shot_timer > 0:
                self.shot_timer -= 1
        
            else:
                dx = player.x - self.x
                dy = player.y - self.y

                distance = math.hypot(dx, dy)

                bullet_speed = 4

                dx = (dx / distance) * bullet_speed
                dy = (dy / distance) * bullet_speed

                enemy_bullets.append(
                    EnemyBullet(self.x, self.y, dx, dy)
                )

                self.shot_timer = self.shot_delay
       
        if self.phase == "final":
            self.final_timer += 1
            if self.final_timer >= 1 and self.final_timer <= 10:
                for i in range(40):
                    angle = random.uniform(0, 360)
                    rad = math.radians(angle)
                    speed = random.uniform(2, 6)

                    dx = math.cos(rad) * speed
                    dy = math.sin(rad) * speed

                    enemy_bullets.append(EnemyBullet(self.x, self.y, dx, dy))
                    
            
                
            if self.final_timer >= 360:
                self.hp = 0
            
    def on_death(self):
        player.clear += 1
        
    def draw(self, screen):
        
        if self.final_timer < 1:
            pygame.draw.circle(screen,WHITE,(int(self.x), int(self.y)),self.radius + 2,2)
            
            if self.hp < self.max_hp * 0.5 and self.hp > self.max_hp*0.1: # Turns red when angry
                pygame.draw.circle(screen, RED, (int(self.x), int(self.y)), self.radius)
                
            else:
                pygame.draw.circle(screen, PURPLE, (int(self.x), int(self.y)), self.radius)


# =====================
# OBJECTS
# =====================

player = Player()
player_bullets = []
enemy_bullets = []
enemies = []

def reset_game():

    global player
    global player_bullets
    global enemy_bullets
    global enemies
    global stage_timer
    global reset_guard_timer
    global quirk
    quirk = random.choice(["sprites", "BPM", "Erin", "genre", "tape", "Q", "enemies"])
    stage_timer = 0

    player = Player()

    player_bullets = []

    enemy_bullets = []

    enemies = []
    


# =====================
# GAME LOOP
# =====================

while running:

    screen.fill(BLACK) # Covers everything that happened in the previous frame.
    keys = pygame.key.get_pressed()

    # =====================
    # EVENTS
    # =====================

    for event in pygame.event.get():

        if event.type == pygame.QUIT:
            running = False


        if game_state == "menu":
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_z or event.key == pygame.K_o:
                    reset_game()
                    game_state = "gameplay"

                if event.key == pygame.K_x or event.key == pygame.K_p:
                    game_state = "instructions1"
                    
        elif game_state == "gameplay":
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_c:
                    game_state = "pause"
                    pause_timer = 0
                    HoldPause = True
                if event.key == pygame.K_z:
                    restart_guard = True
                    clear_guard = True

        elif game_state == "pause":
            if event.type == pygame.KEYDOWN:
                if HoldPause == False:
                    if event.key == pygame.K_z:
                        game_state = "gameplay"
                if event.key == pygame.K_x:
                    game_state = "menu"
                    
                    
        elif game_state == "game over":
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_z or event.key == pygame.K_o:
                    game_state = "gameplay"
            
                
                

    # =====================
    # MENU
    # =====================

    if game_state == "menu":
        menu_timer += 1
        
        shadow = title_font.render("Wonderland Project", True, GREY)
        screen.blit(shadow, (122, 252))
        title = title_font.render("Wonderland Project", True, WHITE)
        screen.blit(title, (120, 250))
        
        if menu_timer % 40 < 30:
            text = font.render("Press Z to start", True, WHITE)
            screen.blit(text, (370, 500))
        
        text = font.render("Press X for instructions", True, WHITE)
        screen.blit(text, (600, 40))
        
        
        if quirk ==  "sprites":
            quirktext = fonttiny.render("(sprites not included)", True, WHITE)
        elif quirk == "BPM":
            quirktext = fonttiny.render("120 BPM! (Bullets per minute, btw)", True, WHITE)
        elif quirk == "Erin":
            quirktext = fonttiny.render("Help me Erin! With my spaghetti code.", True, WHITE)
        elif quirk == "genre":
            quirktext = fonttiny.render("Not a bullet heaven.", True, WHITE)
        elif quirk == "tape":
            quirktext = fonttiny.render("Held together by tape and dreams.", True, WHITE)
        elif quirk == "Q":
            quirktext = fonttiny.render("Clicking Q used to kill the player. (Hopefully patched)", True, WHITE)
        elif quirk == "enemies":
            quirktext = fonttiny.render("Over half of the code is just enemy classes. Don't laugh.", True, WHITE)
            
            
            
        screen.blit(quirktext, ( 180, 350))

    # =====================
    # INSTRUCTIONS
    # =====================
    
    #... I'm not giving instructions on how instructions works that's weird.
    elif game_state == "instructions1":

        text_menu = font.render(f"Go back to main menu: Z", True, WHITE)
        screen.blit(text_menu, (50, 40))
        
        text_move = font.render(f"↑←↓→ or WASD to move", True, WHITE)
        screen.blit(text_move, (100, 200))
        
        text_shoot = font.render(f"Hold/Press Z to shoot", True, WHITE)
        screen.blit(text_shoot, (100, 280))
        
        text_focus = font.render(f"Hold Shift for more precise movement",True,WHITE)
        screen.blit(text_focus, (100, 360))
        
        text_focus = font.render(f"C to pause the game",True,WHITE)
        screen.blit(text_focus, (100, 440))
        
        text = font.render("→ or D to go to next page", True, WHITE)
        screen.blit(text, (300, 600))

        if keys[pygame.K_z]:
            game_state = "menu"
        if keys[pygame.K_RIGHT] or keys[pygame.K_d]:
            game_state = "instructions2"
            
    # Instructions squared.
    elif game_state == "instructions2":
        
        text_menu = font.render(f"Go back to main menu: Z", True, WHITE)
        screen.blit(text_menu, (50, 40))
        
        text_move = font.render(f"If you get hit, you lose a life.", True, WHITE)
        screen.blit(text_move, (100, 200))
        text_move = font.render(f"Just like in real life.", True, WHITE)
        screen.blit(text_move, (100, 240))
        
        text_move = font.render(f"The cyan outline when you shift? That's your graze area.", True, WHITE)
        screen.blit(text_move, (100, 320))
        text_move = font.render(f"...Not as in lifestock. We're grazing bullets.", True, WHITE)
        screen.blit(text_move, (100, 360))
        
        text_move = font.render(f"Grazing bullets rewards you with points!", True, WHITE)
        screen.blit(text_move, (100, 440))
        text_move = font.render(f"Points are more than just for show though.", True, WHITE)
        screen.blit(text_move, (100, 480))
        
        text = font.render("← or A to go to previous page", True, WHITE)
        screen.blit(text, (300, 600))
        
        if keys[pygame.K_z]:
            game_state = "menu"
        if keys[pygame.K_LEFT] or keys[pygame.K_a]:
            game_state = "instructions1"
    
    # =====================
    # PAUSE
    # =====================
    
    # Pause if you feel like going to the bathroom.
    # When you come back you already forgot what happened.
    elif game_state == "pause":
        pause_timer += 1
        
        # Assuming you were holding Z (you were, obviously).
        # This prevents the player from immediately leaving the pause menu.
        if pause_timer >= 30: 
            if keys[pygame.K_z]:
                game_state = "gameplay"
        if keys[pygame.K_x]:
            reset_game()
            game_state = "menu"
            
        # Draws everything happening on-screen every frame.
        # But since nothing's updating everything's paused.
        # Hey, just like the menu name.

        for bullet in player_bullets:
            bullet.draw(screen)

        for bullet in enemy_bullets:
            bullet.draw(screen)

        for enemy in enemies:
            enemy.draw(screen)

        player.draw(screen)
        

        pygame.draw.rect(screen, (20, 20, 20), (700, 0, 300, HEIGHT))

        player.draw(screen)

        score_text = font.render(f"Score: {player.score}", True, WHITE)
        
        lives_text = font.render(f"Lives: {player.lives}", True, WHITE)
        
        # onslaught_text = font.render(f"Onslaught: N/A", True, WHITE)
        
        time_text = font.render(f"Time: {stage_timer / 60}", True, WHITE)

        screen.blit(score_text, (720, 40))
        screen.blit(lives_text, (720, 120))
        screen.blit(onslaught_text, (720, 160))
        # screen.blit(time_text, (720, 380))

        
        text = fontbig.render("Paused", True, WHITE)
        screen.blit(text, (350, 200))
                
        text = font.render("Unpause: Z", True, WHITE)
        screen.blit(text, (350, 400))
            
        text = font.render("Quit: X", True, WHITE)
        screen.blit(text, (350, 460))
        
    # =====================
    # GAME OVER SCREEN
    # =====================
    elif game_state == "game over":
        restart_guard_timer = 0
        restart_guard_timer += 1
        
        # Prevents players from accidentally restarting if they've already been holding Z
        if restart_guard_timer >= 60:
            if keys[pygame.K_z]:
                game_state = "gameplay"
        if keys[pygame.K_x]:
            reset_game()
            game_state = "menu"
            
        text = fontbig.render("So like, you died for reals." , True, WHITE) # Truth Nuke.
        screen.blit(text, (60, 200))
        
        text = font.render("Continue: Z", True, WHITE)
        screen.blit(text, (250, 600))
        
        text = font.render("Main Menu: X", True, WHITE)
        screen.blit(text, (500, 600))
        
    # =====================
    # GAMEPLAY
    # =====================
    elif game_state == "gameplay":
        player.update(keys)
        stage_timer += 1
        if stage_timer >= 6300:
            BossFight1 = True
        else:
            BossFight1 = False
        
        #This entire stage_timer thing is to make enemies spawn properly.
        # enemies.append spawns enemies with 1. Correct variant, and 2. on the right x and y values
        if stage_timer == 120:
            enemies.append(EnemyBaby(100, -20))
            enemies.append(EnemyBaby(600, -20))
            
        if stage_timer == 600:
            enemies.append(EnemyBaby(200, -20))
            enemies.append(EnemyBaby(500, -20))
            
        if stage_timer == 900:
             enemies.append(EnemyBurst(350, -20))
             
        if stage_timer == 1080:
            enemies.append(EnemyStandard(150, -20))
        if stage_timer == 1100:
            enemies.append(EnemyStandard(600, -20))
        
        if stage_timer == 1500:
            enemies.append(EnemyStandardBack(300, -20))
        if stage_timer == 1520:
            enemies.append(EnemyStandardBack(400, -20))
            
        if stage_timer == 1740:
            enemies.append(EnemyBurst(180, -20))
            
        if stage_timer == 1920:
            enemies.append(EnemySpreadDLeft(720, 250))
            
        if stage_timer == 2100:
            enemies.append(EnemyBurst(520, -20))
        if stage_timer == 2280:
            enemies.append(EnemySpreadDRight(-20, 200))
            
        if stage_timer == 2640:
            enemies.append(EnemyStandardBack(200, -20))
            enemies.append(EnemyStandardBack(500, -20))
            
        if stage_timer == 2670:
            enemies.append(EnemyBurstHard(200, -20))
            enemies.append(EnemyBurstHard(500, -20))
            
        if stage_timer == 3300:
            enemies.append(EnemySpreadDRight(-20, 300))
            enemies.append(EnemySpreadDLeft(720, 300))
            
        if stage_timer == 3600:
            enemies.append(EnemyStandard(240, -20))
            enemies.append(EnemyStandard(460, -20))
        
        if stage_timer == 3720:
            enemies.append(EnemyStandardBack(180, -20))
            enemies.append(EnemyStandardBack(520, -20))
            
        if stage_timer == 3900:
            enemies.append(EnemySpreadDDown(350, -20))
            enemies.append(EnemyBurstHard(180, -20))
            enemies.append(EnemyBurstHard(520, -20))
            
        if stage_timer == 4500:
            enemies.append(EnemyBurstHardCorner(20, 780))
            enemies.append(EnemyBurstHardCorner(720, 780))
            
        if stage_timer == 4800:
            enemies.append(EnemyStandard(240, -20))
            enemies.append(EnemyStandard(460, -20))
            enemies.append(EnemyStandardBack(180, -20))
            enemies.append(EnemyStandardBack(520, -20))
            
        if stage_timer == 4860:
            enemies.append(EnemySpreadDLeft(720, 50))
        if stage_timer == 4920:
            enemies.append(EnemySpreadDLeft(720, 50))
            
        if stage_timer == 5640:
            enemies.append(EnemyRecovery(-20, 200))
            
        if stage_timer == 6060:
            enemies.append(Boss(350, -20))
            stage_timer += 0
            BossFight1 = True # (this does nothing I think, just sounds awesome)
            
            

        # shooting
        if keys[pygame.K_z] and player.shot_timer == 0 and player.score <5000:
            player_bullets.append(PlayerBullet(player.x, player.y))
            player.shot_timer = player.shot_delay
            
        # When you have more than 5000 points you get to shoot twice as much,
        # giving the player a feeling of reward and progression
        if keys[pygame.K_z] and player.shot_timer == 0 and player.score >=5000:
            player_bullets.append(PlayerBullet(player.x + 4, player.y))
            player_bullets.append(PlayerBullet(player.x - 4, player.y))
            player.shot_timer = player.shot_delay
            
            

        # enemies
        for enemy in enemies:
            enemy.update(enemy_bullets)
            
            # Removes off-screen enemies, making sure it doesn't turn into an Enemy graveyard.
            # Not a graveyard, actually. They can stil shoot at you (without this code) so they should still be alive.
            # ... Or zombies.
            if (
                enemy.x < -25
                or enemy.x > WIDTH + 25
                or enemy.y < -25
                or enemy.y > HEIGHT + 25
            ):
                enemies.remove(enemy)
                continue

        # player bullets
        for bullet in player_bullets[:]:
            bullet.update()
            if bullet.y < 0:
                player_bullets.remove(bullet) # removes off-screen player bullets.
                
            # Distance, gain score if bullet and enemy radius are close enough.
            # Bullet then disintegrates.
            for enemy in enemies[:]:
                dx = bullet.x - enemy.x
                dy = bullet.y - enemy.y
                if math.hypot(dx, dy) < bullet.radius + enemy.radius:
                    
                    # hasattr() Saving my life. I did not want to paste invulnerable = False in all enemy code.
                    if not hasattr(enemy, "invulnerable") or not enemy.invulnerable:
                        enemy.hp -= 1
                        player.score += 5
                    if bullet in player_bullets:
                        player_bullets.remove(bullet)

                if enemy.hp <= 0:
                    enemy.on_death()
                    "enemy_death" == True
                    enemies.remove(enemy)
                    player.score += enemy.score_value

        # enemy bullets
        for bullet in enemy_bullets[:]:
            bullet.update()
            
            if (
                bullet.x < -50
                or bullet.x > WIDTH -100
                or bullet.y < -50
                or bullet.y > HEIGHT + 50
            ):
                enemy_bullets.remove(bullet)
                continue
            
            # The next 4 or so lines of code make sure the bullet shoots at the coordinates where the player is in this specific frame.
            dx = player.x - bullet.x 
            dy = player.y - bullet.y

            distance = math.hypot(dx, dy)
            
            bullet_speed = 4 # Bullet speed of 4 is the general for most enemies.

            dx = (dx / distance) * bullet_speed
            dy = (dy / distance) * bullet_speed
            
            graze_distance = player.visible_radius + bullet.radius + 12
            hit_distance = player.hitbox_radius + bullet.radius
            
            if distance < hit_distance:
                if player.invincible_timer == 0:
                    player.lives -= 1
                    player.invincible_timer = player.invincible_length
                    hitless = False # Probably unused, since there's no achievements or dialogue.
                    if player.lives > 0:
                        hit_sound.play()
                enemy_bullets.remove(bullet) # Removes the possibility of somehow getting hit by the same bullet twice.
                
            # This distance checks if a bullet has already been grazed.
            # If it isn't, you get points.
            # the print() function was used for testing purposes.
            elif distance < graze_distance and not bullet.grazed:
                player.score += 15
                graze_sound.play()
                bullet.grazed = True
                print("Graze!")
                

        # collisions
        # If you hit an enemy you lose a live, just like in real life.
        for enemy in enemies:
            dx = player.x - enemy.x
            dy = player.y - enemy.y
            if math.hypot(dx, dy) < player.hitbox_radius + enemy.radius:
                if not hasattr(enemy, "invulnerable") or not enemy.invulnerable:
                    if player.invincible_timer == 0:
                        player.lives -= 1
                        if player.lives > 0:
                            hit_sound.play()
                        player.invincible_timer = player.invincible_length
                        hitless = False
        
        # Plays a cool sound that you don't have IRL.
        # And kicks you to the game over screen, I guess.
        if player.lives <= 0:
            death_sound.play()
            game_state = "game over"
            reset_game()
            
        if player.clear > 0:
            game_state = "clear"
            clear_guard_timer = 0

        # =====================
        # DRAW GAMEPLAY
        # =====================

        for bullet in player_bullets:
            bullet.draw(screen)

        for bullet in enemy_bullets:
            bullet.draw(screen)

        for enemy in enemies:
            enemy.draw(screen)
            
         # UI
        pygame.draw.rect(screen, (20, 20, 20), (700, 0, 300, HEIGHT))

        player.draw(screen)

        score_text = font.render(f"Score: {player.score}", True, WHITE)
        
        lives_text = font.render(f"Lives: {player.lives}", True, WHITE)
        
        onslaught_text = font.render(f"Onslaught: N/A", True, WHITE)
        
        logo_text1 = logo_font.render("Wonderland", True, WHITE)
        logo_text2 = logo_font.render("Project", True, WHITE)
        
        
      
      
        #  For developer purposes
        time_text = font.render(f"Time: {stage_timer / 60}", True, WHITE)

        screen.blit(score_text, (720, 40))
        screen.blit(lives_text, (720, 120))
        screen.blit(logo_text1, (740, 600))
        screen.blit(logo_text2, (780, 640))
        # screen.blit(onslaught_text, (720, 160))
        
        # screen.blit(time_text, (720, 380))
    

    # =====================
    # CLEAR
    # =====================

    elif game_state == "clear":
        clear_guard_timer += 1
        if clear_guard_timer >= 150:
            if keys[pygame.K_z]:
                    game_state = "menu"
                    reset_game()
                    
        text = fontbig.render("You've beaten the game!", True, WHITE)
        screen.blit(text, (140, 200))

        text = font.render("Main menu: Z", True, WHITE)
        screen.blit(text, (380, 400))
        
    pygame.display.flip()
    clock.tick(FPS)

pygame.quit()



