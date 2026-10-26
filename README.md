# ArkamysAudio Reference

[English](#english) · [Русский](#русский) · plain-text version: [`ArkamysAudio-Reference.txt`](ArkamysAudio-Reference.txt)

---

## English

### 1. Effect library `libarkamys.so`

C reimplementation of the original BLU `libarkamys_bip_5min.so`. Two builds with the same 352 exported symbols:

| Build | Use |
|---|---|
| `vendor/lib/soundfx/libarkamys.so` | armv7, used by this phone's 32-bit audio HAL |
| `vendor/lib64/soundfx/libarkamys.so` | arm64, for phones with a 64-bit HAL |

Audio output is bit-exact with the original on armv7 and arm64.

#### 1.1 Kept from the original (API compatible)

| Group | Functions |
|---|---|
| Effect entry point | `AELI`, `ArkamysLib_CreateEffect`, `ArkamysLib_ReleaseEffect`, `ArkamysLib_GetDescriptor`, `ArkamysLib_Open`, `ArkamysLib_Close`, `ArkamysLib_Config`, `ArkamysLib_Command`, `ArkamysLib_Process`, `ArkamysLib_CopyConfig`, `ArkamysLib_Effect_GetDescriptor` |
| OptimSpeaker module (C++ ABI) | `AkmOptimSpeaker_Init/Open/Close/Config/Process/Get_Param/Set_Param`, `AkmOptimSpeaker_Get_Param_Wrapper`, `AkmOptimSpeaker_Set_Param_Wrapper`, `applyPreset` |
| `SmartPhone` class | `initialize`, `resetState`, `forcePreset`, `setPreset`, `getCurrentPreset`, `getSPKPreset`, `getHPPreset`, `setModeToDefault/Music/Movie/Game/Bypass`, `setOutputToSpeaker/Headphone/Bluetooth/Earpiece/HDMI/SpeakerCover/GoogleTest` |
| DSP core | `OptimSpeaker_Init/Reinit/Reset/End/Getsize/Process`, `OptimSpeaker_Process_Left/Right/Stereo`, `OptimSpeaker_Set_Param/Get_Param/Set_User_Param/Set_User_Param_Sub_Block`, `OptimSpeaker_Get_User_Param`, `OptimSpeaker_PresetConversion`, `Optimspeaker_Get_User_Preset` |
| Processing modules (each: Init/Reset/Process/Set_Param/Get_Param/Set_User_Param) | `AGC_*`, `BassEx_*`, `MDRC_*`, `SX_*`, `LOUDNESS_*`, `GENERAL_EQ_*`, `LRC_EQ_MIXER_*`, `C_Extract_*`, `Reverb_Set_User_Param` |
| Compressor, delays, EQ, reverb | `ArkamysCompressor*`, `Compressor{Get,Set}{AttackTime,ReleaseTime,Ratio,Threshold,Knee,Makeup,Mute,Bypass,RMSPeak,Preset}`, `ArkamysDelay*`, `DelayMono*`, `MultitapsDelayMono*`, `ArkamysEq*`, `Eq*`, `ArkamysSpatReverb*`, `SpatReverb{Get,Set}*`, `SpatEarly*`, `SpatCluster*`, `SpatLateReverb*` |
| Filters and FFT | `Biquad*`, `IIRComputeCoeff`, `HLShelfComputeCoeff`, `GainComputeCoeff`, `Float_FFT_Process`, `Float_INV_FFT_Process`, `Float_Complex_Init`, `Hadamard4Process`, `Hadamard8Process` |
| Volume and presets (C++ ABI) | `load_preset_files`, `unload_preset`, `set_preset_ID`, `get_current_preset_id`, `volume_load_volume_table`, `volume_set_current_volume_step`, `volume_set_current_output_device`, `volume_get_current_dB/step/attenuation`, `VolumeTable_Table_{dB,lin}_{Speaker,Headphone}` |
| Test tone generator | `generate_stereo_bip_init`, `generate_stereo_bip_sound` |

Not re-exported: only C++ runtime symbols the original exported by accident (`_Unwind_*`, `__aeabi_*`, `_ZTI*`/`_ZTS*`, `__gxx_personality_v0`) and the window table `racinekHanning`, which is present but internal.

#### 1.2 Added in the reimplementation (71 symbols)

| Group | Functions |
|---|---|
| AINUR mode (reverb and loudness as in AINUR) | `OptimSpeaker_Set_Ainur_Chain`, `BassEx_Process_Ainur`, `SX_Ainur_Extract`, `ArkamysSpatReverbSetChain`, `ArkamysSpatReverbRunsInChain`, `ArkamysSpatReverbGetPortableMemSize` |
| C wrappers without C++ name mangling | `AudioEffectLib_CreateEffect/ReleaseEffect/GetDescriptor/QueryEffect/QueryNumberEffects`, `AkmOptimSpeaker_*` (C names), `AkmOptimSpeaker_BootstrapPortable`, `AkmOptimSpeaker_DestroyPortable`, `AkmOptimSpeaker_Get_Param_VendorPacket`, `AkmOptimSpeaker_Set_Param_VendorPacket` |
| Effect parameter parsing | `arkamys_effect_param_parse_set`, `arkamys_effect_param_prepare_get`, `arkamys_effect_param_align4`, `internal_eq_get_param_dispatch`, `internal_eq_set_param_dispatch` |
| Sample formats (PCM16, 8.24, 32-bit, float) | `arkamys_samples_to_float/from_float/copy/accumulate`, `arkamys_sample_bytes`, `get_input_converter`, `get_output_converter`, `pcm16_to_float_block`, `float_to_pcm16_block`, `q8_24_to_float_block`, `float_to_q8_24_block`, `float2short`, `short2float`, `float2longq31`, `longq31_2float`, `float2longq8_24`, `longq8_24_2float` |
| Volume and presets (C names) | `volume_*` (load/unload, step, device, dB↔linear, `volume_find_nearest_step`), `load_preset_files`, `unload_preset`, `set_preset_ID`, `get_current_preset_id` |
| Math helpers | `compute_rms`, `compute_peak`, `soft_limiter`, `fast_inv_sqrt`, `build_ar_lut`, `lut_interp`, `samples_to_spat_preset_delay`, `spat_preset_delay_to_samples` |
| Memory | `arkamys_malloc`, `arkamys_free` |

Internal, not exported: bundled bionic Android 6 `msun` math (`ark_sin`, `ark_cos`, `ark_exp`, `ark_pow`, `ark_log10` …), so results are bit-identical on any Android version, armv7 and arm64.

#### 1.3 Effect commands sent by the app

| Command | Sub-commands |
|---|---|
| `CMD_SET_WRAPPER (2)` | `FORCE_PRESET 0`, `VOLUME_STEP 1`, `DEVICE 2`, `NEW_DEVICE_CONNEXION 3`, `PRESET_STRUCT 4`, `SPEAKERCOVER 5`, `BYPASS 6`, `MODE 7`, `AINUR_CHAIN 8` |
| `CMD_GET_WRAPPER (5)` | `VERSION 0`, `ATTENUATION 1`, `PRESET 2`, `SPK_PRESET 3`, `HP_PRESET 4` |
| `CMD_SET_LIB (0)` | `LIB_BYPASS 0x1000001`, `STREAM_TYPE 0x1000005`, `TUNING_MODE 0x1000007` |

`FORCE_PRESET` takes a preset ID and an output: 0 speaker, 2 headphone, 3 Bluetooth, 6 covered speaker.

### 2. Service app `ArkamysAudio.apk`

Package `com.arkamys.audio`: privileged, persistent system app. Uses the shared library `com.arkamys.audiofx.jar` (class AkmFx).

#### 2.1 Components

| Component | Purpose | Status |
|---|---|---|
| `ArkamysAudioService` | effect service, Binder `ArkamysAudioAPI` | original |
| `BootReceiver` | starts the service after boot | original |
| `ShutDownReceiver` | shutdown handling | original |
| `ArkamysSettingsActivity` + `ArkamysSettingsFragment` | test screen: launcher icon, Settings → Sound tile, player equalizer button | **added** |

#### 2.2 Service methods (`ArkamysAudioAPI`, 26)

| Group | Methods |
|---|---|
| Effect | `setEffectEnable`, `getEffectEnable`, `getArkamysVersion`, `getAttenuation`, `setTuningMode`, `setBroadcastLogEnable` |
| Presets | `getPreset`, `getSPKPreset`, `getHPPreset`, `forceSPKPreset`, `forceHPPreset`, **`forceBTPreset` (added)** |
| Modes | `setMode`, `getAutoMode`, `setAutoMode`, `setSpeakerCover`, `getSpeakerCover` |
| Custom EQ | `setCustomTrebleLevel`/`get`, `setCustomBassLevel`/`get`, `setCustomWidening`/`get`, `setCustomEQ` |
| Raw access | `setParameter(byte[])`, `getParameter(cmd, param, byte[])` |

`forceBTPreset` was appended last (transaction 25), so the codes of the original methods (0–24) are unchanged and existing clients keep working.

#### 2.3 Test screen

| Item | What | Applied |
|---|---|---|
| Status | version, on/off, active preset, speaker, headphone, Bluetooth; tap to refresh (queried on a worker thread) | — |
| Arkamys effect | `EffectEnable` | immediately |
| Speaker preset | files from `/vendor/etc/arkamys/config_presets` | immediately |
| Headphone preset | same | immediately |
| Bluetooth preset | same | immediately |
| `ArkamysAudioService.xml` | every other key: switches for booleans, preset lists for `*Preset*` keys, number fields | after restart |
| Apply | restarts the service so it re-reads the configuration | — |
| Reset to vendor config | restores the original `ArkamysAudioService.xml` | — |

#### 2.4 Files added or changed

| File | Change |
|---|---|
| `service/src/com/arkamys/audio/ArkamysSettingsActivity.java` | **new**: activity shared by the launcher, the Settings tile and the equalizer action |
| `service/src/com/arkamys/audio/ArkamysSettingsFragment.java` | **new**: the test screen |
| `service/src/com/arkamys/audio/ArkamysAudioService.java` | `forceBTPreset` in the Binder stub |
| `service/src/com/arkamys/audio/ArkamysFxManager.java` | `forcePreset` stores the Bluetooth choice |
| `service/src/com/arkamys/audio/ServiceConfigurationManager.java` | `writeLastBTPreset` (key `defaultBluetoothPreset`) |
| `service/aidl/com/arkamys/audio/ArkamysAudioAPI.aidl` | `void forceBTPreset(int)`, appended last |
| `service/AndroidManifest.xml` | activity with `LAUNCHER`, `IA_SETTINGS` (category `ia.sound`), `DISPLAY_AUDIO_EFFECT_CONTROL_PANEL` |
| `service/res/values/strings.xml`, `service/res/values-ru/strings.xml` | screen strings in English and Russian |
| `service/res/drawable/ic_arkamys.xml` | **new**: vector icon |

### 3. Magisk module

beta.1 bootlooped: about 1970 Magic Mount mounts each created a `dev.mnt.blk.*` property, the property area filled up and system_server died on `SystemProperties.set`.

| Version | Change |
|---|---|
| beta.2 | New files merged with overlayfs instead of Magic Mount: about 30 mounts instead of about 1970 (bootloop fixed). jar moved into `priv-app`, `default-permissions` removed, `akm` moved to the module root. |
| beta.3 | Overlay layer copied to `/dev`: overlayfs rejects `/data` (f2fs, `EINVAL`). |
| beta.4 | Overlay layer in an ext4 image `ovl.img` on a loop device: tmpfs has no xattr (`EOPNOTSUPP`). |
| beta.5 | Image labelled `adb_data_file` plus `allow kernel adb_data_file` in `sepolicy.rule`. **The module works.** |
| beta.6 | `service.sh`: `debug.log` no longer grows without limit. |
| beta.7 | Test screen in the app. |
| beta.8 | Screen no longer freezes; Apply can no longer leave the service stopped. |
| beta.9 | Bluetooth preset applied immediately. |

`akm` is now run as: `su -c sh /data/adb/modules/ArkamysAudio/akm`

---

## Русский

### 1. Библиотека эффекта `libarkamys.so`

C-реимплементация оригинальной BLU `libarkamys_bip_5min.so`. Две сборки, обе экспортируют одни и те же 352 символа:

| Сборка | Назначение |
|---|---|
| `vendor/lib/soundfx/libarkamys.so` | armv7, её загружает 32-битный аудио-HAL этого телефона |
| `vendor/lib64/soundfx/libarkamys.so` | arm64, для телефонов с 64-битным HAL |

Обработка звука совпадает с оригиналом бит в бит на armv7 и arm64.

#### 1.1 Сохранено из оригинала (API совместим)

| Группа | Функции |
|---|---|
| Точка входа эффекта | `AELI`, `ArkamysLib_CreateEffect`, `ArkamysLib_ReleaseEffect`, `ArkamysLib_GetDescriptor`, `ArkamysLib_Open`, `ArkamysLib_Close`, `ArkamysLib_Config`, `ArkamysLib_Command`, `ArkamysLib_Process`, `ArkamysLib_CopyConfig`, `ArkamysLib_Effect_GetDescriptor` |
| Модуль OptimSpeaker (C++ ABI) | `AkmOptimSpeaker_Init/Open/Close/Config/Process/Get_Param/Set_Param`, `AkmOptimSpeaker_Get_Param_Wrapper`, `AkmOptimSpeaker_Set_Param_Wrapper`, `applyPreset` |
| Класс `SmartPhone` | `initialize`, `resetState`, `forcePreset`, `setPreset`, `getCurrentPreset`, `getSPKPreset`, `getHPPreset`, `setModeToDefault/Music/Movie/Game/Bypass`, `setOutputToSpeaker/Headphone/Bluetooth/Earpiece/HDMI/SpeakerCover/GoogleTest` |
| Ядро DSP | `OptimSpeaker_Init/Reinit/Reset/End/Getsize/Process`, `OptimSpeaker_Process_Left/Right/Stereo`, `OptimSpeaker_Set_Param/Get_Param/Set_User_Param/Set_User_Param_Sub_Block`, `OptimSpeaker_Get_User_Param`, `OptimSpeaker_PresetConversion`, `Optimspeaker_Get_User_Preset` |
| Модули обработки (у каждого: Init/Reset/Process/Set_Param/Get_Param/Set_User_Param) | `AGC_*`, `BassEx_*`, `MDRC_*`, `SX_*`, `LOUDNESS_*`, `GENERAL_EQ_*`, `LRC_EQ_MIXER_*`, `C_Extract_*`, `Reverb_Set_User_Param` |
| Компрессор, задержки, EQ, реверб | `ArkamysCompressor*`, `Compressor{Get,Set}{AttackTime,ReleaseTime,Ratio,Threshold,Knee,Makeup,Mute,Bypass,RMSPeak,Preset}`, `ArkamysDelay*`, `DelayMono*`, `MultitapsDelayMono*`, `ArkamysEq*`, `Eq*`, `ArkamysSpatReverb*`, `SpatReverb{Get,Set}*`, `SpatEarly*`, `SpatCluster*`, `SpatLateReverb*` |
| Фильтры и FFT | `Biquad*`, `IIRComputeCoeff`, `HLShelfComputeCoeff`, `GainComputeCoeff`, `Float_FFT_Process`, `Float_INV_FFT_Process`, `Float_Complex_Init`, `Hadamard4Process`, `Hadamard8Process` |
| Громкость и пресеты (C++ ABI) | `load_preset_files`, `unload_preset`, `set_preset_ID`, `get_current_preset_id`, `volume_load_volume_table`, `volume_set_current_volume_step`, `volume_set_current_output_device`, `volume_get_current_dB/step/attenuation`, `VolumeTable_Table_{dB,lin}_{Speaker,Headphone}` |
| Генератор тестового сигнала | `generate_stereo_bip_init`, `generate_stereo_bip_sound` |

Не экспортируются: только служебные символы C++-рантайма, которые оригинал экспортировал случайно (`_Unwind_*`, `__aeabi_*`, `_ZTI*`/`_ZTS*`, `__gxx_personality_v0`), и таблица окна `racinekHanning`. Она есть, но внутренняя.

#### 1.2 Добавлено в реимплементации (71 символ)

| Группа | Функции |
|---|---|
| Режим AINUR (реверб и loudness, как в AINUR) | `OptimSpeaker_Set_Ainur_Chain`, `BassEx_Process_Ainur`, `SX_Ainur_Extract`, `ArkamysSpatReverbSetChain`, `ArkamysSpatReverbRunsInChain`, `ArkamysSpatReverbGetPortableMemSize` |
| C-обёртки без C++-манглинга | `AudioEffectLib_CreateEffect/ReleaseEffect/GetDescriptor/QueryEffect/QueryNumberEffects`, `AkmOptimSpeaker_*` (C-имена), `AkmOptimSpeaker_BootstrapPortable`, `AkmOptimSpeaker_DestroyPortable`, `AkmOptimSpeaker_Get_Param_VendorPacket`, `AkmOptimSpeaker_Set_Param_VendorPacket` |
| Разбор параметров эффекта | `arkamys_effect_param_parse_set`, `arkamys_effect_param_prepare_get`, `arkamys_effect_param_align4`, `internal_eq_get_param_dispatch`, `internal_eq_set_param_dispatch` |
| Форматы сэмплов (PCM16, 8.24, 32-bit, float) | `arkamys_samples_to_float/from_float/copy/accumulate`, `arkamys_sample_bytes`, `get_input_converter`, `get_output_converter`, `pcm16_to_float_block`, `float_to_pcm16_block`, `q8_24_to_float_block`, `float_to_q8_24_block`, `float2short`, `short2float`, `float2longq31`, `longq31_2float`, `float2longq8_24`, `longq8_24_2float` |
| Громкость и пресеты (C-имена) | `volume_*` (load/unload, шаг, устройство, dB↔линейная, `volume_find_nearest_step`), `load_preset_files`, `unload_preset`, `set_preset_ID`, `get_current_preset_id` |
| Вспомогательная математика | `compute_rms`, `compute_peak`, `soft_limiter`, `fast_inv_sqrt`, `build_ar_lut`, `lut_interp`, `samples_to_spat_preset_delay`, `spat_preset_delay_to_samples` |
| Память | `arkamys_malloc`, `arkamys_free` |

Внутреннее, без экспорта: встроенная математика `msun` из bionic Android 6 (`ark_sin`, `ark_cos`, `ark_exp`, `ark_pow`, `ark_log10` …). Благодаря ей результат бит в бит одинаков на любой версии Android, armv7 и arm64.

#### 1.3 Команды эффекта, которые шлёт приложение

| Команда | Подкоманды |
|---|---|
| `CMD_SET_WRAPPER (2)` | `FORCE_PRESET 0`, `VOLUME_STEP 1`, `DEVICE 2`, `NEW_DEVICE_CONNEXION 3`, `PRESET_STRUCT 4`, `SPEAKERCOVER 5`, `BYPASS 6`, `MODE 7`, `AINUR_CHAIN 8` |
| `CMD_GET_WRAPPER (5)` | `VERSION 0`, `ATTENUATION 1`, `PRESET 2`, `SPK_PRESET 3`, `HP_PRESET 4` |
| `CMD_SET_LIB (0)` | `LIB_BYPASS 0x1000001`, `STREAM_TYPE 0x1000005`, `TUNING_MODE 0x1000007` |

`FORCE_PRESET` принимает пару «пресет + выход»: 0 — динамик, 2 — наушники, 3 — Bluetooth, 6 — закрытый динамик.

### 2. Приложение `ArkamysAudio.apk`

Пакет `com.arkamys.audio`: системное привилегированное persistent-приложение. Использует библиотеку `com.arkamys.audiofx.jar` (класс AkmFx).

#### 2.1 Компоненты

| Компонент | Назначение | Статус |
|---|---|---|
| `ArkamysAudioService` | сервис эффекта, Binder `ArkamysAudioAPI` | из оригинала |
| `BootReceiver` | запуск после загрузки | из оригинала |
| `ShutDownReceiver` | выключение | из оригинала |
| `ArkamysSettingsActivity` + `ArkamysSettingsFragment` | тестовый экран: значок, «Настройки → Звук», кнопка эквалайзера в плеерах | **добавлено** |

#### 2.2 Методы сервиса (`ArkamysAudioAPI`, 26)

| Группа | Методы |
|---|---|
| Эффект | `setEffectEnable`, `getEffectEnable`, `getArkamysVersion`, `getAttenuation`, `setTuningMode`, `setBroadcastLogEnable` |
| Пресеты | `getPreset`, `getSPKPreset`, `getHPPreset`, `forceSPKPreset`, `forceHPPreset`, **`forceBTPreset` (добавлено)** |
| Режимы | `setMode`, `getAutoMode`, `setAutoMode`, `setSpeakerCover`, `getSpeakerCover` |
| Пользовательский EQ | `setCustomTrebleLevel`/`get`, `setCustomBassLevel`/`get`, `setCustomWidening`/`get`, `setCustomEQ` |
| Прямой доступ | `setParameter(byte[])`, `getParameter(cmd, param, byte[])` |

`forceBTPreset` добавлен в конец (транзакция 25), поэтому номера исходных методов (0–24) не изменились, и существующие клиенты продолжают работать.

#### 2.3 Тестовый экран

| Пункт | Что | Применяется |
|---|---|---|
| Состояние | версия, вкл/выкл, активный пресет, динамик, наушники, Bluetooth; нажатие обновляет (запрос в фоновом потоке) | — |
| Эффект Arkamys | `EffectEnable` | сразу |
| Пресет: динамик | файлы из `/vendor/etc/arkamys/config_presets` | сразу |
| Пресет: наушники | то же | сразу |
| Пресет: Bluetooth | то же | сразу |
| `ArkamysAudioService.xml` | все остальные ключи: переключатели, списки пресетов для ключей `*Preset*`, числовые поля | после перезапуска |
| Применить | перезапускает сервис, он заново читает настройки | — |
| Сбросить к /vendor | возвращает исходный `ArkamysAudioService.xml` | — |

#### 2.4 Добавленные и изменённые файлы

| Файл | Изменение |
|---|---|
| `service/src/com/arkamys/audio/ArkamysSettingsActivity.java` | **новый**: activity, через которую значок, пункт в «Настройках» и кнопка эквалайзера открывают один экран |
| `service/src/com/arkamys/audio/ArkamysSettingsFragment.java` | **новый**: сам тестовый экран |
| `service/src/com/arkamys/audio/ArkamysAudioService.java` | метод `forceBTPreset` в Binder |
| `service/src/com/arkamys/audio/ArkamysFxManager.java` | `forcePreset` сохраняет выбор для Bluetooth |
| `service/src/com/arkamys/audio/ServiceConfigurationManager.java` | `writeLastBTPreset` (ключ `defaultBluetoothPreset`) |
| `service/aidl/com/arkamys/audio/ArkamysAudioAPI.aidl` | `void forceBTPreset(int)` в конце интерфейса |
| `service/AndroidManifest.xml` | activity с `LAUNCHER`, `IA_SETTINGS` (категория `ia.sound`), `DISPLAY_AUDIO_EFFECT_CONTROL_PANEL` |
| `service/res/values/strings.xml`, `service/res/values-ru/strings.xml` | строки экрана на английском и русском |
| `service/res/drawable/ic_arkamys.xml` | **новый**: векторный значок |

### 3. Модуль Magisk

beta.1 уходила в бутлуп: около 1970 монтирований Magic Mount создавали по свойству `dev.mnt.blk.*`, область свойств заполнялась, и system_server падал на `SystemProperties.set`.

| Версия | Изменение |
|---|---|
| beta.2 | Новые файлы подключаются через overlayfs, а не Magic Mount: ~30 монтирований вместо ~1970 (бутлуп устранён). jar перенесён в `priv-app`, `default-permissions` удалён, `akm` в корне модуля. |
| beta.3 | Слой overlay копируется в `/dev`: overlayfs не принимает `/data` (f2fs, `EINVAL`). |
| beta.4 | Слой overlay в ext4-образе `ovl.img` на loop-устройстве: у tmpfs нет xattr (`EOPNOTSUPP`). |
| beta.5 | Метка образа `adb_data_file` и правило `allow kernel adb_data_file` в `sepolicy.rule`. **Модуль заработал.** |
| beta.6 | `service.sh`: `debug.log` больше не растёт без конца. |
| beta.7 | Тестовый экран в приложении. |
| beta.8 | Экран не подвисает; «Применить» больше не может оставить сервис остановленным. |
| beta.9 | Пресет Bluetooth применяется сразу. |

`akm` теперь запускается так: `su -c sh /data/adb/modules/ArkamysAudio/akm`
