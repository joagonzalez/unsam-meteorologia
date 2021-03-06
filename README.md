## Estacion Meteorológica
El proyecto consiste en la utilización del conversor analógico-digial (ADC) de la placa EDU-CIAA con el objetivo de muestrear sensores de temperatura, humedad y presión. 

Esta información será enviada al UART (universal async receiver-transmitter), el cual enviará datos de forma serial con payloads de hasta 8 bits. Como el ADC funciona con datos de 10 bits, se truncarán datos no significativos para ser enviados correctamente por el UART. A la placa EDU-CIAA, se conectará un módulo ESP8266 que recibirá por comuniación serie los datos recolectados con los sensores, esta información será transmitida via Wi-Fi a una base de datos de series de tiempo (influxDB) que luego será graficada con el objetivo de monitorear en tiempo real las variables deseadas.


![Figura 1](https://github.com/joagonzalez/unsam-meteorologia/blob/master/doc/diagrama_high_level_v3.png)


### Notas
En caso de tener error de permisos a la hora de grabar sketch en esp8266, dar permisos correspodientes al puerto sobre el que se esta trabajando

```
chmod 777 /dev/ttyUSB0
```

Para cargar librerias del board esp8266 en Arduino IDE utilizar el siguiente repositorio:
- http://arduino.esp8266.com/stable/package_esp8266com_index.json

De la documentacion de la  libreria sAPI podemos ver que UART2 = USB y UART3 = RS232

```C
 if( ( uart == UART_USB )  && (txIsrCallbackUART2 != 0) )
         (*txIsrCallbackUART2)(0);
      
      if( ( uart == UART_232 )  && (txIsrCallbackUART3 != 0) )
         (*txIsrCallbackUART3)(0);
```

Configuracion de UART en SCU
```C
   // UART not routed
   {  LPC_UART1, { 0, 0, 0     }, { 0, 0, 0     }, UART1_IRQn  }, // 2
   // UART_USB
   { LPC_USART2, { 7, 1, FUNC6 }, { 7, 2, FUNC6 }, USART2_IRQn }, // 3
   // UART_ENET
   { LPC_USART2, { 1,15, FUNC1 }, { 1,16, FUNC1 }, USART2_IRQn }, // 4
   // UART_232
   { LPC_USART3, { 2, 3, FUNC2 }, { 2, 4, FUNC2 }, USART3_IRQn }  // 5  
```

### Sensor de temperatura (KY-013)

![Figura 2](https://github.com/joagonzalez/unsam-meteorologia/blob/master/doc/sensor_temperature.jpg)

Este sensor utiliza la ecuación de Stainhart-Hart (modelo de resistencia de semiconductores para distintas temperaturas). La ecuación con los coeficientes para una resistencia de 10kohm es:

```C
R1 = 10000; // value of R1=10kohm on board
c1 = 0.001129148, c2 = 0.000234125, c3 = 0.0000000876741; //steinhart-hart coeficients for thermistor

R2 = R1 * (1023.0 / (float)Vo - 1.0); //calculate resistance on thermistor
logR2 = log(R2);
T = (1.0 / (c1 + c2*logR2 + c3*logR2*logR2*logR2)); // temperature in Kelvin
T = T - 273.15; //convert Kelvin to Celcius
```

| Description  | Value |
| ------------- | ------------- |
| Operating Voltage  | 5V  |
| Temperature measurement range  | 	-55°C to 125°C [-67°F to 257°F]  |
| Measurement Accuracy  | 	±0.5°C |
| Resistance  | 	10kohm |

Diagrama de conexión 

![Figura 3](https://github.com/joagonzalez/unsam-meteorologia/blob/master/doc/sensor_temp_ky013.png)

Una observación sobre el diagrama de conexion, esta invertido el pinout positivo/negativo. Va a haber mediciones incorrectas si no se inverte la alimentación del sensor (ver enlace adjuntado).

- https://arduinomodules.info/ky-013-analog-temperature-sensor-module/
- https://en.wikipedia.org/wiki/Steinhart%E2%80%93Hart_equation
- https://forum.arduino.cc/index.php?topic=209133.0 (documentacion inversion pinout)


### Sensor de Gas (MQ-135)
La hoja de datos del sensor se encuentra en la carpeta doc de este repositorio.

Curva para detección de distintos gases

![Figura 4](https://github.com/joagonzalez/unsam-meteorologia/blob/master/doc/sensor_gas_mq135.png)

Diagrama de conexión

![Figura 5](https://github.com/joagonzalez/unsam-meteorologia/blob/master/doc/sensor_gas.jpg)

- https://create.arduino.cc/projecthub/karimmufte/arduino-and-mq-135-gas-sensor-with-arduino-code-a8c1c6

### Montaje
![Figura 6](https://github.com/joagonzalez/unsam-telemetria/blob/refactor/doc/montaje_protoboard.jpg)

### Dashboards Grafana
![Figura 6](https://github.com/joagonzalez/unsam-telemetria/blob/refactor/doc/temperature_test_grafana.jpeg)
 
 Descarga dashboard: https://github.com/joagonzalez/unsam-telemetria/blob/refactor/doc/sensores_dashboard.json
 
### Material de consulta
- https://www.youtube.com/watch?v=G6CqvhXpBKM (programming esp8266 with arduino ide)
- https://www.youtube.com/watch?v=6-RXqFS_UtU&list=PLNFq0T6Z3JPsMWtVgmqPnpIu41LNMDPiA&index=8 (serial communication esp8266 and arduino)
- https://www.youtube.com/watch?v=8xqgdXvn3yw&list=PLNFq0T6Z3JPsHwzvPQncip-kMIdWpnnip&index=7 (api access esp8266)
- http://www.circuitbasics.com/basics-uart-communication/
- https://arduino-esp8266.readthedocs.io/en/latest/libraries.html#softwareserial
- https://www.youtube.com/playlist?list=PLNFq0T6Z3JPsHwzvPQncip-kMIdWpnnip (esp8266 projects)
- https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WebServer (webserver esp8266 arduino library)
- https://github.com/tobiasschuerg/InfluxDB-Client-for-Arduino - esp8266 influxdb library
- https://github.com/ciaa/firmware_v2/tree/master/modules/lpc4337_m4/sapi - sAPI (hardware abstraction layer interface for CIAA peripheral)
    - https://github.com/ciaa/firmware_v2/blob/master/modules/lpc4337_m4/sapi/inc/sapi_uart.h - sAPI UART
    - https://github.com/ciaa/firmware_v2/blob/master/sapi_examples/edu-ciaa-nxp/bare_metal/wifi_esp01_esp8266/01_uart_bridge/src/uartBridge_ESP8266.c - sAPI esp8266
    - https://github.com/ciaa/firmware_v2/tree/master/sapi_examples/edu-ciaa-nxp/bare_metal
- https://lastminuteengineers.com/bme280-esp8266-weather-station/ - arduino<->esp8266 integration
