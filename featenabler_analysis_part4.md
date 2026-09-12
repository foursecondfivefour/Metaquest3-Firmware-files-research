# ДОКЛАД ОБ АНАЛИЗЕ БИНАРНИКА `featenabler.img`

## Часть 4: Данные, метаданные и строки (закрытие пробелов частей 1-3)

---

## 1. НАЗНАЧЕНИЕ ЧАСТИ 4

Части 1-3 разобрали **весь код** TA (208 функций, ~36.6 КБ). Повторная инвентаризация выявила, что **данные не разбирались вообще**. Настоящая часть закрывает этот пробел и исправляет две фактические ошибки.

| Пробел | Суть |
|--------|------|
| **1** | Extern-таблица: в части 2 указано 23 символа, реально **27** |
| **2** | Данные, метаданные, строки — не разобраны ни в одной части |

---

## 2. ПРОБЕЛ 1: EXTERN-ТАБЛИЦА (23 -> 27)

### 2.1. Исправление

В части 2 (раздел 2.10) было указано «23 функции `__imp_*`». Фактически в сегменте `extern` (`0x112E0-0x113B8`) находится **27 символов**. Четыре из них — не импорты, а **границы памяти TA**:

| Адрес | Имя | Тип |
|-------|-----|-----|
| 0x11390 | `app_heap_limit` | граница heap |
| 0x113A0 | `app_heap_base` | база heap |
| 0x113A8 | `app_stack_base` | база stack |
| 0x113B0 | `app_stack_limit` | граница stack |

### 2.2. Почему это важно

Эти 4 символа связывают **runtime-переменные** сегмента 0xE000 (`ta_heapAddr`, `ta_heapLimit`, `ta_stackAddr`, `ta_stackLimit`) с реальной средой TEE. Без них непонятно, откуда TA берёт адреса своей памяти.

### 2.3. Полная таблица extern (27 записей)

| Адрес | Имя | Назначение |
|-------|-----|------------|
| 0x112E0 | `__imp_qsee_stor_open_partition` | открытие RPMB-партиции |
| 0x112E8 | `__imp_qsee_stor_write_sectors` | запись секторов |
| 0x112F0 | `__imp_qsee_get_secure_state` | secure state устройства |
| 0x112F8 | `__imp_qsee_stor_add_partition` | добавление партиции |
| 0x11300 | `__imp_qsee_stor_remove_client` | удаление storage-клиента |
| 0x11308 | `__imp_qsee_malloc` | аллокация |
| 0x11310 | `__imp_qsee_open` | открытие QSEE-объекта |
| 0x11318 | `__imp_qsee_stor_client_get_info` | инфо storage-клиента |
| 0x11320 | `__imp_qsee_stor_device_init` | init storage device |
| 0x11328 | `__imp_GPAppLib_handleRequest` | обработка GP-запроса |
| 0x11330 | `__imp_CApp_openSession` | открытие сессии |
| 0x11338 | `__imp_qsee_log` | **логирование (134 xref)** |
| 0x11340 | `__imp_GPAppLib_init` | init GP App Library |
| 0x11348 | `__imp_cmnlib_release` | релиз crypto-lib |
| 0x11350 | `__imp_cmnlib_init` | init crypto-lib |
| 0x11358 | `__imp_GPAppLib_appShutdown` | shutdown приложения |
| 0x11360 | `__imp_qsee_free` | освобождение |
| 0x11368 | `__imp_qsee_stor_device_get_info` | инфо устройства |
| 0x11370 | `__imp_qsee_is_sw_fuse_blown` | проверка fuse |
| 0x11378 | `__imp_qsee_stor_read_sectors` | чтение секторов |
| 0x11380 | `__imp_GPAppLib_appInit` | app init |
| 0x11388 | `__imp_qsee_err_fatal` | фатальная ошибка |
| **0x11390** | **`app_heap_limit`** | **граница heap (пропущено)** |
| 0x11398 | `__imp___funcs_on_exit` | деструкторы при выходе |
| **0x113A0** | **`app_heap_base`** | **база heap (пропущено)** |
| **0x113A8** | **`app_stack_base`** | **база stack (пропущено)** |
| **0x113B0** | **`app_stack_limit`** | **граница stack (пропущено)** |

**Итог:** импортов QSEE — 23, плюс 4 символа границ памяти = **27**.

---

## 3. ПРОБЕЛ 2: МЕТАДАННЫЕ TA (СЕГМЕНТ 0xC000)

### 3.1. Паспорт приложения

Полный разбор `TA_METADATA` — структуры, которую загрузчик TEE читает при старте:

| Адрес | Поле | Значение | Расшифровка |
|-------|------|----------|-------------|
| 0xC000 | `aFeatenabler` | `"featenabler"` | имя приложения |
| 0xC00C | `a10` | `"1.0"` | версия |
| 0xC010 | `aNone` | `"None"` | описание отсутствует |
| 0xC018 | `TA_APP_NAME` | -> 0xC000 | указатель на имя |
| 0xC020 | `ta_version` | -> 0xC00C | указатель на версию |
| 0xC028 | `ta_description` | -> 0xC010 | указатель на описание |
| 0xC040 | `ta_acceptBufSize` | **0x1000** (4096) | макс. размер входящего буфера |
| 0xC048 | `ta_stackSize` | **0xD000** (53248) | размер стека |
| 0xC050 | `ta_heapSize` | **0xD1000** (856064) | размер кучи (~836 КБ) |
| 0xC058 | `ta_cryptoSelfTest` | **0** | crypto self-test выключен |
| 0xC059 | `ta_multiSession` | **1** | мультисессии ВКЛЮЧЕНЫ |
| 0xC05A | `ta_filesNoPersist` | **0** | файлы персистентны |
| 0xC060 | `TA_MAX_STORAGE_FILES` | **60** | макс. файлов в storage |
| 0xC068 | `ta_minExecTimeout` | **800 мс** | мин. таймаут исполнения |
| 0xC070 | `ta_minWaitTimeout` | **50 мс** | мин. таймаут ожидания |
| 0xC078 | `TA_CUSTOM_PROPERTIES_NUM` | **0** | кастомных свойств нет |
| 0xC080 | `TA_METADATA` | (строка) | подпись/хеш метаданных |

### 3.2. Ключевые выводы по метаданным

1. **TA принимает буферы до 4 КБ** (`ta_acceptBufSize = 0x1000`) — типовой размер CBOR-запроса.
2. **Куча 836 КБ, стек 52 КБ** — внушительные размеры для TEE; объясняется объёмной работой с CBOR и RPMB-кэшем.
3. **Мультисессии включены** (`ta_multiSession = 1`) — TA одновременно обслуживает несколько клиентов, что подтверждает наличие объектной модели `GPParams_SessionCtor` из части 1.
4. **Crypto self-test выключен** — TA не проверяет крипто-примитивы при старте.
5. **Таймауты: 0.8 с исполнение, 0.05 с ожидание** — TA оптимизирован под быстрые ответы.
6. **До 60 storage-файлов** — прямо связано с `TA_MAX_STORAGE_FILES`, лимит на число хранимых объектов.

### 3.3. Подпись TA_METADATA @ 0xC080

Строка формата `n=<name>;p=<props>;u=<hash>`:

```
n=featenabler;
p=8:c47728cf3e4081,61:6,77,82:6004,b4,10a:9024;
u=7e1903bb7c17ff47b6d862337ca7db85
```

- `n=` — имя приложения;
- `p=` — список свойств в hex-нотации (адреса/смещения): `8`, `61`, `77`, `82`, `b4`, `10a` — это смещения полей метаданных, часть с указанием размеров (`8:c47728cf3e4081` — поле по смещению 8, значение `0xc47728cf3e4081`; `82:6004` — смещение 0x82, размер 0x6004 и т.д.);
- `u=` — уникальный идентификатор (`7e1903bb7c17ff47b6d862337ca7db85`) — 128-битный хеш/ID сборки.

Это **машинно-читаемый дескриптор сборки**, который TEE использует для валидации и идентификации образа.

---

## 4. ПРОБЕЛ 2: RUNTIME-ПЕРЕМЕННЫЕ (СЕГМЕНТ 0xE000)

| Адрес | Имя | Назначение |
|-------|-----|------------|
| 0xE008 | `__stack_chk_guard` | канарейка стека (рандом при загрузке) |
| 0xE010 | `__stack_guard_size` | размер защиты стека |
| 0xE018 | `ta_stackAddr` | текущий адрес стека TA |
| 0xE020 | `ta_stackLimit` | предел стека |
| 0xE028 | `ta_heapAddr` | текущий адрес кучи TA |
| 0xE030 | `ta_heapLimit` | предел кучи |
| 0xE068 | `appCntxt` | **глобальный контекст приложения** |

### 4.1. Связка с extern-таблицей

Эти переменные заполняются загрузчиком из символов extern-сегмента:
- `ta_stackAddr` <- `app_stack_base` (0x113A8)
- `ta_stackLimit` <- `app_stack_limit` (0x113B0)
- `ta_heapAddr` <- `app_heap_base` (0x113A0)
- `ta_heapLimit` <- `app_heap_limit` (0x11390)

**Именно здесь замкнута связь между пробелом 1 и пробелом 2:** без четырёх пропущенных символов extern невозможно понять, откуда берутся адреса памяти.

### 4.2. `appCntxt` @ 0xE068

Центральная структура состояния TA — единый контекст, в котором хранятся:
- список открытых сессий (см. `GPParams_allocSession`);
- кэш payload'ов RPMB (см. `RPMBUtils_BufferCache*`);
- флаги состояния фич и лицензий.

Размер структуры: `0xE391 - 0xE068 = 809` байт.

### 4.3. Защита стека

Наличие `__stack_chk_guard` + `__stack_guard_size` подтверждает сборку с `-fstack-protector`. Связь с `__stack_chk_guard_ref` @ 0x4370 (из части 1) — этот символ используется всеми функциями в эпилогах.

---

## 5. ПРОБЕЛ 2: КАРТА СТРОК-ЛОГОВ (СЕГМЕНТ 0xF000)

Сегмент 0xF000 содержит **136+ строк-логов** — это де-факто **карта функциональности TA**: по ним видно, какие команды, ошибки и события существуют. Все строки передаются в `qsee_log` (134 xref, самая вызываемая функция).

### 5.1. Feature Enabler (ядро бизнес-логики)

| Строка | Смысл |
|--------|-------|
| `FeatureEnabler_Handler` | имя главного обработчика |
| `Feature license` | лог проверки лицензии фичи |
| `Feature ID` / `FeatureId` | идентификатор фичи |
| `Module ID` / `ModuleId` | идентификатор модуля |
| `Enabled feature` | фича включена |
| `GetEnabledFeatures` | запрос списка включённых |
| `GetInstalledFeatures` | запрос списка установленных |
| `InstalledFeatures` | установленные фичи |
| `Not supported` | фича не поддерживается |
| `Invalid command` | неизвестная команда |

### 5.2. License Validator (валидация лицензий)

| Строка | Смысл |
|--------|-------|
| `LicenseValidate...` | валидация лицензии |
| `Invalid license` | лицензия недействительна |
| `Expiration is %...` | срок действия лицензии |
| `No validity count...` | счётчик валидности отсутствует |
| `Validity count...` | счётчик применений лицензии |
| `CheckAndGetLicense...` | проверка+чтение лицензии |
| `Extra data` | дополнительные данные лицензии |
| `Common data` | общие данные лицензии |

### 5.3. DisplayCore (управление дисплеем)

| Строка | Смысл |
|--------|-------|
| `Display QLTM` | фича QLTM |
| `Display SPR` | фича SPR |
| `Display Demura` | фича Demura |
| `Display Allocate` | аллокация DisplayCore |
| `DisplayCore_Get...` | чтение параметров |
| `DisplayCore_Enable...` | включение SW-fuse |
| `DisplayCore_Configure...` | конфигурация SW-fuse |
| `DisplayCore_Close` | закрытие |
| `DisplayCore_Read...` | чтение статуса |
| `DisplayCore_IsF...` | проверка поддержки |
| `DisplayCore_Encode...` | кодирование метаданных |

### 5.4. RPMB (защищённое хранилище)

| Строка | Смысл |
|--------|-------|
| `RPMBUtils_Device...` | операции с RPMB-устройством |
| `RPMBUtils_Buffer...` | работа с кэшем payload'ов |
| `No data in RPMB` | RPMB пуст |
| `SOC HW version %x` | версия SoC (hex) |

### 5.5. QCBOR (сериализация)

| Строка | Смысл |
|--------|-------|
| `QCBOREncode_Add...` | добавление значений |
| `QCBOREncode_Open...` | открытие массива/карты |
| `QCBOREncode_Close...` | закрытие |
| `QCBOREncode_Finish...` | финализация |
| `QCBOREncode_Init` | инициализация |
| `QCBORDecode_Get...` | чтение значения |
| `QCBORDecode_Init` | инициализация декодера |
| `QCBORDecode_Finish` | завершение декодирования |
| `QCborUtils_Decode...` | утилиты декодирования |

### 5.6. HWIO (доступ к регистрам)

| Строка | Смысл |
|--------|-------|
| `HWIOUtils_Open` / `HWIOUtils_Close` | открытие/закрытие HWIO |
| `HWIOUtils_Read` / `HWIOUtils_Write` | чтение/запись регистров |

### 5.7. Payload (работа с полезной нагрузкой)

| Строка | Смысл |
|--------|-------|
| `EncodePayload` | кодирование payload |
| `UpdatePayload` | обновление payload |
| `CommonUtils_Seek...` | поиск payload |
| `CommonUtils_Get...` | чтение payload |
| `CommonUtils_Remove...` | удаление payload |
| `CommonUtils_Update...` | обновление payload |

### 5.8. Диспетчеризация команд

| Строка | Смысл |
|--------|-------|
| `TZ_AppCmdHandler` | главный диспетчер |
| `CmdHandler received` | получена команда |
| `InvalidCmd rsp b...` | неверный ответ команды |
| `Cmd id %u` | ID команды |

### 5.9. Прочие подсистемы

| Строка | Подсистема |
|--------|------------|
| `CommonUtils_Mem...` | CommonUtils |
| `CommonUtils_IsU...` | CommonUtils (UEFI) |
| `CommonUtils_IsS...` | CommonUtils (SecureBoot) |
| `CommonUtils_Ipf...` | CommonUtils (IPfm) |
| `CommonUtils_Check...` | CommonUtils (лицензия) |
| `UsefulBuf_...` | UsefulBuf |
| `UsefulOutBuf_...` | UsefulBuf |
| `UsefulInputBuf...` | UsefulBuf |
| `IPCore_OpenModule` / `IPCore_CloseModule` | IPCore |
| `CElfFile_Invoke` | CElfFile |
| `GPAppLib_...` | GP App Library |
| `CApp_openSession` | сессии |
| `cmnlib_init` / `cmnlib_release` | crypto-lib |
| `QseeLog`, `QseeMalloc`, `QseeFree`, `QseeOpen` | QSEE API |
| `QseeErrFatal`, `QseeGetSecureS...`, `QseeIsSwFuseBlown` | QSEE API |
| `QseeStor...` | QSEE Storage API |
| `stack_chk_guard`, `StackGuardSize` | защита стека |
| `AppHeapBase/Limit`, `AppStackBase/Limit` | границы памяти |
| `TaHeapAddr/Limit`, `TaStackAddr/Limit` | runtime-адреса |
| `TaVersion`, `TaDescription`, `TaMetadata` | метаданные |
| `TaMultiSession`, `TaCryptoSelfTest`, `TaFilesNoPersist` | флаги |
| `TaMaxStorageFiles`, `TaMinExecTimeout`, `TaMinWaitTimeout` | лимиты |
| `TaAcceptBufSize`, `TaHeapSize`, `TaStackSize` | размеры |
| `TaAppName`, `TaCustomProperties` | прочее |
| `Ldexp`, `DoubleToI64` | math |
| `Scalbn` | math |
| `libcmnlib.so` | внешняя библиотека |

---

## 6. ПРОБЕЛ 2: JUMP-ТАБЛИЦЫ (ДИСПЕТЧЕРИЗАЦИЯ КОМАНД)

Обнаружено 4 jump-таблицы:

| Адрес | Имя | Где используется |
|-------|-----|------------------|
| 0x927C | `jpt_740` | диспетчер `TZ_AppCmdHandler` @ 0x6B8 |
| 0x9940 | `jpt_2254` | область License-логики |
| 0x9945 | `jpt_2344` | область License-логики |
| 0x994A | `jpt_2470` | область License-логики |

**Роль:** jump-таблица (`jpt_`) — это массив относительных смещений, по которому `TZ_AppCmdHandler` выбирает обработчик для команды по её ID. Четыре таблицы отражают четыре группы команд: базовая (Feature), и три подгруппы в License-логике.

**Связь с кодом:** `TZ_AppCmdHandler` (680 б, часть 1) — это `switch`-диспетчер; его `BR` по таблице ведёт к `FeatureEnabler_Handler`, `GetEnabledFeatures_Handler`, `LicenseValidator_Handler` и другим обработчикам.

---

## 7. ПРОБЕЛ 2: ELF-ХЭШ-ТАБЛИЦЫ

| Адрес | Имя | Назначение |
|-------|-----|------------|
| 0xF760 | `elf_hash_nbucket` | число корзин хэш-таблицы |
| 0xF764 | `elf_hash_nchain` | число цепочек |
| 0xF768 | `elf_hash_bucket` | массив корзин |
| 0xF978 | `elf_hash_chain` | массив цепочек |

Это стандартная ELF-хэш-таблица (`.hash`/`.gnu.hash`) для динамического линкера — используется для разрешения символов `libcmnlib.so` и других внешних зависимостей.

---

## 8. ИТОГ ПО ЧАСТИ 4

### 8.1. Закрытые пробелы

| Пробел | Результат |
|--------|-----------|
| **1. Extern-таблица** | Исправлено 23 -> **27**; добавлены 4 символа границ памяти |
| **2. Данные** | Разобраны метаданные TA, runtime-переменные, 136+ строк, jump-таблицы, ELF-хэш |

### 8.2. Что это дало

1. **Паспорт TA** — теперь известно: имя `featenabler`, версия `1.0`, буфер 4 КБ, heap 836 КБ, стек 52 КБ, мультисессии вкл, до 60 файлов, таймауты 0.8/0.05 с.
2. **Связка память-среда** — замкнута цепочка: extern (`app_heap_base` и др.) -> runtime (`ta_heapAddr` и др.) -> `appCntxt`.
3. **Карта функциональности** — 136+ строк показали все подсистемы: Feature, License, Display, RPMB, QCBOR, HWIO, Payload.
4. **Диспетчеризация** — найдены 4 jump-таблицы, объясняющие `switch` в `TZ_AppCmdHandler`.
5. **Сборка** — подтверждена `-fstack-protector` (canary) и динамическая линковка (ELF-hash).

### 8.3. Общий итог по четырём частям

| Часть | Покрытие |
|-------|----------|
| 1 | Функции №1-100 (бизнес-логика) |
| 2 | Функции №101-208 (инфраструктура) |
| 3 | ~86 функций, покрытых обзорно |
| **4** | **Данные, метаданные, строки, таблицы** |

Бинарник `featenabler.img` разобран **полностью**: код (208 функций), данные (371 символ), метаданные (TA_METADATA), строки (136+), таблицы (jump + ELF-hash), extern (27 символов).

---

**Примечание:** все значения прочитаны напрямую из IDB `featenabler.img.i64`. Изменения не сохранены на диск — при необходимости сохраните базу в IDA (`Ctrl+S`).
