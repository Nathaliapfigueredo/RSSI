# Monitoramento de RSSI com ESP32 e Adafruit IO

Este projeto utiliza um ESP32 para monitorar continuamente a intensidade do sinal Wi-Fi (RSSI) e enviar os dados para a plataforma Adafruit IO usando o protocolo MQTT.  

## Objetivo

- Ler periodicamente o RSSI do ESP32  
- Enviar os valores para um *feed* no Adafruit IO  
- Evitar exceder os limites de taxa de dados do serviço  
- Enviar apenas quando houver mudanças no valor lido  

## Componentes Utilizados

### **Hardware**
- ESP32
- Antena
- Rede Wi-Fi

### **Software**
- Arduino IDE  
- Biblioteca `WiFi.h`
- Biblioteca `AdafruitIO_WiFi.h`
- Conta no Adafruit IO


## Arquitetura do Sistema

1. O ESP32 conecta ao Wi-Fi.
2. Conecta ao Adafruit IO via MQTT.
3. A cada 3 segundos:
   - Lê o RSSI
   - Verifica se o valor mudou
   - Envia ao feed somente se houve alteração

## Código Completo

```cpp
#include <WiFi.h>
#include "AdafruitIO_WiFi.h"

#define IO_USERNAME  "USUÁRIO"
#define IO_KEY       "CHAVE"

#define WIFI_SSID    "Inteli.Iot"
#define WIFI_PASS    "SENHA"

AdafruitIO_WiFi io(IO_USERNAME, IO_KEY, WIFI_SSID, WIFI_PASS);

// Feed RSSI
AdafruitIO_Feed *rssiFeed = io.feed("rssi");

// Enviar a cada 3s
const unsigned long PUBLISH_INTERVAL_MS = 3000;
unsigned long lastPublish = 0;

int32_t last_rssi = 9999;

void setup_wifi() {
  Serial.print("\nConectando a: ");
  Serial.println(WIFI_SSID);

  WiFi.mode(WIFI_STA);
  WiFi.begin(WIFI_SSID, WIFI_PASS);

  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(500);
  }

  Serial.println("\nWiFi conectado!");
  Serial.print("IP: ");
  Serial.println(WiFi.localIP());
}

void setup() {
  Serial.begin(115200);
  delay(200);

  setup_wifi();

  Serial.println("Conectando ao Adafruit IO...");
  io.connect();

  while (io.status() < AIO_CONNECTED) {
    Serial.print(".");
    delay(500);
  }

  Serial.println("\nConectado ao Adafruit IO!");
}

void loop() {
  io.run();

  unsigned long now = millis();
  if (now - lastPublish >= PUBLISH_INTERVAL_MS) {
    lastPublish = now;

    int32_t rssi = WiFi.RSSI();

    // Só envia se houver mudança real
    if (rssi != last_rssi) {
      rssiFeed->save(rssi);
      last_rssi = rssi;
    }

    Serial.print("RSSI: ");
    Serial.println(rssi);
  }
}
```

## Limites do Adafruit IO

Plano gratuito permite:
- 30 mensagens/minuto
- 10 feeds
- 30 requisições/segundo no máximo

Para evitar throttling foi configurado o intervalo 3s (20 msgs/min) e envio apenas quando RSSI muda.

## Resultado

[Aqui está o vídeo do sistema funcionando e registrando a mudança de sinal ao redor do campus da faculdade.](https://drive.google.com/file/d/15xrgWLATIy0e90wcCX7ngIciFaTKR9D9/view?usp=drivesdk)

O projeto demonstrou com sucesso a capacidade do ESP32 de monitorar continuamente a intensidade do sinal Wi-Fi (RSSI) e enviar esses dados para a plataforma Adafruit IO utilizando o protocolo MQTT. Após ajustes no intervalo de transmissão e na lógica de envio, o sistema passou a operar dentro dos limites de taxa do Adafruit IO, garantindo estabilidade e evitando throttling.

Além disso, a integração com o dashboard permitiu visualizar em tempo real as oscilações do sinal, evidenciando como ambientes físicos — como o elevador utilizado nos testes — podem impactar drasticamente a recepção de radiofrequência. O experimento reforça a importância do RSSI como métrica em aplicações IoT e demonstra um fluxo completo de coleta, transmissão e visualização de dados em nuvem.


## Autoria: Nathalia Figueredo

