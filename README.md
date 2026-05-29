import pygame
import math
import random #If my memory does not disappoint me, this is only used like, 5 times. and only in enemy code.

pygame.init()
clock = pygame.time.Clock()

WIDTH = 1000
HEIGHT = 800
FPS = 60

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Curse Crush Project")


running = True

game_state = "menu"

WHITE = (255, 255, 255)
GREY = (128, 128, 128)
BLACK = (0, 0, 0)
RED = (255, 60, 60)
CYAN = (80, 255, 255)
GREEN = (100, 255, 100)
ORANGE = (255, 165, 0)
PINK = (100, 40, 80)
PURPLE = (100, 0, 100)

font = pygame.font.SysFont("consolas", 28)
fontbig = pygame.font.SysFont("consolas", 56)

# =====================
# PLAYER
# =====================

class Player:
    def __init__(self):
        self.x = 350
        self.y = 600
        self.speed = 5

        self.visible_radius = 8
        self.hitbox_radius = 3

        self.shot_timer = 0
        self.shot_delay = 5

        self.invincible_timer = 0
        self.invincible_length = 120

        self.lives = 3
        self.score = 5000

    def update(self, keys):
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
            
        #debug purposes, removed in finished version
        if keys[pygame.K_q]:
            self.lives = 0
            
        if keys[pygame.K_e]:
            game_state = "clear"

        if dx and dy:
            dx *= 0.707
            dy *= 0.707

        self.x += dx * speed
        self.y += dy * speed

        self.x = max(self.visible_radius, min(700 - self.visible_radius, self.x))
        self.y = max(self.visible_radius, min(HEIGHT - self.visible_radius, self.y))

        if self.shot_timer > 0:
            self.shot_timer -= 1

        if self.invincible_timer > 0:
            self.invincible_timer -= 1

    def draw(self, screen):
        pygame.draw.circle(screen, GREY, (int(self.x), int(self.y)), self.visible_radius)
        
        keys = pygame.key.get_pressed()


        if self.invincible_timer % 10 < 5:
            pygame.draw.circle(screen, WHITE, (int(self.x), int(self.y)), self.visible_radius)
        
        if keys[pygame.K_LSHIFT]:

            pygame.draw.circle(screen,CYAN,(int(self.x), int(self.y)),self.visible_radius + 12,1)
            
            pygame.draw.circle(screen,RED,(int(self.x), int(self.y)),self.hitbox_radius)
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
        self.y -= self.speed

    def draw(self, screen):
        pygame.draw.circle(screen, CYAN, (int(self.x), int(self.y)), self.radius)


class EnemyBullet:
    def __init__(self, x, y, dx, dy, radius=6):

        self.x = x
        self.y = y

        self.dx = dx
        self.dy = dy

        self.radius = radius

        self.grazed = False

    def update(self):
        self.x += self.dx
        self.y += self.dy

    def draw(self, screen):
        pygame.draw.circle(screen, RED, (int(self.x), int(self.y)), self.radius)
        


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
        self.hp = 10
        if player.score < 5000:
            self.score_value = 5000 - player.score
        else:
            self.score_value = 0
            
    def update(self, enemy_bullet):
        self.x += self.speed
        
    def on_death(self):
        player.lives += 1
            
    def draw(self, screen):
        pygame.draw.circle(screen, PINK, (int(self.x), int(self.y)), self.radius)
            
    
            

#Literally the first 6 enemies in the game.
class EnemyBaby:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.speed = 3.5
        self.radius = 10
        self.hp = 10
        self.score_value = 50
        
        #Tracks how long an enemy attacks (60 FPS)
        self.timer = 0

        self.shot_timer = 0
        self.shot_delay = 120

    def update(self, enemy_bullets):
        if self.y <= 300:
            self.state = "enter"
        elif self.y >= 300 and self.timer < 900:
            self.state = "attack"
        elif self.y >= 300 and self.timer > 900:
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
        
class EnemyStandard:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.speed = 2.5
        self.radius = 10
        self.hp = 15
        self.score_value = 100
        
        #Tracks how long an enemy attacks (60 FPS)
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
            self.x += self.speed

        if self.x < 100 or self.x > 600:
            self.speed *= -1

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
        if self.y <= 200:
            self.state = "enter"
        elif self.y >= 200 and self.timer < 450:
            self.state = "attack"
        elif self.y >= 200 and self.timer > 450:
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
            self.x += self.speed

        if self.x < 100 or self.x > 600:
            self.speed *= -1

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
        if self.y <= 200:
            self.state = "enter"
        elif self.y >= 200 and self.timer < 450:
            self.state = "attack"
        elif self.y >= 200 and self.timer > 450:
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

            if self.x < 100 or self.x > 600:
                self.speed *= -1

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
        self.x = x
        self.y = y
        self.speed = 0.5
        self.radius = 45
        self.shot_timer = 0
        self.shot_delay = 60
        self.burst_timer = 0
        self.burst_delay = 10
        self.burst_attack_delay = 150
        self.burst_left = 5

        self.phase = "enter"
        self.phase_timer = 0

        self.hp = 2400
        self.max_hp = 2400
        
        self.score_value = 10000
        
    
    def update(self, enemy_bullet):
        
        if self.phase == "enter":
            self.speed = 2
            self.y += self.speed
            if self.y >= 100:
                self.phase = "attack1"
                self.speed = 0.5
                self.shoot1 = True
                
        if self.phase == "attack1":
            self.x += self.speed
            if self.x < 200 or self.x > 500:
                 self.speed *= -1
            
            if self.hp <= self.max_hp*0.75:
                self.shoot1 = False

            

            
            if self.shoot1 == True:
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
                        EnemyBullet(self.x, self.y, dx, dy)
                    )

                    self.burst_left -= 1

                    if self.burst_left > 0:
                        self.burst_timer = self.burst_delay

                    else:
                        self.burst_timer = self.burst_attack_delay
                        self.burst_left = 5
                    
                
            
    def on_death(self):
        game_state = "clear"
        
    def draw(self, screen):
        pygame.draw.circle(screen,WHITE,(int(self.x), int(self.y)),self.radius + 2,2)
        pygame.draw.circle(screen, PURPLE, (int(self.x), int(self.y)), self.radius)


# =====================
# OBJECTS
# =====================

player = Player()
player_bullets = []
enemy_bullets = []
enemies = []

stage_timer = 6000
def reset_game():

    global player
    global player_bullets
    global enemy_bullets
    global enemies
    global stage_timer
    stage_timer = 0

    player = Player()

    player_bullets = []

    enemy_bullets = []

    enemies = []


# =====================
# GAME LOOP
# =====================

while running:

    screen.fill(BLACK)
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
                    game_state = "gameplay"

                if event.key == pygame.K_x or event.key == pygame.K_p:
                    game_state = "instructions1"

    # =====================
    # MENU
    # =====================

    if game_state == "menu":

        text = font.render("Press Z to start", True, WHITE)
        screen.blit(text, (350, 400))
        
        text = font.render("Press X for instructions", True, WHITE)
        screen.blit(text, (600, 40))

    # =====================
    # INSTRUCTIONS
    # =====================

    elif game_state == "instructions1":

        text_menu = font.render(f"Go back to main menu: Z", True, WHITE)
        screen.blit(text_menu, (50, 40))
        
        text_move = font.render(f"↑←↓→ or WASD to move", True, WHITE)
        screen.blit(text_move, (100, 200))
        
        text_shoot = font.render(f"Hold/Press Z to shoot", True, WHITE)
        screen.blit(text_shoot, (100, 280))
        
        text_focus = font.render(f"Hold Shift for more precise movement",True,WHITE)
        screen.blit(text_focus, (100, 360))
        
        text_focus = font.render(f"ESC to pause the game",True,WHITE)
        screen.blit(text_focus, (100, 440))
        
        text = font.render("→ or D to go to next page", True, WHITE)
        screen.blit(text, (300, 600))

        if keys[pygame.K_z]:
            game_state = "menu"
        if keys[pygame.K_RIGHT] or keys[pygame.K_d]:
            game_state = "instructions2"
            
    
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
    elif game_state == "pause":
        if keys[pygame.K_z]:
            game_state = "gameplay"
        if keys[pygame.K_x]:
            game_state = "menu"
            
        

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
        
        onslaught_text = font.render(f"Onslaught: N/A", True, WHITE)
        
        time_text = font.render(f"Time: {stage_timer / 60}", True, WHITE)

        screen.blit(score_text, (720, 40))
        screen.blit(lives_text, (720, 120))
        screen.blit(onslaught_text, (720, 160))
        screen.blit(time_text, (720, 380))

        
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
        if keys[pygame.K_z]:
            game_state = "gameplay"
        if keys[pygame.K_x]:
            game_state = "menu"
            
        text = fontbig.render("So like, you died for reals.", True, WHITE)
        screen.blit(text, (60, 200))
        
        text = font.render("Continue: Z", True, WHITE)
        screen.blit(text, (250, 600))
        
        text = font.render("Main Menu: X", True, WHITE)
        screen.blit(text, (500, 600))
        
    # =====================
    # GAMEPLAY
    # =====================
    elif game_state == "gameplay":
        stage_timer += 1
        if stage_timer >= 6300:
            BossFight1 = True
        else:
            BossFight1 = False
        
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
        player.update(keys)
        
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
            
        if stage_timer == 6300:
            enemies.append(Boss(350, -20))
            stage_timer += 0
            BossFight1 = True
            
        
            
            
        #pausing
        if keys[pygame.K_ESCAPE]:
            game_state = "pause"
            

        # shooting
        if keys[pygame.K_z] and player.shot_timer == 0 and player.score <5000:
            player_bullets.append(PlayerBullet(player.x, player.y))
            player.shot_timer = player.shot_delay
            
        if keys[pygame.K_z] and player.shot_timer == 0 and player.score >=5000:
            player_bullets.append(PlayerBullet(player.x + 4, player.y))
            player_bullets.append(PlayerBullet(player.x - 4, player.y))
            player.shot_timer = player.shot_delay
            
            

        # enemies
        for enemy in enemies:
            enemy.update(enemy_bullets)
            
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
                player_bullets.remove(bullet)

            for enemy in enemies[:]:
                dx = bullet.x - enemy.x
                dy = bullet.y - enemy.y
                if math.hypot(dx, dy) < bullet.radius + enemy.radius:
                    enemy.hp -= 1
                    player.score += 5
                    if bullet in player_bullets:
                        player_bullets.remove(bullet)

                if enemy.hp <= 0:
                    enemy.on_death()
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
            
            dx = player.x - bullet.x
            dy = player.y - bullet.y

            distance = math.hypot(dx, dy)
            
            bullet_speed = 4

            dx = (dx / distance) * bullet_speed
            dy = (dy / distance) * bullet_speed
            
            
            
            graze_distance = player.visible_radius + bullet.radius + 12
            hit_distance = player.hitbox_radius + bullet.radius
            
            if distance < hit_distance:
                if player.invincible_timer == 0:
                    player.lives -= 1
                    player.invincible_timer = player.invincible_length
                    hitless = False
                enemy_bullets.remove(bullet)
                
            elif distance < graze_distance and not bullet.grazed:
                player.score += 5
                bullet.grazed = True
                print("Graze!")
                

        # collisions
        for enemy in enemies:
            dx = player.x - enemy.x
            dy = player.y - enemy.y
            if math.hypot(dx, dy) < player.hitbox_radius + enemy.radius:
                if player.invincible_timer == 0:
                    player.lives -= 1
                    player.invincible_timer = player.invincible_length
                    hitless = False
                    
        if player.lives <= 0:
            game_state = "game over"
            reset_game()

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
        
      
      
        #For developer purposes
        time_text = font.render(f"Time: {stage_timer / 60}", True, WHITE)

        screen.blit(score_text, (720, 40))
        screen.blit(lives_text, (720, 120))
        screen.blit(onslaught_text, (720, 160))
        
        screen.blit(time_text, (720, 380))

    elif game_state == "clear":
        reset_game()
        if keys[pygame.K_z]:
                game_state = "menu"
        text = fontbig.render("You've beaten the game!", True, WHITE)
        screen.blit(text, (200, 200))

        text = font.render("Main menu: Z", True, WHITE)
        screen.blit(text, (300, 300))
        
        
    pygame.display.flip()
    clock.tick(FPS)

pygame.quit()



