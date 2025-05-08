# ESP32-2432S028R Touchscreen Button Test

## 📲 Descrizione
Questo progetto dimostra come utilizzare il display integrato touchscreen della scheda ESP32-2432S028R (Cheap Yellow Display) per creare pulsanti interattivi in stile smartphone. Include la gestione dell'inattività per risparmio energetico e l'accensione del display al tocco.

## 🧰 Requisiti
- Scheda ESP32-2432S028R
- Libreria [TFT_eSPI](https://github.com/Bodmer/TFT_eSPI)
- Libreria [XPT2046_Touchscreen](https://github.com/PaulStoffregen/XPT2046_Touchscreen)
- Impostazioni personalizzate nel file `User_Setup.h` di TFT_eSPI come da guida:
  👉 https://randomnerdtutorials.com/cheap-yellow-display-esp32-2432s028r/

## ⚙️ Funzionalità
- Pulsanti "ON" e "OFF" con grafica arrotondata
- Rilevamento touch
- Timeout: spegne il display dopo 30 secondi di inattività
- Riattivazione al tocco

## ✅ Stato del progetto
✔️ Test iniziale completato con successo  
✔️ Display si spegne dopo inattività e si riattiva al tocco  
📌 Prossimo obiettivo: sviluppare un’interfaccia grafica con pulsanti stile smartphone, utilizzando forme, colori ed effetti più avanzati (es. con LVGL o TFT_eSPI)  
📌 Aggiunta menu multi-schermata  
📌 Inizio sviluppo **schermata home** per il progetto di **monitoraggio gas**

## 📂 Codice
```cpp
#include <SPI.h>
#include <TFT_eSPI.h>
#include <XPT2046_Touchscreen.h>

TFT_eSPI tft = TFT_eSPI();

#define XPT2046_IRQ 36
#define XPT2046_MOSI 32
#define XPT2046_MISO 39
#define XPT2046_CLK 25
#define XPT2046_CS 33

SPIClass touchscreenSPI = SPIClass(VSPI);
XPT2046_Touchscreen touchscreen(XPT2046_CS, XPT2046_IRQ);

#define SCREEN_WIDTH 320
#define SCREEN_HEIGHT 240
#define FONT_SIZE 2

int x, y, z;
unsigned long lastTouchTime = 0;
const unsigned long SCREEN_TIMEOUT = 30000; // 30 secondi
bool screenOn = true;

#define BTN_WIDTH 100
#define BTN_HEIGHT 60
#define BTN_Y 150
#define BTN_ON_X 40
#define BTN_OFF_X 180

void drawRoundedButton(int x, int y, int w, int h, const char* label, uint16_t color, uint16_t textColor) {
  tft.fillRoundRect(x, y, w, h, 10, color);
  tft.drawRoundRect(x, y, w, h, 10, TFT_WHITE);
  tft.setTextColor(textColor, color);
  tft.drawCentreString(label, x + w / 2, y + h / 2 - 8, FONT_SIZE);
}

void drawButtons() {
  drawRoundedButton(BTN_ON_X, BTN_Y, BTN_WIDTH, BTN_HEIGHT, "ON", TFT_GREEN, TFT_BLACK);
  drawRoundedButton(BTN_OFF_X, BTN_Y, BTN_WIDTH, BTN_HEIGHT, "OFF", TFT_RED, TFT_WHITE);
}

void showPressedButton(const char* label) {
  tft.fillRect(0, 40, SCREEN_WIDTH, 30, TFT_BLACK);
  tft.setTextColor(TFT_YELLOW, TFT_BLACK);
  tft.drawCentreString(String("Hai premuto: ") + label, SCREEN_WIDTH / 2, 40, FONT_SIZE);
}

void turnOffDisplay() {
  tft.fillScreen(TFT_BLACK);
  screenOn = false;
}

void turnOnDisplay() {
  tft.fillScreen(TFT_BLACK);
  tft.setTextColor(TFT_WHITE, TFT_BLACK);
  tft.drawCentreString("Tocca un pulsante", SCREEN_WIDTH / 2, 20, FONT_SIZE);
  drawButtons();
  screenOn = true;
}

void setup() {
  Serial.begin(115200);

  touchscreenSPI.begin(XPT2046_CLK, XPT2046_MISO, XPT2046_MOSI, XPT2046_CS);
  touchscreen.begin(touchscreenSPI);
  touchscreen.setRotation(1);

  tft.init();
  tft.setRotation(1);

  turnOnDisplay();
  lastTouchTime = millis();
}

void loop() {
  if (touchscreen.tirqTouched() && touchscreen.touched()) {
    TS_Point p = touchscreen.getPoint();
    x = map(p.x, 200, 3700, 1, SCREEN_WIDTH);
    y = map(p.y, 240, 3800, 1, SCREEN_HEIGHT);
    z = p.z;

    Serial.printf("X=%d, Y=%d, Z=%d\n", x, y, z);

    if (!screenOn) {
      turnOnDisplay();
    } else {
      if (x > BTN_ON_X && x < BTN_ON_X + BTN_WIDTH && y > BTN_Y && y < BTN_Y + BTN_HEIGHT) {
        showPressedButton("ON");
      } else if (x > BTN_OFF_X && x < BTN_OFF_X + BTN_WIDTH && y > BTN_Y && y < BTN_Y + BTN_HEIGHT) {
        showPressedButton("OFF");
      }
    }
    lastTouchTime = millis();
    delay(200);
  }

  if (screenOn && millis() - lastTouchTime > SCREEN_TIMEOUT) {
    turnOffDisplay();
  }
}
```

## 🔗 Risorse utili
- Guida completa: [Cheap Yellow Display ESP32-2432S028R - Random Nerd Tutorials](https://randomnerdtutorials.com/cheap-yellow-display-esp32-2432s028r/)
