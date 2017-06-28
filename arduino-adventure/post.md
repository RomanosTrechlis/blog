
I recently bought an Arduino Mega 2560 with a Ramps 1.4 shield the corresponding stepper motor drivers and an LCD screen. The ultimate goal is to build my first 3d printer using spare parts and then use that printer to make parts for a better one.

I try to build it step by step in order to understand how it works. The first thing to try is of course to connect everything together and make the LCD screen print something.

There are a lot of tutorials that explain how to do that, but I couldn't find any that used a wiring with Ramps 1.4. I thing this is due to firmware like Marlin that handles that wiring.

## Parts

For this I used the following parts:

+ Arduino Mega 2560
+ Ramps 1.4
+ LCD adaptor
+ LCD screen

![All parts connected][wired]

## Finding out the pins

The tricky part was to find which pins where used, and for this I needed to see the schematics of Ramps 1.4 and of LCD adaptor.

After wiring everything together I found out that the part of Ramps 1.4 that interest me was the AUX3 and AUX4. AUX3 is the 4x2 part and AUX4 is the 18x1 part.

![Pins for adaptor][adaptor_pins]

Now the AUX3 as far as I understand is for the SD card slot of LCD screen so I won't deal with it right now.

The AUX4 part however connects the digital pins of Mega 2560 with the adaptor. In the previous image the 'D' before the number means digital. Those pins are shown in the following image.

![Mega 2560 LCD screen pins][mega2560]

I had to also find out how the pins on the LCD screen were connected to the pins of the adaptor.

On the LCD screen there are 16 pins that can be connected to the adaptor by two 5x2 arrangement of pins called EXP1 and EXP2. The 16 pins of screen are as follows:

1. VSS (Ground)
2. VDD (5V)
3. VE (Contrast Voltage)
4. RS (Register Select)
5. R/W (Read/Write)
6. E (Enable)
7. Data 0-7
8. Led+
9. Led-

The next image shows the pins arranged in EXP1 and EXP2, and how they are wired on the 18x1 pins of Ramps 1.4.

![Adaptor to Ramps 1.4][exp_adaptor]

The pins needed to write something on the screen are the RS, E, and Data 4-7. The mapping with Mega 2650 is:
+ D16 --> RS
+ D17 --> E
+ D23 --> Data 5
+ D25 --> Data 6
+ D27 --> Data 7
+ D29 --> Data 8

There are several tutorials that explain why these pins are needed.


## Coding

The LiquitCrystal library provides the functionality to write on LCD screen. The key part is to declare the correct pins on initialization.

```cpp
#include "Arduino.h"
#include <LiquidCrystal.h>

#define LCD_RS 16
#define LCD_E 17
#define LCD_5 23
#define LCD_6 25
#define LCD_7 27
#define LCD_8 29

LiquidCrystal lcd(LCD_RS, LCD_E, LCD_5, LCD_6, LCD_7, LCD_8);

void setup() {
  pinMode(LED_BUILTIN, OUTPUT);

  // set up the LCD's number of columns and rows:
  lcd.begin(20, 4);
  // Print a message to the LCD.
  lcd.print("Hello, world!");
  lcd.setCursor(0, 1);
  lcd.print("This is a test.");

}

void loop() {
  // I keep the built-in led blinking
  digitalWrite(LED_BUILTIN, HIGH);
  delay(1000);
  digitalWrite(LED_BUILTIN, LOW);
  delay(1000);

  // set cursor to the last row
  lcd.setCursor(0, 3);
  // prints the number of cycle passed from reset
  lcd.print(millis()/1000);
}
```

Finally the result is shown in the following image.

![Final result][result]


[wired]: images/wired.jpg
[adaptor_pins]: images/adaptor_pins.png
[mega2560]: images/mega_pins.png
[exp_adaptor]: images/exp+adaptor.png
[result]: images/result.jpg
