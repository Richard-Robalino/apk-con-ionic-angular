# Ionic Angular → Android APK (con ícono personalizado, Splash Screen y AndroidManifest)

> Guía paso a paso para dejar lista una APK **funcional** hecha con **Ionic + Angular + Capacitor**, con **ícono** y **splash** personalizados, edición de **AndroidManifest.xml**, y todo documentado. Incluye comandos, fragmentos de configuración, estructura sugerida de repo y referencias oficiales.

---


## 🧰 Requisitos previos

* Node.js LTS y npm
* Ionic CLI y Capacitor
* Android Studio (incluye SDKs, emulador y Gradle wrapper)
* Java JDK compatible con tu versión de Android Gradle Plugin

```bash
npm i -g @ionic/cli
```

> Tip: verifica las versiones ejecutando `ionic --version` y abre Android Studio al menos una vez para que instale componentes.

---

## 1) Crear (o usar) el proyecto Ionic Angular

```bash
# (opcional) crear app nueva
ionic start myapp tabs --type=angular --capacitor
cd myapp

# agregar plataforma Android
ionic capacitor add android
```

* Construye los assets web:

```bash
ionic build
ionic capacitor copy android
```

> También puedes usar `ionic capacitor build android` que hace *build + copy + open*.

---

<img width="926" height="206" alt="image" src="https://github.com/user-attachments/assets/919df300-60c6-4063-95b4-ebe6531e108d" />


## 2) Ícono y Splash Screen (Capacitor)

Desde Capacitor 5/6/7 se recomienda **@capacitor/assets** para generar todos los tamaños de íconos y splash automáticamente.

### 2.1 Instalar herramienta

```bash
npm i -D @capacitor/assets
```

### 2.2 Colocar artes fuente (mínimos sugeridos)

```
assets/
├─ icon-only.png          # 1024x1024 (sin fondo)
├─ icon-foreground.png    # 1024x1024 (si usas capas)
├─ icon-background.png    # 1024x1024 (color/fondo)
├─ splash.png             # 2732x2732 (claro)
└─ splash-dark.png        # 2732x2732 (oscuro)
```

> Consejo: deja suficiente margen (padding) en `splash*.png` para que no se corte en pantallas altas.

### 2.3 Generar assets para Android (y iOS si aplica)

```bash
npx capacitor-assets generate
# o solo Android
npx capacitor-assets generate --android
```

Esto colocará los recursos en `android/app/src/main/res/` y actualizará lo necesario.

### 2.4 Configurar comportamiento del Splash (opcional)

En `capacitor.config.ts`:

```ts
/// <reference types="@capacitor/splash-screen" />
import { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  appId: 'com.ejemplo.miapp',
  appName: 'Mi App',
  webDir: 'www',
  plugins: {
    SplashScreen: {
      launchAutoHide: true,           // o false si quieres ocultarlo manualmente
      launchShowDuration: 1200,       // ms (si autoHide=true)
      launchFadeOutDuration: 200,     // Android 12+
      backgroundColor: '#FFFFFFFF',   // color de fondo
      showSpinner: false
    }
  }
};
export default config;
```

En tu `main.ts` (si usas `launchAutoHide: false`):

```ts
import { SplashScreen } from '@capacitor/splash-screen';

// …cuando la app esté lista
await SplashScreen.hide();
```

> **Android 12+**: el splash del sistema usa un ícono centrado + color de fondo (no imagen full-screen). Asegúrate de definir buen **icono** y **background**.

Añade capturas a `/docs/img/` por ejemplo:

```md
![Ícono generado](docs/img/icon-generated.png)
![Splash en Android 13](docs/img/splash-android13.png)
```

---

## 3) AndroidManifest.xml (ubicación y ediciones comunes)

**Ruta del manifiesto (proyecto Capacitor):** `android/app/src/main/AndroidManifest.xml`

### 3.1 Cambiar *applicationId* (paquete)

En `android/app/build.gradle`:

```gradle
android {
  defaultConfig {
    applicationId "com.ejemplo.miapp" // <— cámbialo
  }
}
```

### 3.2 Cambiar nombre visible de la app

`android/app/src/main/res/values/strings.xml`

```xml
<string name="app_name">Mi App</string>
<string name="title_activity_main">Mi App</string>
```

### 3.3 Permisos (ejemplo: red)

En `AndroidManifest.xml` dentro de `<manifest>`:

```xml
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.INTERNET" />
```

### 3.4 Intent filter de lanzamiento (ya viene por defecto)

```xml
<intent-filter>
  <action android:name="android.intent.action.MAIN" />
  <category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
```

> Plugins de Capacitor suelen indicar qué permisos agregar; revísalos antes de publicar.

---

## 4) Compilar APK

### 4.1 Abrir Android Studio

```bash
ionic capacitor open android
```

* Menú **Build → Make Project** (debug) o **Build → Generate Signed Bundle / APK…** (release)

### 4.2 Línea de comandos (Gradle wrapper)

Desde la carpeta `android/` del proyecto:

```bash
# APK de debug
./gradlew assembleDebug

# APK de release (requiere firma)
./gradlew assembleRelease
```

**Ubicación típica de la salida:**

```
android/app/build/outputs/apk/debug/app-debug.apk
android/app/build/outputs/apk/release/app-release.apk
```

### 4.3 Firma para `release`

1. Crear keystore (una sola vez):

```bash
keytool -genkey -v -keystore release.keystore -alias mi_alias \
  -keyalg RSA -keysize 2048 -validity 10000
```

2. Configura `android/app/build.gradle`:

```gradle
android {
  signingConfigs {
    release {
      storeFile file("release.keystore")
      storePassword System.getenv("STORE_PASSWORD")
      keyAlias System.getenv("KEY_ALIAS")
      keyPassword System.getenv("KEY_PASSWORD")
    }
  }
  buildTypes {
    release {
      signingConfig signingConfigs.release
      minifyEnabled false
      shrinkResources false
    }
  }
}
```

3. Exporta variables y compila:

```bash
export STORE_PASSWORD=********
export KEY_ALIAS=mi_alias
export KEY_PASSWORD=********
cd android && ./gradlew assembleRelease
```

> Para Play Store hoy se prioriza **.aab** (App Bundle), pero puedes entregar **APK** funcional para pruebas internas o distribución directa.

---

## 5) Estructura sugerida del repositorio

```
myapp/
├─ android/
├─ src/
├─ www/
├─ assets/                 # artes fuente para @capacitor/assets
├─ docs/img/               # imágenes del README
├─ release/                # aquí sube tu APK (app-debug.apk o app-release.apk)
├─ capacitor.config.ts
├─ package.json
└─ README.md
```

---

## 6) Troubleshooting rápido

* **Splash no cambia**: vuelve a generar `npx capacitor-assets generate`, borra app previa del dispositivo/emulador, limpia cachés y reconstruye (`Build → Clean Project` en Android Studio).
* **Gradle OK pero no hay APK**: verifica si generaste **Bundle (.aab)** en vez de APK; revisa carpetas `outputs/apk/` vs `outputs/bundle/`.
* **JDK incompatible**: en Android Studio ajusta *Settings → Build Tools → Gradle → Gradle JDK*.
* **Permisos faltantes**: agrega `<uses-permission …>` en el `AndroidManifest.xml` según cada plugin.

---


## 🔗 Referencias útiles (actualizadas)

* Generar íconos y splash con **@capacitor/assets** (guía oficial)

  * [https://capacitorjs.com/docs/guides/splash-screens-and-icons](https://capacitorjs.com/docs/guides/splash-screens-and-icons)
  * [https://github.com/ionic-team/capacitor-assets](https://github.com/ionic-team/capacitor-assets)
* Plugin **Splash Screen** (API y configuración)

  * [https://capacitorjs.com/docs/apis/splash-screen](https://capacitorjs.com/docs/apis/splash-screen)
* Configuración Android en Capacitor (manifest, packageId, permisos)

  * [https://capacitorjs.com/docs/android/configuration](https://capacitorjs.com/docs/android/configuration)
* CLI `ionic capacitor build`/`open`

  * [https://ionicframework.com/docs/cli/commands/capacitor-build](https://ionicframework.com/docs/cli/commands/capacitor-build)
  * [https://ionicframework.com/docs/cli/commands/capacitor-open](https://ionicframework.com/docs/cli/commands/capacitor-open)
* Compilar desde línea de comandos (Gradle)

  * [https://developer.android.com/build/building-cmdline](https://developer.android.com/build/building-cmdline)
* Firmar aplicación (clave y conceptos)

  * [https://developer.android.com/studio/publish/app-signing](https://developer.android.com/studio/publish/app-signing)
* Dónde quedan los artefactos de compilación

  * [https://developer.android.com/build/build-for-release](https://developer.android.com/build/build-for-release)



