# Term-Project-Backup
# Name: Chih-Sen (Jason) Chang; Andrew ID: chihsenc; 
# Section: L; TP Mentor: Josh Korn
# Pygame Framework created by Lukas Peraza 
# for 15-112 F15 Pygame Optional Lecture, 11/11/15

import pygame, random, math

class PygameGame(object):

    def init(self):
        pass

    def mousePressed(self, x, y):
        pass

    def mouseReleased(self, x, y):
        pass

    def mouseMotion(self, x, y):
        pass

    def mouseDrag(self, x, y):
        pass

    def keyPressed(self, keyCode, modifier):
        pass

    def keyReleased(self, keyCode, modifier):
        pass

    def timerFired(self, dt):
        pass

    def redrawAll(self, screen):
        pass

    def isKeyPressed(self, key):
        ''' return whether a specific key is being held '''
        return self._keys.get(key, False)

    def __init__(self, width=900, height=600, fps=10, title="Silent Hill"):
        self.width = width
        self.height = height
        self.fps = fps
        self.title = title
        self.bgColor = (255, 255, 255)
        pygame.init()

    def run(self):

        clock = pygame.time.Clock()
        screen = pygame.display.set_mode((self.width, self.height))
        # set the title of the window
        pygame.display.set_caption(self.title)

        # stores all the keys currently being held down
        self._keys = dict()

        # call game-specific initialization
        self.init()
        playing = True
        while playing:
            time = clock.tick(self.fps)
            self.timerFired(time)
            for event in pygame.event.get():
                if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                    self.mousePressed(*(event.pos))
                elif event.type == pygame.MOUSEBUTTONUP and event.button == 1:
                    self.mouseReleased(*(event.pos))
                elif (event.type == pygame.MOUSEMOTION and
                      event.buttons == (0, 0, 0)):
                    self.mouseMotion(*(event.pos))
                elif (event.type == pygame.MOUSEMOTION and
                      event.buttons[0] == 1):
                    self.mouseDrag(*(event.pos))
                elif event.type == pygame.KEYDOWN:
                    self._keys[event.key] = True
                    self.keyPressed(event.key, event.mod)
                elif event.type == pygame.KEYUP:
                    self._keys[event.key] = False
                    self.keyReleased(event.key, event.mod)
                elif event.type == pygame.QUIT:
                    playing = False
            screen.fill(self.bgColor)
            self.redrawAll(screen)
            pygame.display.flip()

        pygame.quit()

# def main():
#     game = PygameGame()
#     game.run()

# if __name__ == '__main__':
#     main()

class Collectibles(pygame.sprite.Sprite):

    def __init__(self, x, y, image):
        super().__init__()
        self.x = x
        self.y = y
        self.radius = 2
        self.image = image
        self.updateRect()

    def updateRect(self):
        w, h = self.image.get_size()
        self.width, self.height = w, h
        self.rect = pygame.Rect(self.x, self.y, w, h)

    def update(self):
        self.updateRect()

class Health(Collectibles):

    def __init__(self, x, y):
        healthImage = pygame.transform.scale(pygame.image.load
            ("roomFeatures/healthPotion.png").convert_alpha(), (15, 40))
        super().__init__(x, y, healthImage)

class Sanity(Collectibles):

    def __init__(self, x, y):
        sanityImage = pygame.transform.scale(pygame.image.load
            ("roomFeatures/sanityPotion.png").convert_alpha(), (15, 40))
        super().__init__(x, y, sanityImage)

class Gold(Collectibles):

    def __init__(self, x, y):
        goldImage = pygame.transform.scale(pygame.image.load
            ('roomFeatures/gold1.png').convert_alpha(), (40, 40))
        super().__init__(x, y, goldImage)

class Ammo(Collectibles):

    def __init__(self, x, y):
        ammoImage = pygame.transform.scale(pygame.image.load
            ("roomFeatures/ammo.png").convert_alpha(), (10, 25))
        super().__init__(x, y, ammoImage)

class Room(object):

    def healthInit(self):
        self.numHealth = random.randint(0, 2)
        for health in range(self.numHealth):
            x = random.randrange(self.leftBound, self.rightBound)
            y = random.randrange(self.upBound, self.downBound)
            self.healthGroup.add(Health(x, y))
        
    def sanityInit(self):
        self.numSanity = random.randint(0, 2)
        for sanity in range(self.numSanity):
            x = random.randrange(self.leftBound, self.rightBound)
            y = random.randrange(self.upBound, self.downBound)
            self.sanityGroup.add(Sanity(x, y))

    def goldInit(self):
        self.numGold = random.randint(2, 5)
        for gold in range(self.numGold):
            x = random.randrange(self.leftBound, self.rightBound)
            y = random.randrange(self.upBound, self.downBound)
            self.goldGroup.add(Gold(x, y))

    def ammoInit(self):
        self.numAmmo = random.randint(0, 2)
        for ammo in range(self.numAmmo):
            x = random.randrange(self.leftBound, self.rightBound)
            y = random.randrange(self.upBound, self.downBound)
            self.ammoGroup.add(Ammo(x, y))

    def __init__(self, width, height, background):
        self.width, self.height = width, height
        self.leftBound, self.rightBound = self.width // 3, self.width - 90
        self.upBound, self.downBound = 90, self.height - 90
        self.healthGroup = pygame.sprite.Group()
        self.sanityGroup = pygame.sprite.Group()
        self.goldGroup = pygame.sprite.Group()
        self.ammoGroup = pygame.sprite.Group()
        self.background = background
        self.leftSide = 125
        self.healthInit()
        self.sanityInit()
        self.goldInit()
        self.ammoInit()

    def drawObjs(self, screen):
        screen.blit(self.background, (self.leftSide, 0))
        self.healthGroup.draw(screen)
        self.sanityGroup.draw(screen)
        self.ammoGroup.draw(screen)
        self.goldGroup.draw(screen)

class Player(pygame.sprite.Sprite):

    def imagesInit(self):
        self.rightImages = ['Right/right1.png', 'Right/right2.png', 
                            'Right/right3.png', 'Right/right4.png']
        self.rightCount = 0
        self.shootRight = 'Right/rightShooting1.png'
        self.leftImages = ['Left/left1.png', 'Left/left2.png',
                           'Left/left3.png', 'Left/left4.png']
        self.leftCount = 0
        self.shootLeft = 'Left/leftShooting1.png'
        self.upImages = ['Up/up1.png', 'Up/up2.png', 'Up/up3.png', 'Up/up4.png']
        self.upCount = 0
        self.shootUp = 'Up/upShooting1.png'
        self.downImages = ['Down/down1.png', 'Down/down2.png',
                           'Down/down3.png', 'Down/down4.png']
        self.downCount = 0
        self.shootDown = 'Down/downShooting1.png'

    def __init__(self, x, y, width, height):
        super().__init__()
        self.imagesInit()
        self.x = x
        self.y = y
        self.width, self.height = width, height
        self.leftBound, self.rightBound = 120, self.width - 50
        self.upBound, self.downBound = 20, self.height - 50
        self.currDir = 'right'
        self.dx, self.dy = 0, 0
        self.size = 50
        self.image = pygame.transform.scale(pygame.image.
            load(self.rightImages[self.rightCount]).convert_alpha(), 
            (self.size, self.size))
        self.firing = False
        self.updateRect()

    def firingImgs(self):
        if self.currDir == 'right':
            self.image = pygame.transform.scale(pygame.image.
                load(self.shootRight).convert_alpha(), (self.size, self.size))
        elif self.currDir == 'left':
            self.image = pygame.transform.scale(pygame.image.
                load(self.shootLeft).convert_alpha(), (self.size, self.size))
        elif self.currDir == 'up':
            self.image = pygame.transform.scale(pygame.image.
                load(self.shootUp).convert_alpha(), (self.size + 10, self.size))
        elif self.currDir == 'down':
            self.image = pygame.transform.scale(pygame.image.
                load(self.shootDown).convert_alpha(), (self.size, self.size))

    def moveUp(self):
        self.currDir = 'up'
        self.dx, self.dy = 0, -6
        self.image = pygame.transform.scale(pygame.image.
            load(self.upImages[self.upCount % len(self.upImages)]).
            convert_alpha(), (self.size, self.size))
        self.upCount += 1

    def moveDown(self):
        self.currDir = 'down'
        self.dx, self.dy = 0, 6
        self.image = pygame.transform.scale(pygame.image.
            load(self.downImages[self.downCount % len(self.downImages)]).
            convert_alpha(), (self.size, self.size))
        self.downCount += 1

    def moveLeft(self):
        self.currDir = 'left'
        self.dx, self.dy = -6, 0
        self.image = pygame.transform.scale(pygame.image.
            load(self.leftImages[self.leftCount % len(self.leftImages)]).
            convert_alpha(), (self.size, self.size))
        self.leftCount += 1

    def moveRight(self):
        self.currDir = 'right'
        self.dx, self.dy = 6, 0
        self.image = pygame.transform.scale(pygame.image.
            load(self.rightImages[self.rightCount % len(self.rightImages)]).
            convert_alpha(), (self.size, self.size))
        self.rightCount += 1

    def outOfBounds(self, x, y):
        if (x < self.leftBound or x + self.size > self.rightBound or 
            y < self.upBound or y + self.size > self.downBound):
            return True

    def updateRect(self):
        w, h = self.image.get_size()
        self.width, self.height = w, h
        self.rect = pygame.Rect(self.x, self.y, w, h)

    def update(self, keysDown):
        walking = False
        if self.firing == True:
            self.firingImgs()
        else:
            if (keysDown(pygame.K_w) or keysDown(pygame.K_s) or 
                keysDown(pygame.K_a) or keysDown(pygame.K_d)):
                walking = True
            if keysDown(pygame.K_w): self.moveUp()
            if keysDown(pygame.K_s): self.moveDown()
            if keysDown(pygame.K_a): self.moveLeft()
            if keysDown(pygame.K_d): self.moveRight()
            if walking == False: self.dx, self.dy = 0, 0
        if not self.outOfBounds(self.x + self.dx, self.y + self.dy):
            self.x += self.dx
            self.y += self.dy
        self.firing = False
        self.updateRect()

class Wall(object):

    def __init__(self, x, y):
        self.x, self.y = x, y
        self.black = (0, 0, 0)
        self.wallImage = pygame.transform.scale(pygame.image.
            load('walls.png').convert_alpha(), (25, 25))

    def draw(self):
        Game.screen.blit(self.wallImage, (self.x, self.y))

class Monster(pygame.sprite.Sprite):

    def monsterInit(self):
        self.rightEnemy = ['pyramidHead/right1.png', 'pyramidHead/right2.png',
                           'pyramidHead/right3.png', 'pyramidHead/right4.png']
        self.leftEnemy = ['pyramidHead/left1.png', 'pyramidHead/left2.png',
                          'pyramidHead/left3.png', 'pyramidHead/left4.png']
        self.upEnemy = ['pyramidHead/up1.png', 'pyramidHead/up2.png',
                        'pyramidHead/up3.png', 'pyramidHead/up4.png']
        self.downEnemy = ['pyramidHead/down1.png', 'pyramidHead/down2.png',
                          'pyramidHead/down3.png', 'pyramidHead/down4.png']

    def __init__(self, width, height):
        super().__init__()
        self.monsterInit()
        self.x = width - 70
        self.y = random.randrange(70, height - 100)
        self.currDir = 'left'
        self.move = (-4, 0)
        self.radius = 10
        self.leftCount, self.rightCount = 0, 0
        self.upCount, self.downCount = 0, 0
        self.image = pygame.transform.scale(pygame.image.load
            (self.leftEnemy[self.leftCount % len(self.leftEnemy)]).
            convert_alpha(), (40, 50))
        self.updateRect()

    def updateRect(self):
        w, h = self.image.get_size()
        self.width, self.height = w, h
        self.rect = pygame.Rect(self.x, self.y, w, h)

    def moveLeft(self):
        self.image = pygame.transform.scale(pygame.image.load
            (self.leftEnemy[self.leftCount % len(self.leftEnemy)]).
            convert_alpha(), (40, 50))
        self.leftCount += 1

    def moveRight(self):
        self.image = pygame.transform.scale(pygame.image.load
            (self.rightEnemy[self.rightCount % len(self.rightEnemy)]).
            convert_alpha(), (40, 50))
        self.rightCount += 1

    def moveUp(self):
        self.image = pygame.transform.scale(pygame.image.load
            (self.upEnemy[self.upCount % len(self.upEnemy)]).
            convert_alpha(), (40, 50))
        self.upCount += 1

    def moveDown(self):
        self.image = pygame.transform.scale(pygame.image.load
            (self.downEnemy[self.downCount % len(self.downEnemy)]).
            convert_alpha(), (40, 50))
        self.downCount += 1

    def update(self):
        if self.currDir == 'left': self.moveLeft()
        elif self.currDir == 'right': self.moveRight()
        elif self.currDir == 'up': self.moveUp()
        elif self.currDir == 'down': self.moveDown()
        dx, dy = self.move
        self.x += dx
        self.y += dy
        self.updateRect()

class Bullet(pygame.sprite.Sprite):

    def bulletInit(self):
        self.rightBullet = 'Bullets/bulletRight.png'
        self.leftBullet = 'Bullets/bulletLeft.png'
        self.upBullet = 'Bullets/bulletUp.png'
        self.downBullet = 'Bullets/bulletDown.png'

    def __init__(self, x, y, direction):
        super().__init__()
        self.bulletInit()
        self.direction = direction
        if self.direction == 'right':
            self.image = pygame.transform.scale(pygame.image.load
                (self.rightBullet).convert_alpha(), (30, 20))
        elif self.direction == 'left':
            self.image = pygame.transform.scale(pygame.image.load
                (self.leftBullet).convert_alpha(), (30, 20))
        elif self.direction == 'up':
            self.image = pygame.transform.scale(pygame.image.load
                (self.upBullet).convert_alpha(), (30, 20))
        elif self.direction == 'down':
            self.image = pygame.transform.scale(pygame.image.load
                (self.downBullet).convert_alpha(), (30, 20))
        self.x, self.y = x, y
        self.bulletVel = 20
        self.updateRect()

    def updateRect(self):
        w, h = self.image.get_size()
        self.width, self.height = w, h
        self.rect = pygame.Rect(self.x, self.y, w, h)

    def update(self):
        if self.direction == 'right': self.x += self.bulletVel
        elif self.direction == 'left': self.x -= self.bulletVel
        elif self.direction == 'up': self.y -= self.bulletVel
        elif self.direction == 'down': self.y += self.bulletVel
        self.updateRect()

class Game(PygameGame):

    def colors(self):
        self.red = (255, 0, 0)
        self.bgColor = (255, 255, 255)
        self.black = (0, 0, 0)
        self.yellow = (255, 255, 0)
        self.green = (0, 255, 0)
        self.gray = (50, 50, 50)
        self.white = (255, 255, 255)

    def roomInit(self):
        self.room1 = Room(self.width, self.height, 
            pygame.transform.scale(pygame.image.load('Backgrounds/room1.png').
            convert_alpha(), (self.width - self.boxBound, self.height)))
        self.room2 = Room(self.width, self.height, 
            pygame.transform.scale(pygame.image.load('Backgrounds/room2.png').
            convert_alpha(), (self.width - self.boxBound, self.height)))
        self.room3 = Room(self.width, self.height, 
            pygame.transform.scale(pygame.image.load('Backgrounds/room3.png').
            convert_alpha(), (self.width - self.boxBound, self.height)))
        self.room4 = Room(self.width, self.height, 
            pygame.transform.scale(pygame.image.load('Backgrounds/room4.png').
            convert_alpha(), (self.width - self.boxBound, self.height)))

    def sprites(self):
        self.playerSize = 50
        self.playerGroup = pygame.sprite.Group(Player(150, 
            self.height // 2 - self.playerSize // 2, self.width, self.height))
        self.bulletsGroup = pygame.sprite.Group()
        self.monstersGroup = pygame.sprite.Group()
        self.numMon = 5
        self.dirs = [(0, -4), (2, -2), (4, 0), (2, 2), (0, 4), 
                     (-2, 2), (-4, 0), (-2, -2), (0, -4)]
        for mon in range(self.numMon):
            self.monstersGroup.add(Monster(self.width, self.height))

    def init(self):
        self.sprites()
        self.boxBound = 125
        self.colors()
        self.wave = 0
        self.barColors = [self.red, self.green]
        self.barLen = [111, 111]
        self.bars = []
        self.barMargin = 60
        self.walls = []
        self.health, self.sanity, self.gold = 100, 100, 100
        self.ammo = 100
        self.roomInit()
        self.currRoom = self.room1
        self.switchRooms = False
        self.transparency = 0
        self.switchTimer = 0
        self.nextRoom = None
        for wall in range(20):
            self.walls += [Wall(300, 57 + wall * 24.5)]

    def mousePressed(self, x, y):
        print(x, y)

    def mouseReleased(self, x, y):
        pass

    def mouseMotion(self, x, y):
        pass

    def mouseDrag(self, x, y):
        pass

    def fireBullet(self, player):
        if self.ammo == 0: return
        elif player.currDir == 'right':
            self.bulletsGroup.add(Bullet(player.x + player.size - 10,
                player.y + player.size // 2 - 12, 'right'))
        elif player.currDir == "left":
            self.bulletsGroup.add(Bullet(player.x - 15, player.y + 
                player.size // 2 - 12, 'left'))
        elif player.currDir == "up":
            self.bulletsGroup.add(Bullet(player.x + player.size // 2 - 5,
                player.y - 5, 'up'))
        elif player.currDir == "down":
            self.bulletsGroup.add(Bullet(player.x, player.y + 
                player.size // 2, 'down'))

    def openDoor(self, player):
        centerX = player.x + player.size // 2
        centerY = player.y + player.size // 2
        if (centerX > 680 and centerX < 760 and
            centerY > 20 and centerY < 80):
            self.nextRoom = self.room2
            player.x, player.y = 688, 500
            self.switchRooms = True

    def keyPressed(self, keyCode, modifier):
        player = self.playerGroup.sprites()[0]
        if keyCode == pygame.K_SPACE and self.ammo != 0:
            player.firing = True
            self.fireBullet(player)
            self.ammo -= 1
        if keyCode == pygame.K_TAB:
            self.openDoor(player)

    def keyReleased(self, keyCode, modifier):
        pass

    def checkBullets(self):
        for bullet in self.bulletsGroup.sprites():
            if (bullet.x <= self.boxBound - 17 or bullet.x >= 850 or 
                bullet.y <= 28 or bullet.y >= 550):
                self.bulletsGroup.remove(bullet)

    def calculateDist(self, newX, newY):
        player = self.playerGroup.sprites()[0]
        return math.sqrt((newX - player.x) ** 2 + (newY - player.y) ** 2)

    def moveMonsters(self, monster):
        shortestDist, shortestMove = None, None
        for direction in self.dirs:
            (dx, dy) = direction
            newX = monster.x + dx
            newY = monster.y + dy
            currDist = self.calculateDist(newX, newY)
            if shortestDist == None or currDist < shortestDist:
                shortestDist = currDist
                shortestMove = direction
        return shortestMove

    def monstersDir(self):
        for monster in self.monstersGroup.sprites():
            (dx, dy) = self.moveMonsters(monster)
            monster.move = (dx, dy)
            if (dx, dy) == self.dirs[0]:
                monster.currDir = 'up'
            elif (dx, dy) == self.dirs[2]:
                monster.currDir = 'right'
            elif (dx, dy) == self.dirs[4]:
                monster.currDir = 'down'
            elif (dx, dy) == self.dirs[6]:
                monster.currDir = 'left'

    def collisions(self):
        if pygame.sprite.groupcollide(self.bulletsGroup, self.monstersGroup, 
            True, True, pygame.sprite.collide_circle):
            self.numMon += 1
        if pygame.sprite.groupcollide(self.currRoom.goldGroup, self.playerGroup, 
            True, False, pygame.sprite.collide_circle):
            self.gold += 10
        if pygame.sprite.groupcollide(self.currRoom.healthGroup,
            self.playerGroup, True, False, pygame.sprite.collide_circle):
            if self.health + 20 > 100: self.health = 100
            else: self.health += 20
        if pygame.sprite.groupcollide(self.currRoom.sanityGroup,
            self.playerGroup, True, False, pygame.sprite.collide_circle):
            if self.sanity + 20 > 100: self.sanity = 100
            else: self.sanity += 20
        if pygame.sprite.groupcollide(self.currRoom.ammoGroup, self.playerGroup,
            True, False, pygame.sprite.collide_circle):
            self.ammo += 20

    def switch(self):
        if self.switchRooms == True:
            self.switchTimer += 1
            if self.switchTimer < 4:
                self.transparency += 85
            elif self.switchTimer >= 10:
                self.currRoom = self.nextRoom
                self.transparency -= 85
            if self.switchTimer == 12:
                self.transparency = 0
                self.switchRooms = False
                self.switchTimer = 0

    def timerFired(self, dt):
        self.playerGroup.update(self.isKeyPressed)
        self.bulletsGroup.update()
        self.monstersGroup.update()
        self.checkBullets()
        self.collisions()
        self.monstersDir()
        self.currRoom.healthGroup.update()
        self.switch()

    def drawBars(self, screen):
        for color in range(len(self.barColors)):
            currRect = [7, self.barMargin + color * 50, self.barLen[color], 20]
            pygame.draw.rect(screen, self.black, currRect, 7)
            pygame.draw.rect(screen, self.barColors[color], currRect)

    def drawText(self, screen):
        font = pygame.font.Font(None, 35)
        waves = font.render("Waves: %d" % self.wave, True, self.white)
        screen.blit(waves, (10, 5))
        pygame.draw.line(screen, self.white, (5, 30), (120, 30))
        font = pygame.font.Font(None, 25)
        text = font.render("HEALTH", True, self.white)
        screen.blit(text, (7, self.barMargin - 22))
        text = font.render("%d %%" % self.health, True, self.black)
        screen.blit(text, (9, self.barMargin + 2))
        text = font.render("SANITY", True, self.white)
        screen.blit(text, (7, self.barMargin + 28))
        text = font.render("%d %%" % self.sanity, True, self.black)
        screen.blit(text, (9, self.barMargin + 52))

    def drawInven(self, screen):
        font = pygame.font.Font(None, 35)
        inventory = font.render("Inventory", True, self.white)
        screen.blit(inventory, (7, self.barMargin + 78))
        pygame.draw.line(screen, self.white, (5, 165), (120, 165))
        font = pygame.font.Font(None, 25)
        gold = font.render("Gold: %d" % self.gold, True, self.white)
        screen.blit(gold, (7, 170))
        ammo = font.render("Ammo: %d" % self.ammo, True, self.white)
        screen.blit(ammo, (7, 190))

    def drawWalls(self):
        for wall in self.walls:
            wall.draw()

    def redrawAll(self, screen):
        self.currRoom.drawObjs(screen)
        pygame.draw.rect(screen, self.gray, 
            [0, 0, self.boxBound, self.height])
        pygame.draw.line(screen, self.black, 
            (self.boxBound, 0), (self.boxBound, self.height))
        self.drawInven(screen)
        self.drawBars(screen)
        self.drawText(screen)
        self.playerGroup.draw(screen)
        self.bulletsGroup.draw(screen)
        self.monstersGroup.draw(screen)
        # code adapted from http://stackoverflow.com/questions/6339057/
        # draw-a-transparent-rectangle-in-pygame
        if self.switchRooms:
            switchScreen = pygame.Surface((self.width - self.boxBound, 
                self.height), pygame.SRCALPHA)
            switchScreen.fill((0, 0, 0, self.transparency))
            screen.blit(switchScreen, (self.boxBound, 0))

Game(900, 600).run()
