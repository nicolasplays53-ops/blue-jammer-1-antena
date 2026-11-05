# blue-jammer-1-antena

Projeto simples de jammer usando ESP32 + NRF24L01.  
Aqui tem o que precisa e os códigos pra rodar.

---

### 1. Itens que você vai precisar

- 1x Placa ESP32-Dev
- 1x Módulo NRF24L01 com PA/LNA (aquele com antena mesmo)
- 1x Capacitor de 15uF até 100uF (pode ser eletrolítico)
- 1x Botão (ou um switch se quiser deixar ligado direto)

---

### 2. Como conectar as coisas

Liga o NRF24L01 no ESP32 assim:

- VCC → 3.3V (não coloca no 5V senão queima)
- GND → GND
- CE → GPIO 4
- CSN → GPIO 5
- SCK → GPIO 18
- MOSI → GPIO 23
- MISO → GPIO 19

O capacitor você coloca entre o VCC e o GND do NRF24L01 (bem perto do módulo).  
O botão você pode ligar no GPIO 0 com um resistor de pull-up interno, ou usar um switch pra deixar sempre ligado.

---

### 3. Códigos e extensões

<details>
<summary><b>📦 Código principal (cola esse no Arduino IDE)</b></summary>

```cpp
// #include "RF24.h"
#include <ezButton.h>
#include "esp_bt.h"
#include "esp_wifi.h"

SPIClass *sp = nullptr;

//HSPI=SCK = 14, MISO = 12, MOSI = 13, CS = 15 , CE = 16
//VSPI=SCK = 18, MISO =19, MOSI = 23 ,CS =21 ,CE = 22


RF24 radio(22, 21, 16000000);  ///DEFAULT 10000000

byte i = 45;  ///37-50 CHANNEL (CHANNEL NRF NEEDS TO START )

unsigned int flag = 0;


ezButton toggleSwitch(33);


void initSP() {
  sp = new SPIClass(VSPI);
  sp->begin();
  if (radio.begin(sp)) {
    delay(1000);
    Serial.println("Sp Started !!!");
    radio.setAutoAck(false);
    radio.stopListening();
    radio.setRetries(0, 0);
    radio.setPayloadSize(5);   ////SET VALUE ON RF24.CPP
    radio.setAddressWidth(3);  ////SET VALUE ON RF24.CPP
    radio.setPALevel(RF24_PA_MAX, true);
    radio.setDataRate(RF24_2MBPS);
    radio.setCRCLength(RF24_CRC_DISABLED);
    radio.printPrettyDetails();
    radio.startConstCarrier(RF24_PA_MAX, i);  ////EDITED VALUES ON LIBRARY ....SET 5 BYTES PAYLOAD SIZE//REDUCE RF24_MAX TO RF24_HIGH OR LOW IF RF NOT STABLE



  } else {
    Serial.println("SP couldn't start !!!");
  }
}
void two() {

  ///CHANNEL WITH 2 SPACING HOPPING
  if (flag == 0) {
    i += 2;
  } else {
    i -= 2;
  }

  if ((i > 79) && (flag == 0)) {
    flag = 1;
  } else if ((i < 2) && (flag == 1)) {
    flag = 0;
  }

  radio.setChannel(i);
  // Serial.println(i);
}

void one() {
  for (int i = 0; i < 15; i++) {
    radio.setChannel(i);
  }
}

// radio.setChannel(random(79));




void setup(void) {
  pinMode(33, INPUT_PULLUP);  // BUILD-IN RESISTOR
  esp_bt_controller_deinit();
  esp_wifi_stop();
  esp_wifi_deinit();
  Serial.begin(115200);
  delay(1000);
  Serial.println("Setup started...");
  toggleSwitch.setDebounceTime(50);
  initSP();
}


void loop(void) {

  toggleSwitch.loop();  // MUST call the loop() function first

  if (toggleSwitch.isPressed())
    Serial.println("one");
  if (toggleSwitch.isReleased())
    Serial.println("two");

  int state = toggleSwitch.getState();


  if (state == HIGH)
    two();

  else {
    one();
  }
}
```
</details>

<details>
<summary><b>⚙️ Bibliotecas que você precisa instalar</b></summary>

No Arduino IDE vai em:
Sketch → Include Library → Manage Libraries
E instala essas:

RF24 (do TMRh20); 
ezButton (de ArduinoGetStarde.com); 
Adafruit GFX Library (E todas suas extensões, de Adafruit).

</details>

---

### 5. Drivers e programas que você precisa ter

Antes de subir qualquer código, você precisa preparar o PC.

**1. Instalar o Arduino IDE**
- Entra no site: [https://www.arduino.cc/en/software](https://www.arduino.cc/en/software)
- Baixa a versão pro seu sistema (Windows, Linux ou Mac)
- Instala normal

**2. Adicionar a placa ESP32 no Arduino IDE**
- Abre o Arduino IDE
- Vai em **File → Preferences**
- No campo **Additional boards manager URLs** cola isso: 
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json

- Clica em OK
- Agora vai em **Tools → Board → Boards Manager**
- Pesquisa por **esp32** e instala o pacote da Espressif
- Depois seleciona a placa: **Tools → Board → ESP32 Arduino → ESP32 Dev Module**

**3. Instalar a biblioteca do NRF24**
- Vai em **Sketch → Include Library → Manage Libraries**
- Pesquisa por **RF24**
- Instala a do **TMRh20** (é a mais usada)

**4. Driver da placa (se o PC não reconhecer)**
- Se quando você conectar o ESP32 não aparecer nenhuma porta COM, baixa o driver do chip da sua placa:
  - A maioria usa **CP210x** → [driver aqui](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers)
  - Algumas usam **CH340** → pesquisa “CH340 driver” e instala

Depois de instalar tudo, seleciona a porta certa em **Tools → Port** e manda o código.

⚠️ATENÇÃO, USE COM MODERAÇÃO OU NEM USO, POIS É CRIME!!!!⚠️