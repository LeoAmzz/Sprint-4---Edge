# Sprint-4---Edge

# Descrição:
O projeto apresenta uma solução inovadora para tornar os exames de sangue pediátricos mais reconfortantes, envolvendo o uso de óculos de realidade virtual e um sistema baseado em Arduino. Enquanto as crianças exploram um mundo imaginário dentro da realidade virtual, o Arduino monitora seus sinais vitais (nesse projeto simulamos um sensor de sinais vitais atraves de um potenciometro) e ativa estímulos calmantes, como luzes suaves e música tranquilizadora, caso detecte sinais de ansiedade. Essa abordagem visa proporcionar uma experiência mais positiva e menos estressante para as crianças durante os procedimentos médicos, promovendo um ambiente hospitalar mais acolhedor e reconfortante.

# Link Wokwi:
https://wokwi.com/projects/398791637916504065

# Código
#include <WiFi.h>
#include <HTTPClient.h>
#include "pitches.h"

#define NOTE_B0  31
#define NOTE_C1  33
#define NOTE_CS1 35
#define NOTE_D1  37
#define NOTE_DS1 39
#define NOTE_E1  41
#define NOTE_F1  44
#define NOTE_FS1 46
#define NOTE_G1  49
#define NOTE_GS1 52
#define NOTE_A1  55
#define NOTE_AS1 58
#define NOTE_B1  62
#define NOTE_C2  65
#define NOTE_CS2 69
#define NOTE_D2  73
#define NOTE_DS2 78
#define NOTE_E2  82
#define NOTE_F2  87
#define NOTE_FS2 93
#define NOTE_G2  98
#define NOTE_GS2 104
#define NOTE_A2  110
#define NOTE_AS2 117
#define NOTE_B2  123
#define NOTE_C3  131
#define NOTE_CS3 139
#define NOTE_D3  147
#define NOTE_DS3 156
#define NOTE_E3  165
#define NOTE_F3  175
#define NOTE_FS3 185
#define NOTE_G3  196
#define NOTE_GS3 208
#define NOTE_A3  220
#define NOTE_AS3 233
#define NOTE_B3  247
#define NOTE_C4  262
#define NOTE_CS4 277
#define NOTE_D4  294
#define NOTE_DS4 311
#define NOTE_E4  330
#define NOTE_F4  349
#define NOTE_FS4 370
#define NOTE_G4  392
#define NOTE_GS4 415
#define NOTE_A4  440
#define NOTE_AS4 466
#define NOTE_B4  494
#define NOTE_C5  523
#define NOTE_CS5 554
#define NOTE_D5  587
#define NOTE_DS5 622
#define NOTE_E5  659
#define NOTE_F5  698
#define NOTE_FS5 740
#define NOTE_G5  784
#define NOTE_GS5 831
#define NOTE_A5  880
#define NOTE_AS5 932
#define NOTE_B5  988
#define NOTE_C6  1047
#define NOTE_CS6 1109
#define NOTE_D6  1175
#define NOTE_DS6 1245
#define NOTE_E6  1319
#define NOTE_F6  1397
#define NOTE_FS6 1480
#define NOTE_G6  1568
#define NOTE_GS6 1661
#define NOTE_A6  1760
#define NOTE_AS6 1865
#define NOTE_B6  1976
#define NOTE_C7  2093
#define NOTE_CS7 2217
#define NOTE_D7  2349
#define NOTE_DS7 2489
#define NOTE_E7  2637
#define NOTE_F7  2794
#define NOTE_FS7 2960


#define ssid "Wokwi-GUEST"
#define password ""
const char* tago_token = "848781a1-2ed5-4616-b6c6-7bd564f31062";

// Pinos do LED RGB
int red_led = 25;
int green_led = 26;
int blue_led = 27;

// Pino do potenciômetro
const int potentiometerPin = 34;

// Pino do buzzer
const int buzzerPin = 14;

// Pino do LED de controle
const int controlLedPin = 2;

int tempo = 114;

int melody[] = {
  NOTE_E4, 4, NOTE_E4, 4, NOTE_F4, 4, NOTE_G4, 4, // 1
  NOTE_G4, 4, NOTE_F4, 4, NOTE_E4, 4, NOTE_D4, 4,
  NOTE_C4, 4, NOTE_C4, 4, NOTE_D4, 4, NOTE_E4, 4,
  NOTE_E4, -4, NOTE_D4, 8, NOTE_D4, 2,
  NOTE_E4, 4, NOTE_E4, 4, NOTE_F4, 4, NOTE_G4, 4, // 4
  NOTE_G4, 4, NOTE_F4, 4, NOTE_E4, 4, NOTE_D4, 4,
  NOTE_C4, 4, NOTE_C4, 4, NOTE_D4, 4, NOTE_E4, 4,
  NOTE_D4, -4, NOTE_C4, 8, NOTE_C4, 2,
  NOTE_D4, 4, NOTE_D4, 4, NOTE_E4, 4, NOTE_C4, 4, // 8
  NOTE_D4, 4, NOTE_E4, 8, NOTE_F4, 8, NOTE_E4, 4, NOTE_C4, 4,
  NOTE_D4, 4, NOTE_E4, 8, NOTE_F4, 8, NOTE_E4, 4, NOTE_D4, 4,
  NOTE_C4, 4, NOTE_D4, 4, NOTE_G3, 2,
  NOTE_E4, 4, NOTE_E4, 4, NOTE_F4, 4, NOTE_G4, 4, // 12
  NOTE_G4, 4, NOTE_F4, 4, NOTE_E4, 4, NOTE_D4, 4,
  NOTE_C4, 4, NOTE_C4, 4, NOTE_D4, 4, NOTE_E4, 4,
  NOTE_D4, -4, NOTE_C4, 8, NOTE_C4, 2
};

int notes = sizeof(melody) / sizeof(melody[0]) / 2;
int wholenote = (60000 * 4) / tempo;

void setup() {
  // Configuração dos pinos de saída
  pinMode(controlLedPin, OUTPUT);
  pinMode(red_led, OUTPUT);
  pinMode(green_led, OUTPUT);
  pinMode(blue_led, OUTPUT);

  // Configuração do pino de entrada
  pinMode(potentiometerPin, INPUT);

  // Configuração do canal PWM para o buzzer
  ledcSetup(0, 2000, 8); // Canal 0, 2kHz, 8-bit resolution
  ledcAttachPin(buzzerPin, 0);

  // Conectar ao Wi-Fi
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");
}

void loop() {
  // Leitura do potenciômetro
  int potentiometerValue = analogRead(potentiometerPin);

  // Controle do LED RGB baseado no valor do potenciômetro
  if (potentiometerValue > 800) {
    digitalWrite(controlLedPin, HIGH);  // Ativa o LED de controle
    analogWrite(red_led, 255);          // LED vermelho
    analogWrite(green_led, 0);
    analogWrite(blue_led, 0);

    // Enviar dados para TagoIO
    sendDataToTagoIO(potentiometerValue, "HIGH");
  } else {
    digitalWrite(controlLedPin, LOW);   // Desativa o LED de controle
    analogWrite(red_led, 0);
    analogWrite(green_led, 255);        // LED verde
    analogWrite(blue_led, 0);

    // Enviar dados para TagoIO
    sendDataToTagoIO(potentiometerValue, "LOW");
  }

  // Tocar a melodia se o LED de controle estiver aceso
  if (digitalRead(controlLedPin) == HIGH) {
    playMelody();
  }

  delay(10);  // Pequeno atraso para estabilidade
}

void sendDataToTagoIO(int value, String status) {
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;
    http.begin("https://api.tago.io/data");
    http.addHeader("Content-Type", "application/json");
    http.addHeader("Device-Token", tago_token);

    String payload = "{\"variable\":\"potentiometer\",\"value\":" + String(value) + ",\"status\":\"" + status + "\"}";

    int httpResponseCode = http.POST(payload);
    if (httpResponseCode > 0) {
      String response = http.getString();
      Serial.println(httpResponseCode);
      Serial.println(response);
    } else {
      Serial.print("Error on sending POST: ");
      Serial.println(httpResponseCode);
    }
    http.end();
  } else {
    Serial.println("Error in WiFi connection");
  }
}

void playMelody() {
  for (int thisNote = 0; thisNote < notes * 2; thisNote += 2) {
    int divider = melody[thisNote + 1];
    int noteDuration = divider > 0 ? wholenote / divider : wholenote / abs(divider) * 1.5;

    ledcWriteTone(0, melody[thisNote]);  // Inicia a nota
    delay(noteDuration * 0.9);           // Toca a nota
    ledcWriteTone(0, 0);                 // Para a nota
    delay(noteDuration * 0.1);           // Pequena pausa entre notas
  }
}

