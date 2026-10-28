# UIN'T — mod de Windhawk

Personalización del escritorio de Windows 11 con mods de Windhawk. El trabajo principal es
**UIN'T** (`@id uint`), un fork en C++ del mod **Dynamic Island for Windows** de devcode90 (MIT).
Repo público: https://github.com/fsanchezux/UINT (cuenta `fsanchezux`, no la activa en `gh`:
usar `GH_TOKEN=$(gh auth token -u fsanchezux)`).

## Archivos

- `uint.wh.cpp` — el mod actual (C++, ~17.500 líneas). Es el que se pega en Windhawk.
  Al publicar una versión, subir `@version` y apuntarla en `CHANGELOG.md`.
- El primer commit del repo es el fork tal como estaba antes de los cambios (v1.3.1); `git diff`
  contra él muestra qué se ha añadido. Copia local sin versionar en `reference/` (ignorada).
- `settings/*.yaml` — ajustes actuales (pestaña **Avanzado** de cada mod en Windhawk):
  - `uint.yaml`
  - `windows-11-taskbar-styler.yaml` (dock centrado, sin bandeja ni botón Inicio)
  - `taskbar-height-and-icon-size.yaml` (altura 60, iconos 32 para que salgan nítidos)

## Cómo se prueba

No hay compilador local: el mod se compila **desde Windhawk** (Editar mod → pegar el código →
Compilar mod). Windhawk usa su propio toolchain (clang/mingw con C++/WinRT). Los errores de
compilación hay que copiarlos desde Windhawk. El log del mod (`Wh_Log`) se ve en Windhawk.

Para comprobar sintaxis sin abrir Windhawk se puede usar su propio clang con el archivo entero
(~25 s, sin stubs):

```
"/c/Program Files/Windhawk/Compiler/bin/clang++.exe" -fsyntax-only -x c++ -std=c++23 \
  -target x86_64-w64-mingw32 -DUNICODE -D_UNICODE -DWINVER=0x0A00 -D_WIN32_WINNT=0x0A00 \
  -D_WIN32_IE=0x0A00 -DNTDDI_VERSION=0x0A000008 -D__USE_MINGW_ANSI_STDIO=0 -DWH_MOD \
  '-DWH_MOD_ID=L"uint"' -include windhawk_api.h \
  -Wno-pragma-pack -Wno-pragma-system-header-outside-header -Wall uint.wh.cpp
```

El archivo usa finales de línea CRLF.

## Convenciones del usuario

- Habla en español. Respuestas concisas.
- **Cada vez que pida un cambio de configuración, devolver el YAML / código completo**, no solo el
  fragmento modificado.
- Interfaz del mod en español (textos nuevos escritos en castellano directamente).
- Tiene Resource Redirect activo para el pack de iconos del sistema.

## Qué se ha añadido al fork (todo propio, no upstream)

### Barra superior al maximizar (siempre activa, `MaximizedTopBarHeight` = 26)
- Ya no es opción: `maximizedTopBar` se fuerza a `true` en `LoadSettings()` y se quitó del
  esquema de ajustes y de la tarjeta Widgets.
- Si hay una ventana maximizada en el monitor de la isla, la pastilla se oculta y aparece una
  barra de lado a lado registrada como **AppBar** (`SHAppBarMessage` ABM_NEW / ABM_SETPOS), así que
  las ventanas maximizadas quedan debajo. Al no quedar maximizadas, se quita (ABM_REMOVE).
- Debounce de 2 ticks (timer de 300 ms en la ventana de la barra, hilo de render).
- Layout en 3 grupos que no se solapan (`TopBarLayout`): primero la derecha, el reloj se desplaza
  a la izquierda si choca, y la izquierda solo usa lo que queda hasta la alerta/reloj.
  - Izquierda: las **4 apps usadas más recientemente** (`g_topBarRecentApps`, MRU que actualiza
    `TopBarTrackForeground`; `g_topBarApps` = apps abiertas, una por ejecutable, las de Store una
    por ventana). Marca acento = app delante, marca gris = la anterior; botón ⇄ (`kBarSwitchApp`)
    vuelve a la anterior; botón … (`kBarMoreApps`) hace aparecer el dock con Win+T. Clic en la
    app delante → siguiente ventana o minimizar. Después, chip de la nota fijada.
  - Centro: hora. Clic → **desplegable**: la isla expandida aparece bajo la barra
    (`g_barDropdownOpen`, cuenta como `pinned`; clic fuera de isla/barra/paneles la recoge, se
    comprueba en el bucle de render). Al abrir crece desde la barra hacia abajo (los muelles
    arrancan en una franja de ~120×2) y al cerrar se recoge hacia la barra antes de aparcarse
    (`g_barDropdownFolding`). A su izquierda, chip de **alerta** (`TopBarAlert`: copiado,
    Bloq Mayús/Num, dispositivo, Bluetooth, No molestar, volumen, notificación, batería baja).
  - Derecha: atajos a las tarjetas activas (`kBarWidget`: Calendario, Tiempo, Notas, Pomodoro;
    Calendario y Tiempo abren la tarjeta de hora en esa vista) que abren el desplegable en esa tarjeta, y luego Pomodoro en marcha, red,
    Bluetooth, volumen, batería y Centro de control.
- Red y Bluetooth abren un **panel propio** (el mismo `g_ccHwnd` en `g_ccMode` Wifi/Bluetooth,
  `CcDrawMini`): interruptor, lista de redes (WLAN API cargada en runtime, `CcWlan`; conecta con
  perfiles guardados, las nuevas van a `ms-availablenetworks:`) o de dispositivos vinculados
  (`CcQueryBtDevices`, WinRT; conectar no tiene API pública → Configuración).
  En Windows 11 24H2+ la lista de redes necesita permiso de Ubicación (`wifiDenied`).
- Si otra copia del mod (el original) dibuja su pastilla, se oculta mientras está la barra.
- La detección de pantalla completa (`IsForegroundFullscreen`) solo mira el monitor de la isla.

### Centro de control (estilo macOS)
- Panel layered con Direct2D (`CcRender`), se abre desde la barra o desde la tarjeta del notch.
- Wi-Fi/Bluetooth vía `Windows.Devices.Radios` (hilo worker MTA `CcWorkerProc`), red con
  Network List Manager, brillo por DDC/CI (dxva2 cargado dinámicamente; no hay WMI para
  portátiles), volumen con `IAudioEndpointVolume`, media, Segundo plano (taskmgr), Ajustes.
- No molestar y Luz nocturna solo abren Configuración (no hay API pública para cambiarlos).

### Tarjetas del notch (pestañas)
- Las pestañas van por **identificador** (`enum TabId`, `TabOrder()`), no por posición.
  `g_idleTab` guarda el TabId seleccionado y se persiste en `Wh_SetIntValue("SelectedTab")`.
  Ya no se resetea al recoger el notch ni al cambiar de canción.
- `g_tabsMediaMode` lo fija la superficie expandida que se pintó la última (media o idle).
- Orden: Primaria (media o reloj) → Hora (solo con media) → Centro de control (reutiliza el
  ajuste `HardwareMonitorModule`) → Notas → Pomodoro → Widgets.
- **Tarjetas fusionadas** (`kTabCalendar`, `kTabWeather` y `kTabFileTray` siguen en el enum para
  los atajos de la barra y selecciones antiguas, pero nunca están en `TabOrder()`):
  - Hora → Calendario → Tiempo: la tarjeta de hora (`TimeCardTab()`: Primaria en reposo,
    `kTabClock` con media) rota con cada clic entre las vistas activas (`TimeCardViews()`,
    `g_timeCardView`, persistido en `TimeCardView`; `DrawTimeCard`).
  - Notas + archivos en **una sola lista** (`DrawNotesCard` / `NotesHandleClick`): primero los
    archivos soltados en la isla (más nuevo arriba), luego las notas, 3 por página. Fila de
    archivo: copiar (CF_HDROP + ruta, `CopyFileToClipboard`), ✕ = quitar de la lista (no borra
    del disco), resto = abrir. Se aceptan archivos si Notas o Bandeja están activas; se guardan por
    ruta en `TrayFiles` (`FileTraySave` / `FileTrayLoad`, se saltan los que ya no existen).
- **Notas**: guardadas con `Wh_SetStringValue("Notes")` separadas por `\x1E`; `PinnedNote` es el
  índice. Editor = popup Win32 con EDIT. Nota fijada = tarjeta arriba a la derecha (sticky).
- **Pomodoro**: estado en `g_pomo`, se avanza en `PomoTick()` (timer de la barra). Notificación
  en la isla + MessageBeep. Barra fina en la pastilla recogida solo si está en marcha.
- **Widgets**: activa/desactiva tarjetas; los overrides (`Widget.*` con `Wh_SetIntValue`) se
  aplican al final de `LoadSettings()` y tienen prioridad sobre los ajustes de Windhawk.
- Recogido con otra tarjeta seleccionada, la pastilla muestra esa tarjeta en compacto en vez de
  la canción (`SelectedCardIsNotMedia`, `CollapsedCardText`).

### Otros ajustes propios
- **Foco al pasar el ratón** (`FocusFollowsMouse`, `...Raise`, `...Delay`): activa el "active window
  tracking" de Windows con `SystemParametersInfo` solo para la sesión (sin `SPIF_UPDATEINIFILE`) y
  restaura los valores previos al desactivarlo o al descargar el mod (`RestoreFocusFollowsMouse`).
- **Paleta propia** (`Themes.CustomPalette`): hasta 5 hex separados por comas — fondo, texto,
  secundario, acento, borde. Si el primero es válido, manda sobre el preset y los colores sueltos.

### 1.1.0 (Linear FSA-115..121)
- **Paleta UIN'T** (`ThemePreset::Uint`, `kDefaultThemeIndex`, por defecto): fondo #170312. Acento de la isla
  #F62B0A (`AccentColorMode` custom por defecto). `SecondaryAccent()` (#FF9A44, `Themes.SecondaryAccentHex`)
  es el acento de barra superior, centro de control, editor de notas y nota fijada (antes el acento de Windows).
- **Forma**: notch por defecto (también con `ShapeStyle` vacío), radio inferior `kNotchRadius` = 14.
- **Fuente**: `FontFamily` vacío → `InstalledInterFamily()` (Inter / Inter Variable si están instaladas).
  Si falta, `FontThreadProc` → `InstallInterForUser()` la baja de `raw.githubusercontent.com/fsanchezux/UINT/main/
  fonts/InterVariable.ttf` (en el repo, con `fonts/LICENSE-Inter.txt`), la copia a
  `%LOCALAPPDATA%\Microsoft\Windows\Fonts`, la registra en HKCU (`...\CurrentVersion\Fonts`) y avisa al hilo de
  render con `WM_APP_FONT_INSTALLED`.
- **Centro de control**: 4 atajos abajo en 2×2 (Energía, Segundo plano, Notificaciones = Win+N, Ajustes);
  `kCcHeight` 462. La fila de modo avión dibuja su círculo como las otras (su hit es toda la fila).
- **Pomodoro**: `kPomoYellow`, `DrawPomoRing()`. Pastilla recogida (idle) = anillo a la izquierda
  (`IdleStripLayout::kPomoRingReserve`); barra superior = anillo con Direct2D sobre el bitmap GDI
  (`TopBarDrawPomoRing`), tiempo solo con `g_topBarPomoHover`. Tarjeta: progreso por el borde de la isla
  (`CreateOutlinePath` + dash), hora de fin con campana, botones play/pausa y parar (`PomoButtonOffset`).
- **Archivos**: `IslandDropTarget` (OLE, `RegisterDragDrop`; `WM_DROPFILES` solo de reserva) abre Notas
  en `DragEnter`. Arrastrar una fila de archivo (`NotesFileAt`, umbral `SM_CXDRAG`) → `FileTrayDragOut`
  (`SHDoDragDrop` con el data object del shell). `addedAt` por archivo (guardado en `TrayFiles` como
  `ruta\x1Fsegundos`); `FileTrayPrune()` cada minuto quita los de >24 h o que ya no existen.

### Clics en la isla
- Clic en el notch recogido → se despliega (como hover).
- Desplegado: solo la portada/título abre la app que suena.
- Doble clic → abre la app que suena (en las tarjetas interactivas cuenta como dos clics).
- Fijar abierto sigue en el menú del botón derecho.

## Pendiente / sin verificar

- Compila sin errores con el clang de Windhawk (`-fsyntax-only -Wall`), pero nada de lo añadido
  se ha probado aún en el equipo real.
- Linear (proyecto UI'NT): FSA-5..8 y FSA-84..90 en Done (2026-10-22); FSA-115..121 en In Review
  (2026-10-28, sin probar en el equipo). El historial de cada cambio está en sus descripciones y comentarios.
- Además de `-fsyntax-only`, se puede compilar y enlazar una DLL de prueba añadiendo `-shared -O2`,
  `"/c/Program Files/Windhawk/Engine/<versión>/64/windhawk.lib"` y las librerías de `@compilerOptions`.
- Glifos de Segoe Fluent Icons elegidos de memoria; alguno puede salir distinto.
- Posible duplicado de pastilla si el mod original sigue activo junto al fork (desactivarlo).
