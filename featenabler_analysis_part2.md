# ДОКЛАД ОБ АНАЛИЗЕ БИНАРНИКА `featenabler.img`

## Часть 2: Функции №101-208 (диапазон 0x437C - 0x113B8)

---

## 1. ГРАНИЦЫ И ОБЪЁМ РАССМАТРИВАЕМОГО ДИАПАЗОНА

Вторая часть охватывает функции №101-208 - адреса **`0x437C` - `0x113B8`**. Это ровно половина всего кода TA.

| Метрика | Значение |
|---------|----------|
| Кол-во функций | **108** |
| Диапазон адресов | `0x437C` - `0x113B8` |
| Суммарный объём кода | **19 200 байт** (~53% кода TA) |
| Мин. размер | 4 байта |
| Макс. размер | **2508 байт** (`qsee_vsnprintf`) |
| Средний размер | 177 байт |
| Медиана | 98 байт |
| Функций с 0 xref | **0** (все достижимы) |

**Структурная граница:** функции 101-185 занимают код вплоть до `0x8DF0`; функции 186-208 - это таблица импорта `__imp_*` в сегменте `extern` (`0x112E0-0x11398`).

---

## 2. КАТЕГОРИЗАЦИЯ ФУНКЦИЙ №101-208

### 2.1. QCBOREncode - кодирование CBOR (20 функций, 2132 байта)

Это «писатель» CBOR-ответов, дополняющий `QCBOREncode_EncodeBuffer` из части 1.

| Адрес | Имя | Размер |
|-------|-----|--------|
| 0x50bc | `QCBOREncode_AddBytes_Internal` | 136 |
| 0x5364 | `QCBOREncode_Init` | 108 |
| 0x53d8 | `QCBOREncode_AddBytes_3` | 12 |
| 0x53ec | `QCBOREncode_AddTag` | 204 |
| 0x54c0 | `QCBOREncode_AddText_3` | 12 |
| 0x54d4 | `QCBOREncode_AddRaw` | 28 |
| 0x54f8 | `QCBOREncode_OpenArray_3` | 196 |
| 0x55c4 | `QCBOREncode_OpenMap_3` | 32 |
| 0x55ec | `QCBOREncode_CloseArray` | 148 |
| 0x5688 | `QCBOREncode_AddUInt64_Internal` | 468 |
| 0x585c | `QCBOREncode_AddUInt64_3` | 12 |
| 0x5870 | `QCBOREncode_AddInt64_Internal` | 124 |
| 0x58f4 | `QCBOREncode_AddInt64_3` | 12 |
| 0x5908 | `QCBOREncode_AddRawSimple_3` | 144 |
| 0x59a0 | `QCBOREncode_AddSimple_3` | 44 |
| 0x59d4 | `QCBOREncode_AddFloat_3` | 12 |
| 0x59e8 | `QCBOREncode_AddDouble_3` | 12 |
| 0x59fc | `QCBOREncode_Finish2` | 128 |
| 0x5a7c | `QCBOREncode_Finish` | 108 |
| 0x5ae8 | `QCBOREncode_AddText_Internal` | 192 |

**Особенность:** суффикс `_3` у 10 функций - это ABI-варианты (thunk-обёртки по 12 байт), которые передают 3-й аргумент; `_Internal` - рабочие реализации. Полный набор энкодеров: строки, байты, целые (int64/uint64), float/double, массивы, карты, теги.

### 2.2. QCBORDecode - декодирование CBOR (7 функций, 924 байта)

| Адрес | Имя | Размер |
|-------|-----|--------|
| 0x5144 | `QCBORDecode_TailCall` | 12 |
| 0x5150 | `QCBORDecode_GetNextInternal` | 248 |
| 0x5248 | `QCBORDecode_RestoreRegs` | 16 |
| 0x5310 | `QCBORDecode_GetTypeHint` | 72 |
| 0x855c | `QCBORDecode_Init` | 88 |
| 0x8644 | `QCBORDecode_GetNext` | 436 |
| 0x8d1c | `QCBORDecode_Finish` | 52 |

«Читатель» CBOR - TA не только формирует ответы, но и **парсит входящие CBOR-запросы** (например, лицензии или списки фич).

### 2.3. QCborUtils - утилиты CBOR (8 функций, 2572 байта)

| Адрес | Имя | Размер |
|-------|-----|--------|
| 0x4e8c | `QCborUtils_GetHeadByte` | 32 |
| 0x4eac | `QCborUtils_DecodeTypeHint` | 96 |
| 0x4f0c | `QCborUtils_IntToText` | 432 |
| 0x6f84 | `QCborUtils_DecodeBytesFromMap` | 300 |
| 0x70b0 | `QCborUtils_DecodeInt64FromMap` | 288 |
| 0x87f8 | `QCborUtils_DecodeValue` | **1316** |
| 0x8d58 | `QCborUtils_DecodeLength` | 64 |
| 0x8da0 | `QCborUtils_SkipBytes` | 44 |

Самый мощный слой CBOR: `DecodeValue` (1316 б) - центральный рекурсивный декодер; `DecodeInt64FromMap` (11 xref) и `DecodeBytesFromMap` - извлечение типизированных значений из CBOR-карт (по ключам) - типичный паттерн разбора запросов.

### 2.4. RPMBUtils - стек работы с RPMB-хранилищем (15 функций, 4072 байта)

Самый крупный модуль второй части - полноценный драйвер RPMB (Replay Protected Memory Block).

**Кэш-буфер (BufferCache) - 9 функций:**

| Адрес | Имя | Размер |
|-------|-----|--------|
| 0x71d0 | `RPMBUtils_BufferCacheSeekStart` | 48 |
| 0x7208 | `RPMBUtils_BufferCacheSeekNextPayload` | 116 |
| 0x727c | `RPMBUtils_BufferCacheGetCurrPayload` | 124 (7 xref) |
| 0x72f8 | `RPMBUtils_BufferCacheRemoveCurrPayload` | 308 |
| 0x742c | `RPMBUtils_BufferCacheAddPayload` | 396 |
| 0x75b8 | `RPMBUtils_BufferCacheRecreate` | 436 |
| 0x776c | `RPMBUtils_BufferCacheReplaceCurrPayload` | 504 |
| 0x7964 | `RPMBUtils_BufferCacheGetNextPayload` | 104 |
| 0x79d4 | `RPMBUtils_BufferCacheFlush` | 200 |

**Устройство и партиции - 6 функций:**

| Адрес | Имя | Размер |
|-------|-----|--------|
| 0x7a9c | `RPMBUtils_DeviceCreate` | 696 |
| 0x7d54 | `RPMBUtils_InitPartition` | 460 |
| 0x7f20 | `RPMBUtils_DeinitPartition` | 100 |
| 0x7f8c | `RPMBUtils_DeviceDestroy` | 176 (7 xref) |
| 0x803c | `RPMBUtils_BufferCacheDestroy` | 96 |
| 0x80a4 | `RPMBUtils_LogClientInfo` | 308 |

**Вывод:** RPMB-модуль - это «жёсткий диск» TA. Через него хранятся установленные фичи и лицензии. Реализован кэш payload'ов с операциями seek/get/add/remove/replace/flush/recreate.


### 2.5. HWIOUtils - работа с HWIO (4 функции, 912 байт)

| Адрес | Имя | Размер |
|-------|-----|--------|
| 0x6bdc | `HWIOUtils_Open` | 476 |
| 0x6db8 | `HWIOUtils_Close` | 188 |
| 0x6e7c | `HWIOUtils_Read` | 136 |
| 0x6f0c | `HWIOUtils_Write` | 112 |

HWIO (Hardware I/O) - интерфейс доступа к регистрам устройств через QSEE. Используется для чтения/записи fuse-регистров и secure-состояния.

### 2.6. CommonUtils - утилиты общего назначения (13 функций, 4108 байт)

Второй по объёму кода модуль после RPMB.

| Адрес | Имя | Размер | Назначение |
|-------|-----|--------|------------|
| 0x5bb0 | `CommonUtils_MemSCopy` | 120 (12 xref) | безопасное копирование памяти |
| 0x5c30 | `CommonUtils_GetSocHwVersion` | 252 (6 xref) | версия SoC |
| 0x5d2c | `CommonUtils_IsUEFIMode` | 108 (6 xref) | проверка UEFI-режима |
| 0x5d98 | `CommonUtils_IsSecureBoot` | 116 | проверка secure boot |
| 0x5e0c | `CommonUtils_CheckAndGetLicenseInfo` | 476 | проверка+чтение лицензии |
| 0x5fe8 | `CommonUtils_SeekPayload` | 284 | поиск payload |
| 0x6104 | `CommonUtils_UpdatePayload` | 324 | обновление payload |
| 0x6248 | `CommonUtils_RemovePayload` | 60 | удаление payload |
| 0x628c | `CommonUtils_GetPayload` | 104 | чтение payload |
| 0x62fc | `CommonUtils_IPfmOpen` | 360 | открытие IPfm |
| 0x6464 | `CommonUtils_IPfmClose` | 28 | закрытие IPfm |
| 0x6488 | `CommonUtils_CheckFeatureIds` | 548 | проверка ID фич |
| 0x66ac | `CommonUtils_GetLicenseInfo` | **1328** | ядро чтения лицензий |

Ключевые функции здесь - `GetLicenseInfo` (1328 б), `CheckFeatureIds` (548 б) и группа работы с payload (Seek/Update/Remove/Get). Это сердце логики лицензирования.

### 2.7. UsefulBuf - библиотека безопасных буферов (10 функций, 1100 байт)

| Адрес | Имя | Размер |
|-------|-----|--------|
| 0x8210 | `UsefulBuf_Copy` | 80 |
| 0x8268 | `UsefulBuf_Compare` | 52 |
| 0x82a4 | `UsefulBuf_Set` | 16 |
| 0x82bc | `UsefulBuf_FindBytes` | 120 |
| 0x833c | `UsefulOutBuf_Init` | 44 |
| 0x8370 | `UsefulOutBuf_InsertUsefulBuf` | 196 (7 xref) |
| 0x8434 | `UsefulOutBuf_OutUBuf` | 52 |
| 0x8470 | `UsefulOutBuf_CopyOut` | 124 |
| 0x84ec | `UsefulInputBuf_GetBytes` | 104 (7 xref) |
| 0x8dd4 | `UsefulBuf_Compare_Internal` | 312 |

Это базовый слой QCBOR (QCBOR использует UsefulBuf как абстракцию входных/выходных буферов). `UsefulOutBuf` - растущий выходной буфер, `UsefulInputBuf` - читающий входной.


### 2.8. qsee_lib - системные функции QSEE (2 функции, 2832 байта)

| Адрес | Имя | Размер |
|-------|-----|--------|
| 0x437c | `qsee_snprintf_core` | 324 |
| 0x44c0 | `qsee_vsnprintf` | **2508** |

`qsee_vsnprintf` - **самая крупная функция всего бинарника** (2508 байт). Это реализация форматирования строк `printf`-семейства внутри TEE-образа (собственный libc для логов). `qsee_snprintf_core` - ядро одиночного форматирования.

### 2.9. Libc-функции (4 функции, 352 байта)

| Адрес | Имя | Размер |
|-------|-----|--------|
| 0x5258 | `strnlen` | 184 |
| 0x81e0 | `memcmp` | 48 (11 xref) |
| 0x85bc | `ldexp` | 4 |
| 0x85c8 | `double_to_i64` | 116 |

`memcmp` с 11 xref - активно используется в сравнениях (проверка лицензий, хешей, ID фич). `double_to_i64` - вспомогательная для CBOR float-кодирования.

### 2.10. Импорт-таблица `__imp_*` (23 функции, 184 байта)

Функции №186-208 - это элементы **`extern`-сегмента** (`0x112E0-0x11398`), по 8 байт каждая. Не код, а **таблица импортируемых символов** (GOT-подобные стабы на внешние API QSEE):

`__imp_qsee_stor_open_partition`, `__imp_qsee_stor_write_sectors`, `__imp_qsee_get_secure_state`, `__imp_qsee_stor_add_partition`, `__imp_qsee_stor_remove_client`, `__imp_qsee_malloc`, `__imp_qsee_open`, `__imp_qsee_stor_client_get_info`, `__imp_qsee_stor_device_init`, `__imp_GPAppLib_handleRequest`, `__imp_CApp_openSession`, `__imp_qsee_log`, `__imp_GPAppLib_init`, `__imp_cmnlib_release`, `__imp_cmnlib_init`, `__imp_GPAppLib_appShutdown`, `__imp_qsee_free`, `__imp_qsee_stor_device_get_info`, `__imp_qsee_is_sw_fuse_blown`, `__imp_qsee_stor_read_sectors`, `__imp_GPAppLib_appInit`, `__imp_qsee_err_fatal`, `__imp___funcs_on_exit`.

Каждая такая «функция» - 8-байтовый указатель в таблице импорта. Реальный код-обёртки для них - в сегменте кода (qsee_api-заглушки из части 1).

### 2.11. Прочее

| Адрес | Имя | Размер | Примечание |
|-------|-----|--------|------------|
| 0x5358 | `GPParams_ReturnTrue` | 8 | возврат true |
| 0x5360 | `nullsub_1` | 4 | пустышка |

---

## 3. СТАТИСТИКА ПО РАЗМЕРАМ

| Диапазон | Кол-во |
|----------|--------|
| 1-16 байт (thunk/заглушки) | **35** |
| 17-100 байт (утилиты) | 20 |
| 101-300 байт (средние) | **33** |
| 301-600 байт (крупные) | 16 |
| 600+ байт (тяжёлые) | **4** |

**Топ-10 самых крупных функций части 2:**
| Адрес | Имя | Размер |
|-------|-----|--------|
| 0x44c0 | `qsee_vsnprintf` | **2508** |
| 0x66ac | `CommonUtils_GetLicenseInfo` | **1328** |
| 0x87f8 | `QCborUtils_DecodeValue` | **1316** |
| 0x7a9c | `RPMBUtils_DeviceCreate` | 696 |
| 0x6488 | `CommonUtils_CheckFeatureIds` | 548 |
| 0x776c | `RPMBUtils_BufferCacheReplaceCurrPayload` | 504 |
| 0x5e0c | `CommonUtils_CheckAndGetLicenseInfo` | 476 |
| 0x6bdc | `HWIOUtils_Open` | 476 |
| 0x5688 | `QCBOREncode_AddUInt64_Internal` | 468 |
| 0x7d54 | `RPMBUtils_InitPartition` | 460 |

---

## 4. ГРАФ ВЫЗОВОВ - «ГОРЯЧИЕ» ФУНКЦИИ ЧАСТИ 2

| Ранг | Адрес | Имя | Xref | Роль |
|------|-------|-----|------|------|
| 1 | 0x58f4 | `QCBOREncode_AddInt64_3` | **14** | добавление int64 в CBOR |
| 2 | 0x5bb0 | `CommonUtils_MemSCopy` | 12 | копирование памяти |
| 3 | 0x70b0 | `QCborUtils_DecodeInt64FromMap` | 11 | извлечение int64 из карты |
| 4 | 0x81e0 | `memcmp` | 11 | сравнение буферов |
| 5 | 0x53d8 | `QCBOREncode_AddBytes_3` | 10 | добавление байтов |
| 6 | 0x55ec | `QCBOREncode_CloseArray` | 9 | закрытие массива |
| 7 | 0x6f84 | `QCborUtils_DecodeBytesFromMap` | 8 | извлечение байтов из карты |
| 8 | 0x4e8c | `QCborUtils_GetHeadByte` | 7 | чтение головного байта CBOR |
| 9 | 0x5364 | `QCBOREncode_Init` | 7 | инициализация энкодера |
| 10 | 0x55c4 | `QCBOREncode_OpenMap_3` | 7 | открытие карты |

Здесь доминируют CBOR-функции: TA постоянно кодирует/декодирует CBOR при обмене с клиентом.

---

## 5. АРХИТЕКТУРНЫЕ ВЫВОДЫ ПО ЧАСТИ 2

1. **Вторая половина - это «инфраструктурный слой» TA.** Если часть 1 (0x0-0x4370) - это логика приложения (обработчики фич, лицензий, сессий), то часть 2 (0x437C-0x113B8) - библиотеки и драйверы: QCBOR (полный кодер/декодер), RPMB-драйвер, HWIO-доступ, UsefulBuf, собственный libc.

2. **QCBOR - ключевой протокол обмена.** Суммарно 35 функций в трёх модулях (QCBOREncode 20 + QCBORDecode 7 + QCborUtils 8) на 5628 байт - треть кода части 2. TA формирует CBOR-ответы и парсит CBOR-запросы.

3. **RPMB-стек (4072 б) - защищённое хранилище.** Полноценный драйвер с кэшем payload'ов (9 функций BufferCache) и управлением устройством/партициями (6 функций). Через него хранятся установленные фичи и лицензии.

4. **Лицензирование - самая тяжёлая логика.** `CommonUtils_GetLicenseInfo` (1328 б), `CommonUtils_CheckFeatureIds` (548 б), `CommonUtils_CheckAndGetLicenseInfo` (476 б) образуют ядро проверки прав на фичи.

5. **Собственный libc.** `qsee_vsnprintf` (2508 б - абсолютный максимум) реализует форматирование строк для логирования внутри TEE, поскольку полноценный libc в TEE-образах недоступен.

6. **Все функции достижимы** - 0 функций без xref во второй части (в отличие от части 1, где было 2 изолированных).

7. **23 импорт-стаба `__imp_*`** замыкают таблицу внешних зависимостей QSEE - все реальные системные вызовы TA проходят через них.

---

## 6. ИТОГ ПО ЧАСТИ 2

Диапазон **0x437C - 0x113B8** (108 функций, 19 200 байт) содержит:
- **QCBOR-кодирование** - 20 функций,
- **QCBOR-декодирование** - 7 функций,
- **QCborUtils** - 8 функций,
- **RPMB-стек** - 15 функций (драйвер защищённого хранилища),
- **HWIO-утилиты** - 4 функции,
- **CommonUtils** - 13 функций (включая ядро лицензирования),
- **UsefulBuf** - 10 функций,
- **qsee_lib** - 2 функции (в т.ч. `qsee_vsnprintf`, 2508 б),
- **libc** - 4 функции,
- **таблица импорта** `__imp_*` - 23 стаба,
- **прочее** - 2 функции.

---

## 7. ОБЩИЙ ИТОГ ПО ОБЕИМ ЧАСТЯМ

Бинарник `featenabler.img` - **Trusted Application Qualcomm**, 208 функций, ~36.6 КБ кода. Структурно делится на:
- **Бизнес-слой** (часть 1, 0x0-0x4370): обработчики фич/лицензий, сессии, DisplayCore, IPCore, ObjectList;
- **Инфраструктурный слой** (часть 2, 0x437C-0x113B8): QCBOR, RPMB, HWIO, UsefulBuf, libc, импорт-таблица.

TA реализует полный цикл: приём CBOR-запроса -> валидация лицензии (RPMB+HWIO) -> включение фич через fuse-регистры -> формирование CBOR-ответа.

---

**Примечание:** доклад составлен на основе данных текущей IDB `featenabler.img.i64` (208 функций после удаления 86 ложных `sub_*`-thunk'ов). Изменения не сохранены на диск - при необходимости сохраните базу в IDA (`Ctrl+S`).
