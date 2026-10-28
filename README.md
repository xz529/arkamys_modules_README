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

#### 1.4 Preset descriptions

The following descriptions refer to the restored DSP implementation and the mapped BLU V4 preset fields. *Enabled* and *bypassed* describe the preset configuration; actual playback behavior also depends on the selected processing chain and runtime controls.

##### `SPK` — Base BLU profile

The base BLU audio profile. It handles channel gain (+4.5 dB for the left channel, 0 dB for the right), BassEx enhancement (+3 dB of added bass), frequency correction through a 5-band LRC and 10-band output EQ, as well as AGC and MDRC processing. Reverb, SX, and Loudness are disabled.

##### `0_TUNING` — Neutral profile

A neutral audio profile. AGC, SX, BassEx, Reverb, Loudness, and MDRC are configured for bypass. The output EQ has no active bands. No additional bass enhancement or spatial effects are applied through those modules.

##### `1_TUNING` — Bass and stereo enhancement

Provides bass enhancement, stereo processing, equalization, and dynamic compression.

- **BassEx:** Enabled, +5 dB of added bass.
- **SX:** Enabled.
- **Output EQ:** 5 bands, including +5 dB at 700 Hz and −4 dB at 9000 Hz.
- **MDRC:** Enabled.
- **Reverb:** Disabled.

##### `2_TUNING` — Enhanced bass and multiband processing

Provides stronger bass enhancement, stereo processing, multiband frequency correction, and dynamic compression.

- **BassEx:** Enabled, +8 dB of added bass.
- **SX:** Enabled, with a +9 dB filter at 200 Hz.
- **LRC:** 3-band frequency correction.
- **Output EQ:** 6 bands.
- **MDRC:** Enabled.
- **Reverb:** Disabled.

##### `8_BYPASS` — Processing bypass

A bypass configuration intended to disable additional audio processing.

- **BassEx, AGC, SX, Reverb, Loudness, MDRC:** Bypassed.
- **Output EQ:** 0 active bands.

##### `9_BYPASS` — Processing bypass

Identical in parameters to `8_BYPASS`. A bypass configuration with no additional bass enhancement, spatial effects, or active output EQ bands.

##### `10_TUNING` — Advanced frequency correction

Primarily focuses on detailed frequency correction and LRC balance adjustments.

- **Output EQ:** 9 bands, including +5.5 dB at 800 Hz and multiple mid/high-frequency cuts.
- **SX, BassEx, Reverb, MDRC:** Bypassed.
- **Loudness:** Contains a +9 dB setting at 100 Hz, but Loudness is not executed in the standard BLU processing chain.

##### `12_TUNING` — Bass, compression, and spatial reverb

Combines bass enhancement, equalization, compression, and reverberation for the AINUR processing mode.

- **BassEx:** Enabled, +4 dB of added bass.
- **Output EQ:** 9 bands, including +6 dB at 1000 Hz.
- **MDRC:** Enabled.
- **SX:** Disabled.
- **Reverb:** Enabled in the preset.
- **Cluster gain:** −14.30 dB.
- **Late reverb gain:** −10.43 dB.
- **Maximum delay:** 202.85 ms.

##### `14_TUNING` — Bass, stereo, and reverb enhancement

Combines bass enhancement, stereo processing, equalization, compression, and reverberation for the AINUR processing mode.

- **BassEx:** Enabled, +2 dB of added bass.
- **SX:** Enabled, with a +6 dB filter at 800 Hz.
- **Output EQ:** 10 bands.
- **MDRC:** Enabled.
- **Reverb:** Enabled in the preset.
- **Cluster gain:** −9.30 dB.
- **Late reverb gain:** −10.92 dB.
- **Maximum delay:** 45.98 ms.

##### `16_TUNING` — Advanced EQ and extended reverb

Combines bass enhancement, extensive frequency correction, dynamic compression, and a pronounced late reverberation tail for the AINUR processing mode.

- **BassEx:** Enabled, +5 dB of added bass.
- **Output EQ:** 10 bands, including +6 dB at 600 Hz.
- **MDRC:** Enabled.
- **SX:** Disabled.
- **Reverb:** Enabled in the preset.
- **Cluster gain:** −11.31 dB.
- **Late reverb gain:** −1.32 dB.
- **Maximum delay:** 166.66 ms.

##### `19_BYPASS` — Processing bypass

A bypass configuration in which the main DSP enhancement modules are disabled by their respective control flags.

- **BassEx, SX, Reverb, Loudness, MDRC:** Bypassed.
- **Output EQ:** 0 active bands.
- **Note:** A +10 dB setting at 600 Hz remains in the bypassed Loudness block and therefore does not produce amplification through that block.

##### Important notes

- Reverberation in `12_TUNING`, `14_TUNING`, and `16_TUNING` requires the **AINUR DSP processing chain** to be active.
- The reverberator is not invoked in the standard BLU processing chain.
- All specified dB values represent gain settings of individual DSP modules, **not** the overall device output volume.
- Enabled/bypassed states describe the preset configuration; audible results depend on the active DSP chain and runtime settings.
- These descriptions are based on the restored Arkamys Audio implementation and preset mappings. Full behavioral equivalence with the original proprietary BLU library requires additional verification.


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

#### 1.4 Описание пресетов

Описания относятся к восстановленной реализации DSP и сопоставленным полям пресетов BLU V4. Статусы *включён* и *обход* описывают конфигурацию пресета; реальное поведение при воспроизведении также зависит от выбранной цепочки обработки и настроек во время выполнения.

##### `SPK` — Базовый профиль BLU

Базовый звуковой профиль BLU. Отвечает за усиление каналов (+4,5 дБ для левого канала, 0 дБ для правого), усиление баса BassEx (+3 дБ добавляемого баса), частотную коррекцию через 5-полосный LRC и 10-полосный выходной эквалайзер, а также обработку AGC и MDRC. Реверберация, SX и Loudness отключены.

##### `0_TUNING` — Нейтральный профиль

Нейтральный звуковой профиль. AGC, SX, BassEx, Reverb, Loudness и MDRC настроены на обход обработки (Bypass). Выходной эквалайзер не содержит активных полос. Дополнительное усиление баса и пространственные эффекты через эти модули не применяются.

##### `1_TUNING` — Усиление баса и стереообработка

Отвечает за усиление баса, стереообработку, эквализацию и динамическую компрессию.

- **BassEx:** Включён, +5 дБ добавляемого баса.
- **SX:** Включён.
- **Выходной EQ:** 5 полос, включая +5 дБ на 700 Гц и −4 дБ на 9000 Гц.
- **MDRC:** Включён.
- **Реверберация:** Отключена.

##### `2_TUNING` — Усиленный бас и многополосная обработка

Отвечает за более сильное усиление баса, стереообработку, многополосную частотную коррекцию и динамическую компрессию.

- **BassEx:** Включён, +8 дБ добавляемого баса.
- **SX:** Включён, с фильтром +9 дБ на 200 Гц.
- **LRC:** 3-полосная частотная коррекция.
- **Выходной EQ:** 6 полос.
- **MDRC:** Включён.
- **Реверберация:** Отключена.

##### `8_BYPASS` — Обход обработки

Профиль, предназначенный для отключения дополнительной обработки звука.

- **BassEx, AGC, SX, Reverb, Loudness, MDRC:** Обход.
- **Выходной EQ:** 0 активных полос.

##### `9_BYPASS` — Обход обработки

По параметрам идентичен `8_BYPASS`. Профиль обхода обработки без дополнительного усиления баса, пространственных эффектов и активных полос выходного эквалайзера.

##### `10_TUNING` — Расширенная частотная коррекция

Основное назначение — детальная коррекция частот и настройка баланса LRC.

- **Выходной EQ:** 9 полос, включая +5,5 дБ на 800 Гц и несколько фильтров ослабления средних и высоких частот.
- **SX, BassEx, Reverb, MDRC:** Обход.
- **Loudness:** Содержит настройку +9 дБ на 100 Гц, однако Loudness не выполняется в стандартной цепочке обработки BLU.

##### `12_TUNING` — Бас, компрессия и пространственная реверберация

Объединяет усиление баса, эквалайзер, компрессию и реверберацию для режима обработки AINUR.

- **BassEx:** Включён, +4 дБ добавляемого баса.
- **Выходной EQ:** 9 полос, включая +6 дБ на 1000 Гц.
- **MDRC:** Включён.
- **SX:** Отключён.
- **Реверберация:** Разрешена в пресете.
- **Cluster Gain:** −14,30 дБ.
- **Late Reverb Gain:** −10,43 дБ.
- **Максимальная задержка:** 202,85 мс.

##### `14_TUNING` — Бас, стереообработка и реверберация

Объединяет усиление баса, стереообработку, эквалайзер, компрессию и реверберацию для режима AINUR.

- **BassEx:** Включён, +2 дБ добавляемого баса.
- **SX:** Включён, с фильтром +6 дБ на 800 Гц.
- **Выходной EQ:** 10 полос.
- **MDRC:** Включён.
- **Реверберация:** Разрешена в пресете.
- **Cluster Gain:** −9,30 дБ.
- **Late Reverb Gain:** −10,92 дБ.
- **Максимальная задержка:** 45,98 мс.

##### `16_TUNING` — Расширенный EQ и длительная реверберация

Объединяет усиление баса, расширенную частотную коррекцию, динамическую компрессию и выраженный поздний хвост реверберации для режима AINUR.

- **BassEx:** Включён, +5 дБ добавляемого баса.
- **Выходной EQ:** 10 полос, включая +6 дБ на 600 Гц.
- **MDRC:** Включён.
- **SX:** Отключён.
- **Реверберация:** Разрешена в пресете.
- **Cluster Gain:** −11,31 дБ.
- **Late Reverb Gain:** −1,32 дБ.
- **Максимальная задержка:** 166,66 мс.

##### `19_BYPASS` — Обход обработки

Профиль обхода, в котором основные модули улучшения звука отключены соответствующими управляющими флагами.

- **BassEx, SX, Reverb, Loudness, MDRC:** Обход.
- **Выходной EQ:** 0 активных полос.
- **Примечание:** Настройка +10 дБ на 600 Гц сохраняется в блоке Loudness, находящемся в режиме обхода, поэтому не создаёт усиления через этот блок.

##### Важные примечания

- Реверберация в `12_TUNING`, `14_TUNING` и `16_TUNING` работает только при активной **цепочке обработки AINUR DSP**.
- В стандартной цепочке обработки BLU ревербератор не вызывается.
- Все указанные значения в дБ относятся к отдельным DSP-модулям, **а не к общей громкости устройства**.
- Состояния включения и обхода описывают конфигурацию пресета; слышимый эффект зависит от активной DSP-цепочки и настроек во время выполнения.
- Описания основаны на восстановленной реализации Arkamys Audio и сопоставлении пресетов. Полная эквивалентность поведения с оригинальной проприетарной библиотекой BLU требует дополнительной проверки.


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
