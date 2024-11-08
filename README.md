from machine import Pin, SPI
from tft import TFT_GREEN
_init()
machine.freq(160000000)
 
dc  = Pin(4, Pin.OUT)
cs  = Pin(2, Pin.OUT)
rst = Pin(5, Pin.OUT)
spi = SPI(1, baudrate=40000000, polarity=0, phase=0)
 
tft = TFT_GREEN(128, 160, spi, dc, cs, rst, rotate=0)
 
Map = [
    [1,1,0,1,1,1,0,1],
    [0,1,1,1,1,1,1,0],
    [1,1,0,0,0,1,1,1],
    [0,1,0,1,0,1,0,1],
    [0,1,0,0,0,1,0,1],
    [1,1,1,1,0,0,0,1],
    [1,0,0,0,0,0,0,1],
    [1,0,0,0,1,0,0,1],
    [1,0,0,0,1,1,1,1],
    [1,1,1,1,1,0,0,0]
]
 
Gates = []
Boxes = []
 
class Box:
    def _init_(self, tft, x, y):
        self.tft = tft
        self.x = x
        self.y = y
        self.picture = 'box.bmp'
        self.picture_onGate = 'boxngate.bmp'
        self.onGate = False
        self.draw()
 
    def draw(self):
        if (self.onGate):
            self.tft.draw_bmp(self.x * 16,self.y * 16,  self.picture_onGate)
        else:
            self.tft.draw_bmp(self.x * 16,self.y * 16, self.picture)
 
    def setOnGate(self, state):
        self.onGate = state
 
    def getOnGate(self):
        return self.onGate
 
    def getPos(self):
        return (self.x, self.y)
 
    def setPos(self, x, y):
        self.x = x
        self.y = y
        self.draw()
 
class Gate:
    def _init_(self, tft, x, y):
        self.tft = tft
        self.x = x
        self.y = y
        self.picture = 'gate.bmp'
        self.draw()
 
    def draw(self):
        self.tft.draw_bmp(self.x * 16,self.y * 16, self.picture)
 
    def getPos(self):
        return (self.x, self.y)
 
class Man:
    def _init_(self, tft, x, y):
        self.tft = tft
        self.x = x
        self.y = y
        self.picture = 'man.bmp'
        self.draw()
 
    def draw(self):
        self.tft.draw_bmp(self.x * 16,self.y * 16, self.picture)
 
    def getPos(self):
        return (self.x, self.y)
 
    def setPos(self, x, y):
        self.tft.rect(self.x * 16, self.y * 16, 16, 16, tft.COLOR_BLACK)
        self.x = x
        self.y = y
        self.draw()
 
class Button:
    def _init_(self, p, pressSate):
        self.pin = Pin(p, Pin.IN)
        self.pressSate = pressSate
        self.oldState = 0
 
    def onPress(self):
        state = self.pin.value()
        if state != self.oldState:
            self.oldState = state
            if state == self.pressSate:
                return True
        return False
 
tft.initr(tft.BGR) # tft.initr(tft.RGB) #Если вместо синего цвета отображается красный, а вместо красного синий
tft.clear(tft.COLOR_BLACK)
 
x = 0
y = 0
 
for row in Map:
    for col in row:
        if col:
            tft.draw_bmp(x * 16, y * 16,'brick.bmp')
        x+=1
    x=0
    y+=1
 
Boxes.append(Box(tft, 3,4))
Boxes.append(Box(tft, 4,6))
Boxes.append(Box(tft, 2,7))
 
Gates.append(Gate(tft, 6,3))
Gates.append(Gate(tft, 6,4))
Gates.append(Gate(tft, 6,5))
 
man = Man(tft, 5, 6)
 
btnUp = Button(16, 1)
btnDown = Button(15, 1)
btnLeft = Button(12, 1)
btnRight = Button(0, 0)
 
def canMove(x,y):
    if (Map[y][x]):
        return False
    else:
        return True
 
def feelBox(x,y):
    for B in Boxes:
        if B.getPos() == (x,y):
            return B
    return False
 
def boxFeelGate(x,y):
    for G in Gates:
        if G.getPos() == (x,y):
            return True
    return False
 
while True:
    mPos = man.getPos()
    newPos = (-1,-1)
 
    if btnUp.onPress():
        newPos = (mPos[0], mPos[1]-1)
        newPosNext = (mPos[0], mPos[1]-2)
 
    if btnDown.onPress():
        newPos = (mPos[0], mPos[1]+1)
        newPosNext = (mPos[0], mPos[1]+2)
 
    if btnLeft.onPress():
        newPos = (mPos[0]-1, mPos[1])
        newPosNext = (mPos[0]-2, mPos[1])
 
    if btnRight.onPress():
        newPos = (mPos[0]+1, mPos[1])
        newPosNext = (mPos[0]+2, mPos[1])
 
    if newPos != (-1,-1):
        B = feelBox(newPos[0], newPos[1])
        if B and canMove(newPosNext[0], newPosNext[1]):
            if boxFeelGate(newPosNext[0], newPosNext[1]):
                B.setOnGate(True)
            else:
                B.setOnGate(False)
            B.setPos(newPosNext[0], newPosNext[1])
            man.setPos(newPos[0], newPos[1])
        if not B and canMove(newPos[0], newPos[1]):
            man.setPos(newPos[0], newPos[1])
 
        for G in Gates:
            gPos = G.getPos()
            if not (feelBox(gPos[0], gPos[1])) and gPos != man.getPos():
                G.draw()
 
        win = 1
        for B in Boxes:
            if (not B.getOnGate()):
                win = 0
                break
 
        if (win):
            tft.draw_bmp(0, 16,  'win.bmp')
            raise SystemExit
