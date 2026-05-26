# Coderen-PO-Alex-en-Ethan

    self.visible_radius = 8
    self.hitbox_radius = 3

    self.shot_timer = 0
    self.shot_delay = 5

    self.invincible_timer = 0
    self.invincible_length = 120

    self.lives = 5
    self.score = 0

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
        
    #insta-death, debug purposes
    if keys[pygame.K_q]:
        self.lives = 0

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


