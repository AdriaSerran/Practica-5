# Pràctica-5 BUSES DE COMUNICACIÓN I

L'objectiu d'aquesta pràctica és entendre el funcionament dels busos sistemes de comunicació entre perifèrics.

Penso que ha estat una pràctica relativament senzilla però amb un resultat molt bo ja que s'han introduït nous materials els quals aporten bastant visualment.

# Material

- Display OLED i2c SSD1306
- Sensor temperatura HTU21D
- ESP32

# Funcionament

El que fa aquest codi és enviar la temperatura i la humitat ambiental al display després d'haver obtingut aquesta informaciño gràcies a l'ajuda del sensor de temperatura i humitat.

Un cop fet tots els includes

```c++
#include <Arduino.h>
#include "SSD1306Wire.h"
#include <Wire.h>
#include "images.h"
#include "SparkFunHTU21D.h"
```

Creem una instància de l'objecte i inicialitzem el display:

```c++
HTU21D myHumidity;
SSD1306Wire display(0x3c, SDA, SCL);
```

A continuació tenim el void setup el qual ens ho iniacilitzarà tot juntament amb bastants voids els quals ens permetran estructurar el text al nostre display:

```c++
#define DEMO_DURATION 3000
typedef void (*Demo)(void);

int demoMode = 0;
int counter = 1;

void setup() {
  Serial.begin(115200);
  Serial.println();
  Serial.println();

   Serial.println("HTU21D Example!");

   myHumidity.begin();
//}

  //init_temp_hum_task();
  // Initialising the UI will init the display too.
  display.init();

  display.flipScreenVertically();
  display.setFont(ArialMT_Plain_10);

}

void drawFontFaceDemo() {
  // Font Demo1
  // create more fonts at http://oleddisplay.squix.ch/
  display.setTextAlignment(TEXT_ALIGN_LEFT);
  display.setFont(ArialMT_Plain_10);
  display.drawString(0, 0, "Hello world");
  display.setFont(ArialMT_Plain_16);
  display.drawString(0, 10, "Hello world");
  display.setFont(ArialMT_Plain_24);
  display.drawString(0, 26, "Hello world");
}

void drawTextFlowDemo() {
  display.setFont(ArialMT_Plain_10);
  display.setTextAlignment(TEXT_ALIGN_LEFT);
  display.drawStringMaxWidth(0, 0, 128,
                             "Lorem ipsum\n dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore." );
}

void drawTextAlignmentDemo() {
  // Text alignment demo
  display.setFont(ArialMT_Plain_10);

  // The coordinates define the left starting point of the text
  display.setTextAlignment(TEXT_ALIGN_LEFT);
  display.drawString(0, 10, "Left aligned (0,10)");

  // The coordinates define the center of the text
  display.setTextAlignment(TEXT_ALIGN_CENTER);
  display.drawString(64, 22, "Center aligned (64,22)");

  // The coordinates define the right end of the text
  display.setTextAlignment(TEXT_ALIGN_RIGHT);
  display.drawString(128, 33, "Right aligned (128,33)");
}

void drawRectDemo() {
  // Draw a pixel at given position
  for (int i = 0; i < 10; i++) {
    display.setPixel(i, i);
    display.setPixel(10 - i, i);
  }
  display.drawRect(12, 12, 20, 20);

  // Fill the rectangle
  display.fillRect(14, 14, 17, 17);

  // Draw a line horizontally
  display.drawHorizontalLine(0, 40, 20);

  // Draw a line horizontally
  display.drawVerticalLine(40, 0, 20);
}

void drawCircleDemo() {
  for (int i = 1; i < 8; i++) {
    display.setColor(WHITE);
    display.drawCircle(32, 32, i * 3);
    if (i % 2 == 0) {
      display.setColor(BLACK);
    }
    display.fillCircle(96, 32, 32 - i * 3);
  }
}

void drawProgressBarDemo() {
  int progress = (counter / 5) % 100;
  // draw the progress bar
  display.drawProgressBar(0, 32, 120, 10, progress);

  // draw the percentage as String
  display.setTextAlignment(TEXT_ALIGN_CENTER);
  display.drawString(64, 15, String(progress) + "%");
}

void drawImageDemo() {
  // see http://blog.squix.org/2015/05/esp8266-nodemcu-how-to-create-xbm.html
  // on how to create xbm files
  display.drawXbm(34, 14, WiFi_Logo_width, WiFi_Logo_height, WiFi_Logo_bits);
}

Demo demos[] = {drawFontFaceDemo, drawTextFlowDemo, drawTextAlignmentDemo, drawRectDemo, drawCircleDemo, drawProgressBarDemo, drawImageDemo};
int demoLength = (sizeof(demos) / sizeof(Demo));
long timeSinceLastModeSwitch = 0;
```

I per últim entrem al loop, el qual veiem com utilitza funcions que ens deuen proporcionar algunes llibraries ja incloses al nostre projecte per a 
llegir tant la humitat com la temperatura, myHumidity.readHumidity() i myHumidity.readTemperature() com podem veure a continuació.

```c++
void loop() {
  float humd = myHumidity.readHumidity();
  float temp = myHumidity.readTemperature();
```

Tot seguit veiem una sèrie de Serial.print(), aquesta és la informació que sortirà per la pantalla del nostre ordinador

```c++
  Serial.print("Time:");
  Serial.print(millis());
  Serial.print(" Temperature:");
  Serial.print(temp, 1);
  Serial.print("C");
  Serial.print(" Humidity:");
  Serial.print(humd, 1);
  Serial.print("%");

  Serial.println();
```

A continuacio veiem tot una sèrie de display.funcions(), a través d'aquestes enviarem la informació del que volem escriure a través del nostre display.

```c++
display.setTextAlignment(TEXT_ALIGN_CENTER);
  display.setFont(ArialMT_Plain_10);
  display.drawString(128/2, 0, "HUMEDAD");
  display.setFont(ArialMT_Plain_16);
  display.drawString(128/2, 11, String(humd)+ "%");
  display.setFont(ArialMT_Plain_10);
  display.drawString(128/2, 30, "TEMPERATURA");
  display.setFont(ArialMT_Plain_16);
  display.drawString(128/2, 41, String(temp)+ "ºC");

  display.setFont(ArialMT_Plain_10);
  display.setTextAlignment(TEXT_ALIGN_RIGHT);
  display.drawString(128, 54, String(millis()/3600000)+String(":")\
          +String((millis()/60000)%60)+String(":")\
          +String((millis()/1000)%(60)));

  display.display();
  delay(100);
}
```

Per últim veiem el delay amb el qual es realitzarà aquest loop, 100 ms, és a dir, cada 100 ms obtindrem nova informació tant per la pantalla del nostre ordinador com
a través del display.
