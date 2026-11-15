# Protocolo entre el módulo y la placa

Lo usa Marlin (`Marlin/src/lcd/extui/mks_ui/wifi_module.cpp`) para hablar con el módulo MKS Robin WiFi. Se reconstruyó a partir del código de Marlin 2.1.3-b3 y del documento de protocolo de Makerbase; las pruebas (`pruebas/test_protocolo.cpp`) llevan una copia fiel del lector de Marlin para comprobar que lo que arma MagnAir se lee bien del otro lado.

## Conexión

| Señal | ESP8266 | Uso |
|---|---|---|
| TX/RX | UART0 | 115200 baudios; 1.958.400 durante la subida de archivos |
| Pedido | GPIO0 (salida) | El módulo lo pone en bajo mientras manda |
| Listo | GPIO4 (entrada) | La placa lo pone en bajo cuando puede recibir (en la subida, cuando tiene listo el DMA para un bloque) |

El puerto serie es de la placa: MagnAir no escribe mensajes de depuración.

## Trama

`A5 | tipo | largo (2 bytes, LE) | datos | FC` — máximo 1024 bytes.

Del módulo a la placa:

| Tipo | Datos |
|---|---|
| 0 red | IP (4), estado del enlace (0x0A ok / 0x05 no), modo (1 AP / 2 cliente), largo + nombre de la red, largo + clave, campos de la nube en cero, largo + versión |
| 1 G-code | texto terminado en `\r\n` |
| 2 primer bloque | largo del nombre, tamaño (4, LE), nombre |
| 3 fragmento | número (4, LE; bit 31 = último) + hasta 1015 bytes. Tramas siempre de 1024 bytes |
| 4 lista de redes | cantidad, y por red: largo + nombre + RSSI |

De la placa al módulo:

| Tipo | Datos |
|---|---|
| 0 config de red | modo (1 AP / 2 cliente), largo + nombre, largo + clave |
| 2 texto | respuestas (`ok`, `T:…`, `M997 IDLE`, lista de archivos, …) |
| 3 excepción | 1 error / 2 ok de transferencia (no se usa) |
| 4, 5 | nube de Makerbase (no se usa) |
| 6 | pedido de lista de redes |
| 9 | 1 conectar, 2 desconectar, 3 olvidar |

## Consultas de estado

Cada 2 segundos MagnAir manda `M991` (temperaturas) y `M997` (estado: `M997 IDLE/PRINTING/PAUSE`); imprimiendo, además `M27` (`M27 54` = %), `M992` (`M992 01:07:42`) y `M994` (`M994 nombre;tamaño`). Al principio, `M115` hasta saber el nombre de la placa. La placa contesta esos comandos directamente, sin pasar por la cola de G-code.

`M20 L` (solo con la impresora libre) devuelve `Begin file list`, una línea por archivo (`CORTO.GCO 0 nombre largo`, `.DIR` para carpetas) y `End file list`.

Los demás comandos pasan por un buffer de 288 bytes de la placa, por eso MagnAir manda **una línea por trama con 60 ms entre tramas**. La placa contesta `ok` apenas recibe cada línea. Las líneas que empiezan con `wait`, `busy` o `X:` no se reenvían por WiFi.

## Comandos largos

Marlin atiende el módulo (`wifi_looping()`, que lee las tramas y contesta) solo desde `printer_state_polling()`, al final de su bucle principal, y **no desde `idle()`**. Mientras corre un comando que bloquea (M306, M303, M109/M190 esperando temperatura, G28, G29, G34, M48, M600, G4, M400) la placa no contesta nada. MagnAir recuerda el último de esos comandos que mandó (`mks::comando_bloqueante`) y, si la placa deja de contestar después, lo informa como `ocupada`. Mientras no contesta, consulta cada 10 s en lugar de cada 2 s para no llenarle el buffer de recepción.

## Subida de archivos

1. Trama 2 a 115200 (nombre y tamaño). La placa espera 1 segundo, abre el archivo en la SD y pasa a 1.958.400 baudios con DMA.
2. MagnAir espera 1,8 segundos y cambia también de velocidad.
3. Por cada bloque de 1015 bytes: esperar GPIO4 en bajo (hasta 4 s), mandar la trama de 1024 bytes completa.
4. El último fragmento lleva el bit 31 (aunque tenga 0 bytes).
5. MagnAir vuelve a 115200 y, si se pidió, manda `M23 /NOMBRE~1.GCO` y `M24`.

La placa abandona la subida si pasan 8 segundos sin datos. Guarda el archivo con nombre 8.3: `longName2DosName()` de Marlin, copiada en `mks::nombre_dos()`.

## Actualización del módulo desde la SD

Al arrancar, si la SD tiene `MksWifi.bin`, Marlin graba el módulo por el bootloader del ESP8266 (con GPIO0 y reset), pero **solo si el módulo le mandó al menos 20 bytes en los primeros ~2 segundos**. Por eso MagnAir manda la trama de red en `setup()`, antes de encender el WiFi, y la repite cada segundo los primeros 15 segundos.

## Ajustes guardados

EEPROM emulada de 1024 bytes. MagnAir usa desde la posición 512 (marca `MAG1`). El firmware original de Makerbase guarda la red en 0 (nombre, 32), 32 (clave, 64), 160 (modo) y 176 (0x0A = válido); MagnAir la lee al instalarse para no pedirla de nuevo, y no la borra (así, volviendo al original, la red sigue cargada).

## Comandos propios de MagnAir (firmware de la impresora `mm2`)

El firmware MKS de la placa no reenvía al WiFi las respuestas normales de Marlin (`M503`, `M114`, `M119`…). Por eso el firmware de la impresora agrega dos comandos que contestan directo por WiFi:

| Comando | Respuesta | Etapa de `mm2` |
|---|---|---|
| `M256` / `M256 B0` / `M256 B255` | `M256 B255` o `M256 B0` (luz de la pantalla) | 22 |
| `M257` | una línea por grupo, `M257 <grupo> CLAVE=valor …`, y al final `M257 fin` | 23 |

Grupos de `M257`: `mpc` (P C R A F H) o `pidhotend` (P I D), `pidcama` (P I D), `pasos` (X Y Z E0 E1), `avance` (K0 K1), `vibraciones` (XF XD YF YD), `altura` (S Z), `velmax`, `acelmax` (X Y Z E) y `acel` (P R T). El módulo junta las líneas hasta `M257 fin` (o 5 s) y las entrega en `/api/calibracion`.

## Carpetas de la SD

- `M20 L /PIEZAS` lista esa carpeta (las carpetas vienen con `/` al final). El firmware original ignora la ruta y lista siempre la raíz; la etapa 23 de `mm2` entra a la carpeta y vuelve a la raíz al terminar.
- `M23 /PIEZAS/BENCH~1.GCO` abre el archivo con la ruta completa (la etapa 23 cambia el `strrchr` por la ruta entera, con el comportamiento viejo como respaldo).
- Las rutas van con nombres cortos 8.3.
