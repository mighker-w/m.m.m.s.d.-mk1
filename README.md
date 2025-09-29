# m.m.m.s.d.-mk1
An Arduino-controlled device, capable of scanning RFID cards/tags and obtain their identification. The device has an OLED interface, joystick for interaction, and speaker for noise output. You can also play pong on it.

Link to the video of the build process: [https://www.youtube.com/watch?v=4Ir3_Ja1QmI](url) (It doesn't go into too much detail)

So Mica: what can it do?
 - Interface: To create this device, I considered how the user will interact and use this device. Which later brought upon the gameboy/console side of things. I settled on a 128x128 RGB OLED screen since that was visually pleasing, combined with a joystick for the user to     navigate the UI (specified in my code under "void menu()"). I also added a speaker but we'll come back to that. Selection for menu bars works by pressing down on the joystick module.
 - Customisation: I wanted the display to be sci-fi/technologic-esque, and initially gave the display a green and black grid background, but later added a settings menu that allowed for 3 different backgrounds to be chosen. More backgrounds can be made by using the            Adafruit TFT library, although it is a bit of work.
 - Pong/other games: Currently, I've programmed this device to play a version of pong against an AI. The AI is very basic, so if anyone else knows how to program useful AIs in C++, hit me up. The player controls their thing with the joystick, just up and down/y-axis       
   movement. I also implemented momentum/ball speed altering, but ball movement is still overall buggy to say in the least. Pong can be quit out of by pressing down on the joystick. There is also sound feedback for when the ball is hit, and a scoring system.
 - Radio-frequency Identification (RFID) Reading: Integrated on the the back portion of this device is an MFRC522 RFID Read/Write module, which I programmed to obtain the tag ID from scanned cards. Later, I plan to implement writing data to RFID tags.
 - Lasts a while on 2 AAA batteries that supply a constant 3V. Not sure about the actual energy consumption though.

And that's it so far. Soon, I'll replace my microcontroller and then update my code and this repo. Code written by me, with the inclusion of some external libraries.
