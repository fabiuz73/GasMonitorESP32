# GasMonitorESP3

# ESP32 Touchscreen Buttons Demo (ESP32-2432S028R)


🔗 [Scheda tecnica e guida dettagliata - Random Nerd Tutorials](https://randomnerdtutorials.com/cheap-yellow-display-esp32-2432s028r/)

---

## 🧰 Hardware utilizzato

- ✅ ESP32-2432S028R (2.8" TFT 240x320)
- ✅ Display ILI9341 SPI
- ✅ Touchscreen resistivo XPT2046 (già cablato via SPI sulla scheda)

---

## 🎯 Obiettivo

Visualizzare due pulsanti (ON/OFF) sullo schermo. Quando l'utente tocca uno dei due pulsanti, il display mostra quale pulsante è stato premuto e stampa le coordinate sul monitor seriale.

---

## 📦 Librerie richieste

Installa queste librerie tramite il Library Manager di Arduino IDE:

- [`TFT_eSPI`](https://github.com/Bodmer/TFT_eSPI)
- [`XPT2046_Touchscreen`](https://github.com/PaulStoffregen/XPT2046_Touchscreen)

> ⚠️ Per `TFT_eSPI`, **configura il file `User_Setup.h`** per la tua scheda seguendo questa guida:
> https://randomnerdtutorials.com/cheap-yellow-display-esp32-2432s028r/

---

## 🖥️ Sketch di test

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

// Coordinate dei pulsanti
#define BTN_WIDTH 100
#define BTN_HEIGHT 60
#define BTN_Y 150
#define BTN_ON_X 40
#define BTN_OFF_X 180

void drawButtons() {
  tft.fillRect(BTN_ON_X, BTN_Y, BTN_WIDTH, BTN_HEIGHT, TFT_GREEN);
  tft.setTextColor(TFT_BLACK);
  tft.drawCentreString("ON", BTN_ON_X + BTN_WIDTH / 2, BTN_Y + BTN_HEIGHT / 2 - 8, FONT_SIZE);

  tft.fillRect(BTN_OFF_X, BTN_Y, BTN_WIDTH, BTN_HEIGHT, TFT_RED);
  tft.setTextColor(TFT_WHITE);
  tft.drawCentreString("OFF", BTN_OFF_X + BTN_WIDTH / 2, BTN_Y + BTN_HEIGHT / 2 - 8, FONT_SIZE);
}

void showPressedButton(const char* label) {
  tft.fillRect(0, 40, SCREEN_WIDTH, 30, TFT_WHITE);
  tft.setTextColor(TFT_BLACK);
  tft.drawCentreString(String("Hai premuto: ") + label, SCREEN_WIDTH / 2, 40, FONT_SIZE);
}

void setup() {
  Serial.begin(115200);

  touchscreenSPI.begin(XPT2046_CLK, XPT2046_MISO, XPT2046_MOSI, XPT2046_CS);
  touchscreen.begin(touchscreenSPI);
  touchscreen.setRotation(1);

  tft.init();
  tft.setRotation(1);

  tft.fillScreen(TFT_WHITE);
  tft.setTextColor(TFT_BLACK, TFT_WHITE);
  tft.drawCentreString("Tocca un pulsante", SCREEN_WIDTH / 2, 20, FONT_SIZE);

  drawButtons();
}

void loop() {
  if (touchscreen.tirqTouched() && touchscreen.touched()) {
    TS_Point p = touchscreen.getPoint();
    x = map(p.x, 200, 3700, 1, SCREEN_WIDTH);
    y = map(p.y, 240, 3800, 1, SCREEN_HEIGHT);
    z = p.z;

    Serial.printf("X=%d, Y=%d, Z=%d\n", x, y, z);

    if (x > BTN_ON_X && x < BTN_ON_X + BTN_WIDTH && y > BTN_Y && y < BTN_Y + BTN_HEIGHT) {
      showPressedButton("ON");
    } else if (x > BTN_OFF_X && x < BTN_OFF_X + BTN_WIDTH && y > BTN_Y && y < BTN_Y + BTN_HEIGHT) {
      showPressedButton("OFF");
    }

    delay(200);  // debounce touch
  }
}
```



---

## ✅ Stato del progetto

✔️ Test iniziale completato con successo  
📌 Prossimo obiettivo: sviluppare un’interfaccia grafica con pulsanti stile smartphone, utilizzando forme, colori ed effetti più avanzati (es. con **LVGL** o **TFT_eSPI**).  
📌 Aggiunta menu multi-schermata

---

## 📄 Licenza

MIT – Usa liberamente per scopi didattici e personali.
