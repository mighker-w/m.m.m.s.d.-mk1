#include <Arduino.h>
#include <U8g2lib.h>
#include <BobaBlox.h>

#ifdef U8X8_HAVE_HW_SPI
#include <SPI.h>
#endif
#ifdef U8X8_HAVE_HW_I2C
#include <Wire.h>
#endif

#include <MFRC522.h>

#define SS_PIN 10
#define RST_PIN 9
MFRC522 mfrc522(SS_PIN, RST_PIN); 

#define SCREEN_WIDTH  128
#define SCREEN_HEIGHT 128 // Change this to 96 for 1.27" OLED.

// You can use any (4 or) 5 pins 
#define SCLK_PIN 2
#define MOSI_PIN 3
#define DC_PIN   4
#define CS_PIN   5
#define RST_PIN  6

// Color definitions
#define	BLACK           0x0000
#define	BLUE            0x001F
#define	RED             0xF800
#define	GREEN           0x07E0
#define DARK_GREEN      0x0be1
#define CYAN            0x07FF
#define MAGENTA         0xF81F
#define YELLOW          0xFFE0  
#define WHITE           0xFFFF
#define PINK            0xf01f

#include <Adafruit_GFX.h>
#include <Adafruit_SSD1351.h>
#include <SPI.h>

// Option 1: use any pins but a little slower
Adafruit_SSD1351 tft = Adafruit_SSD1351(SCREEN_WIDTH, SCREEN_HEIGHT, CS_PIN, DC_PIN, MOSI_PIN, SCLK_PIN, RST_PIN);

#include <Servo.h>
Servo myServo;

Speaker sp(7);

#include "SparkFun_MMA8452Q.h"
MMA8452Q accel;

U8G2_SSD1306_128X64_ALT0_F_HW_I2C u8g2(U8G2_R0, /* reset=*/ U8X8_PIN_NONE);   // same as the NONAME variant, but may solve the "every 2nd line skipped" problem

// set up pins for ultrasonic sensor
const int trig = 12;
const int echo = 11;

const int button = 8;
const int pot = A0;

int xStick = A6;
int yStick = A1;

float durationX, distanceX;

int mode = 0;

int increment = 1;

int selected = 1;

int gridType = 1;

int mmultiplier;

int skib;

int xu;

void grid() {
  //tft.fillScreen(DARK_GREEN);
  if (gridType == 1) {
    for (int i = 0; i < 17; i++) {
      mmultiplier = 16 * i;
      tft.drawFastHLine(0, mmultiplier, 128, GREEN);
    }

    tft.drawFastHLine(0, 127, 128, GREEN);

    for (int u = 0; u < 17; u++) {
      mmultiplier = 16 * u;
      tft.drawFastVLine(mmultiplier, 0, 128, GREEN);
    }

    tft.drawFastVLine(127, 0, 128, GREEN);

  }
  else if (gridType == 2) {
    tft.fillRect(0, 0, 128, 80, CYAN);
    tft.fillRect(25, 17, 46, 13, PINK);
    tft.fillRect(33, 14, 57, 4, PINK);
    for (int i = 0; i < 128; i += 32) {
      tft.fillRect(i + 16, 80, 16, 16, WHITE);
      tft.fillRect(i, 96, 16, 16, WHITE);
      tft.fillRect(i + 16, 112, 16, 16, WHITE);
    }
  }
  else if (gridType == 3) {
    /*for (int i = 0; i < 128; i += 32) {
      tft.fillRect(i, 0, 16, 16, PINK);
      tft.fillRect(i + 16, 16, 16, 16, PINK);
      tft.fillRect(i, 32, 16, 16, PINK);
      tft.fillRect(i + 16, 48, 16, 16, PINK);
      tft.fillRect(i, 64, 16, 16, PINK);
      tft.fillRect(i + 16, 80, 16, 16, PINK);
      tft.fillRect(i, 96, 16, 16, PINK);
      tft.fillRect(i + 16, 112, 16, 16, PINK);
    } */
    tft.fillCircle(64, 50, 34, YELLOW);
    tft.fillRect(25, 52, 90, 6, BLACK);
    tft.fillRect(25, 64, 90, 6, BLACK);
    tft.fillRect(25, 76, 80, 6, BLACK);
    tft.drawFastHLine(0, 86, 128, PINK); // horizon, will gradually make the lines move further apart.
    tft.drawFastHLine(0, 88, 128, PINK);
    tft.drawFastHLine(0, 90, 128, PINK);
    tft.drawFastHLine(0, 93, 128, PINK);
    tft.drawFastHLine(0, 97, 128, PINK);
    tft.drawFastHLine(0, 102, 128, PINK);
    tft.drawFastHLine(0, 108, 128, PINK);
    tft.drawFastHLine(0, 115, 128, PINK);
    tft.drawFastHLine(0, 122, 128, PINK);
    tft.drawFastVLine(4, 86, 42, PINK);
    tft.drawFastVLine(8, 86, 42, PINK);
    tft.drawFastVLine(16, 86, 42, PINK);
    tft.drawFastVLine(32, 86, 42, PINK);
    tft.drawFastVLine(64, 86, 42, PINK);
    tft.drawFastVLine(96, 86, 42, PINK);
    tft.drawFastVLine(112, 86, 42, PINK);
    tft.drawFastVLine(120, 86, 42, PINK);
    tft.drawFastVLine(124, 86, 42, PINK);
  }
}

void setup() {
  Serial.begin(9600);
  u8g2.begin();
  pinMode(trig, OUTPUT);
  pinMode(echo, INPUT);
  pinMode(button, INPUT);
  pinMode(pot, INPUT);
  pinMode(yStick, INPUT);

  randomSeed(analogRead(A0));

  SPI.begin(); // Initialise the SPI communication
  mfrc522.PCD_Init(); // Initialise the RFID reader. 
  Serial.println("Hold your card close to the reader...");
  
  tft.begin();

  tft.fillScreen(BLACK);

  grid();

  tft.setTextColor(WHITE);
  tft.setCursor(5, 34);
  tft.setTextSize(2);
  tft.setFont();
  tft.print("m.m.m.s.d.          by mighker add snake"); // mighker's modular multifunctional specialisation device

}

int multiplier;

void cardRead() {

  tft.setCursor(0, 0);
  tft.setTextSize(1);
  tft.setTextColor(CYAN);
  String init = "Hold your card close to the reader...";
  tft.print(init); // IMPORTANT: 21 characters/line (size 1), second part of this is 16 btw
  tft.setCursor(0, 17);
  // Check if a new card is placed near the reader.
  if (!mfrc522.PICC_IsNewCardPresent()) {
    return; // Exit the loop if no new card is detected.
  }

  // Read the information from the card.
  if (!mfrc522.PICC_ReadCardSerial()) {
    return; // Exit the loop if card reading fails. 
  }

  tft.fillRect(0, 0, 128, 34, BLACK);

  tft.setCursor(0, 0);
  tft.print(init);

  tft.setCursor(0, 17);
  tft.setTextColor(BLUE);

  tft.print("Card ID: ");
  String cardID = ""; // Store the card's ID. 
  for (byte i = 0; i < mfrc522.uid.size; i++) {
    tft.print(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ");          // Format the ID with the leading byte
    tft.print(mfrc522.uid.uidByte[i], HEX);                         // Print the ID in hexadecimal
    cardID.concat(String(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ")); // Store the ID
    cardID.concat(String(mfrc522.uid.uidByte[i], HEX));
  }
  tft.setCursor(0, 25);
  Serial.println();
  
  // Card ID: 8C 0E 03 E1
  // Keyring ID: 23 CE 07 B7
  // Opal Card ID: 04 40 

  /*cardID.toUpperCase(); // Convert the cardID to uppercase
  if (cardID.substring(1) == "8C 0E 03 E1") { // Change this to the authorised card's ID
    tft.setTextColor(YELLOW);
    tft.print("Access granted.");
    sp.beep(300, 200);
    Serial.println();
  }
  else {
    tft.setTextColor(RED);
    tft.print("Access denied.");
    sp.beep(100, 100);
    delay(10);
    sp.beep(100, 100);
  } */

  delay(2000);
}

void ultrasonic() {
  digitalWrite(trig, LOW);
  delayMicroseconds(2);
  digitalWrite(trig, HIGH);
  delayMicroseconds(10);
  digitalWrite(trig, LOW);

  durationX = pulseIn(echo, HIGH);
  distanceX = (durationX*.0343)/2;

  u8g2.clearBuffer();					// clear the internal memory
  u8g2.setFont(u8g2_font_ncenB08_tr);
  u8g2.drawStr(0, 10, "Distance X: ");
  u8g2.setCursor(65, 10);
  u8g2.print(distanceX);
  u8g2.sendBuffer();					// transfer internal memory to the display
  delay(10);  
}

void gyrating() {
  //gyrating

  u8g2.clearBuffer();
  u8g2.setFont(u8g2_font_ncenB08_tr);
  u8g2.drawStr(0, 10, "X: ");
  u8g2.setCursor(65, 10);
  u8g2.print(accel.getCalculatedX());
  u8g2.sendBuffer();
  delay(10);
}

int dx = 1;
int dy = 1;

int xPos = random(28, 100);
int yPos = random(10, 76);

int linePos = 26;
int opLinePos = 26;

int points = 0;
int botPoints = 0;

int dLine;

int colour = CYAN;

void bounce() {
  xPos += dx;
  yPos += dy;

  tft.drawRect(0, 0, 126, 86, BLUE); // bounds have to be even

  if (analogRead(yStick) < 300) {
    dLine = -3; // precision controls
  }
  else if (analogRead(yStick) < 80) {
    dLine = -6;
  } 
  else if (analogRead(yStick) > 600) {
    dLine = 3;
  }
  else if (analogRead(yStick) > 700) {
    dLine = 6;
  }
  else {
    dLine = 0;
  }

  linePos += dLine;

  if (linePos + 24 >= 85) {
    linePos = 61; // too lazy to do the maths
  }
  else if (linePos <= 1) {
    linePos = 1;
  }

  if (xPos + 5 >= 120) { //box width // oppenent's court
    if (yPos >= opLinePos && yPos <= opLinePos + 24) {
      dx = -2; 
      sp.beep(350, 50);
      colour = WHITE;
    }
    else {
      points += 1;
      tft.setCursor(20, 20);
      tft.setTextColor(GREEN);
      tft.print("Your");
      tft.setCursor(20, 40);
      tft.print("point!");

      tft.fillRect(10, 90, 118, 18, BLACK);
      tft.setCursor(10, 90);
      tft.setTextSize(2);
      tft.setTextColor(WHITE);
      tft.print(points);
      tft.print(" : ");
      tft.print(botPoints);

      delay(1000);

      tft.fillScreen(BLACK);

      xPos = random(28, 100);
      yPos = random(10, 76);

      dx = 2;
      dy = 2;
    }
  }

  if (xPos <= 6) { // the ball is in your court now, buddy
    if (yPos >= linePos && yPos <= linePos + 24) { // NEXT: introduce momentum transfer
      if (dLine >= 3 && dy == 2) {
        dx = 4;
        dy *= 2;
      }
      else if (dLine <= -3 && dy == -2) {
        dx = 4;
        dy *= 2;
      }
      else if (dLine >= 3 && dy == -2) {
        dx = 2;
        dy = -1;
      }
      else if (dLine <= -3 && dy == 2) {
        dx = 2;
        dy = 1;
      }
      else {
        dx = 2;
        dy = 2;
      }

      sp.beep(350, 50);
    }
    else {
      botPoints += 1;
      tft.setCursor(20, 20);
      tft.setTextColor(RED);
      tft.setTextSize(2);
      tft.print("Bot's");
      tft.setCursor(20, 40);
      tft.print("point!");
      tft.fillRect(10, 90, 118, 18, BLACK);
      tft.setCursor(10, 90);
      tft.setTextSize(2);
      tft.setTextColor(WHITE);
      tft.print(points);
      tft.print(" : ");
      tft.print(botPoints);

      delay(1000);

      tft.fillScreen(BLACK);

      xPos = random(28, 100);
      yPos = random(10, 76);

      dx = 2;
      dy = 2;
    }
  }

  if (yPos + 6 >= 85) {
    dy = -2;
    sp.beep(350, 50);
    colour = CYAN;
  }

  if (yPos <= 2) {
    dy = 2;
    sp.beep(350, 50);
    colour = RED;
  }

  if (random(1, 5) == 3) {
    opLinePos -= 4;
  }
  else {
    opLinePos = yPos + random(-4, 4) - random(1, 24); // random falloff rate, but tries to follow ball with midpoint.
  }

  // bounds
  if (opLinePos + 24 >= 85) {
    opLinePos = 61; 
  }
  else if (opLinePos <= 1) {
    opLinePos = 1;
  }

  tft.fillRect(xPos, yPos, 5, 6, colour);

  tft.drawFastVLine(6, linePos, 24, WHITE);

  tft.drawFastVLine(120, opLinePos, 24, WHITE);

  //Serial.print("Vertical Joystick Value: "); // only care about vertical movement for the sake of pong
  //Serial.println(analogRead(yStick));
  /*Serial.print(", Horizontal Joystick Value: ");
  Serial.println(analogRead(xStick));*/

  delay(6);

  tft.fillRect(xPos, yPos, 5, 6, BLACK);
  tft.drawFastVLine(6, linePos, 30, BLACK);
  tft.drawFastVLine(120, opLinePos, 30, BLACK);
}

int snakeLength = 1; // has to be manually updated i think
int snakeDeltaX = 2;
int snakeDeltaY = 0; // so that the snake starts by moving to the right (maybe adjust the values?)
int snX = 40;
int snY = 40;

int snakeParts[] = {40, 40, /*44, 40, 48, 40, 52, 40, 56, 40*/}; // individual snake parts are 2x2 in pixel length, parts are grouped together as x, y, x, y... 
// also, when the snake eats an apple, next frame will just add another segment in the same last tail spot
void snake() {

  /*for (int p = 0; p <= snakeLength; p++) {
    if (snakeParts[p] % 2 == 0 || snakeParts[p] == 0) {
      snakeParts[p] += snakeDeltaX;
    }
    else if (snakeParts[p] % 2 == 1) {
      snakeParts[p] += snakeDeltaY;
    }
  } */

  if (analogRead(yStick) <= 300) {                         
    snakeDeltaY = -2;                                     //| 
    snakeDeltaX = 0;                                     // |
  }                                                     //  |
  else if (analogRead(yStick) >= 600) {                //   |
    snakeDeltaY = 2;                                  //    |
    snakeDeltaX = 0;                                 //     |
  }                                                 //______| (delta)

  if (analogRead(xStick) <= 300) {
    snakeDeltaX = 2;
    snakeDeltaY = 0;
  }
  else if (analogRead(xStick) >= 600) {
    snakeDeltaX = -2;
    snakeDeltaY = 0;
  }

  snX += snakeDeltaX;
  snY += snakeDeltaY;

  tft.fillRect(snX, snY, 8, 8, BLUE);

  /*for (int i = 0; i <= snakeLength * 2; i += 2) {
    int u = i + 1;
    i += snakeDeltaX;
    u += snakeDeltaY;
  } */ //idk if ths even works

  /*for (int i = 0; i <= snakeLength * 2; i += 2) {
    int u = i + 1;
    tft.fillRect(snakeParts[i], snakeParts[u], 2, 2, BLUE);
  } */

  delay(10); // 10ms delay to refresh position

  tft.fillRect(snX, snY, 8, 8, BLACK);

  /*for (int i = 0; i <= snakeLength - 1; i += 2) {
    int u = i + 1;
    tft.fillRect(snakeParts[i], snakeParts[u], 2, 2, BLACK);
  } */
}

/*bool enableGrid = true;
bool enableSurreal = false;
bool enableRetro = false; */
bool enableSetRet = false;

void settings() {
  
  if (analogRead(yStick) <= 300){
    if (selected == 1) {
      selected = 1;
    }
    else {
      selected -= 1;
      // remember to clear buttons right here
      tft.fillRect(33, 17, 64, 12, BLACK);
      tft.fillRect(33, 49, 64, 12, BLACK);
      tft.fillRect(33, 82, 64, 12, BLACK);
      tft.fillRect(33, 114, 64, 12, BLACK);
    }
  }
  else if (analogRead(yStick) >= 600) {
    if (selected == 4) {
      selected == 4;
    }
    else {
      selected += 1;
      tft.fillRect(33, 17, 64, 12, BLACK);
      tft.fillRect(33, 49, 64, 12, BLACK);
      tft.fillRect(33, 82, 64, 12, BLACK);
      tft.fillRect(33, 114, 64, 12, BLACK);
    }
  }

  if (selected == 1) {
    tft.fillRect(33, 17, 64, 12, WHITE);

    tft.setCursor(36, 20);
    tft.setTextColor(BLACK);
    tft.setTextSize(1);
    tft.print("Green Grid");

    tft.drawRect(33, 49, 64, 12, WHITE);

    tft.setCursor(36, 52);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print("Surreal");

    tft.drawRect(33, 82, 64, 12, WHITE);

    tft.setCursor(36, 85);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print("Retro"); // texture

    tft.drawRect(33, 114, 64, 12, WHITE);

    tft.setCursor(36, 117);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print(" <-Return");
  }
  else if (selected == 2) {
    tft.drawRect(33, 17, 64, 12, WHITE);

    tft.setCursor(36, 20);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print("Green Grid");

    tft.fillRect(33, 49, 64, 12, WHITE);

    tft.setCursor(36, 52);
    tft.setTextColor(BLACK);
    tft.setTextSize(1);
    tft.print("Surreal");

    tft.drawRect(33, 82, 64, 12, WHITE);

    tft.setCursor(36, 85);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print("Retro"); // texture

    tft.drawRect(33, 114, 64, 12, WHITE);

    tft.setCursor(36, 117);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print(" <-Return");
  }
  else if (selected == 3) {
    tft.drawRect(33, 17, 64, 12, WHITE);

    tft.setCursor(36, 20);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print("Green Grid");
    tft.drawRect(33, 49, 64, 12, WHITE);

    tft.setCursor(36, 52);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print("Surreal");

    tft.fillRect(33, 82, 64, 12, WHITE);

    tft.setCursor(36, 85);
    tft.setTextColor(BLACK);
    tft.setTextSize(1);
    tft.print("Retro"); // texture

    tft.drawRect(33, 114, 64, 12, WHITE);

    tft.setCursor(36, 117);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print(" <-Return");
  }
  else if (selected == 4) {
    tft.drawRect(33, 17, 64, 12, WHITE);

    tft.setCursor(36, 20);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print("Green Grid");
    tft.drawRect(33, 49, 64, 12, WHITE);

    tft.setCursor(36, 52);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print("Surreal");

    tft.drawRect(33, 82, 64, 12, WHITE);

    tft.setCursor(36, 85);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print("Retro"); // texture

    tft.fillRect(33, 114, 64, 12, WHITE);

    tft.setCursor(36, 117);
    tft.setTextColor(BLACK);
    tft.setTextSize(1);
    tft.print(" <-Return");
  }

  if (digitalRead(button) == 1) {
    if (selected != 4) {
      gridType = selected;
    }
    else {
      enableSetRet = true;
    }
  }
}

int currentBar = 1;

bool enablePong = false;
bool enableRFID = false;
bool enableRet = false;
bool enableSettings = false;

void menu() {

  // create the menu and highlight which bar is selected

  if (analogRead(yStick) <= 300) {
    if (currentBar == 1) {
      currentBar == 1;
    }
    else {
      currentBar -= 1;
      tft.fillRect(33, 17, 64, 12, BLACK);
      tft.fillRect(33, 49, 64, 12, BLACK);
      tft.fillRect(33, 82, 64, 12, BLACK);
      tft.fillRect(33, 114, 64, 12, BLACK);
    }
  }
  else if (analogRead(yStick) >= 600) {
    if (currentBar == 4) {
      currentBar = 4;
    }
    else {
      currentBar += 1;
      tft.fillRect(33, 17, 64, 12, BLACK);
      tft.fillRect(33, 49, 64, 12, BLACK);
      tft.fillRect(33, 82, 64, 12, BLACK);
      tft.fillRect(33, 114, 64, 12, BLACK);
    }
  }

  if (currentBar == 1) {
    tft.fillRect(33, 17, 64, 12, WHITE);

    tft.setCursor(36, 20);
    tft.setTextColor(BLACK);
    tft.setTextSize(1);
    tft.print("Play PONG");
    tft.drawRect(33, 49, 64, 12, WHITE);

    tft.setCursor(36, 52);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print("Scan RFID");

    tft.drawRect(33, 82, 64, 12, WHITE);

    tft.setCursor(36, 85);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print("Settings");

    tft.drawRect(33, 114, 64, 12, WHITE);

    tft.setCursor(36, 117);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print(" <-Return");
  }
  else if (currentBar == 2) {
    tft.drawRect(33, 17, 64, 12, WHITE);

    tft.setCursor(36, 20);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print("Play PONG");

    tft.fillRect(33, 49, 64, 12, WHITE);

    tft.setCursor(36, 52);
    tft.setTextColor(BLACK);
    tft.setTextSize(1);
    tft.print("Scan RFID");

    tft.drawRect(33, 82, 64, 12, WHITE);

    tft.setCursor(36, 85);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print("Settings");

    tft.drawRect(33, 114, 64, 12, WHITE);

    tft.setCursor(36, 117);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print(" <-Return");
  }
  else if (currentBar == 3) {
    tft.drawRect(33, 17, 64, 12, WHITE);

    tft.setCursor(36, 20);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print("Play PONG");

    tft.drawRect(33, 49, 64, 12, WHITE);

    tft.setCursor(36, 52);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print("Scan RFID");

    tft.fillRect(33, 82, 64, 12, WHITE);

    tft.setCursor(36, 85);
    tft.setTextColor(BLACK);
    tft.setTextSize(1);
    tft.print("Settings");

    tft.drawRect(33, 114, 64, 12, WHITE);

    tft.setCursor(36, 117);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print(" <-Return");
  }
  else if (currentBar == 4) {
    tft.drawRect(33, 17, 64, 12, WHITE);

    tft.setCursor(36, 20);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print("Play PONG");

    tft.drawRect(33, 49, 64, 12, WHITE);

    tft.setCursor(36, 52);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print("Scan RFID");

    tft.drawRect(33, 82, 64, 12, WHITE);

    tft.setCursor(36, 85);
    tft.setTextColor(WHITE);
    tft.setTextSize(1);
    tft.print("Settings");

    tft.fillRect(33, 114, 64, 12, WHITE);

    tft.setCursor(36, 117);
    tft.setTextColor(BLACK);
    tft.setTextSize(1);
    tft.print(" <-Return");
  }

  if (digitalRead(button) == 1) {
    if (currentBar == 1) {
      enablePong = true;
    }
    else if (currentBar == 2) {
      enableRFID = true;
    }
    else if (currentBar == 3) {
      enableSettings = true;
    }
    else if (currentBar == 4) {
      enableRet = true;
    }
  }
}

void text() {
  tft.setCursor(0, 0);
  tft.setTextColor(RED);
  tft.setTextSize(3);
  tft.print("some   "); 
  tft.setTextColor(YELLOW);
  tft.print("lame    "); 
  tft.setTextColor(GREEN);
  tft.print("oled    "); 
  tft.setTextColor(CYAN);
  tft.print("text    "); 
  tft.setTextColor(WHITE);
  tft.print("here"); 
  delay(10000);

}

int isEnabled = 0;

void loop() {
  /*u8g2.clearBuffer();					// clear the internal memory
  u8g2.setFont(u8g2_font_ncenB08_tr);
  u8g2.drawStr(0, 10, "hhhh");
  u8g2.sendBuffer(); */

  // Basically just gonna press the button to switch modes
  //gyrating();
  
  //ultrasonic();

  //text();
  if (digitalRead(button) == 1) {
    if (isEnabled == 0) {
      tft.fillScreen(BLACK);
      grid();
      millis();
    }
    isEnabled = 1;
  }
  if (isEnabled == 1) {
    //cardRead();
    //bounce();
    menu();
    if (enablePong == true || enableRFID == true ||  enableSettings == true || enableRet == true) {
      isEnabled = 3;
      tft.fillScreen(BLACK);
      xPos = random(28, 100);
      yPos = random(10, 76);
      dx = 2;
      tft.setTextSize(2);
      if (enablePong == false) {
        grid();
        if (enableRFID == true) {
          tft.fillRect(0, 0, 128, 34, BLACK);
        }
      }
    }
  }
  else if (isEnabled == 3) {
    if (enablePong == true) {
      bounce();
      //snake();
    }
    else if (enableRFID == true) {
      cardRead();
    }
    else if (enableSettings == true) {
      settings();
      if (enableSetRet == true) {
        tft.fillScreen(BLACK);
        isEnabled = 1;
        enableSettings = false;
        enableSetRet = false;
      }
    }
    else if (enableRet == true) {
      /*if (gridType == 1 || gridType == 3) {
        tft.setTextColor(WHITE);
        tft.setCursor(5, 34);
        tft.setTextSize(2);
        tft.print("m.m.m.s.d.          by mighker"); // mighker's modular multifunctional specialisation device
      }
      else {
        /*tft.setTextColor(BLACK);
        tft.setCursor(5, 36);
        tft.print("m.m.m.s.d.          by mighker");
      } */
    }
    if (digitalRead(button) == 1 && enableSetRet == false) {
      enablePong = false;
      enableRFID = false;
      enableRet = false;
      enableSettings = false;
      isEnabled = 1;
      tft.fillScreen(BLACK);

      grid();
    }
  }

  Serial.println(analogRead(A1));
  //grid();
  //delay(1000000000);

}
