# Саймон — голосовой ИИ-помощник для Android

Аналог «Джарвиса»: открывает приложения, управляет музыкой и громкостью,
ищет в интернете и на YouTube, отвечает голосом (TextToSpeech) и текстом.
Работает на **Android 8.0+ (API 26)**, интерфейс и команды — на русском.

---

## ⚠️ Важно: про «готовый APK»

Скомпилировать `.apk` можно только в среде с установленным **Android SDK**.
В этом проекте уже всё готово к сборке — нужно лишь нажать «Build» в одной из сред ниже.
Самый простой и надёжный способ для телефона — **GitHub Actions** (собирает APK в облаке
автоматически, вам не нужен SDK на телефоне). Способ через AIDE тоже описан.

---

## Способ A. GitHub Actions (рекомендуется — APK без установки SDK)

1. Создайте репозиторий на github.com (через браузер телефона).
2. Загрузите туда все файлы из этого проекта (можно через «Add file → Upload files»,
   распаковав ZIP заранее, либо через GitHub-приложение).
3. GitHub сам запустит сборку (файл `.github/workflows/build.yml`).
   Откройте вкладку **Actions** → дождитесь зелёной галочки (≈3–5 минут).
4. Зайдите в завершённый запуск → раздел **Artifacts** → скачайте
   `Simon-debug-apk` → внутри `app-debug.apk`.
5. Установите APK на телефон (разрешите «установку из неизвестных источников»).

> Если Actions не запустились автоматически: вкладка Actions → выберите
> «Build Simon APK» → кнопка **Run workflow**.

---

## Способ B. Termux (сборка прямо на телефоне)

```bash
pkg update && pkg upgrade -y
pkg install -y openjdk-17 wget unzip
# Распакуйте проект, например в ~/Simon, затем:
cd ~/Simon
# Установите Android command-line tools и SDK (см. ниже), затем:
./gradlew assembleDebug
# Готовый APK: app/build/outputs/apk/debug/app-debug.apk
```

Для сборки нужен Android SDK. Кратко:
```bash
mkdir -p ~/android-sdk/cmdline-tools && cd ~/android-sdk/cmdline-tools
wget https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip
unzip commandlinetools-linux-11076708_latest.zip && mv cmdline-tools latest
export ANDROID_HOME=~/android-sdk
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
yes | sdkmanager --licenses
sdkmanager "platform-tools" "platforms;android-34" "build-tools;34.0.0"
echo "sdk.dir=$ANDROID_HOME" > ~/Simon/local.properties
```
> Примечание: на ARM-телефоне `aapt2` из Google SDK может не запуститься.
> Если так — используйте Способ A (GitHub Actions). Он надёжнее.

---

## Способ C. AIDE (на телефоне)

1. Установите **AIDE** из Play Market.
2. Распакуйте ZIP проекта в память телефона.
3. Откройте папку `Simon` в AIDE как Gradle-проект.
4. Нажмите **Run / Build**.

> Внимание: бесплатная версия AIDE исторически плохо дружит с современным
> Kotlin/Gradle. Если AIDE выдаёт ошибки сборки — используйте **Способ A**.

---

## Структура проекта

```
Simon/
├── app/
│   ├── build.gradle
│   └── src/main/
│       ├── AndroidManifest.xml
│       ├── java/com/simon/assistant/
│       │   ├── MainActivity.kt              # Экран, кнопки настройки прав
│       │   ├── VoiceActivity.kt             # Быстрый голосовой запуск (прозрачный)
│       │   ├── SimonService.kt              # Foreground-сервис: уведомление + оверлей
│       │   ├── SimonEngine.kt               # Распознавание речи (RU) + TTS
│       │   ├── CommandProcessor.kt          # Разбор команд → действия
│       │   ├── MediaController.kt           # Медиакнопки Play/Pause/Next/Prev
│       │   ├── LocalAI.kt                   # Офлайн-ответчик (заглушка ИИ)
│       │   └── SimonAccessibilityService.kt # Глобальные действия системы
│       └── res/...
├── .github/workflows/build.yml             # Авто-сборка APK
├── build.gradle, settings.gradle, gradle.properties
└── gradlew, gradlew.bat, gradle/wrapper/
```

---

## Первый запуск (настройка прав — один раз)

Откройте «Саймон» и нажмите по очереди кнопки внизу экрана:
1. **Спец. возможности (Accessibility)** → включите «Саймон».
2. **Оверлей** → разрешите отображение поверх других окон.
3. **Экономия батареи** → отключите для Саймона.
4. **Запустить фоновый сервис** → появится уведомление и плавающая кнопка-микрофон.

Дайте разрешения на **микрофон** и **уведомления**, когда система их запросит.

При первом открытии Саймон поздоровается: «Саймон к вашим услугам».

---

## Команды (примеры)

| Скажите / напишите | Действие |
|---|---|
| `открой Телеграм` | запуск приложения |
| `включи музыку` / `пауза` | Play / Pause |
| `следующий трек` / `предыдущий трек` | Next / Previous |
| `громче` / `тише` / `выключи звук` | громкость |
| `найди погода в Минске в интернете` | поиск в Google |
| `найди новый клип на Ютубе` | поиск в YouTube |
| `который час` / `какое сегодня число` | офлайн-ответ |
| `сколько будет 12 умножить на 8` | калькулятор |

Активация голосом: нажмите кнопку **🎤 Слушать** в приложении, кнопку в уведомлении,
или плавающую кнопку-оверлей. Можно начинать команду со слова «Саймон» — оно отбрасывается.

---

## Замечания по функционалу

- **Управление музыкой** реализовано через системный `AudioManager.dispatchMediaKeyEvent`
  (стандартный способ без root) — работает с большинством плееров (YouTube Music,
  Spotify, VK, штатный плеер и т.п.).
- **«Повтор»** на уровне системы глобальной клавиши не имеет — режим повтора
  включается в самом плеере; Саймон сообщает об этом.
- **ИИ-разговоры** работают офлайн (`LocalAI`) — простые ответы, время/дата,
  калькулятор. Для развёрнутых ответов можно позже подключить онлайн-API.
- **Постоянно слушающий микрофон** намеренно не включён (расход батареи и
  ограничения Android) — активация по кнопке/уведомлению/оверлею.

---

## Технологии
Kotlin · SpeechRecognizer · TextToSpeech · AccessibilityService ·
Foreground Service · SYSTEM_ALERT_WINDOW overlay · AudioManager media keys.
Минимум зависимостей (только Kotlin stdlib, без AndroidX).
