# Práctica 2A – Interrupción por GPIO en ESP32

## Descripción

Este repositorio contiene la **Práctica 2A: Interrupción por GPIO**, realizada para la asignatura **Procesadores Digitales**.

El objetivo principal de la práctica es comprender el funcionamiento de las **interrupciones hardware** en una placa **ESP32**, utilizando un pin GPIO para detectar pulsaciones simuladas mediante un cable jumper. Cada interrupción incrementa un contador y muestra el resultado por el monitor serial. Además, la interrupción se desconecta automáticamente después de 1 minuto.

## Autor

**Marco Carrasco Carmona**  
Fecha: **10 de marzo de 2026**

## Objetivo de la práctica

Comprender cómo funcionan las interrupciones hardware en el ESP32 mediante el uso de la función `attachInterrupt()`, detectando cambios en un pin GPIO y ejecutando una rutina de interrupción o ISR cuando se produce el evento.

## Materiales utilizados

- Placa ESP32 DevKit
- Cable jumper
- Ordenador con VS Code
- PlatformIO
- Monitor serial a 115200 baudios

## Montaje

El montaje utilizado es sencillo:

- El pin **GPIO 18** se configura como entrada con resistencia interna pull-up mediante `INPUT_PULLUP`.
- La pulsación se simula conectando momentáneamente el pin **18** a **GND** con un cable jumper.
- No se utiliza botón físico; se sustituye por cables Arduino.

## Funcionamiento

El programa configura una interrupción en el pin GPIO 18.  
Cuando el pin pasa de estado HIGH a LOW, se dispara la interrupción en modo `FALLING`.

La función ISR realiza dos acciones:

1. Incrementa el contador de pulsaciones.
2. Activa una bandera booleana llamada `pressed`.

Después, en el `loop()`, si la bandera está activa, se muestra por el monitor serial el número de veces que se ha detectado la pulsación.

Tras 60 segundos, la interrupción se desactiva mediante `detachInterrupt()`.

## Código principal

```cpp
#include <Arduino.h>

struct Button {
  const uint8_t PIN;
  uint32_t numberKeyPresses;
  bool pressed;
};

Button button1 = {18, 0, false};

void IRAM_ATTR isr() {
  button1.numberKeyPresses += 1;
  button1.pressed = true;
}

void setup() {
  Serial.begin(115200);
  pinMode(button1.PIN, INPUT_PULLUP);
  attachInterrupt(button1.PIN, isr, FALLING);
}

void loop() {
  if (button1.pressed) {
    Serial.printf("Button 1 has been pressed %u times\n",
                  button1.numberKeyPresses);
    button1.pressed = false;
  }

  static uint32_t lastMillis = 0;
  if (millis() - lastMillis > 60000) {
    lastMillis = millis();
    detachInterrupt(button1.PIN);

Salida esperada:

Button 1 has been pressed 1 times
Button 1 has been pressed 2 times
Button 1 has been pressed 3 times
Button 1 has been pressed 4 times
Button 1 has been pressed 5 times
Interrupt Detached!

Conceptos trabajados
Interrupciones hardware
Uso de attachInterrupt()
Uso de detachInterrupt()
Rutinas de servicio de interrupción, ISR
Configuración de pines con INPUT_PULLUP
Detección de flanco de bajada con FALLING
Uso de IRAM_ATTR
Comunicación por monitor serial
Ventajas frente al polling
Resultados

La interrupción respondió correctamente a las pulsaciones simuladas con el cable jumper. Cada conexión momentánea entre el pin 18 y GND generó una interrupción, incrementando el contador y mostrando el número de pulsaciones por pantalla.

Después de un minuto de ejecución, la interrupción se desconectó correctamente y apareció el mensaje:

Interrupt Detached!
Observaciones
La respuesta de la interrupción fue inmediata.
No se observaron rebotes significativos durante la simulación con cable.
El uso de una estructura struct Button permite organizar mejor la información asociada al botón.
El programa funcionó correctamente tras la compilación y subida a la placa.
Conclusión

Esta práctica demuestra la utilidad de las interrupciones hardware frente al polling. Gracias a las interrupciones, el procesador no necesita comprobar constantemente el estado del pin, sino que reacciona únicamente cuando ocurre el evento.

Se ha comprobado el funcionamiento correcto de attachInterrupt(), detachInterrupt() y de una ISR en el entorno ESP32.
    Serial.println("Interrupt Detached!");
  }
}
