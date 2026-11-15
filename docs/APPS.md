# Apps de Android y Windows

Las dos apps tienen **el mismo código**: Kotlin + Compose Multiplatform. La interfaz es la misma que la de la página del módulo (Inicio, Archivos, Control, Consola, Ajustes), pero nativa: anda más fluida, recuerda el módulo y lo busca sola en la red. Usan la misma API JSON que la página ([API.md](API.md)), así que no hace falta nada nuevo en el módulo.

## Instalar

- **Android:** bajá `MagnAir-….apk` (artefacto **MagnAir-Android** o la versión publicada en Releases) y abrilo en el celular. La primera vez Android pide permiso para instalar apps de esa fuente (el navegador o el administrador de archivos). Funciona desde Android 7.
- **Windows:** bajá `MagnAir-Setup-….exe` (artefacto **MagnAir-Windows**) e instalalo. No necesita Java ni permisos de administrador. Windows puede avisar "Windows protegió su PC" porque el instalador no está firmado: *Más información → Ejecutar de todos modos*.

Las versiones nuevas se instalan encima de las viejas (el .apk siempre se firma con la misma clave).

## Primer uso

La app busca sola el módulo:
1. la última dirección usada,
2. `magnair.local`,
3. `192.168.4.1` (la red propia del módulo, MAGNAIR-XXXX),
4. toda la red local del equipo (las 254 direcciones, de a 48 a la vez; tarda unos segundos).

Si no lo encuentra, se puede escribir la IP a mano (la muestra la pantalla de la impresora en Ajustes → WiFi). En Ajustes → *Esta app* → **Cambiar de módulo** se vuelve a esa pantalla.

## Organización del código (`apps/`)

```
apps/
├─ gradle/libs.versions.toml      versiones de todo (Kotlin, Compose, Ktor, Android)
├─ version.gradle.kts             versión = MAGNAIR_VERSION de ../src/config.h; compilación = GITHUB_RUN_NUMBER
├─ shared/                        TODO lo común (Android y Windows)
│  └─ src/commonMain/kotlin/ar/magnair/
│     ├─ api/          Modelos.kt (JSON de la API) · ClienteMagnAir.kt (Ktor: pedidos, subidas con avance)
│     ├─ dominio/      Formato (horas, tamaños, nombre 8.3) · Comandos (G-code, atajos, textos de "ocupada")
│     │                Extras (cámara por IP, avisos de fin, Inicio modular, botones de consola)
│     │                Calibracion (textos de M257 y funciones de Marlin sin menú)
│     ├─ camara/       Mjpeg.kt: separa los cuadros JPEG del video del celular
│     ├─ conexion/     Buscador.kt: busca módulos en la red · BuscadorCamara.kt: busca IP Webcam (puerto 8080)
│     ├─ estado/       AppMagnAir (qué módulo) · Sesion (consulta cada 2 s, consola, archivos, acciones)
│     ├─ plataforma/   Plataforma.kt: lo único que cambia entre sistemas (interfaz)
│     └─ ui/           tema (colores, fuentes, íconos) · componentes · pantallas · App.kt
│  └─ src/commonTest/  pruebas (formato, G-code, MJPEG, JSON, cámara, avisos, Inicio, ajustes de Marlin)
├─ androidApp/                    MainActivity + PlataformaAndroid (selector de documentos, imágenes, red)
│                                 ServicioSeguimiento + Notificaciones (avisos con la app cerrada)
└─ desktopApp/                    Main.kt (ventana + ícono en la bandeja) + PlataformaEscritorio (archivos, imágenes, red, avisos)
   └─ instalador/magnair.iss      instalador de Inno Setup
```

Reglas:
- La interfaz **solo observa** `StateFlow` de `Sesion` y llama a sus métodos; no habla con la red.
- Lo que depende del sistema entra por `Plataforma` (no hay `expect/actual`): elegir un archivo, decodificar un JPEG, abrir el navegador, direcciones IP propias, ajustes guardados y avisos del sistema.
- Los G-code y textos están en `dominio/`, una sola vez, con las mismas cifras que la página y que los botones de la pantalla de la impresora (las pruebas lo comprueban).
- Colores y fuentes de la identidad en `ui/tema/` (Michroma, Space Grotesk y JetBrains Mono, en `shared/src/commonMain/composeResources/font/`, convertidas a TTF desde `web/fuentes/`).

## Compilar en la computadora

Necesita JDK 17 (y el SDK de Android para el .apk).

```
cd apps
./gradlew :shared:desktopTest                 # pruebas
./gradlew :desktopApp:run                     # abre la app de escritorio
./gradlew :androidApp:assembleRelease         # .apk en androidApp/build/outputs/apk/release/
./gradlew :desktopApp:createDistributable     # MagnAir.exe con su Java, en desktopApp/build/compose/binaries/main/app/
```

## Tarjetas plegables y ordenables en todas las secciones

- **Plegar:** toda tarjeta con título se pliega tocando el título o la flechita (`componentes/Basicos.kt` → `Tarjeta`). Se recuerda por título en la app (`Plegadas`, claves `plegada_<título>` del `Almacen`; la da `App` con `LocalPlegadas`). Plegada muestra solo el título.
- **Ordenar:** cada sección (Archivos, Control, Consola, Ajustes) tiene **Editar** arriba, igual que Inicio: arrastrar, flechas, pasar de columna, ocultar y agregar. `ui/Ordenable.kt`: `SeccionOrdenable(ctx, clave, tarjetas, predeterminada)` con una lista de `TarjetaSeccion(id, título) { contenido }`; el orden se guarda en la app como `"izquierda|derecha|ocultas"` (clave `seccion_<clave>`, `dominio/Extras.kt` → `OrdenSeccion`). El Inicio usa el mismo `SeccionConEdicion` pero guarda en el módulo. Los movimientos son `Edicion` (genérico; `EdicionInicio` lo usa). Una tarjeta agregada en una versión nueva aparece en su columna de fábrica.
- La Consola se separó en dos tarjetas (Consola y Funciones), una debajo de la otra como antes.

## Ventana de Windows

- **Barra de título del color de la app** (`BarraWindows.kt`): `DwmSetWindowAttribute` por JNA (dependencia `net.java.dev.jna:jna`). Windows 11: color de fondo de la app, texto claro, borde igual; Windows 10: barra oscura del sistema (no deja elegir color). Sigue siendo la ventana normal (arrastrar, acomodar, maximizar).
- **Cerrar a la bandeja** (Ajustes → Esta app → "Al cerrar la ventana"): de fábrica la X la deja junto al reloj (sigue con avisos y timelapse); "Cerrar del todo" la cierra. La primera vez que se esconde avisa cómo cerrarla del todo (clic derecho → Salir). `Plataforma.cerrarABandeja` (null en Android), guardado en `cerrar_a_bandeja`.

## Editar Inicio arrastrando

- **Editar inicio** (arriba en Inicio) pone una capa sobre cada tarjeta: se arrastra con el dedo o el mouse y una línea roja marca dónde va a quedar. En la computadora se puede pasar a la otra columna; en el celular es una sola lista (izquierda y después derecha). Las flechas mueven de a un lugar (útil en listas largas en el celular) y la cruz la oculta. **Listo** guarda en el módulo (config `inicio`, la misma que usa la página).
- La lógica (mover, subir/bajar, cambiar de columna, ocultar, agregar) está en `dominio/Extras.kt` → `EdicionInicio`, con pruebas en `ExtrasTest.editarInicio`. La pantalla (`pantallas/Inicio.kt` → `InicioEditable`, `CapaEdicion`) solo mide dónde está cada tarjeta y calcula el destino.
- Ojo al tocarlo: `pointerInput` no se reinicia al recomponer; las funciones del arrastre se leen con `rememberUpdatedState`. El origen del arrastre se guarda al empezar, porque los límites de la tarjeta se mueven con ella.

## Timelapse

- `timelapse/Timelapse.kt`: sigue a la sesión (lo arranca `AppMagnAir.vigilar()`); cuando empieza una impresión y hay intervalo elegido y cámara configurada, pide a la plataforma un archivo (`Plataforma.crearVideo`) y cada N segundos baja una foto (`DireccionCamara.cuadroPara`: `/shot.jpg` de IP Webcam, rápido) y la agrega. Al volver a "libre" (o si se cierra la app) cierra el video y avisa.
- `timelapse/VideoMjpeg.kt`: AVI con cuadros JPEG tal cual (sin recomprimir), 24 por segundo; completa el encabezado y el índice al terminar. Verificado con ffprobe/ffmpeg y OpenCV: 640×360, 24 fps, todos los cuadros decodifican.
- Dónde queda: Android 10 o más en `Películas/MagnAir` (MediaStore, se ve en la galería); antes, en la carpeta de la app. Windows en `Videos\\MagnAir` del usuario.
- El intervalo (0 = apagado, 10, 30, 60 s) se guarda en la app (`Almacen`, clave `timelapse_intervalo`).

## Historial, Telegram, protecciones y nivelación

- Las mismas funciones que la página: `pantallas/Historial.kt` (historial y estadísticas, en Archivos), `pantallas/Calibracion.kt` → `TarjetaProtecciones` y `TarjetaNivelar` (en Control), `pantallas/Ajustes.kt` → `TarjetaCostos`, `TarjetaTimelapse`; Telegram tiene su propia sección (abajo).
- Las cuentas (gramos, costo, textos) están en `dominio/Extras.kt` → `Filamento`, iguales a las del módulo (`src/seguimiento.cpp`) y la página. Los puntos de nivelación en `NivelarCama`.
- Las fechas del historial se muestran en la hora del equipo (`Plataforma.fechaLocal`).
- La actualización del módulo manda la huella MD5 del archivo (`api/Md5.kt`, probada contra la de Java).

## Sección Telegram

- `pantallas/Telegram.kt`, `Seccion.TELEGRAM` (ícono `Iconos.mensaje`). Tarjetas ordenables (clave "telegram", fábrica `pasos,bot|comandos,saber`): *Cómo crear el bot* (pasos con @BotFather), *Tu bot* (token, chat, avisos, Buscar mi chat, prueba, quitar), *Comandos* y *Bueno saber*.
- La lista de comandos es `dominio/Extras.kt` → `ComandosTelegram`, igual a `COMANDOS[]` de `src/bot.cpp` (lo controla `pruebas/verificar-comandos-telegram.py`).

## Avisos con la app cerrada

- La sesión (código común) detecta el fin de una calibración, un calentamiento, un home o una impresión (`Eventos.detectar`) y lo publica en `Sesion.finalizados`. `AppMagnAir.vigilar()` lo pasa a `Plataforma.notificar()` y avisa si hay trabajo en curso con `Plataforma.trabajoEnCurso()`. Con la app a la vista el aviso sale adentro (barra de abajo).
- **Android:** al pasar a segundo plano con algo en curso arranca `ServicioSeguimiento` (si después cierran la app, sigue consultando por su cuenta; ver Widget más abajo), un servicio en primer plano (tipo `specialUse`; `dataSync` en Android 10 a 13) con un aviso fijo "Siguiendo la impresora". Así Android no corta la consulta al módulo. Cuando termina, `Notificaciones.mostrar()` manda la notificación (canal "Impresión o calibración terminada") y el servicio se detiene. En Android 13 o más la app pide el permiso de notificaciones al abrirse; si se niega, todo funciona igual pero sin avisos afuera. Si el teléfono tiene un ahorro de batería agresivo, conviene sacar a MagnAir de la optimización de batería.
- **Windows:** ícono en la bandeja (junto al reloj) con menú *Abrir / Salir*. Con la ventana minimizada o sin foco, el aviso sale como notificación de Windows. Cerrar la ventana cierra la app.

## Aspecto y visor 3D

- La app se ve como la pantalla de la impresora (firmware mm2): fondo degradé, título "M A G N A 2", reloj y WiFi arriba, íconos de la impresora y barra de avance con brillo. Las temperaturas cambian de color y titilan mientras calientan.
- **Archivos → Visor 3D y editor**: abrí un .gcode (hasta 80 MB) para verlo capa por capa (paredes rojo, relleno amarillo, soportes celeste). Arrastrá para girar, pellizcá o usá la ruedita para acercar.
- Elegí una capa con la barra, una acción (pausa, cambio de filamento, temperatura, ventilador, velocidad o mensaje) y tocá "Agregar". "Mandar a la impresora" sube una copia con los cambios (el archivo original no se toca). Con "marcas para la pantalla" no hacen falta los scripts de Cura de `docs-magna/CURA-PANTALLA.md` (mm2).

## Capturas automáticas

Cada compilación en GitHub saca capturas de todas las pantallas (computadora y celular) contra un módulo simulado y las deja en el artefacto **Capturas-apps**. Sirven para ver cómo queda la app antes de instalarla. En la compu: `python3 pruebas/servidor_simulado.py 8000 &` y `./gradlew :desktopApp:capturas` (quedan en `apps/desktopApp/build/capturas`).

## Firma del .apk

`androidApp/firma/magnair.jks` (alias `magnair`, clave `magnair-impresora`) para que todas las versiones se puedan instalar encima. Si preferís una clave privada, cargá en GitHub (Settings → Secrets → Actions) `MAGNAIR_FIRMA_CLAVE`, `MAGNAIR_FIRMA_ALIAS` y `MAGNAIR_FIRMA_CLAVE_ALIAS`, y poné el archivo en el runner con `MAGNAIR_FIRMA_ARCHIVO`. Ojo: cambiar la clave obliga a desinstalar la app una vez.

## Cómo está ordenada la app

Cuatro pestañas abajo (o al costado en Windows y Linux):

- **Inicio**: lo que está pasando. Si no imprime, muestra la próxima de la cola o la programada.
- **Imprimir**: archivos de la SD, cola de impresión (y programar una a una hora), subir y visor 3D.
- **Máquina**: mover, temperaturas, filamento, dormir, cambio de filamento y nivelar. La tarjeta "Más" lleva a Avanzado (calibración, copia de la calibración, ajustes de Marlin, ajustes en vivo, memoria y protecciones), Consola y Rollos.
- **Más**: Historial, Telegram, Cámara y timelapse, Pantalla de la impresora, Módulo WiFi, Esta app y Acerca de.

Las subpantallas tienen una flecha para volver.

## Cola, rollos, tiempo real y copia de la calibración

- La cola vive en el módulo (hasta ~200 caracteres de nombres). La siguiente no arranca sola porque hay que sacar la pieza de la cama; se arranca desde Inicio, Imprimir → Cola, la página o `/siguiente`.
- Los rollos se guardan en la app (no en el módulo). El rollo marcado como puesto descuenta los gramos de cada impresión nueva del historial.
- El tiempo estimado del slicer se lee del `.gcode` al subirlo desde la app (`;TIME:` de Cura, `estimated printing time` de PrusaSlicer y Orca). Con 1 o más impresiones terminadas la app calcula cuánto más o menos tarda tu máquina.
- La copia de la calibración es un `.gcode` de texto con M92, M301, M304, M306, M593, M851/M206, M900, M201, M203, M204 y M500. Al cargarla la app manda solo esas líneas.

## Android: widget, accesos directos y compartir

- Widget: mantener apretado el escritorio → Widgets → MagnAir. Se actualiza así:
  - con la app abierta o en segundo plano, con cada lectura de la sesión;
  - con la app cerrada y una impresión o calibración en curso, desde `ServicioSeguimiento`, que sigue solo (pregunta cada 20 s, actualiza widget y notificación, avisa al terminar y se detiene; si el módulo no contesta 30 minutos, se rinde);
  - al tocar ↻, al ponerlo y cada 30 minutos (lo mínimo que deja Android): una consulta al último módulo usado (`ClienteMagnAir.estadoUnaVez`).
  Solo funciona en la misma red que la impresora; afuera queda lo último que supo, con la hora.
- Accesos directos: mantener apretado el ícono de MagnAir.
- Compartir un `.gcode` (desde Drive, Telegram, el explorador…) → MagnAir: pregunta y lo sube; se puede marcar para que empiece a imprimir.

## Linux

`./gradlew :desktopApp:packageDeb` arma `magnair_X_amd64.deb`. Se instala con `sudo apt install ./magnair-X.deb` y queda en el menú como MagnAir. Los archivos exportados van a `~/Documentos/MagnAir` (o `~/Documents/MagnAir`).

## Varias impresoras

Arriba de cada sección está la impresora abierta ("Taller · Magna 2 400D · placa V1.3"). Tocándola aparece la lista de las que usaste (hasta 10): tocás una y la app se pasa a esa; con **Nombre** le ponés cómo se llama; con **Quitar** la sacás de la lista; **Agregar impresora** vuelve a la pantalla de buscar módulos. Se mira una a la vez: los avisos, el widget y el timelapse son de la abierta. Cada módulo tiene su propio bot de Telegram.
