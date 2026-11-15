# API del módulo

Todo lo que hace la página pasa por esta API, así que una app (Android, Windows, un script) puede hacer lo mismo. Las respuestas son JSON y llevan `Access-Control-Allow-Origin: *`.

- Los `POST` reciben los datos como formulario (`application/x-www-form-urlencoded`) o en la dirección (`?clave=valor`).
- Los errores vienen con código HTTP 4xx/5xx y `{"ok":false,"error":"texto"}`.
- El módulo atiende **un pedido por vez**. Mientras sube un archivo no responde otra cosa.

## Estado

`GET /api/estado`

```json
{
  "version": "MagnAir-1.0.0", "placa": "Robin_nano", "conectada": true,
  "ocupada": "",                      // "M306", "G28", ... si la placa no contesta por un comando largo que se le mandó
  "ocupada_s": 0, "sin_datos": 1,     // segundos desde ese comando / desde el último dato de la placa (-1 = nunca)
  "estado": "imprimiendo",            // libre | imprimiendo | pausada | desconocido
  "progreso": 54, "tiempo": 4062,     // % y segundos transcurridos
  "archivo": "soporte.gcode", "tamano": 2516582,
  "hotend": 210, "hotend_obj": 210, "cama": 60, "cama_obj": 60,
  "pantalla": 1,                      // luz de la pantalla: 1 prendida, 0 apagada, -1 sin dato (firmware viejo)
  "consola": 132,                     // número de la última línea de consola
  "red": {"nombre": "Casa", "ip": "192.168.1.42", "senal": -58, "casa": true, "propia": false, "equipo": "magnair.local"},
  "subida": {"activa": false, "nombre": "", "enviado": 0, "tamano": 0, "error": ""},
  "memoria": 31544, "encendido": 3600, "reinicio": "Software/System restart", "flash": 4194304
}
```

`conectada` = hubo respuesta de la impresora en los últimos 10 segundos. Mientras corre un comando largo (calibrar, calentar esperando, home, nivelar) Marlin no atiende el WiFi: `conectada` queda en `false` y `ocupada` dice qué comando es. Los datos se actualizan cada 2 segundos.

## Comandos

| Pedido | Datos | Qué hace |
|---|---|---|
| `POST /api/gcode` | `c` = una o varias líneas separadas por `\n` | Manda G-code (máx. 96 caracteres por línea, 40 líneas en cola) |
| `POST /api/pantalla` | `encender` = `1` \| `0` | Prende o apaga la luz de la pantalla de la impresora (M256; necesita el firmware de la impresora con la etapa 22) |
| `POST /api/accion` | `a` = `pausar` \| `seguir` \| `detener` | M25 / M24 / M26 |
| `POST /api/imprimir` | `nombre` = ruta con nombres cortos (`/PIEZAS/BENCH~1.GCO`, o solo `BENCH~1.GCO` en la raíz) | M23 + M24 |
| `POST /api/borrar` | `nombre` = igual que imprimir | M30 |

Imprimir y borrar devuelven 409 si la impresora está imprimiendo. Imprimir desde una carpeta necesita el firmware de la impresora con la etapa 23 (`mm2`).

## Consola

`GET /api/consola?desde=N` → líneas con número mayor que `N`:

```json
{"ultima": 134, "lineas": [{"n": 133, "d": ">", "t": "G28"}, {"n": 134, "d": "<", "t": "ok"}]}
```

`d`: `>` enviado, `<` recibido, `*` aviso del módulo. Guarda las últimas 60 líneas.

## Archivos

`GET /api/archivos[?carpeta=/PIEZAS]` → lee una carpeta de la SD (tarda 1 o 2 segundos; sin `carpeta`, la raíz). Con `?guardada=1`, o mientras imprime, devuelve la última lista leída sin preguntarle a la impresora.

```json
{"guardada": false, "edad": 0, "carpeta": "/PIEZAS",
 "archivos": [{"corto": "VIEJOS", "nombre": "viejos", "carpeta": true},
              {"corto": "BENCH~1.GCO", "nombre": "benchy.gcode", "carpeta": false}]}
```

La ruta de la carpeta va con los nombres cortos (`corto`). Leer subcarpetas necesita el firmware de la impresora con la etapa 23 (`mm2`); con uno viejo siempre devuelve la raíz. Las subidas van siempre a la raíz.

## Calibración guardada

`GET /api/calibracion` → los valores que la impresora tiene en uso (le pide `M257`, tarda hasta 4 segundos):

```json
{"disponible": true, "grupos": {
  "mpc": {"P": 40, "C": 16.7, "R": 0.18, "A": 0.06, "F": 0.08, "H": 0.0036},
  "pidcama": {"P": 54.1, "I": 7.2, "D": 270.5},
  "pasos": {"X": 80, "Y": 80, "Z": 400, "E0": 415, "E1": 415},
  "avance": {"K0": 0, "K1": 0},
  "vibraciones": {"XF": 0, "XD": 0.15, "YF": 0, "YD": 0.15},
  "altura": {"S": 0, "Z": 0},
  "velmax": {"X": 300, "Y": 300, "Z": 5, "E": 70},
  "acelmax": {"X": 3000, "Y": 3000, "Z": 100, "E": 5000},
  "acel": {"P": 1000, "R": 1500, "T": 1500}}}
```

`disponible: false` = la impresora no contestó (firmware sin la etapa 23, o está ocupada). En `altura`, `S` 1 = el valor es el de la sonda (M851), 0 = el desplazamiento del inicio (M206). Para cambiar un valor se manda el G-code correspondiente por `/api/gcode` y después `M500` para guardarlo.

### Subir

`POST /upload?nombre=archivo.gcode[&imprimir=1]` con el archivo como **cuerpo crudo** (`Content-Type: application/octet-stream`) y `Content-Length`. También acepta `?X-Filename=` (complemento MKS de Cura).

```
curl --data-binary @benchy.gcode -H "Content-Type: application/octet-stream" "http://magnair.local/upload?nombre=benchy.gcode"
```

Respuesta: `{"ok":true,"nombre_sd":"BENCH~1.GCO"}`. La impresora guarda con nombre corto (ver README, limitaciones).

## Historial y estadísticas

`GET /api/historial` → las últimas 50 impresiones que anotó el módulo (de la más vieja a la más nueva) y los totales de Marlin:

```json
{"entradas": [{"fecha": 1731590000, "duracion": 9840, "filamento": 12650, "terminada": true, "nombre": "soporte.gcode"}],
 "estadisticas": {"disponible": true, "impresiones": 37, "terminadas": 31, "tiempo": 412380, "mas_larga": 52200, "filamento": 486250},
 "precio_kg": 18000, "densidad": 124, "hora": 1731600000}
```

- `fecha`: segundos desde 1970 en UTC (la hora la toma de internet; 0 = todavía no tenía hora). `duracion` en segundos, `filamento` en mm (-1 = no se sabe).
- `estadisticas` sale de M257 (grupo `estadisticas`, firmware `mm2` etapa 27); `disponible: false` con un firmware viejo. Si los datos tienen más de un minuto se vuelven a pedir (hasta 4 s).
- `POST /api/historial/borrar` borra la lista del módulo (los totales de Marlin no).

## Telegram

- `GET /api/telegram/chat` → `{"ok":true,"chat":"123456789"}`: el último chat que le escribió al bot (el módulo le pregunta a Telegram). Error 400 con el motivo si no hay token o no hay mensajes.
- `POST /api/telegram/probar` → manda un mensaje de prueba con el bot y el chat guardados.

### Comandos del bot (no son de la API HTTP)

El módulo lee los mensajes de Telegram con `getUpdates` cada 15 s (cada 3 s durante 2 minutos después de un comando) y contesta **solo al chat guardado** (`tg_chat`). Carga el menú con `setMyCommands` una vez por token. La lista está en `src/bot.cpp` (`COMANDOS[]`) y se repite en las apps (`ComandosTelegram`); `pruebas/verificar-comandos-telegram.py` controla que sean iguales.

| Comando | Qué hace |
|---|---|
| `/estado` | estado, progreso, temperaturas |
| `/foto` | foto de la cámara configurada (`/shot.jpg` de IP Webcam) |
| `/pausar` | M25 (solo si imprime) |
| `/seguir` | M24 (solo si está en pausa) |
| `/detener` | pide confirmar; `/detener_si` en 60 s manda M26 |
| `/enfriar` | `M104 S0` y `M140 S0` (no mientras imprime) |
| `/mensaje texto` | `M117 texto` (limpio: sin saltos, `;` ni `*`, hasta 90 bytes): lo muestra la pantalla de la impresora |
| `/ayuda`, `/start` | lista de comandos |

## Configuración

| Pedido | Datos |
|---|---|
| `GET /api/config` | → `nombre, modo (casa/propia), red, tiene_clave, red_propia, clave_propia, camara, foto, giro, espejo, ip_fija, ip, mascara, puerta, dns, precalentar[{material, hotend, cama}], inicio` |
| `POST /api/config` | cualquiera de: `nombre`, `camara`, `foto`, `giro` (0/90/180/270), `espejo` (0/1), `pla_h`, `pla_c`, `petg_h`, `petg_c`, `abs_h`, `abs_c`, `inicio`, `tg_token`, `tg_chat`, `tg_avisos`, `precio_kg`, `densidad` |
| `POST /api/red` | `modo` = `casa` \| `propia`, `red`, `clave` (vacía = conservar), `ip_fija` (0/1), `ip`, `mascara`, `puerta`, `dns`. Responde y después reconecta |
| `GET /api/redes[?buscar=1]` | `{"buscando": false, "redes": [{"nombre": "Casa", "senal": -58}]}`. Con `buscar=1` empieza una búsqueda (unos 3 segundos) |
| `POST /api/reiniciar` | reinicia el módulo |
| `POST /api/fabrica` | borra los ajustes y reinicia |
| `tg_token` | token del bot de Telegram. **No se devuelve nunca**: `GET` trae `tg_configurado` (true/false). Para borrarlo: `tg_token=` vacío con `tg_borrar=1` |
| `tg_chat`, `tg_avisos` | número de chat y qué avisar: suma de 1 terminó, 2 se detuvo, 4 pausa, 8 calibración, 16 empezó (de fábrica 15) |
| `precio_kg`, `densidad` | precio del rollo por kilo (0 = sin costos) y densidad del filamento en g/cm³ × 100 (PLA 124) |
| `inicio` | qué tarjetas muestra Inicio y en qué columna: `"camara,trabajo,consola\|temperaturas,atajos,archivos"` (izquierda`\|`derecha). Tarjetas: `camara`, `trabajo`, `consola`, `temperaturas`, `atajos`, `archivos`, `calibracion`. La comparten la página y las apps |
| `POST /actualizar` | firmware `.bin` del ESP8266 como cuerpo crudo; responde y reinicia. Con la cabecera `X-MD5` (huella del archivo, 32 letras) el módulo comprueba que llegó intacto y si no, no lo instala (la página y las apps la mandan siempre). `reinicio` y `flash` en `/api/estado` sirven para diagnosticar un arranque fallido |

## Cura (puerto 8080)

Conexión TCP de texto: cada línea que llega se manda a la impresora como G-code y todo lo que responde la impresora se reenvía a la conexión. Es lo que usa el complemento **MKS WiFi** de Cura (además de `/upload`). Una sola conexión por vez.

## Hora para la pantalla de la impresora (M258)

El módulo le manda a Marlin, por el enlace serie, `M258 S<segundos desde la medianoche local> D<día de la semana, 0 = domingo> F<día> O<mes> T<1 si el bot de Telegram está configurado> A<opciones de la pantalla: 1 animaciones, 2 melodías, 4 siempre encendida> P<precio del filamento por kilo> G<densidad × 100>` cuando la placa empieza a responder y cada 10 minutos. La zona horaria es `zona_min` de `/api/config` (minutos respecto de UTC; −180 = Argentina).


`pantalla_op` de `/api/config`: las opciones de la pantalla (la letra A del M258). `M117 texto` por `/api/gcode` muestra un mensaje en la pantalla de la impresora (firmware mm2 etapa 31).
