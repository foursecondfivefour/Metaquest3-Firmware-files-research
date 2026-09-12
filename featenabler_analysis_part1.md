# ДОКЛАД ОБ АНАЛИЗЕ БИНАРНИКА `featenabler.img`

## Часть 1: Функции №1–100 (диапазон 0x0 – 0x4370)

---

## 1. ОБЩИЕ СВЕДЕНИЯ О БИНАРНИКЕ

| Параметр | Значение |
|----------|----------|
| Файл | `C:\questerthree\featenabler.img` |
| База (IDB) | `featenabler.img.i64` |
| Архитектура | **ARM64 (AArch64)**, ELF-образ TEE-модуля |
| Image base | `0x0` |
| Размер образа | `0x113B8` (~70 КБ) |
| MD5 | `57ed3ce39a66f4c7dc46015f0093b048` |
| SHA-256 | `133c801f29520f439e4f07191f55583097cd895a303bf3e962cbac9abd102a7b` |
| Всего функций (после очистки) | **208** |
| Точка входа | `__module_entry_thunk` @ `0x0` |

**Назначение:** Trusted Application (TA) «Feature Enabler» для QSEE/TEE-окружения Qualcomm — приложение, управляющее включением/валидацией лицензируемых фич устройства (Feature Enabler / License Validator). Реализует GP TEE Internal Core API, работу с RPMB, QCBOR-кодирование ответов и проверку лицензий.

**Сегменты:**

| Сегмент | Начало | Конец | Права |
|---------|--------|-------|-------|
| LOAD (код) | `0x0` | `0xB8A4` | r-x |
| LOAD | `0xC000` | `0xC0F0` | rw- |
| LOAD | `0xD000` | `0xD1D8` | rw- |
| LOAD | `0xE000` | `0xE391` | rw- |
| LOAD | `0xF000` | `0x112E0` | rw- |
| extern | `0x112E0` | `0x113B8` | --- |

---

## 2. ГРАНИЦЫ РАССМАТРИВАЕМОГО ДИАПАЗОНА

Функции №1–100 занимают адреса **0x0 – 0x4370** (функция №100 `__stack_chk_guard_ref` заканчивается на `0x437C`).
Суммарный объём кода: **~17 400 байт** (~47% всего кода TA).

**Функции с нулевым количеством xref (2 шт.):**
- `__funcs_on_exit_wrapper` @ `0x27A0` — обёртка, вызывается через указатель
- `TA_InvokeCommandEntryPoint_CGP` @ `0x2E80` — вызывается косвенно через таблицу интерфейсов


---

## 3. КАТЕГОРИЗАЦИЯ ФУНКЦИЙ №1–100

### 3.1. Импорт-стабы и системные обёртки (24 функции)

**`__module_entry_thunk` @ 0x0** — точка входа ELF-образа (18 xref).

| Адрес | Имя | Назначение |
|-------|-----|------------|
| 0x20 | `qsee_log` | **самая вызываемая (134 xref)** — логирование |
| 0x30 | `qsee_malloc` | аллокация (13 xref) |
| 0x40 | `qsee_free` | освобождение (21 xref) |
| 0x50 | `qsee_err_fatal` | фатальная ошибка |
| 0x60 | `__funcs_on_exit` | деструкторы при выходе |
| 0x70 | `cmnlib_init` | инициализация crypto-lib |
| 0x80 | `GPAppLib_init` | инициализация GP App Library |
| 0x90 | `GPAppLib_appInit` | app init |
| 0xa0 | `GPAppLib_appShutdown` | app shutdown |
| 0xb0 | `cmnlib_release` | релиз crypto-lib |
| 0xc0 | `GPAppLib_handleRequest` | обработка запроса |
| 0xd0 | `CApp_openSession` | открытие сессии |
| 0xe0 | `qsee_open` | открытие QSEE-объекта (4 xref) |
| 0xf0 | `qsee_is_sw_fuse_blown` | проверка перегоревшего fuse |
| 0x100 | `qsee_get_secure_state` | чтение secure state (3 xref) |
| 0x110 | `qsee_stor_write_sectors` | запись секторов |
| 0x120 | `qsee_stor_device_init` | init storage device |
| 0x130 | `qsee_stor_device_get_info` | инфо устройства |
| 0x140 | `qsee_stor_read_sectors` | чтение секторов |
| 0x150 | `qsee_stor_open_partition` | открытие партиции |
| 0x160 | `qsee_stor_add_partition` | добавление партиции |
| 0x170 | `qsee_stor_client_get_info` | инфо клиента |
| 0x180 | `qsee_stor_remove_client` | удаление клиента |


### 3.2. Jump-трамплины регистров (7 функций)

`jump_x8` (26 xref), `jump_x9` (10), `jump_x10` (3), `jump_x11`, `jump_x20`, `jump_x21`, `jump_x23` @ `0x630`–`0x690` — по 8 байт (`MOV Xn,X0; BR Xn`). Непрямые диспетчеры вызовов по регистру.

### 3.3. TA-точки входа и командные обработчики (7 функций)

| Адрес | Имя | Размер | Роль |
|-------|-----|--------|------|
| 0x6a0 | `TA_LogAppInit` | 16 | логирование старта |
| 0x6b8 | `TZ_AppCmdHandler` | 680 | главный диспетчер команд (3 xref) |
| 0x968 | `TA_LogAppShutdown` | 16 | логирование завершения |
| 0x2ae8 | `TA_OpenCloseSessionEntryPoint` | 256 | GP-точка входа open/close session |
| 0x2be8 | `TA_InvokeCommandEntryPoint` | 648 | GP-точка входа invoke command |
| 0x2e80 | `TA_InvokeCommandEntryPoint_CGP` | 316 | вариант для CGP-профиля |

### 3.4. Бизнес-логика Feature Enabler — ядро TA (9 функций)

| Адрес | Имя | Размер | Назначение |
|-------|-----|--------|------------|
| 0x190 | `FeatureEnabler_Handler` | **1180** | главный обработчик FE-команд |
| 0x980 | `GetEnabledFeatures_Handler` | 592 | возврат списка включённых фич |
| 0xbd0 | `GetInstalledFeatures_Prepare` | 656 | подготовка списка установленных фич |
| 0xe60 | `GetInstalledFeatures_FromRPMB` | 628 | чтение установленных фич из RPMB |
| 0x10d4 | `GetInstalledFeatures_Handler` | 232 | обработчик запроса |
| 0x11bc | `LicenseValidateAndEnable_Handler` | 820 | валидация лицензии + включение |
| 0x14f0 | `LicenseValidator_Handler` | **1156** | ядро валидации лицензий |
| 0x1974 | `LicenseValidatorTest_Handler` | 632 | тестовая валидация |
| 0x27d0 | `GetOptionalFeatureFlag` | 52 | флаг опциональной фичи |

**Вывод:** это «мозг» TA — полный цикл: чтение статуса из RPMB -> валидация лицензии -> включение фич.

### 3.5. DisplayCore — управление SW-fuse дисплея (11 функций, 0x1bec–0x251c)

| Адрес | Имя | Размер |
|-------|-----|--------|
| 0x1bec | `DisplayCore_GetFeatureString` | 100 |
| 0x1c58 | `DisplayCore_EnableSwFuse` | 168 |
| 0x1d08 | `DisplayCore_ReadSwFuseStatus` | 132 |
| 0x1d8c | `DisplayCore_IsFeatureSupported` | 388 (4 xref) |
| 0x1f18 | `DisplayCore_GetFeatureList` | — |
| 0x20e8 | `DisplayCore_EncodeMetaData` | — |
| 0x2174 | `DisplayCore_Close` | — |
| 0x22ec | `DisplayCore_GetSwFuseStatus` | 332 (4 xref) |
| 0x2438 | `DisplayCore_GetFeatureName` | 228 (4 xref) |
| 0x251c | `DisplayCore_ConfigureSwFuse` | — |
| 0x2524 | `DisplayCore_Open` | — |

Модуль управляет программными fuse-битами дисплея: чтение/запись статуса, активация, получение имён/строк фич, кодирование метаданных.


### 3.6. IPCore и CElfFile (3 функции)

| Адрес | Имя | Размер | xref |
|-------|-----|--------|------|
| 0x260c | `IPCore_OpenModule` | 104 | 7 |
| 0x267c | `IPCore_CloseModule` | 72 | 7 |
| 0x280c | `CElfFile_invoke` | 732 | — |

`CElfFile_invoke` — вызов ELF-модуля (загрузка/исполнение IP-ядра).

### 3.7. GPParams — система сессий/параметров (18 функций, 0x278c–0x3aa8)

**Утилиты-возвраты:**

| Адрес | Имя | Размер |
|-------|-----|--------|
| 0x278c | `GPParams_ReturnUnsupported` | 8 |
| 0x32b4 | `GPParams_SessionListReset` | 8 |
| 0x32dc | `GPParams_ReturnOk` | 8 |
| 0x32ec | `GPParams_ReturnErr` | 8 |
| 0x32fc | `GPParams_ReturnErr2` | 8 |
| 0x5358 | `GPParams_ReturnTrue` | 8 |

**Управление списком сессий (thunks):** `0x2fc4 GPParams_SessionListClearThunk` (12), `0x2fd8 GPParams_SessionListClearThunk2` (12).

**Жизненный цикл сессий:**

| Адрес | Имя | Размер |
|-------|-----|--------|
| 0x2e70 | `GPParams_InitListNode` | 16 |
| 0x330c | `GPParams_allocSession` | 584 |
| 0x3554 | `GPParams_freeSession` | 512 |
| 0x3754 | `GPParams_EntryInit` | 56 |
| 0x3794 | `GPParams_newFromMinkInternal` | 340 |
| 0x38f0 | `GPParams_EntryDtor` | 124 |
| 0x3974 | `GPParams_SessionCtor` | 172 |
| 0x3a28 | `GPParams_SessionDtor` | 120 |
| 0x3aa8 | `GPParams_Validate` | 64 |

Это полноценная объектная модель сессий GP: конструкторы/деструкторы, пул, валидация, интеграция с MINK (Mink IPC).

### 3.8. ObjectList — связный список объектов (6 функций)

| Адрес | Имя | Размер |
|-------|-----|--------|
| 0x2fec | `ObjectList_lookup_code` | 44 |
| 0x3018 | `ObjectList_status_to_code` | 32 |
| 0x3040 | `ObjectList_get` | 64 |
| 0x3088 | `ObjectList_find` | 92 |
| 0x30ec | `ObjectList_append` | 308 |
| 0x3228 | `ObjectList_clear` | 132 |

Классический контейнер: поиск, добавление, очистка, преобразование статусов в GP-коды.

### 3.9. CommonUtils — утилиты общего назначения (6 функций)

| Адрес | Имя | Размер | Назначение |
|-------|-----|--------|------------|
| 0x3af0 | `CommonUtils_Base64Encode` | 100 | Base64-кодирование |
| 0x3b54 | `CommonUtils_BytesToHex` | 148 | hex-кодирование |
| 0x3be8 | `CommonUtils_HexToBytes` | 200 | hex-декодирование |
| 0x3cb0 | `CommonUtils_LoadPair` | 20 | загрузка пары (ключ-значение) |
| 0x4164 | `CommonUtils_StrDup` | 92 | дублирование строк |
| 0x41c0 | `CommonUtils_StrToU64` | 280 (5 xref) | строка -> u64 |


### 3.10. QCBOR-кодирование (в диапазоне 1-100 только 1 функция)

| Адрес | Имя | Размер |
|-------|-----|--------|
| 0x3d34 | `QCBOREncode_EncodeBuffer` | 820 (10 xref) |

Ключевая функция сериализации ответов TA - использует библиотеку QCBOR (Qualcomm CBOR).

### 3.11. Libc-функции (6 функций, слинкованы статически)

| Адрес | Имя | Размер |
|-------|-----|--------|
| 0x26cc | `scalbn` | 148 |
| 0x3cc4 | `strlcpy` | 112 |
| 0x4068 | `memchr` | 52 |
| 0x409c | `memset` | 200 (10 xref) |
| 0x42d8 | `strlen` | 100 (8 xref) |
| 0x433c | `strchr` | 52 |

### 3.12. Прочее

| Адрес | Имя | Размер | Примечание |
|-------|-----|--------|------------|
| 0x279c | `j_qsee_err_fatal` | 4 | самая горячая ветвь (26 xref) |
| 0x27a0 | `__funcs_on_exit_wrapper` | 40 | обёртка exit |
| 0x32c4 | `nullsub_2` | 4 | пустышка |
| 0x32d0 | `nullsub_3` | 4 | пустышка |
| 0x4370 | `__stack_chk_guard_ref` | 12 | ссылка на stack canary |


---

## 4. СТАТИСТИКА ПО РАЗМЕРАМ

| Диапазон | Кол-во |
|----------|--------|
| 1-16 байт (thunk/заглушки) | ~42 |
| 17-100 байт (утилиты) | ~18 |
| 101-300 байт (средние) | ~20 |
| 301-600 байт (крупные) | ~12 |
| 600+ байт (тяжёлые) | 8 |

**Топ-10 самых крупных функций первой сотни:**

| Адрес | Имя | Размер |
|-------|-----|--------|
| 0x190 | `FeatureEnabler_Handler` | 1180 |
| 0x14f0 | `LicenseValidator_Handler` | 1156 |
| 0x11bc | `LicenseValidateAndEnable_Handler` | 820 |
| 0x3d34 | `QCBOREncode_EncodeBuffer` | 820 |
| 0x280c | `CElfFile_invoke` | 732 |
| 0x6b8 | `TZ_AppCmdHandler` | 680 |
| 0xbd0 | `GetInstalledFeatures_Prepare` | 656 |
| 0x2be8 | `TA_InvokeCommandEntryPoint` | 648 |
| 0x1974 | `LicenseValidatorTest_Handler` | 632 |
| 0xe60 | `GetInstalledFeatures_FromRPMB` | 628 |


---

## 5. ГРАФ ВЫЗОВОВ - ГОРЯЧИЕ ФУНКЦИИ

| Ранг | Адрес | Имя | Xref | Роль |
|------|-------|-----|------|------|
| 1 | 0x20 | `qsee_log` | 134 | логирование повсюду |
| 2 | 0x630 | `jump_x8` | 26 | непрямой диспетчер |
| 3 | 0x279c | `j_qsee_err_fatal` | 26 | ветвь фатальной ошибки |
| 4 | 0x40 | `qsee_free` | 21 | освобождение памяти |
| 5 | 0x0 | `__module_entry_thunk` | 18 | точка входа |
| 6 | 0x30 | `qsee_malloc` | 13 | аллокация |
| 7 | 0x640 | `jump_x9` | 10 | непрямой диспетчер |
| 8 | 0x3d34 | `QCBOREncode_EncodeBuffer` | 10 | CBOR-сериализация |
| 9 | 0x409c | `memset` | 10 | инициализация буферов |
| 10 | 0x42d8 | `strlen` | 8 | работа со строками |


---

## 6. АРХИТЕКТУРНЫЕ ВЫВОДЫ ПО ДИАПАЗОНУ 1-100

1. **Ядро TA сосредоточено в 0x190-0x1974**: три тяжёлых обработчика (`FeatureEnabler_Handler`, `LicenseValidator_Handler`, `LicenseValidateAndEnable_Handler`) реализуют бизнес-логику включения фич и проверки лицензий. Это самая насыщенная часть образа.

2. **DisplayCore (0x1bec-0x251c)** - самостоятельный модуль управления SW-fuse дисплея, с полным API (open/close, enable/read/configure, получение имён и списков).

3. **Объектная модель GPParams + ObjectList (0x2e70-0x3aa8)** - полноценная инфраструктура управления сессиями GP TEE: конструкторы, деструкторы, пулы, связный список объектов.

4. **QCBOR-сериализация** начинается в первой сотне (`QCBOREncode_EncodeBuffer`) и продолжается во второй половине - то есть код TA активно формирует CBOR-ответы клиенту.

5. **Интенсивность логирования**: `qsee_log` вызывается 134 раза - TA подробно логирует своё поведение.

6. **Память**: pair `qsee_malloc`/`qsee_free` (13/21 xref) - ручное управление кучей TEE, критичное для security-контекста.

7. **Libc слинкован статически** (`memset`, `strlen`, `strchr`, `memchr`, `strlcpy`, `scalbn`) - стандартно для TEE-образов без внешних зависимостей.

---

## 7. ИТОГ ПО ЧАСТИ 1

Диапазон **0x0 - 0x4370** (100 функций) - это **полный каркас доверенного приложения**:
- 24 импорт-стаба (интерфейс с QSEE),
- 7 jump-трамплинов,
- 6 точек входа TEE/GP,
- ~10 бизнес-обработчиков Feature Enabler и License Validator (ядро логики),
- 13 функций DisplayCore,
- 18 функций GPParams (сессии),
- 6 функций ObjectList,
- 10 функций CommonUtils,
- QCBOR-кодирование,
- 6 libc-функций.

Дальнейшие 108 функций (№101-208, диапазон `0x437C - 0x113B8`) содержат остальную часть QCBOR (`QCBOREncode_*`, `QCBORDecode_*`, `QCborUtils_*`), полный **RPMB-стек** (`RPMBUtils_*`), **HWIO-утилиты** (`HWIOUtils_*`), `UsefulBuf`-семейство и **24 импорт-табличных заглушки** `__imp_*` в сегменте `extern` (`0x112E0-0x113B8`).

---

**Примечание:** доклад составлен на основе восстановленных имён функций, размеров, таблицы xref и категоризации по префиксам в текущей IDB `featenabler.img.i64` (после удаления 86 ложных `sub_*`-thunk'ов). Изменения не сохранены на диск - при необходимости сохраните базу в IDA (`Ctrl+S`).

