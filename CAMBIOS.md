# Cambios de MagnAir

Lo que cambió en cada versión, para quien usa MagnAir. El workflow `publicar-release.yml` copia en
las notas de cada versión publicada **todas** las secciones más nuevas que la última versión
publicada (si saltaste versiones, aparecen todas juntas).

Formato: una sección `## X.Y.Z (fecha)` por versión, la más nueva arriba; el número es el de
`MAGNAIR_VERSION` en `src/config.h` sin "MagnAir-". Dentro, viñetas agrupadas en **Módulo WiFi**,
**Página** y **Apps** (solo los grupos que cambiaron).

## 1.6.1 (2026-11-24)

**Módulo WiFi**
- Arreglado: al pasar a la SD un archivo con nombre largo, la impresora se reiniciaba justo al recibirlo y el archivo no quedaba. Ahora el módulo manda el nombre recortado (con la extensión), y en la SD queda con el mismo nombre corto de siempre. Conviene también el firmware mm2 nuevo, que trae el mismo arreglo del lado de la impresora.

## 1.6.0 (2026-11-15)

**Módulo WiFi y página**
- Cola de impresión: una lista de archivos de la SD para imprimir uno atrás del otro. Cuando termina una pieza, Telegram avisa cuál sigue; la siguiente se arranca a mano (hay que sacar la pieza): desde la app, la página o `/siguiente`. `/cola` muestra la lista.
- Imprimir a una hora: se elige un archivo y la hora, y el módulo la arranca solo (si la impresora está ocupada espera hasta 30 minutos).
- Aviso de error grave por Telegram: si la impresora se detiene por un error (falla de temperatura, termistor, etc.) llega un mensaje con el motivo. Necesita el firmware mm2 nuevo para los errores que muestra la pantalla.
- Los precalentados de PLA y ABS de la pantalla de la impresora usan las temperaturas que pusiste en la app (con mm2 nuevo).
- Sabe qué Magna 2 es (230, 300, 400, 400D, 500 o 500D) cuando la impresora tiene el firmware mm2 nuevo.
- Placa MKS Robin Nano V1.2 (firmware mm12, sin probar): el módulo le pregunta el estado cada 5 segundos en vez de cada 2, para no cargarla, y la página esconde las opciones que esa placa no tiene.

**Apps**
- Todo ordenado en 4 secciones: **Inicio**, **Imprimir** (SD, cola, subir y visor 3D), **Máquina** (mover, temperaturas, filamento, nivelar; lo menos usado en Avanzado) y **Más** (historial, Telegram, cámara, pantalla, módulo, la app y Acerca de).
- Rollos de filamento: cargás tus rollos, marcás cuál está puesto y la app va descontando lo que usa cada impresión.
- Tiempo real: la app compara lo que dice el slicer con lo que tardó de verdad y en el visor y el historial muestra cuánto suele tardar en tu máquina.
- Historial: exportar a planilla (CSV) y foto de cómo quedó cada impresión si hay cámara.
- Copia de la calibración: guarda los valores de la impresora en un archivo y los vuelve a cargar después de flashear.
- Android: widget con el avance (sigue andando con la app cerrada mientras hay una impresión, y tiene un botón ↻ para actualizarlo cuando quieras), notificación con barra de progreso, compartir un `.gcode` a la app para subirlo y accesos directos (pausar, cámara, precalentar PLA) manteniendo apretado el ícono.
- Versión para Linux (`.deb`).
- Con el firmware mm2 nuevo la app sabe el modelo de Magna 2: el visor 3D dibuja la cama del tamaño correcto, la nivelación a mano va a las esquinas de esa cama y en Acerca de dice el modelo. Sin mm2 nuevo se supone la 230.
- Arriba de cada sección (al lado de Editar, y en Más al lado del título) dice qué máquina está conectada: "Magna 2 400D · placa V1.3".
- Varias impresoras: tocando esa etiqueta se elige otra de las que ya usaste (hasta 10), se le pone un nombre ("Taller") o se agrega una nueva. Se mira una a la vez.
- Con una placa V1.2 (firmware mm12) no aparecen las animaciones, "siempre encendida" ni la compensación de vibraciones, que esa versión no tiene.

## 1.5.1 (2026-11-15)

**Módulo WiFi y página**
- Dormir la impresora: apaga hotend, cama, ventilador y motores y oscurece la pantalla (Control en la página y en las apps, y `/dormir` en el bot de Telegram). La fuente sigue prendida; se despierta tocando la pantalla.

**Apps**
- Ajustes, Telegram y la calibración guardada se ven aunque el módulo esté apagado: la app guarda una copia de lo último que leyó (para cambiar algo, el módulo tiene que estar prendido).
- Botón "Continuar sin conexión" en la pantalla de conectar: entra a la app aunque no esté la impresora (por ejemplo para usar el visor 3D).
- Arriba dice MagnAir otra vez, con "MAGNA 2" chico abajo (el 2 en rojo, como en la impresora).
- Windows: la barra de la ventana tiene el mismo color que arriba de la app y el borde es bordó.
- La consola ocupa todo el ancho de la pantalla.
- Arreglo: el logo y los títulos se veían oscuros sobre el fondo oscuro.
- Sección Telegram: cada comando se ve siempre en un solo renglón.
- Mensajes de error entendibles: en vez de textos técnicos ("Connect timeout has expired…") dice qué pasa y qué revisar (por ejemplo "El módulo no contesta (192.168.1.42). Revisá que la impresora esté prendida…").
- Si el módulo no contesta, la app pregunta cada vez más espaciado (hasta cada 15 s) y gasta menos batería; apenas contesta vuelve a cada 2 s.
- Visor 3D más liviano: ocupa mucha menos memoria con piezas grandes.
- Inicio: el ícono de la cama muestra el calor que sube y el del hotend una gota que cae mientras están calientes (los mismos cuadros que la pantalla de la impresora); el ventilador del hotend se ve siempre y gira cuando anda.
- Botones del visor con nombre para el lector de pantalla; los números de los pasos de Telegram no se cortan con letra grande.

## 1.5.0 (2026-11-14)

**Apps**
- Aspecto nuevo, como la pantalla de la impresora: fondo degradé gris → bordó, título "M A G N A 2" con reloj y WiFi arriba, los íconos de la impresora en la barra de abajo y en las temperaturas, línea roja en las tarjetas y barra de avance con brillo.
- Temperaturas que cambian de color según lo caliente (gris, amarillo, naranja, rojo) y titilan mientras calientan; el ventilador del hotend gira cuando anda (arriba de 50 °C).
- Visor 3D de gcode (Archivos → "Visor 3D y editor"): la pieza capa por capa, con colores para paredes, relleno y soportes; se gira arrastrando, se acerca pellizcando o con la ruedita, vista desde arriba o en 3D.
- Editor por capas: agregá pausas, cambio de filamento, temperatura, ventilador, velocidad o un mensaje en la pantalla en la capa que elijas, y mandalo a la impresora (opcional: empezar a imprimir al terminar). Puede agregar solo las marcas para la pantalla (capa actual y cancelar piezas) sin usar los scripts de Cura. El archivo original no se toca.

## 1.4.0 (2026-11-14)

**Módulo WiFi (bot de Telegram)**
- Botones en el chat (/menu): estado, foto, pausar, seguir, detener (con confirmación), archivos, precalentar PLA y enfriar.
- Archivos: mandale un .gcode al bot (hasta 20 MB) y lo pasa a la SD; al terminar ofrece un botón para imprimirlo. `/archivos` lista la SD con un botón por archivo.
- Comandos nuevos: `/precalentar PLA|PETG|ABS`, `/temp 210 60`, `/historial`, `/estadisticas`, `/menu`.
- Fotos automáticas al 25, 50 y 75 %, al terminar y cuando queda en pausa (opción nueva en los avisos).
- Otros chats autorizados: más de una persona puede usar el bot y recibir los avisos.

**Apps**
- Lector y editor de gcode (base del visor 3D y del editor por capas que vienen después).

## 1.3.0 (2026-11-14)

**Módulo WiFi**
- Le pasa la hora a la pantalla de la impresora (la saca de internet) cada 10 minutos, junto con si el bot de Telegram está configurado. Hace falta el firmware de la impresora con la hora (mm2, etapa 30).
- Zona horaria configurable (por defecto Argentina, UTC−3).

- Pasa también a la pantalla las opciones (animaciones, melodías, siempre encendida) y el precio y material del filamento (para el resumen al terminar).
- Bot de Telegram: comando nuevo `/mensaje texto`, que muestra el texto en la pantalla de la impresora.

**Página y apps**
- Ajustes: tarjeta nueva **Pantalla de la impresora**: zona horaria, animaciones, melodías, siempre encendida mientras imprime y un campo para mostrar un mensaje en la pantalla.

## 1.2.1 (2026-11-14)

**Módulo WiFi**
- *Buscar mi chat* ya no deja la app desconectada: pide solo el último mensaje a Telegram y controla que haya memoria antes de conectarse (si no alcanza, avisa en vez de reiniciarse).

**Apps**
- No muestran "No se puede conectar con el módulo" mientras el módulo habla con Telegram, y el aviso aparece recién con dos fallas seguidas.

## 1.2.0 (2026-11-14)

**Módulo WiFi**
- Bot de Telegram con comandos, desde cualquier lugar y sin VPN: `/estado`, `/foto`, `/pausar`, `/seguir`, `/detener` (pide confirmación), `/enfriar`, `/ayuda`. Solo contesta a tu chat y carga solo el menú de comandos.

**Página**
- La tarjeta de Telegram muestra la lista de comandos.

**Apps**
- Sección nueva **Telegram**: cómo crear el bot paso a paso, configuración, lista de comandos y notas.

## 1.1.0 (2026-11-14)

**Módulo WiFi**
- Historial de impresiones (últimas 50) y estadísticas, con filamento y costo de cada una.
- Avisos por Telegram: impresión empezada, terminada, detenida, en pausa y calibración terminada.
- Actualización del módulo por WiFi más segura: verifica la huella MD5 del archivo y muestra el motivo del último reinicio.

**Página y apps**
- Archivos: Historial y Estadísticas. Control: Protecciones (corte de luz, sensor de filamento) y Nivelar la cama a mano. Ajustes: Telegram y Filamento y costos.
- Timelapse (apps): arma un video con fotos de la cámara mientras imprime.
- Apps: todas las tarjetas se pueden plegar y mover en todas las secciones; en Windows la barra de la ventana toma el color de la app y se puede cerrar a la bandeja.

## 1.0.0 (2026-11-14)

- Primera versión: firmware propio para el módulo MKS Robin WiFi con página web, app de Android y app de Windows. Estado, cámara, archivos (con carpetas), control, consola, pantalla de la impresora, calibración y ajustes; avisos al terminar; Inicio armable arrastrando las tarjetas.
