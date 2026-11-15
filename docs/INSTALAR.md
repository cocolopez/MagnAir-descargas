# Instalar MagnAir en el módulo WiFi

El módulo WiFi de la Magna 2 230 es una placa chiquita (MKS Robin WiFi, con un ESP8266) enchufada en la placa principal. La impresora sabe grabarle el firmware desde la tarjeta SD.

## Primera vez (desde la SD)

1. Copiá **`MksWifi.bin`** a la raíz de la tarjeta SD (formato FAT32). Puede ir junto con el firmware de la impresora.
2. Poné la SD y **encendé la impresora**. Después de cargar los íconos aparece la pantalla de actualización del WiFi. **No la apagues** hasta que termine (menos de un minuto).
3. Cuando termina, el archivo se renombra a `MKSWIFI.CUR` (así no se vuelve a grabar en cada arranque).
4. En la pantalla de la impresora, entrá a **Ajustes → WiFi**:
   - Si antes estaba conectado a tu red, MagnAir **conserva la red y la clave** del firmware anterior y se conecta solo. La IP aparece en esa pantalla.
   - Si no, el módulo crea su propia red **MAGNAIR-XXXX** (clave **magna2230**). Conectate con el celular y se abre la página sola (si no, entrá a `http://192.168.4.1`). Ahí, en **Ajustes → Red WiFi**, elegí la red de tu casa.
5. Desde la computadora o el celular, en la misma red: `http://magnair.local` o la IP que muestra la pantalla.

> En Android `magnair.local` a veces no funciona (depende del celular): usá la IP. Conviene reservarle una IP fija en el router o activar **Usar IP fija** en Ajustes.

## Actualizaciones (desde la página)

**Ajustes → Módulo WiFi → Actualizar el firmware del módulo**, elegir el `MksWifi.bin` nuevo. Tarda alrededor de un minuto; después el módulo se reinicia solo y los ajustes se conservan.

## Volver al firmware original de Makerbase

En la carpeta `volver-al-oficial/` está el firmware original de Makerbase (versión CS1.0.4_200227). Se instala igual que MagnAir: copiarlo como `MksWifi.bin` a la raíz de la SD y encender la impresora. También se puede cargar desde la página de MagnAir (Ajustes → Módulo WiFi).

## Si algo sale mal

- **La pantalla de actualización no aparece:** revisá que el archivo se llame exactamente `MksWifi.bin`, que esté en la raíz y que la SD sea FAT32. Si ya se renombró a `MKSWIFI.CUR`, borralo y volvé a copiar el `.bin`.
- **Para grabar desde la SD, el módulo tiene que estar funcionando:** la impresora solo lo actualiza si el módulo le "habla" en los primeros segundos. MagnAir lo hace apenas arranca, igual que el firmware original, así que siempre se puede volver al original desde la SD. Si el módulo quedara totalmente trabado (por ejemplo, porque se cortó la luz en medio de una grabación), la SD no alcanza: hay que sacarlo y grabarlo con un adaptador USB-serie de 3,3 V (TX, RX, GND y GPIO0 a GND al encender) con esptool o con el programa de Espressif.
- **La página dice "El módulo WiFi no recibe respuesta de la impresora":** en la pantalla, Ajustes → WiFi tiene que estar activado. Si recién se encendió, esperá unos segundos.
- **No encuentro la IP:** está en la pantalla de la impresora (Ajustes → WiFi). Si el módulo no logra conectarse a tu red en 40 segundos, además enciende su red propia MAGNAIR-XXXX para que puedas entrar y corregir la clave.
- **Valores de fábrica:** desde la página, Ajustes → Módulo WiFi → Valores de fábrica. Se borran la red, la cámara y los ajustes, y vuelve a crear su propia red.
