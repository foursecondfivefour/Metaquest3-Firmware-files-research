# АУДИТ БЕЗОПАСНОСТИ БИНАРНИКА `featenabler.img`
## Static Security Assessment (IOActive / NCC Group style)

---

## 0. МЕТОДОЛОГИЯ И ОГРАНИЧЕНИЯ

| Параметр | Значение |
|----------|----------|
| Объект | `C:\\questerthree\\featenabler.img` — Qualcomm QSEE TA «Feature Enabler» |
| Объём | 208 функций, ~36 КБ, ARM64/AArch64, base 0x0 |
| Тип | Статический реверс-инжиниринг (IDA Pro + Hex-Rays) |
| Дата | 2026-09-13 |
| Класс | Security Assessment (static) — НЕ пентест, НЕ dynamic fuzzing |

ОГРАНИЧЕНИЯ:
1. Нет динамики/эмуляции TEE, нет устройства и ключей.
2. Severity — предварительная, не подтверждена эксплойтом.
3. Часть контрактов (GP TEE framework) — вне образа.
4. IDB могла меняться; findings привязаны к текущему состоянию.

---

## 1. ATTACK SURFACE

### 1.1. Точки входа GP TEE (TA<->REE boundary)
| Адрес | Функция | Роль |
|-------|---------|------|
| 0x2be8 | TA_InvokeCommandEntryPoint | главная точка входа команд |
| 0x2ae8 | TA_OpenCloseSessionEntryPoint | open/close session |
| 0x2e80 | TA_InvokeCommandEntryPoint_CGP | CGP-вариант |
| 0x6b8  | TZ_AppCmdHandler | диспетчер команд |

### 1.2. Интерфейсы недоверенных данных
- CBOR-запросы -> QCborUtils_DecodeValue (0x87f8)
- Параметры команд a3[] (от GP)
- Строки/hex -> CommonUtils_HexToBytes, CommonUtils_StrToU64
- RPMB -> RPMBUtils_*; HWIO/fuse -> HWIOUtils_*, DisplayCore_*

### 1.3. Trust boundaries
TZ_AppCmdHandler -> LicenseValidator_Handler -> CommonUtils_GetLicenseInfo (CBOR parse)
                                    -> CommonUtils_CheckFeatureIds -> IPFM (external)

---

## 2. СВОДНАЯ ТАБЛИЦА FINDINGS

| ID | Severity | CWE | Адрес | Описание | Статус |
|----|----------|-----|-------|----------|--------|
| FE-2026-01 | Medium | CWE-125 | 0x2be8 | OOB read a3[1..28] в case 2 без проверки NELEM | Требует контракта GP |
| FE-2026-02 | Low-Med | CWE-125 | 0x6104 | memcmp с фикс. длиной 2048 | Подтверждено |
| FE-2026-03 | Medium | CWE-674/400 | 0x87f8 | CBOR recursion без depth-limit (85 blocks, CC=50) | Подтверждено |
| FE-2026-04 | Info | - | 0x6b8 | Логирование cmd_id до валидации | Подтверждено |
| FE-2026-05 | Medium | CWE-690/476 | 0x160c | qsee_malloc без NULL-check в крипто-пути | Подтверждено |
| FE-2026-06 | - | - | - | ОБЪЕДИНЁН с FE-03 | Закрыт |
| FE-2026-07 | Low | design | 0x1700 | Необратимый disable лицензии при ошибке | Подтверждено |
| FE-2026-08 | Low-Med | CWE-125/787 | 0x16e8 | Фикс. 2048 для буфера лицензии в стеке | Подтверждено |
| FE-2026-09 | - | - | 0x16b8 | Цикл по feature_id | FALSE POSITIVE (см. ниже) |
| FE-2026-10 | Info | - | 0x3c70 | Anti-tamper: порча canary при overflow | ЗАЩИТНЫЙ механизм |
| FE-2026-11 | Low-Med | CWE-125 | 0x61d0 | memcmp фикс. 2048 в UpdatePayload | Подтверждено |
| FE-2026-12 | Low | CWE-770 | 0x1634 | Жёсткие лимиты 2048/4096 | Подтверждено |
| FE-2026-13 | Info | - | 0x1590 | Логирование hw_version/feature_id | Подтверждено |
| FE-2026-14 | Medium | CWE-197 | 0x675c | Усечение FID до 16 бит (v6 = (uint16)v31) | Подтверждено |
| FE-2026-15 | Low | CWE-126 | 0x689c | memcmp строк без проверки длины | Подтверждено |
| FE-2026-16 | - | - | 0x68ac | MemSCopy в 32-байт буфер | FALSE POSITIVE (safe memmove) |
| FE-2026-17 | Low | CWE-476 | 0x66dc | Не проверяются a2/a3 в GetLicenseInfo | Подтверждено |

---

## 3. ПОДРОБНОЕ ОПИСАНИЕ FINDINGS

### FE-2026-01 (Medium, CWE-125) — OOB read массива параметров
Адрес: TA_InvokeCommandEntryPoint @ 0x2be8, case 2u (0x2d08)
В case 2u проверяется a4 != 4693 и два маркерных указателя a3[1]/a3[11], после чего БЕЗУСЛОВНО читаются a3[1]..a3[28] и передаются в CApp_openSession.
Impact: OOB read из памяти TEE при NELEM(a3) < 24. Info-leak/crash.
Условие: [dependencies: GP framework] — нужно подтвердить, что a4 НЕ привязан к числу элементов.
Рекомендация: проверять NELEM(a3) >= 24 до чтения.

### FE-2026-02 / FE-2026-11 (Low-Med, CWE-125) — фиксированная длина memcmp 2048
Адреса: CommonUtils_UpdatePayload @ 0x61d0, CommonUtils_SeekPayload @ 0x5fe8
v13 = 2048; memcmp(v12, a3, v13) — сравнение ровно 2048 байт.
Impact: OOB read до 2048 байт, если фактическая длина a3 < 2048.
Рекомендация: передавать реальную длину; проверять len <= 2048.

### FE-2026-03 (Medium, CWE-674/400) — CBOR recursion без depth-limit
Адрес: QCborUtils_DecodeValue @ 0x87f8 (1316 б, 85 blocks, CC=50)
3 вложенных switch, рекурсия по array/map. Проверка *(a1+26) — флаг ошибки (uErr), НЕ глубина.
Impact: stack exhaustion при глубоко-вложенном CBOR (ta_stackSize = 53248 б). DoS.
Рекомендация: depth-limit 16-32 + лимит числа элементов.

### FE-2026-05 (Medium, CWE-690/476) — malloc без NULL-check
Адрес: LicenseValidator_Handler @ 0x160c
v22 = qsee_malloc(a1: 0);  // без проверки результата и размера
... v10 = *(_QWORD *)(v22 + 8*v9);  // разыменование
Impact: NULL-deref -> panic TA при OOM.
Рекомендация: if (v22 == nullptr) return error; проверять размер.

### FE-2026-07 (Low, design) — необратимый disable лицензии
Адрес: LicenseValidator_Handler @ 0x1700
При неудачной валидации: CommonUtils_RemovePayload(...) -> "Invalid License!! Disable feature_id"
Impact: ложно-отрицательная валидация (сбой RPMB) -> потеря лицензии навсегда.
Рекомендация: различать транзиентные ошибки и невалидные лицензии.

### FE-2026-08 (Low-Med, CWE-125/787) — фикс. 2048 в стеке
Адрес: LicenseValidator_Handler @ 0x16e8
v18 = &v34; v19 = 2048; v17 = 2048; (буфер лицензии)
Рекомендация: sizeof + проверка возвращённой длины.

### FE-2026-10 (Informational) — anti-tamper порча canary
Адрес: CommonUtils_HexToBytes @ 0x3c70
Ассемблер: CBZ X9; BL __stack_chk_guard_ref; MOV W9,#75; MOVN W0,#0; STR W9,[X8]; B ret
При a2 >= 0x80000000 функция намеренно пишет 75 в &__stack_chk_guard и возвращает -1.
Это ЗАЩИТНЫЙ механизм (fail-safe), НЕ уязвимость. Severity: Informational.

### FE-2026-12 (Low, CWE-770) — жёсткие лимиты буферов
Адрес: LicenseValidator_Handler @ 0x1634 (лицензия 2048, feature_ids 4096)
Рекомендация: документировать; при усечении возвращать ошибку.

### FE-2026-13 (Info) — логирование hw_version/feature_id
Адрес: LicenseValidator_Handler @ 0x1590, 0x1730
TEE-лог обычно недоступен из REE. Понизить уровень для release.

### FE-2026-14 (Medium, CWE-197) — усечение FID до 16 бит
Адрес: CommonUtils_GetLicenseInfo @ 0x675c
v6 = (unsigned __int16)v31;  // флаг "FID найден", усечён
if (v6 != 0) { ... }  // для feature_id >= 0x10000 v6 == 0 -> лицензия "не найдена"
Impact: отказ валидной лицензии для FID >= 0x10000.
Рекомендация: хранить v6 как int/__int64 либо отдельный bool found.

### FE-2026-15 (Low, CWE-126) — memcmp строк без длины
Адрес: CommonUtils_GetLicenseInfo @ 0x689c
memcmp(v33, "FID", 3); memcmp(v33, "LicenseeHash", 12); memcmp(v13, "Response", 8)
Длина CBOR-строки (v32) не проверяется до memcmp.
Impact: OOB read до 12 байт. Рекомендация: v32 >= strlen(key).

### FE-2026-17 (Low, CWE-476) — нет проверки a2/a3
Адрес: CommonUtils_GetLicenseInfo @ 0x66dc
Проверяется только a4; a2 (CBOR buf) и a3 (len) — без проверки перед QCBORDecode_Init.

---

## 4. FALSE POSITIVES (закрыто после дочитывания)

### FE-2026-09 — FALSE POSITIVE
Функция: CommonUtils_CheckFeatureIds @ 0x6488
Цикл: for (i = a4; i != 0; --i) { v9 = *v6++; QCBOREncode_AddInt64_3(..., v9); }
Явная граница a4. Корректный bounded-loop. Функция формирует CBOR-запрос
{blobVersion:1, FeatureIDs:[[id,bytes]...]} и вызывает IPFM_CheckFIDAndGetAllSerialNums.

### FE-2026-16 — FALSE POSITIVE
Функция: CommonUtils_MemSCopy @ 0x5bb0
result = a2 <= a4 ? a2 : a4;  // min(capacity, src_len)
Безопасная реализация memmove с обработкой перекрытия (копирование назад).
NULL-проверки есть. Переполнения v36[32] НЕ происходит (лимит v26=32).

---

## 5. RISK RATING

| Категория | Оценка | Обоснование |
|-----------|--------|-------------|
| Critical | 0 | Нет RCE/обхода подписи |
| High | 0 | - |
| Medium | 4 | FE-01, FE-03, FE-05, FE-14 |
| Low-Med | 3 | FE-02, FE-08, FE-11 |
| Low | 4 | FE-07, FE-12, FE-15, FE-17 |
| Informational | 2 | FE-10, FE-13 |
| ОБЩИЙ УРОВЕНЬ | MEDIUM | Преимущественно DoS + отказ лицензий |

---

## 6. SECURITY STRENGTHS

| Область | Наблюдение |
|---------|-----------|
| Bounds-check диспетчера | TZ_AppCmdHandler проверяет a2/a4 перед КАЖДЫМ handler |
| NULL-checks | a1/a3 в диспетчере; Bytes в CBOR; a1/a3 в MemSCopy |
| Stack canary | -fstack-protector во всех крупных функциях |
| Безопасные строки | strlcpy/qsee_snprintf_core, нет strcpy/sprintf |
| Overflow-check | HexToBytes: if (a2>>31) -> fail |
| Изоляция ключей | Крипто делегировано cmnlib/IPFM |
| Cleanup | корректные qsee_free/IPCore_CloseModule/IPfmClose |
| Anti-tamper | намеренная порча canary при аномальном входе |
| Safe memmove | MemSCopy с min() и обработкой перекрытия |

---

## 7. АРХИТЕКТУРНЫЙ ВЫВОД ПО ЛИЦЕНЗИЯМ

TA НЕ валидирует подпись лицензии сам. Доверяет:
1. RPMB (защищённое хранилище) — источник лицензии;
2. IPFM/IPCore (cmnlib) — внешний модуль (IPFM_CheckFIDAndGetAllSerialNums);
3. Response-код в CBOR — сервер уже проверил лицензию.

Следствие: компрометация RPMB или подмена ответа IPFM -> включение фич.
Это свойство threat model, не статическая уязвимость TA.

---

## 8. РЕКОМЕНДАЦИИ (приоритизированные)

| Приоритет | Рекомендация |
|-----------|-------------|
| P1 | Проверить qsee_malloc на NULL (FE-05) |
| P1 | Ввести depth-limit в CBOR-декодер (FE-03) |
| P1 | Валидировать NELEM(a3) >= 24 в case 2 (FE-01) |
| P1 | Исправить усечение FID до 16 бит (FE-14) |
| P2 | Передавать реальные длины вместо 2048 (FE-02/08/11) |
| P2 | Различать транзиентные ошибки RPMB и невалидные лицензии (FE-07) |
| P3 | Проверять длину строк перед memcmp (FE-15) |
| P3 | Понизить уровень логов с hw_version/feature_id (FE-13) |

---

## 9. ИТОГ

Проведён статический аудит безопасности TA featenabler.img.
Прочитано и проанализировано 14 ключевых функций (точки входа, диспетчер,
CBOR-декодер, лицензионный валидатор, парсеры, MemSCopy, CheckFeatureIds).

Findings: 15 (после объединения FE-03/06 и закрытия FE-09/16 как false positives).
- 0 Critical, 0 High
- 4 Medium (FE-01, FE-03, FE-05, FE-14)
- 7 Low/Low-Med
- 2 Informational
- 2 False Positive (закрыты)

Общая оценка: MEDIUM. Преимущественно DoS (recursion, NULL-deref, OOB read)
и функциональный отказ лицензий (FE-14, FE-07). Критических уязвимостей
(RCE/обход проверки лицензий) в прочитанном коде НЕ обнаружено.

ОГОВОРКА: severity предварительные; FE-01, FE-03, FE-08 требуют либо
контракта GP-фреймворка, либо полной (неусечённой) декомпиляции.
Динамический анализ (эмулятор/устройство) выходит за рамки статического реверса.

---

**Артефакты:** IDB featenabler.img.i64; отчёты part1-part5 (анализ 208 функций).
