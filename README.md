# Pi-Pico-Simon-Game
A simple microcontroller based Simon game to test your memory 


## How it works
- when powered on the OLED display will ask the player to press the button under the display to start the game 
- then microcontroller will flash a random led 
- next the user will need to press the button that is the same color as the led that was just flashed 
- if the user pressed the correct button they will get a point and it will be added to the score and shown on the OLED display
- now the same led will be flashed along with another one, once the LEDs are all off the user will need to press the buttons in the same order that the LEDs were flashed in and if the user pressed the buttons in the right order they will get another point ect
- than if the user presses the wrong button that is not in or out of order from the pattern the game will reset asking them if they want to play again
- this will continue until the pio is disconnected from power 




## Parts used 
- Raspberry Pi Pico 
- 128x64 OLED
- 5 NO buttons 
- 4 leds
- 4 resistors 

## Images

- Front (when off)
![front](https://github.com/AndrewSae/Pi-Pico-Simon-Game/blob/main/images/20220622_113938.jpg?raw=true)

- Back (with microcontrollerO)
![back](https://github.com/AndrewSae/Pi-Pico-Simon-Game/blob/main/images/20220622_114000.jpg?raw=true)

- Startup
![startup](https://github.com/AndrewSae/Pi-Pico-Simon-Game/blob/main/images/20220622_114057.jpg?raw=true)

- When playing
![playing](https://github.com/AndrewSae/Pi-Pico-Simon-Game/blob/main/images/20220622_114210.jpg?raw=true)

- When wrong button was pressed
![lost](https://github.com/AndrewSae/Pi-Pico-Simon-Game/blob/main/images/20220622_114239.jpg?raw=true)

## Video demo
Video can be found [here](https://drive.google.com/file/d/1--hTG3fuaXCabdZLHSP3pG-j4fPuJu1Z/view)

## Code 
``` python 
from machine import Pin, I2C
from time import sleep
from ssd1306 import SSD1306_I2C
import random
import freesans20
import writer

# led pins
leds = [15,14,13,12]
# sets up the button under the oled
SB = Pin(9, Pin.IN, Pin.PULL_DOWN)

# used to store the sequence of the leds
robot = []
# used to store the score 
score = 0

# oled size
width = 128
height = 64

# sets up the display to be used 
i2c = I2C(0, scl = Pin(1), sda = Pin(0))
oled = SSD1306_I2C(width, height, i2c)
font_writer = writer.Writer(oled, freesans20)



# gets the users input from the buttons
def getUserInput():
    userSel=True
    YB = Pin(11, Pin.IN, Pin.PULL_DOWN)
    BB = Pin(10, Pin.IN, Pin.PULL_DOWN)
    RB = Pin(16, Pin.IN, Pin.PULL_DOWN)
    GB = Pin(17, Pin.IN, Pin.PULL_DOWN)

    while userSel:
        if RB.value():
            return 15
            userSel = False
            print("b1")
        elif GB.value():
            return 14
            userSel = False
            print("b2")
        elif YB.value():
            return 13
            userSel = False
            print("b3")
        elif BB.value():
            return 12
            userSel = False
            print("b4")
        

# generates a new led to be used
def getLed():
    ledIndex = random.choice(leds)
    robot.append(ledIndex)

# flashes all the leds in the robot list one at a time 
def relightLEDS():
    count=0
    while count<len(robot):
        num = robot[count]
        led = Pin(num,Pin.OUT)
        led.toggle()
        sleep(1.5)
        led.value(0)
        sleep(.1)
        count=count+1


oled.fill(0)
font_writer.set_textpos(32, 0)
font_writer.printstring("Press To")
font_writer.set_textpos(32, 20)
font_writer.printstring("Play")
oled.show()


x = True
while x:
    # if the user pressed the button to play start to execute the main loop
    if SB.value():
        x = False
        run = True
        oled.fill(0)
        font_writer.set_textpos(32, 0)
        font_writer.printstring("Score:")
        oled.show()


while run:
    getLed() # get a new led to be used
    relightLEDS() # relight all the leds that are in the robot list
    count = 0
    # will get the user's input and checks if it matches the random order that is stored in the robot list
    while count < len(robot):
        input =  getUserInput()
        
        # if the user entered the correct button increment the loop to check the next input 
        if input == robot[count]:
            count=count+1
            
            # if the user entered the correct sequence add a point to the score and display the new score on the oled (if the loop checked all of the items in the list and the user got the pattern correct) 
            if count==len(robot):
                score=score+1
                oled.fill(0)
                font_writer.set_textpos(32, 0)
                font_writer.printstring("Score:"+ " " + str(score))
                oled.show()

        # if the user enters the incorrect button ask if they want to play again via the oled 
        else: 
            
            oled.fill(0)
            font_writer.set_textpos(32, 0)
            font_writer.printstring("Press To")
            font_writer.set_textpos(32, 20)
            font_writer.printstring("Play")
            font_writer.set_textpos(32, 40)
            font_writer.printstring("Again")
            oled.show()
            
            fail = True
            
            while fail:
                # if the user pressed the button under the oled the game will restart
                if SB.value():
                    robot.clear()
                    score = 0
                    oled.fill(0)
                    font_writer.set_textpos(32, 0)
                    font_writer.printstring("Score:")
                    oled.show()

                    fail = False
                    
        sleep(.3 ) # pause the code for .3 seconds 
```

