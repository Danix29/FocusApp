# FocusLock

App Android nativa (Kotlin) que bloquea las apps que te distraen. Para volver a usarlas tienes que ganarle a la casa en un juego — Blackjack, Cara o Cruz o Alto/Bajo. Si ganas, te llevas un rato de uso libre. Si pierdes, la próxima espera obligatoria se multiplica. Teléfono, SMS y WhatsApp nunca se bloquean, ni siquiera en plena penalización.

## Índice

- [Qué hace](#qué-hace)
- [Características](#características)
- [Cómo funciona](#cómo-funciona)
- [Los juegos](#los-juegos)
- [Requisitos](#requisitos)
- [Compilar desde el código](#compilar-desde-el-código)
- [Instalar en un móvil](#instalar-en-un-móvil)
- [Primer uso](#primer-uso)
- [Estructura del proyecto](#estructura-del-proyecto)
- [Configuración](#configuración)
- [Permisos y privacidad](#permisos-y-privacidad)
- [Limitaciones conocidas](#limitaciones-conocidas)
- [Licencia](#licencia)

## Qué hace

1. Eliges qué apps quieres bloquear (redes sociales, juegos, lo que distraiga).
2. Cuando abres una de esas apps, FocusLock la cubre con una pantalla de bloqueo.
3. Para desbloquear, juegas una partida. Si ganas, esa app queda libre durante un rato. Si pierdes, no puedes reintentar hasta pasar una penalización — y la siguiente penalización será más larga que la anterior.
4. Teléfono, SMS y WhatsApp están excluidos del bloqueo siempre, para que nunca te quedes sin forma de contactar a alguien en una emergencia.

## Características

- Tres juegos jugables: **Blackjack**, **Cara o Cruz** y **Alto/Bajo**, todos con una ventaja ligera a favor de la casa.
- Penalización que **se multiplica en cada derrota** (configurable) y se reinicia cada día a una hora fija.
- Acceso de emergencia siempre disponible a Teléfono, SMS y WhatsApp, resuelto en tiempo de ejecución (no depende de nombres de paquete fijos por fabricante).
- **Cero permisos peligrosos**: nada de superposición sobre otras apps, nada de acceso al uso de apps, nada de llamadas, nada de internet.
- Historial local de partidas (victorias/derrotas y evolución de la penalización).
- Interfaz simple en español, tema oscuro fijo.

## Cómo funciona

El bloqueo se basa en un **Servicio de Accesibilidad** (`AccessibilityService`) que escucha cambios de ventana en primer plano. Cuando detecta que se abrió una app marcada como bloqueada (y no está en la lista de apps siempre permitidas), lanza una `Activity` a pantalla completa que la cubre — así se evita pedir el permiso de "superposición sobre otras apps".

No hay servicio en segundo plano, `WorkManager`, `AlarmManager` ni arranque automático: el estado (libre / bloqueado, penalización actual, próximo intento disponible) se calcula al vuelo a partir de marcas de tiempo guardadas en `SharedPreferences`, cada vez que se consulta.

## Los juegos

- **Blackjack**: baraja continua de 6 mazos (sin conteo de cartas viable), el Blackjack paga 1:1 en vez de 3:2, el crupier pide carta en "17 blando" y los empates los gana la banca.
- **Cara o Cruz** y **Alto/Bajo**: el resultado se decide primero por una probabilidad de victoria configurable (45% por defecto) y después se genera el detalle narrativo (qué cara salió, qué número salió) de forma consistente con ese resultado.

La ventaja de la casa es intencionada pero deliberadamente ligera, no aplastante.

## Requisitos

- Android 8.0 (API 26) o superior.
- Para compilar: JDK 17, Android SDK (`platform-34`, `build-tools;34.0.0`), Gradle (incluido vía wrapper).

## Compilar desde el código

```bash
git clone <url-de-este-repositorio>
cd FocusLock
```

Crea `local.properties` en la raíz del proyecto (no se versiona) apuntando a tu Android SDK:

```properties
sdk.dir=C:\\Android\\Sdk
```

Y compila:

```bash
.\gradlew.bat assembleDebug
```

El APK queda en `app\build\outputs\apk\debug\app-debug.apk`.

> Si Gradle usa un JDK distinto al 17 y falla con `Unsupported class file major version`, fija la ruta del JDK 17 en `gradle.properties` con `org.gradle.java.home=<ruta-al-jdk-17>`.

## Instalar en un móvil

**Con cable y depuración USB:**

1. En el móvil: Ajustes → Acerca del teléfono → pulsa 7 veces "Número de compilación" para activar Opciones de desarrollador.
2. Ajustes → Opciones de desarrollador → activa "Depuración USB".
3. Conecta el móvil y acepta "¿Confiar en este ordenador?".
4. `adb install -r app\build\outputs\apk\debug\app-debug.apk`

**Sin cable:**

1. Copia `app-debug.apk` al móvil (Drive, USB, lo que sea).
2. Ábrelo desde el móvil y permite "instalar apps desconocidas" para esa app de origen cuando lo pida.

## Primer uso

1. Abre FocusLock. Si falta el permiso de accesibilidad, te lo avisa en la pantalla principal.
2. Pulsa **"Activar permiso"** → busca FocusLock en Ajustes de Accesibilidad del sistema → actívalo → confirma el diálogo.
3. Pulsa **"Elegir apps a bloquear"** y marca las que quieras restringir. Teléfono, SMS, WhatsApp y el launcher no aparecen en la lista porque nunca se bloquean.
4. (Opcional) Ajusta probabilidad de ganar, minutos libres, multiplicador de penalización, etc. en **"Ajustes"**.

## Estructura del proyecto

```
app/src/main/java/com/focuslock/app/
├── FocusLockApp.kt            # Application: inicializa Prefs, HistoryDb, RestrictionManager
├── data/
│   ├── Prefs.kt                # Configuración y estado persistente (SharedPreferences)
│   └── HistoryDb.kt             # Historial de partidas y eventos (SQLite plano)
├── restrict/
│   └── RestrictionManager.kt    # Máquina de estados: libre / bloqueado / penalización
├── games/
│   ├── QuickGame.kt             # Interfaz + Cara o Cruz + Alto/Bajo
│   └── BlackjackGame.kt         # Motor de Blackjack
├── service/
│   ├── AppBlockerService.kt     # AccessibilityService: detecta y bloquea
│   └── SafeApps.kt              # Resuelve qué apps nunca se bloquean
└── ui/
    ├── MainActivity.kt          # Inicio / estado actual
    ├── AppSelectionActivity.kt  # Elegir apps a bloquear
    ├── SettingsActivity.kt      # Ajustes
    ├── HistoryActivity.kt       # Historial
    ├── BlockOverlayActivity.kt  # Pantalla de bloqueo
    ├── BlackjackActivity.kt     # UI del Blackjack
    ├── SimpleGameActivity.kt    # UI de Cara o Cruz / Alto-Bajo
    └── EmergencyActivity.kt     # Teléfono / SMS / WhatsApp
```

## Configuración

Valores ajustables desde **Ajustes**, con su valor por defecto:

| Parámetro | Clave interna | Por defecto |
|---|---|---|
| Probabilidad de ganar (Cara/Cruz, Alto-Bajo) | `winProbabilityPercent` | 45% |
| Minutos libres al ganar | `freeWindowMinutes` | 10 min |
| Multiplicador de penalización al perder | `penaltyMultiplierPercent` | 180% (x1.8) |
| Penalización base | `basePenaltyMinutes` | 5 min |
| Tope máximo de penalización | `maxPenaltyMinutes` | 120 min |
| Hora de reinicio diario | `dailyResetHour` | 4 (04:00) |

El Blackjack no usa `winProbabilityPercent` — su ventaja para la casa viene de las reglas (pago 1:1, empates a favor de la banca, crupier pide en 17 blando).

## Permisos y privacidad

FocusLock no declara ningún permiso peligroso: nada de `SYSTEM_ALERT_WINDOW`, `PACKAGE_USAGE_STATS`, `QUERY_ALL_PACKAGES`, `CALL_PHONE` ni `INTERNET`. El único permiso del sistema involucrado es `BIND_ACCESSIBILITY_SERVICE`, protegido por el propio sistema y que el usuario activa y puede desactivar en cualquier momento desde Ajustes.

Todos los datos (configuración, estado e historial de partidas) se guardan solo en el dispositivo, en `SharedPreferences` y una base SQLite local. La app no se conecta a internet ni envía nada a ningún servidor.

## Limitaciones conocidas

- Es una herramienta de autodisciplina, no un control parental a prueba de manipulaciones: cualquiera puede desactivar el Servicio de Accesibilidad desde Ajustes y desbloquearlo todo.
- Al detectar la apertura de una app bloqueada hay un margen mínimo (el tiempo que tarda la accesibilidad en notificar y lanzar la pantalla de bloqueo) en el que la app puede verse una fracción de segundo.
- El APK se firma con la clave de depuración (debug); está pensado para instalación personal, no para publicar en Google Play tal cual.

## Licencia

Este repositorio no tiene licencia definida todavía. Si quieres permitir reutilización o contribuciones externas, añade un archivo `LICENSE` con la que prefieras (MIT, Apache 2.0, etc.).
