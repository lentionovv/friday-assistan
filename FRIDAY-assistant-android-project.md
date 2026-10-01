# FRIDAY — голосовой ассистент Android (стиль Marvel)

Полный проект: все файлы, дизайн «дуговой реактор», голос в стиле FRIDAY, автоматическая сборка APK через GitHub Actions.

---

## СОДЕРЖИМОЕ ЭТОГО ФАЙЛА

1. [Честно про голос FRIDAY из Marvel](#1-честно-про-голос-friday-из-marvel)
2. [Как получить готовый APK (2 пути)](#2-как-получить-готовый-apk)
3. [Структура проекта](#3-структура-проекта)
4. [Файлы проекта](#4-файлы-проекта)
   - GitHub Actions workflow (сборка APK без Android Studio)
   - Gradle-файлы
   - Манифест
   - Ресурсы (тёмный дизайн с циановым свечением)
   - Kotlin-код: MainActivity, AssistantService, CommandProcessor, FridayTts
5. [Настройка голоса пошагово](#5-настройка-голоса)

---

## 1. ЧЕСТНО ПРО ГОЛОС FRIDAY ИЗ MARVEL

Озвучку FRIDAY из фильмов Marvel (актриса Керри Кондон) **нельзя легально встроить** в приложение — голос проприетарный. Но есть рабочие варианты «в стиле FRIDAY»:

| Вариант | Звучание | Стоимость |
|---|---|---|
| Системный TTS (рус. женский голос + настройки FRIDAY: спокойный, чуть выше тон) | Приближение, офлайн, встроено в проект | 0 ₽ |
| RHVoice + русский женский голос «Elena» | Самый близкий бесплатный женский ИИ-голос, офлайн | 0 ₽ |
| ElevenLabs (любой женский голос, можно свой) | Максимально «настоящий ИИ», онлайн | ~$5 подписка |

В проекте по умолчанию стоит **системный женский голос с настройками FRIDAY** (лёгкий тон, спокойный темп). Если установлен RHVoice с голосом Elena — ассистент заговорит им автоматически, потому что использует системный TTS. Для ElevenLabs достаточно вписать API-ключ и ID голоса.

---

## 2. КАК ПОЛУЧИТЬ ГОТОВЫЙ APK

Сам я APK собрать не могу (нет Android SDK). Но проект спроектирован так, что APK получается двумя способами:

### Путь А — Android Studio (локально, 10 минут)
1. Установить Android Studio.
2. New Project → Empty Views Activity (Kotlin), имя пакета `com.example.friday`.
3. Заменить/добавить файлы из этого документа по путям из раздела 3.
4. Build → Build Bundle(s)/APK(s) → Build APK(s).
5. APK лежит в `app/build/outputs/apk/debug/`.

### Путь Б — GitHub Actions (бесплатно, без Android Studio)
1. Создать репозиторий на GitHub, залить туда все файлы проекта (включая `.github/workflows/build-apk.yml`).
2. На вкладке Actions → workflow «Build APK» → Run workflow.
3. Через ~3 минуты в Artifacts будет готовый `FRIDAY-apk.zip` с APK внутри.
4. Скачать, установить на телефон (нужно разрешить установку из неизвестных источников).

> ВАЖНО: для CI-сборки в репозитории должен быть gradle wrapper. Android Studio создаёт его автоматически. Если заливаете без студии — workflow сам сгенерирует wrapper (`gradle wrapper`).

---

## 3. СТРУКТУРА ПРОЕКТА

```
FRIDAYAssistant/
├── .github/workflows/build-apk.yml      ← сборка APK в облаке
├── settings.gradle.kts
├── build.gradle.kts
├── gradle.properties
├── local.properties                      ← НЕ заливать на GitHub (путь к SDK)
├── gradle/wrapper/gradle-wrapper.properties
└── app/
    ├── build.gradle.kts
    └── src/main/
        ├── AndroidManifest.xml
        ├── res/
        │   ├── values/strings.xml
        │   ├── values/colors.xml
        │   ├── values/themes.xml
        │   ├── drawable/arc_core.xml
        │   ├── drawable/arc_button.xml
        │   └── layout/activity_main.xml
        └── java/com/example/friday/
            ├── MainActivity.kt
            ├── AssistantService.kt
            ├── CommandProcessor.kt
            └── FridayTts.kt
```

---

## 4. ФАЙЛЫ ПРОЕКТА

### 4.1 `.github/workflows/build-apk.yml`

```yaml
name: Build APK

on:
  push:
    branches: [ main ]
  workflow_dispatch:        # позволяет запускать сборку вручную кнопкой

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Setup Gradle
        uses: gradle/actions/setup-gradle@v3
        with:
          gradle-version: '8.7'

      - name: Generate wrapper if missing
        run: |
          if [ ! -f gradlew ]; then gradle wrapper --gradle-version 8.7; fi
          chmod +x gradlew

      - name: Build debug APK
        run: ./gradlew assembleDebug

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: FRIDAY-apk
          path: app/build/outputs/apk/debug/app-debug.apk
```

### 4.2 `settings.gradle.kts` (корень)

```kotlin
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
rootProject.name = "FRIDAYAssistant"
include(":app")
```

### 4.3 `build.gradle.kts` (корень)

```kotlin
plugins {
    id("com.android.application") version "8.5.2" apply false
    id("org.jetbrains.kotlin.android") version "1.9.24" apply false
}
```

### 4.4 `gradle.properties`

```properties
org.gradle.jvmargs=-Xmx2048m
android.useAndroidX=true
android.nonTransitiveRClass=true
# Ключ ElevenLabs (необязательно). Без него работает системный женский TTS.
# elevenlabs.key=
```

### 4.5 `gradle/wrapper/gradle-wrapper.properties`

```properties
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-8.7-bin.zip
networkTimeout=10000
validateDistributionUrl=true
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
```

### 4.6 `app/build.gradle.kts`

```kotlin
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
}

val elevenLabsKey: String = (project.findProperty("elevenlabs.key") as String?) ?: ""

android {
    namespace = "com.example.friday"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.example.friday"
        minSdk = 26
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"

        buildConfigField("String", "ELEVENLABS_API_KEY", "\"$elevenLabsKey\"")
    }

    buildFeatures {
        viewBinding = true
        buildConfig = true
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    kotlinOptions {
        jvmTarget = "17"
    }
}

dependencies {
    implementation("androidx.core:core-ktx:1.13.1")
    implementation("androidx.appcompat:appcompat:1.7.0")
    implementation("com.google.android.material:material:1.12.0")
    implementation("com.squareup.okhttp3:okhttp:4.12.0")
}
```

### 4.7 `app/src/main/AndroidManifest.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.CALL_PHONE" />
    <uses-permission android:name="android.permission.SEND_SMS" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE_MICROPHONE" />
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
    <uses-permission android:name="android.permission.INTERNET" />

    <uses-feature android:name="android.hardware.microphone" android:required="true" />

    <!-- CAMERA не требуется: снимки делаются через системное приложение камеры -->

    <application
        android:allowBackup="true"
        android:label="@string/app_name"
        android:theme="@style/Theme.Friday">

        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <service
            android:name=".AssistantService"
            android:exported="false"
            android:foregroundServiceType="microphone" />
    </application>
</manifest>
```

### 4.8 `app/src/main/res/values/strings.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">FRIDAY</string>
</resources>
```

### 4.9 `app/src/main/res/values/colors.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="background">#000000</color>
    <color name="cyan">#00E5FF</color>
    <color name="cyan_dark">#006064</color>
    <color name="cyan_glow">#3300E5FF</color>
    <color name="text_secondary">#B3FFFFFF</color>
</resources>
```

### 4.10 `app/src/main/res/values/themes.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="Theme.Friday" parent="Theme.MaterialComponents.DayNight.NoActionBar">
        <item name="android:windowBackground">@color/background</item>
        <item name="android:statusBarColor">@color/background</item>
        <item name="android:navigationBarColor">@color/background</item>
        <item name="colorPrimary">@color/cyan</item>
        <item name="colorOnPrimary">@color/background</item>
    </style>
</resources>
```

### 4.11 `app/src/main/res/drawable/arc_core.xml`

Пульсирующее «ядро реактора» — круг с циановым свечением.

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- внешнее свечение -->
    <item>
        <shape android:shape="oval">
            <solid android:color="@color/cyan_glow" />
        </shape>
    </item>
    <!-- кольцо -->
    <item android:left="10dp" android:top="10dp" android:right="10dp" android:bottom="10dp">
        <shape android:shape="oval">
            <solid android:color="#1100E5FF" />
            <stroke android:width="3dp" android:color="@color/cyan" />
        </shape>
    </item>
    <!-- ядро -->
    <item android:left="42dp" android:top="42dp" android:right="42dp" android:bottom="42dp">
        <shape android:shape="oval">
            <gradient
                android:startColor="@color/cyan"
                android:endColor="@color/cyan_dark"
                android:angle="0" />
            <stroke android:width="1dp" android:color="#FFFFFF" />
        </shape>
    </item>
</layer-list>
```

### 4.12 `app/src/main/res/drawable/arc_button.xml`

Стилизованная кнопка-«шарнир» под ядром.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="@color/cyan_glow">
    <item>
        <shape android:shape="rectangle">
            <corners android:radius="12dp" />
            <solid android:color="#1AFFFFFF" />
            <stroke android:width="1dp" android:color="@color/cyan" />
        </shape>
    </item>
</ripple>
```

### 4.13 `app/src/main/res/layout/activity_main.xml`

Дизайн FRIDAY: чёрный фон, ядро реактора, статус «Слушаю, босс.»

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.core.widget.NestedScrollView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fillViewport="true">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:gravity="center_horizontal"
        android:padding="24dp">

        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:orientation="vertical"
            android:gravity="center">

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="FRIDAY"
                android:textColor="#FFFFFF"
                android:textSize="40sp"
                android:textStyle="bold"
                android:letterSpacing="0.3" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Тони Старк · ИИ-ассистент"
                android:textColor="@color/text_secondary"
                android:textSize="13sp" />
        </LinearLayout>

        <!-- Ядро реактора -->
        <ImageView
            android:id="@+id/core"
            android:layout_width="220dp"
            android:layout_height="220dp"
            android:layout_marginTop="28dp"
            android:src="@drawable/arc_core"
            android:contentDescription="Ядро реактора" />

        <TextView
            android:id="@+id/status"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="12dp"
            android:text="ОЖИДАНИЕ"
            android:textColor="@color/cyan"
            android:textSize="16sp"
            android:textStyle="bold"
            android:letterSpacing="0.2" />

        <com.google.android.material.button.MaterialButton
            android:id="@+id/btnToggle"
            android:layout_width="240dp"
            android:layout_height="48dp"
            android:layout_marginTop="20dp"
            android:background="@drawable/arc_button"
            android:text="Активировать"
            android:textColor="#FFFFFF"
            android:textSize="15sp" />

        <com.google.android.material.button.MaterialButton
            android:id="@+id/btnWake"
            style="@style/Widget.MaterialComponents.Button.OutlinedButton"
            android:layout_width="240dp"
            android:layout_height="48dp"
            android:layout_marginTop="10dp"
            android:text="Режим ожидания: ВКЛ"
            android:textColor="@color/cyan"
            android:textSize="13sp"
            app:strokeColor="@color/cyan_dark" />

        <com.google.android.material.button.MaterialButton
            android:id="@+id/btnPermissions"
            style="@style/Widget.MaterialComponents.Button.TextButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="4dp"
            android:text="Разрешения"
            android:textColor="@color/text_secondary" />

        <TextView
            android:id="@+id/log"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:fontFamily="monospace"
            android:gravity="bottom|start"
            android:minHeight="140dp"
            android:text="БОСС?.."
            android:textColor="@color/cyan"
            android:textSize="12sp" />
    </LinearLayout>
</androidx.core.widget.NestedScrollView>
```

### 4.14 `app/src/main/java/com/example/friday/MainActivity.kt`

Экран: запрос разрешений, кнопка активации, пульс ядра, журнал команд.

```kotlin
package com.example.friday

import android.Manifest
import android.animation.ObjectAnimator
import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.content.IntentFilter
import android.content.pm.PackageManager
import android.os.Build
import android.os.Bundle
import android.view.animation.DecelerateInterpolator
import android.widget.Toast
import androidx.activity.result.contract.ActivityResultContracts
import androidx.appcompat.app.AppCompatActivity
import androidx.core.content.ContextCompat
import com.example.friday.databinding.ActivityMainBinding

class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding
    private var serviceRunning = false
    private var pulse: ObjectAnimator? = null

    private val permissions = buildList {
        add(Manifest.permission.RECORD_AUDIO)
        add(Manifest.permission.CALL_PHONE)
        add(Manifest.permission.SEND_SMS)
        add(Manifest.permission.FOREGROUND_SERVICE_MICROPHONE)
        if (Build.VERSION.SDK_INT >= 33) add(Manifest.permission.POST_NOTIFICATIONS)
    }

    private val permissionLauncher =
        registerForActivityResult(ActivityResultContracts.RequestMultiplePermissions()) { result ->
            val denied = result.filterValues { !it }.keys
            if (denied.isEmpty()) {
                toast("Разрешения получены, босс.")
            } else {
                toast("Нужны разрешения: $denied")
            }
        }

    private val transcriptReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context?, intent: Intent?) {
            val text = intent?.getStringExtra(AssistantService.EXTRA_TEXT) ?: return
            runOnUiThread { appendLog(text) }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        binding.btnPermissions.setOnClickListener { permissionLauncher.launch(permissions.toTypedArray()) }

        binding.btnToggle.setOnClickListener {
            if (serviceRunning) {
                stopService(Intent(this, AssistantService::class.java).setAction(AssistantService.ACTION_STOP))
            } else {
                if (ContextCompat.checkSelfPermission(this, Manifest.permission.RECORD_AUDIO)
                    != PackageManager.PERMISSION_GRANTED
                ) {
                    toast("Сначала дайте доступ к микрофону.")
                    permissionLauncher.launch(permissions.toTypedArray())
                    return@setOnClickListener
                }
                startService(Intent(this, AssistantService::class.java).setAction(AssistantService.ACTION_START))
            }
        }

        binding.btnWake.setOnClickListener {
            startService(Intent(this, AssistantService::class.java)
                .setAction(AssistantService.ACTION_TOGGLE_WAKE_WORD))
        }
    }

    override fun onStart() {
        super.onStart()
        val filter = IntentFilter(AssistantService.BROADCAST_TRANSCRIPT)
        registerReceiver(transcriptReceiver, filter, Context.RECEIVER_NOT_EXPORTED)
    }

    override fun onStop() {
        unregisterReceiver(transcriptReceiver)
        super.onStop()
    }

    /** Вызывается из сервиса через трансляцию */
    fun setServiceRunning(running: Boolean) {
        serviceRunning = running
        binding.btnToggle.text = if (running) "Деактивировать" else "Активировать"
        binding.status.text = if (running) "СЛУШАЮ, БОСС" else "ОЖИДАНИЕ"
        if (running) startPulse() else stopPulse()
    }

    private fun startPulse() {
        if (pulse != null) return
        pulse = ObjectAnimator.ofFloat(binding.core, "scaleX", 1f, 1.12f).apply {
            duration = 900
            repeatCount = ObjectAnimator.INFINITE
            repeatMode = ObjectAnimator.REVERSE
            interpolator = DecelerateInterpolator()
        }
        pulse?.start()
        ObjectAnimator.ofFloat(binding.core, "scaleY", 1f, 1.12f).apply {
            duration = 900
            repeatCount = ObjectAnimator.INFINITE
            repeatMode = ObjectAnimator.REVERSE
            interpolator = DecelerateInterpolator()
            start()
        }
    }

    private fun stopPulse() {
        pulse?.cancel()
        pulse = null
        binding.core.animate().scaleX(1f).scaleY(1f).setDuration(300).start()
    }

    private fun appendLog(text: String) {
        binding.log.append("\n> " + text)
    }

    private fun toast(msg: String) = Toast.makeText(this, msg, Toast.LENGTH_LONG).show()
}
```

### 4.15 `app/src/main/java/com/example/friday/FridayTts.kt`

Голос FRIDAY: женский системный TTS с настройками FRIDAY + необязательный ElevenLabs.

```kotlin
package com.example.friday

import android.content.Context
import android.media.MediaPlayer
import android.os.Handler
import android.os.Looper
import android.speech.tts.TextToSpeech
import android.speech.tts.UtteranceProgressListener
import android.util.Log
import okhttp3.Call
import okhttp3.Callback
import okhttp3.MediaType.Companion.toMediaType
import okhttp3.OkHttpClient
import okhttp3.Request
import okhttp3.RequestBody.Companion.toRequestBody
import okhttp3.Response
import org.json.JSONObject
import java.io.File
import java.io.IOException
import java.util.Locale
import java.util.UUID

/**
 * Голос FRIDAY.
 * Режим 1: системный TTS — женский голос, лёгкий тон, спокойный темп. Работает офлайн.
 * Режим 2: ElevenLabs — любой женский голос из аккаунта (максимально «настоящий»).
 */
class FridayTts(context: Context) : TextToSpeech.OnInitListener {

    private val appContext = context.applicationContext
    private val tts = TextToSpeech(context, this)
    private val handler = Handler(Looper.getMainLooper())
    private val pendingDone = HashMap<String, () -> Unit>()
    private var ready = false

    /** Ключ из gradle.properties: elevenlabs.key=... Пусто = системный голос. */
    private val elevenKey: String = BuildConfig.ELEVENLABS_API_KEY

    /** ID женского голоса в вашем аккаунте ElevenLabs (раздел VoiceLab). Замените на свой. */
    private val elevenVoiceId: String = "EXAVITQu4vr4xnSDxMaL"

    override fun onInit(status: Int) {
        if (status != TextToSpeech.SUCCESS) return

        val lang = tts.setLanguage(Locale("ru", "RU"))
        if (lang == TextToSpeech.LANG_MISSING_DATA || lang == TextToSpeech.LANG_NOT_SUPPORTED) {
            tts.setLanguage(Locale.getDefault())
        }

        // Настройки FRIDAY: чуть выше тон (женский), спокойный темп
        tts.setPitch(1.05f)
        tts.setSpeechRate(0.92f)

        // Если установлен отдельный женский голос (RHVoice Elena и т.п.) — берём его
        val voices = tts.voices ?: emptySet()
        val female = voices.firstOrNull {
            val n = it.name.lowercase()
            (n.contains("female") || n.contains("elena") || n.contains("friday"))
                    && !n.contains("male")
        }
        if (female != null) tts.voice = female

        tts.setOnUtteranceProgressListener(object : UtteranceProgressListener() {
            override fun onStart(utteranceId: String?) {}
            override fun onDone(utteranceId: String?) {
                pendingDone.remove(utteranceId)?.invoke()
            }

            @Deprecated("Deprecated in Java")
            override fun onError(utteranceId: String?) {
                pendingDone.remove(utteranceId)?.invoke()
            }
        })
        ready = true
    }

    /** Проговорить текст; onDone вызывается после окончания речи. */
    fun speak(text: String, onDone: () -> Unit) {
        if (elevenKey.isNotBlank()) {
            speakViaElevenLabs(text, onDone)
        } else {
            speakLocal(text, onDone)
        }
    }

    private fun speakLocal(text: String, onDone: () -> Unit) {
        if (!ready) {
            handler.postDelayed({ speakLocal(text, onDone) }, 100)
            return
        }
        val id = UUID.randomUUID().toString()
        pendingDone[id] = onDone
        val result = tts.speak(text, TextToSpeech.QUEUE_FLUSH, null, id)
        if (result == TextToSpeech.ERROR) {
            pendingDone.remove(id)
            onDone()
        }
    }

    private fun speakViaElevenLabs(text: String, onDone: () -> Unit) {
        val client = OkHttpClient()
        val body = JSONObject()
            .put("text", text)
            .put("model_id", "eleven_multilingual_v2")
            .toString()
            .toRequestBody("application/json; charset=utf-8".toMediaType())

        val request = Request.Builder()
            .url("https://api.elevenlabs.io/v1/text-to-speech/$elevenVoiceId?output_format=mp3_44100_128")
            .header("xi-api-key", elevenKey)
            .post(body)
            .build()

        client.newCall(request).enqueue(object : Callback {
            override fun onFailure(call: Call, e: IOException) {
                Log.e("FridayTts", "ElevenLabs недоступен: ${e.message}")
                handler.post { speakLocal(text, onDone) }
            }

            override fun onResponse(call: Call, response: Response) {
                val bytes = response.use { it.body?.bytes() }
                if (response.isSuccessful && bytes != null) {
                    handler.post { playMp3(bytes, onDone) }
                } else {
                    Log.e("FridayTts", "ElevenLabs HTTP ${response.code}")
                    handler.post { speakLocal(text, onDone) }
                }
            }
        })
    }

    private fun playMp3(bytes: ByteArray, onDone: () -> Unit) {
        val file = File(appContext.cacheDir, "friday_${System.currentTimeMillis()}.mp3")
        file.writeBytes(bytes)
        val player = MediaPlayer()
        try {
            player.setDataSource(file.absolutePath)
            player.setOnCompletionListener {
                player.release(); file.delete(); onDone()
            }
            player.setOnErrorListener { _, _, _ ->
                player.release(); file.delete(); onDone(); true
            }
            player.prepare()
            player.start()
        } catch (e: Exception) {
            player.release()
            speakLocal("Простите, голосовой модуль не сработал", onDone)
        }
    }

    fun shutdown() {
        ready = false
        tts.stop()
        tts.shutdown()
    }
}
```

### 4.16 `app/src/main/java/com/example/friday/AssistantService.kt`

Фоновый сервис FRIDAY: прослушивание, слово-триггер «Пятница»/«FRIDAY», обработка команд.

```kotlin
package com.example.friday

import android.Manifest
import android.app.Notification
import android.app.NotificationChannel
import android.app.NotificationManager
import android.app.PendingIntent
import android.app.Service
import android.content.Context
import android.content.Intent
import android.content.pm.PackageManager
import android.os.Bundle
import android.os.Handler
import android.os.IBinder
import android.os.Looper
import android.speech.RecognitionListener
import android.speech.RecognizerIntent
import android.speech.SpeechRecognizer
import androidx.core.app.NotificationCompat
import androidx.core.content.ContextCompat

class AssistantService : Service() {

    companion object {
        const val ACTION_START = "com.example.friday.START"
        const val ACTION_STOP = "com.example.friday.STOP"
        const val ACTION_TOGGLE_WAKE_WORD = "com.example.friday.TOGGLE_WAKE_WORD"

        const val BROADCAST_TRANSCRIPT = "com.example.friday.TRANSCRIPT"
        const val EXTRA_TEXT = "text"

        private const val CHANNEL_ID = "friday_channel"
        private const val NOTIF_ID = 1001
    }

    private val handler = Handler(Looper.getMainLooper())
    private var recognizer: SpeechRecognizer? = null
    private var tts: FridayTts? = null
    private var listening = false

    /** true = команда только после «Пятница, ...»; false = любые фразы */
    private var wakeWordEnabled = true

    private val listener = object : RecognitionListener {
        override fun onReadyForSpeech(params: Bundle?) = broadcast("Слушаю, босс...")
        override fun onBeginningOfSpeech() {}
        override fun onRmsChanged(rmsdB: Float) {}
        override fun onBufferReceived(buffer: ByteArray?) {}
        override fun onEndOfSpeech() {}

        override fun onError(error: Int) {
            broadcast("—")
            restartWithDelay(500)
        }

        override fun onResults(results: Bundle?) {
            val text = results
                ?.getStringArrayList(SpeechRecognizer.RESULTS_RECOGNITION)
                ?.firstOrNull()
            if (text == null) {
                restartWithDelay(300)
                return
            }
            broadcast(text)
            processCommand(text)
        }

        override fun onPartialResults(partialResults: Bundle?) {}
        override fun onEvent(eventType: Int, params: Bundle?) {}
    }

    override fun onBind(intent: Intent?): IBinder? = null

    override fun onCreate() {
        super.onCreate()
        createChannel()
        tts = FridayTts(this)
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        when (intent?.action) {
            ACTION_STOP -> {
                stopListening()
                stopForeground(STOP_FOREGROUND_REMOVE)
                stopSelf()
            }

            ACTION_TOGGLE_WAKE_WORD -> {
                wakeWordEnabled = !wakeWordEnabled
                broadcast(
                    if (wakeWordEnabled) "Режим ожидания включён. Скажите: Пятница"
                    else "Режим ожидания отключён — выполняю все команды подряд"
                )
            }

            else -> {
                startForeground(NOTIF_ID, buildNotification())
                startListening()
            }
        }
        return START_STICKY
    }

    private fun startListening() {
        if (listening || recognizer != null) return
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.RECORD_AUDIO)
            != PackageManager.PERMISSION_GRANTED
        ) return

        val sr = SpeechRecognizer.createSpeechRecognizer(this)
        sr.setRecognitionListener(listener)
        sr.startListening(
            Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH).apply {
                putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL, RecognizerIntent.LANGUAGE_MODEL_FREE_FORM)
                putExtra(RecognizerIntent.EXTRA_LANGUAGE, "ru-RU")
                putExtra(RecognizerIntent.EXTRA_PARTIAL_RESULTS, true)
            }
        )
        recognizer = sr
        listening = true
        updateNotification("Слушаю, босс...")
    }

    private fun restartWithDelay(delayMs: Long) {
        handler.postDelayed({
            recognizer?.destroy()
            recognizer = null
            listening = false
            if (tts != null) startListening()
        }, delayMs)
    }

    private fun processCommand(raw: String) {
        val cmd = raw.lowercase().trim()

        val clean: String = if (wakeWordEnabled) {
            if (cmd.contains("пятница") || cmd.contains("фрайдей") || cmd.contains("friday")) {
                cmd.replace(Regex("пятница|фрайдей|friday"), "").trim()
            } else {
                "" // без триггера игнорируем
            }
        } else {
            cmd
        }

        if (clean.isBlank()) {
            restartWithDelay(400)
            return
        }

        val reply = CommandProcessor.process(clean, this)
        tts?.speak(reply.text) {
            restartWithDelay(600)
        }
    }

    private fun broadcast(text: String) {
        sendBroadcast(
            Intent(BROADCAST_TRANSCRIPT)
                .setPackage(packageName)
                .putExtra(EXTRA_TEXT, text)
        )
    }

    private fun updateNotification(text: String) {
        if (listening) {
            val nm = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
            nm.notify(NOTIF_ID, buildNotification(text))
        }
    }

    private fun buildNotification(text: String = "Слушаю, босс..."): Notification {
        val stopIntent = PendingIntent.getService(
            this, 0,
            Intent(this, AssistantService::class.java).setAction(ACTION_STOP),
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
        )
        return NotificationCompat.Builder(this, CHANNEL_ID)
            .setSmallIcon(android.R.drawable.ic_btn_speak_now)
            .setContentTitle("FRIDAY")
            .setContentText(text)
            .setOngoing(true)
            .addAction(0, "Стоп", stopIntent)
            .setPriority(NotificationCompat.PRIORITY_LOW)
            .build()
    }

    private fun stopListening() {
        listening = false
        recognizer?.destroy()
        recognizer = null
        tts?.shutdown()
        tts = null
        handler.removeCallbacksAndMessages(null)
    }

    override fun onDestroy() {
        stopListening()
        super.onDestroy()
    }

    private fun createChannel() {
        val channel = NotificationChannel(
            CHANNEL_ID, "FRIDAY", NotificationManager.IMPORTANCE_LOW
        )
        (getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager)
            .createNotificationChannel(channel)
    }
}
```

### 4.17 `app/src/main/java/com/example/friday/CommandProcessor.kt` (ПОЛНЫЙ ФАЙЛ)

Парсер русских команд + выполнение: звонки, SMS, камера, приложения, время. Включая поиск контактов.

```kotlin
package com.example.friday

import android.content.Context
import android.content.Intent
import android.net.Uri
import android.provider.ContactsContract
import android.provider.MediaStore
import android.telephony.SmsManager
import java.text.SimpleDateFormat
import java.util.Date
import java.util.Locale

object CommandProcessor {

    data class Reply(val text: String)

    private val appAliases = mapOf(
        "ютуб" to "com.google.android.youtube",
        "youtube" to "com.google.android.youtube",
        "телефон" to "com.android.dialer",
        "контакты" to "com.google.android.contacts",
        "карты" to "com.google.android.apps.maps",
        "браузер" to "com.android.chrome",
        "хром" to "com.android.chrome",
        "калькулятор" to "com.google.android.calculator",
        "настройки" to "com.android.settings",
        "галерея" to "com.google.android.apps.photos",
        "фото" to "com.google.android.apps.photos",
        "плей маркет" to "com.android.vending",
        "маркет" to "com.android.vending",
        "телеграм" to "org.telegram.messenger",
        "вотсап" to "com.whatsapp",
        "вконтакте" to "com.vkontakte.android",
        "вк" to "com.vkontakte.android",
        "инстаграм" to "com.instagram.android",
        "спотифай" to "com.spotify.music"
    )

    fun process(raw: String, context: Context): Reply {
        val t = raw.lowercase().trim()
        return when {
            t.contains("позвони") || t.contains("набери") -> handleCall(t, context)

            t.contains("смс") || t.contains("сообщение") ||
                    t.contains("напиши") || t.contains("отправь") -> handleSms(t, context)

            t.contains("сфотографируй") || t.contains("сними") ||
                    (t.contains("камеру") && t.contains("открой")) -> Reply(openCamera(context))

            t.contains("открой") || t.contains("запусти") -> handleOpen(t, context)

            t.contains("который час") || t.contains("время") ->
                Reply("Сейчас " + SimpleDateFormat("HH:mm", Locale("ru", "RU")).format(Date()))

            t.contains("привет") || t == "friday" || t == "пятница" ->
                Reply("Слушаю, босс.")

            else -> Reply(
                "Босс, я не поняла команду. " +
                "Попробуйте: позвони маме, отправь смс, открой ютуб, сфотографируй."
            )
        }
    }

    // ---------- ЗВОНКИ ----------

    private fun handleCall(t: String, context: Context): Reply {
        val rest = t.replace(Regex("позвони|набери"), "").trim()
        val number = extractNumber(rest) ?: findContactNumber(context, rest)
        if (number == null) return Reply("Босс, я не нашла контакт «$rest».")

        return try {
            val intent = Intent(Intent.ACTION_CALL, Uri.parse("tel:$number"))
            intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
            context.startActivity(intent)
            Reply("Звоню на номер $number.")
        } catch (e: SecurityException) {
            Reply("Нет разрешения на звонки, босс.")
        }
    }

    // ---------- SMS ----------

    private fun handleSms(t: String, context: Context): Reply {
        var rest = t
            .replace(Regex("отправь смс|отправить смс|напиши смс|напиши сообщение|отправь сообщение|отправить сообщение"), "")
            .replace("смс", "")
            .trim()

        // «... маме текст приду поздно» → получатель до «текст», сообщение после
        val parts = rest.split(Regex("(текст|напиши|сообщение)"))
        var recipient = parts[0].trim()
        var message = if (parts.size > 1) parts[1].trim() else ""

        // Без разделителя: первое слово — получатель, дальше — текст
        if (message.isEmpty()) {
            val words = recipient.split(" ")
            if (words.size >= 2) {
                recipient = words[0]
                message = words.drop(1).joinToString(" ")
            }
        }

        if (recipient.isEmpty()) return Reply("Кому отправить сообщение, босс?")
        if (message.isEmpty()) return Reply("Что написать в сообщении, босс?")

        val number = extractNumber(recipient) ?: findContactNumber(context, recipient)
        if (number == null) return Reply("Босс, я не нашла контакт «$recipient».")

        return try {
            SmsManager.getDefault().sendTextMessage(number, null, message, null, null)
            Reply("Сообщение отправлено пользователю $recipient.")
        } catch (e: SecurityException) {
            Reply("Нет разрешения на отправку SMS, босс.")
        }
    }

    // ---------- КАМЕРА ----------

    private fun openCamera(context: Context): String {
        val intent = Intent(MediaStore.ACTION_IMAGE_CAPTURE)
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
        context.startActivity(intent)
        return "Открываю камеру."
    }

    // ---------- ПРИЛОЖЕНИЯ ----------

    private fun handleOpen(t: String, context: Context): Reply {
        val name = t
            .replace(Regex("открой|открыть|запусти|запустить|пожалуйста"), "")
            .trim()
            .trimEnd('.', '!', '?', ' ')
        if (name.isEmpty()) return Reply("Что открыть, босс?")

        val pkg = appAliases[name] ?: findInstalledApp(context, name)
        if (pkg == null) return Reply("Босс, я не нашла приложение «$name».")

        val launch = context.packageManager.getLaunchIntentForPackage(pkg)
        if (launch == null) return Reply("Не удалось запустить $name.")

        launch.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
        context.startActivity(launch)
        return Reply("Открываю $name.")
    }

    /** Поиск установленного приложения по его названию (метке). */
    private fun findInstalledApp(context: Context, name: String): String? {
        val pm = context.packageManager
        val intent = Intent(Intent.ACTION_MAIN).addCategory(Intent.CATEGORY_LAUNCHER)
        for (ri in pm.queryIntentActivities(intent, 0)) {
            val label = ri.loadLabel(pm).toString().lowercase()
            if (label.contains(name)) return ri.activityInfo.packageName
        }
        return null
    }

    /** Поиск номера телефона по имени контакта в телефонной книге. */
    private fun findContactNumber(context: Context, name: String): String? {
        val uri = Uri.withAppendedPath(
            ContactsContract.PhoneLookup.CONTENT_FILTER_URI,
            Uri.encode(name)
        )
        context.contentResolver.query(
            uri,
            arrayOf(ContactsContract.PhoneLookup.NUMBER),
            null, null, null
        )?.use { cursor ->
            if (cursor.moveToFirst()) {
                return cursor.getString(0)
            }
        }
        return null
    }

    /** Извлечение номера из строки: «позвони +7 900 123-45-67» → +79001234567 */
    private fun extractNumber(text: String): String? {
        val found = Regex("[+\\d][\\d\\s()\\-]{5,}").find(text)?.value ?: return null
        return found.replace(Regex("[^+\\d]"), "")
    }
}
```

---

## 5. НАСТРОЙКА ГОЛОСА

### Шаг 1. Проверьте системный TTS (обязательно)
Настройки → Спец. возможности → Синтез речи / Вывод текста в речь → Установите голос:
- Выберите **русский голос (Google)** — по умолчанию женский.
- Или установите **RHVoice** (бесплатно, Play Market) → в настройках RHVoice выберите русский голос **Elena** → установите RHVoice системным движком TTS. Приложение автоматически заговорит этим голосом, и это будет очень близко к стилю FRIDAY.

### Шаг 2 (необязательно) — ElevenLabs для «настоящего» голоса
1. elevenlabs.io → зарегистрироваться → API key.
2. VoiceLab → выбрать/создать женский голос → скопировать Voice ID.
3. В `gradle.properties`: `elevenlabs.key=ВАШ_КЛЮЧ`.
4. В `FridayTts.kt` заменить `elevenVoiceId` на свой.
5. Пересобрать APK. Если облако недоступно — автоматически вернётся системный голос.

### Шаг 3. Команды после установки
- «**Пятница, позвони маме**»
- «**Пятница, позвони +7 900 111-22-33**»
- «**Пятница, отправь смс маме текст я буду поздно**»
- «**Пятница, открой ютуб**»
- «**Пятница, запусти телеграм**»
- «**Пятница, сфотографируй**»
- «**Пятница, который час**»
- «**Пятница, привет**»

---

## ЧАСТЫЕ ВОПРОСЫ

**Почему приложение не собирается на GitHub?**
Проверьте, что в репозитории есть `gradle/wrapper/gradle-wrapper.jar`. Android Studio создаёт его автоматически при создании проекта. Можно сгенерировать локально: `gradle wrapper --gradle-version 8.7`.

**Работает ли без интернета?**
Распознавание речи — онлайн (Google). Голос — офлайн (системный TTS). Рекомендую стабильное интернет-соединение.

**Почему я не слышу оригинальный голос FRIDAY?**
Потому что он принадлежит Marvel и встроить его легально нельзя. Максимально близкие бесплатные варианты описаны в разделе 1.

**Где искать собранный APK?**
Локально: `app/build/outputs/apk/debug/app-debug.apk`. На GitHub: вкладка Actions → последний запуск → Artifacts → FRIDAY-apk.