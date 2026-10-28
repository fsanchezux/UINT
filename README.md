# UIN'T

Mod de [Windhawk](https://windhawk.net) para Windows 11: una isla tipo Dynamic Island que reacciona a la música,
las descargas, el portapapeles, la batería y más. Cuando maximizas una ventana, se convierte en una barra superior.

Es un fork de [Dynamic Island for Windows](https://github.com/devcode90/Dynamic-Island-for-Windows) de
Himanshu (devcode90). Todo lo del original sigue funcionando. Este fork añade lo siguiente.

## Qué añade este fork

- **Barra superior con ventanas maximizadas.** La pastilla se oculta y aparece una barra de lado a lado
  (registrada como AppBar, así que las ventanas quedan debajo) con:
  - las 4 apps usadas más recientemente, y un botón para volver a la anterior;
  - la hora, que al hacer clic despliega la isla bajo la barra;
  - avisos rápidos (copiado, Bloq Mayús, volumen, batería baja, notificaciones…);
  - atajos a Calendario, Tiempo, Notas y Pomodoro, más red, Bluetooth, volumen, batería y Centro de control.
- **Centro de control estilo macOS.** Wi-Fi, Bluetooth, brillo (DDC/CI), volumen y multimedia en un panel, con
  listas de redes y de dispositivos Bluetooth.
- **Tarjetas nuevas:**
  - **Notas**, con una nota fijada que se queda visible.
  - **Archivos** soltados en la isla, en la misma lista que las notas.
  - **Pomodoro**.
  - **Widgets**, para activar o desactivar tarjetas.
  - La tarjeta de **hora** rota entre reloj, calendario y tiempo.
- **Foco al pasar el ratón** (opcional) y **paleta de colores propia**.
- Interfaz en castellano.

## Instalación

1. Instala [Windhawk](https://windhawk.net).
2. En Windhawk: **Explorar → Crear un nuevo mod**.
3. Borra la plantilla, pega el contenido de [`uint.wh.cpp`](uint.wh.cpp) y pulsa **Compilar mod**.
4. Opcional: en la pestaña **Ajustes** del mod, abre la vista de texto y pega [`settings/uint.yaml`](settings/uint.yaml).

Si tienes instalado el mod original (*Dynamic Island for Windows*), desactívalo para que no salgan dos islas.

### Para que las notificaciones funcionen

Activa **Configuración → Privacidad y seguridad → Notificaciones → "Permitir que las aplicaciones accedan a las
notificaciones"**.

### Ajustes de la barra de tareas (opcional)

En [`settings/`](settings) también están los ajustes que uso para tener la barra de tareas como un dock centrado.
Son para estos dos mods, que se instalan desde el catálogo de Windhawk:

- `windows-11-taskbar-styler.yaml` → *Windows 11 Taskbar Styler*
- `taskbar-height-and-icon-size.yaml` → *Taskbar height and icon size*

## Desarrollo

Todo el mod está en un único archivo, [`uint.wh.cpp`](uint.wh.cpp), en C++ con Direct2D y C++/WinRT. Para probar
un cambio, pégalo en el editor de Windhawk y compílalo. Para comprobar solo la sintaxis, puedes usar el clang que
trae Windhawk (el comando está en [`CLAUDE.md`](CLAUDE.md)).

Los issues y los pull requests son bienvenidos.

## Créditos y licencia

- Mod original: [Dynamic Island for Windows](https://github.com/devcode90/Dynamic-Island-for-Windows), de
  Himanshu (devcode90) y sus colaboradores (ver los créditos dentro del mod).
- Fork: [fsanchezux](https://github.com/fsanchezux).

Licencia [MIT](LICENSE).
