# Cámara con un celular Android viejo

El módulo WiFi no tiene cámara (el ESP8266 no puede manejar una). En cambio, la página muestra el video de un celular viejo que hace de cámara de red. El video va **directo del celular al navegador**: no pasa por el módulo, así que no lo carga ni frena las impresiones.

## Preparar el celular

1. Instalá **IP Webcam** (de Pavel Khlebovich) desde Google Play. Funciona desde Android 4.
2. En la app:
   - **Video preferences → Video resolution:** 1280×720 o 640×480 alcanzan y gastan menos WiFi.
   - **Power management:** activá que no se apague la pantalla o que siga funcionando con la pantalla apagada.
   - (Opcional) **Local broadcasting → Login/password:** si le ponés usuario y clave, la dirección queda `http://usuario:clave@IP:8080/video` (algunos navegadores no aceptan clave en la dirección de una imagen; en ese caso, mejor dejarla sin clave en la red de tu casa).
3. Tocá **Start server**. Abajo de la pantalla aparece una dirección, por ejemplo `http://192.168.1.50:8080`.
4. Dejalo enchufado al cargador y fijo con un soporte apuntando a la cama.

> Conviene reservarle una IP fija al celular en el router, así la dirección no cambia.

## Cargarla en la página

**Ajustes → Cámara**:

| Campo | Qué poner |
|---|---|
| Dirección del video | `http://192.168.1.50:8080/video` (la dirección de la app + `/video`) |
| Dirección de una foto | Vacío: se usa sola `…/photo.jpg`. Para otra app, la dirección de una foto JPG |
| Girar / Espejar | Según cómo quedó el celular. También se puede girar con el botón sobre el video |

Sirve cualquier cámara o app que dé video **MJPEG** por HTTP (por ejemplo, otras apps de cámara IP o una cámara de red que tenga dirección MJPEG).

## Detalles

- El video se corta solo cuando la pestaña queda oculta (ahorra batería del celular y WiFi) y vuelve al mostrarla.
- Si el celular no responde, la página reintenta cada 10 segundos.
- El celular tiene que estar en la misma red que quien mira. Si entrás con la red propia del módulo (MAGNAIR-XXXX), el celular también tendría que estar conectado a esa red.
- Desde fuera de casa no se ve (ni la página ni la cámara) salvo que armes una VPN o algo similar: el módulo no está pensado para quedar expuesto a internet.
