# Telegram iOS Source Code Compilation Guide

We welcome all developers to use our API and source code to create applications on our platform.
There are several things we require from **all developers** for the moment.

# Creating your Telegram Application

1. [**Obtain your own api_id**](https://core.telegram.org/api/obtaining_api_id) for your application.
2. Please **do not** use the name Telegram for your app — or make sure your users understand that it is unofficial.
3. Kindly **do not** use our standard logo (white paper plane in a blue circle) as your app's logo.
3. Please study our [**security guidelines**](https://core.telegram.org/mtproto/security_guidelines) and take good care of your users' data and privacy.
4. Please remember to publish **your** code too in order to comply with the licences.

# Quick Compilation Guide

## Get the Code

```
git clone --recursive -j8 https://github.com/TelegramMessenger/Telegram-iOS.git
```

## Setup Xcode

Install Xcode (directly from https://developer.apple.com/download/applications or using the App Store).

## Adjust Configuration

1. Generate a random identifier:
```
openssl rand -hex 8
```
2. Create a new Xcode project. Use `Telegram` as the Product Name. Use `org.{identifier from step 1}` as the Organization Identifier.
3. Open `Keychain Access` and navigate to `Certificates`. Locate `Apple Development: your@email.address (XXXXXXXXXX)` and double tap the certificate. Under `Details`, locate `Organizational Unit`. This is the Team ID.
4. Edit `build-system/template_minimal_development_configuration.json`. Use data from the previous steps.
   - **Important:** `bundle_id` and `team_id` must not contain curly braces `{ }` (use e.g. `org.yourname.telegram` and your 10-character Team ID).

## Generate an Xcode project

```
python3 build-system/Make/Make.py \
    --cacheDir="$HOME/telegram-bazel-cache" \
    generateProject \
    --configurationPath=build-system/template_minimal_development_configuration.json \
    --xcodeManagedCodesigning
```

# Advanced Compilation Guide

## Xcode

1. Copy and edit `build-system/appstore-configuration.json`.
2. Copy `build-system/fake-codesigning`. Create and download provisioning profiles, using the `profiles` folder as a reference for the entitlements.
3. Generate an Xcode project:
```
python3 build-system/Make/Make.py \
    --cacheDir="$HOME/telegram-bazel-cache" \
    generateProject \
    --configurationPath=configuration_from_step_1.json \
    --codesigningInformationPath=directory_from_step_2
```

## IPA

1. Repeat the steps from the previous section. Use distribution provisioning profiles.
2. Run:
```
python3 build-system/Make/Make.py \
    --cacheDir="$HOME/telegram-bazel-cache" \
    build \
    --configurationPath=...see previous section... \
    --codesigningInformationPath=...see previous section... \
    --buildNumber=100001 \
    --configuration=release_arm64
```

# Building without a Mac

You need **macOS only for building** the app (Xcode). You do **not** need a Mac to get your Team ID or to install the app on your iPhone.

**Team ID (without Mac):**
1. Open [developer.apple.com](https://developer.apple.com) in a browser and sign in with your Apple ID.
2. Go to **Account** (or **Membership**).
3. Your **Team ID** is shown there (10 characters, e.g. `A1B2C3D4E5`). Copy it into `team_id` in `build-system/template_minimal_development_configuration.json`.

**Building:** Use a Mac in the cloud, for example:
- **GitHub Actions:** push your repo and use a `macos-latest` runner to run the project generation and build; upload the IPA as an artifact and download it.
- **Rented Mac** (e.g. MacinCloud, MacStadium): run the same `Make.py` commands there and download the IPA.

**Installing on iPhone (from Windows):** See below.

## Установка на iPhone через Sideloadly (Windows)

1. **Скачайте IPA** — готовый файл приложения (например, из артефактов CI или с облачного Mac).
2. **Установите Sideloadly** — [sideloadly.io](https://sideloadly.io/), скачайте версию для Windows и установите.
3. **Подключите iPhone** к ПК кабелем. На iPhone при запросе «Доверять этому компьютеру?» нажмите **Доверять**.
4. **Запустите Sideloadly.** В поле **App / IPA** укажите путь к вашему `.ipa` файлу (или перетащите файл в окно).
5. **Apple ID:** введите свой Apple ID и пароль (или выберите сохранённый аккаунт). Sideloadly подпишет приложение вашим аккаунтом — для бесплатного Apple ID подпись действует **7 дней**, после чего установку нужно повторить.
6. Нажмите **Start** (или **Sideload**). Дождитесь окончания установки.
7. На iPhone: **Настройки → Основные → VPN и управление устройством** — при необходимости нажмите на профиль разработчика и выберите **Доверять**.
8. Приложение появится на домашнем экране; можно открывать и пользоваться.

**Если приложение перестало открываться** (например, через 7 дней) — снова откройте тот же IPA в Sideloadly и повторите установку.

# Building for distribution (e.g. Sideloadly)

If you build **one IPA and share it with other users** (they install via Sideloadly with their own Apple ID):

- Use **one** configuration: set `bundle_id` (e.g. `org.telegram.sideload`), `api_id`, `api_hash`, and `team_id` to your values. Build the IPA once; the same IPA can be installed by many users. Each user signs it with their Apple ID in Sideloadly (typically 7-day limit for free accounts).
- You can keep a single `api_id`/`api_hash` from [my.telegram.org](https://my.telegram.org/apps) for your app; all users of your build will use that application identity.

If **other people will build from your source** (each builds their own app):

- Each builder must edit `build-system/template_minimal_development_configuration.json` (or their own copy) with:
  - Their own `bundle_id` (e.g. `org.{identifier}` from `openssl rand -hex 8`) — **no `{ }` in the value.**
  - Their own `api_id` and `api_hash` from [my.telegram.org](https://my.telegram.org/apps).
  - Their own `team_id`: from Keychain on Mac, or from [developer.apple.com](https://developer.apple.com) → Account → Membership (no Mac needed).
- Document for them that `bundle_id` and `team_id` must not contain curly braces.

# FAQ

## Xcode is stuck at "build-request.json not updated yet"

Occasionally, you might observe the following message in your build log:
```
"/Users/xxx/Library/Developer/Xcode/DerivedData/Telegram-xxx/Build/Intermediates.noindex/XCBuildData/xxx.xcbuilddata/build-request.json" not updated yet, waiting...
```

Should this occur, simply cancel the ongoing build and initiate a new one.

## Telegram_xcodeproj: no such package 

Following a system restart, the auto-generated Xcode project might encounter a build failure accompanied by this error:
```
ERROR: Skipping '@rules_xcodeproj_generated//generator/Telegram/Telegram_xcodeproj:Telegram_xcodeproj': no such package '@rules_xcodeproj_generated//generator/Telegram/Telegram_xcodeproj': BUILD file not found in directory 'generator/Telegram/Telegram_xcodeproj' of external repository @rules_xcodeproj_generated. Add a BUILD file to a directory to mark it as a package.
```

If you encounter this issue, re-run the project generation steps in the README.


# Tips

## Codesigning is not required for simulator-only builds

Add `--disableProvisioningProfiles`:
```
python3 build-system/Make/Make.py \
    --cacheDir="$HOME/telegram-bazel-cache" \
    generateProject \
    --configurationPath=path-to-configuration.json \
    --codesigningInformationPath=path-to-provisioning-data \
    --disableProvisioningProfiles
```

## Versions

Each release is built using a specific Xcode version (see `versions.json`). The helper script checks the versions of the installed software and reports an error if they don't match the ones specified in `versions.json`. It is possible to bypass these checks:

```
python3 build-system/Make/Make.py --overrideXcodeVersion build ... # Don't check the version of Xcode
```
