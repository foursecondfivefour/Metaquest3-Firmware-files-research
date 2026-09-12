# ДОКЛАД ОБ АНАЛИЗЕ БИНАРНИКА `featenabler.img`

## Часть 5: Полный аннотированный реестр 208 функций (сводный индекс)

---

## 1. НАЗНАЧЕНИЕ ЧАСТИ 5

Части 1-4 разобрали код, данные и метаданные TA. Настоящая часть — **сводный индекс**: единая таблица всех 208 функций с адресом, именем, размером, числом xref и категорией. Служит навигатором по частям 1-4.

**Сводные метрики:**

| Метрика | Значение |
|---------|----------|
| Всего функций | **208** |
| Суммарный объём кода | **36080 байт** (~35.2 КБ) |
| Самая крупная | `qsee_vsnprintf` @ 0x44c0 — 2508 б |
| Функций с 0 xref | 2 (`__funcs_on_exit_wrapper`, `TA_InvokeCommandEntryPoint_CGP`) |

---

## 2. РАСПРЕДЕЛЕНИЕ ПО РАЗМЕРАМ

| Диапазон | Кол-во |
|----------|--------|
| 1-16 байт (thunk/заглушки) | 80 |
| 17-100 байт (утилиты) | 37 |
| 101-300 байт (средние) | 51 |
| 301-600 байт (крупные) | 26 |
| 601+ байт (тяжёлые) | 14 |

---

## 3. ПОЛНЫЙ РЕЕСТР ФУНКЦИЙ (№1-208)

| № | Адрес | Имя | Размер | Xref | Категория |
|---|-------|-----|--------|------|-----------|
| 1 | `0x0` | `__module_entry_thunk` | 20 | 18 | other |
| 2 | `0x20` | `qsee_log` | 16 | 134 | QSEE/import |
| 3 | `0x30` | `qsee_malloc` | 16 | 13 | QSEE/import |
| 4 | `0x40` | `qsee_free` | 16 | 21 | QSEE/import |
| 5 | `0x50` | `qsee_err_fatal` | 16 | 1 | QSEE/import |
| 6 | `0x60` | `__funcs_on_exit` | 16 | 1 | other |
| 7 | `0x70` | `cmnlib_init` | 16 | 1 | other |
| 8 | `0x80` | `GPAppLib_init` | 16 | 1 | other |
| 9 | `0x90` | `GPAppLib_appInit` | 16 | 1 | other |
| 10 | `0xa0` | `GPAppLib_appShutdown` | 16 | 1 | other |
| 11 | `0xb0` | `cmnlib_release` | 16 | 1 | other |
| 12 | `0xc0` | `GPAppLib_handleRequest` | 16 | 1 | other |
| 13 | `0xd0` | `CApp_openSession` | 16 | 1 | other |
| 14 | `0xe0` | `qsee_open` | 16 | 4 | QSEE/import |
| 15 | `0xf0` | `qsee_is_sw_fuse_blown` | 16 | 1 | QSEE/import |
| 16 | `0x100` | `qsee_get_secure_state` | 16 | 3 | QSEE/import |
| 17 | `0x110` | `qsee_stor_write_sectors` | 16 | 1 | QSEE/import |
| 18 | `0x120` | `qsee_stor_device_init` | 16 | 1 | QSEE/import |
| 19 | `0x130` | `qsee_stor_device_get_info` | 16 | 1 | QSEE/import |
| 20 | `0x140` | `qsee_stor_read_sectors` | 16 | 1 | QSEE/import |
| 21 | `0x150` | `qsee_stor_open_partition` | 16 | 2 | QSEE/import |
| 22 | `0x160` | `qsee_stor_add_partition` | 16 | 1 | QSEE/import |
| 23 | `0x170` | `qsee_stor_client_get_info` | 16 | 1 | QSEE/import |
| 24 | `0x180` | `qsee_stor_remove_client` | 16 | 1 | QSEE/import |
| 25 | `0x190` | `FeatureEnabler_Handler` | 1180 | 3 | FE-core |
| 26 | `0x630` | `jump_x8` | 8 | 26 | jump-trampoline |
| 27 | `0x640` | `jump_x9` | 8 | 10 | jump-trampoline |
| 28 | `0x650` | `jump_x10` | 8 | 3 | jump-trampoline |
| 29 | `0x660` | `jump_x11` | 8 | 1 | jump-trampoline |
| 30 | `0x670` | `jump_x20` | 8 | 1 | jump-trampoline |
| 31 | `0x680` | `jump_x21` | 8 | 1 | jump-trampoline |
| 32 | `0x690` | `jump_x23` | 8 | 1 | jump-trampoline |
| 33 | `0x6a0` | `TA_LogAppInit` | 16 | 1 | TA-entry |
| 34 | `0x6b8` | `TZ_AppCmdHandler` | 680 | 3 | TA-entry |
| 35 | `0x968` | `TA_LogAppShutdown` | 16 | 1 | TA-entry |
| 36 | `0x980` | `GetEnabledFeatures_Handler` | 592 | 2 | FE-core |
| 37 | `0xbd0` | `GetInstalledFeatures_Prepare` | 656 | 3 | FE-core |
| 38 | `0xe60` | `GetInstalledFeatures_FromRPMB` | 628 | 3 | FE-core |
| 39 | `0x10d4` | `GetInstalledFeatures_Handler` | 232 | 3 | FE-core |
| 40 | `0x11bc` | `LicenseValidateAndEnable_Handler` | 820 | 4 | FE-core |
| 41 | `0x14f0` | `LicenseValidator_Handler` | 1156 | 4 | FE-core |
| 42 | `0x1974` | `LicenseValidatorTest_Handler` | 632 | 3 | FE-core |
| 43 | `0x1bec` | `DisplayCore_GetFeatureString` | 100 | 2 | DisplayCore |
| 44 | `0x1c58` | `DisplayCore_EnableSwFuse` | 168 | 2 | DisplayCore |
| 45 | `0x1d08` | `DisplayCore_ReadSwFuseStatus` | 132 | 1 | DisplayCore |
| 46 | `0x1d8c` | `DisplayCore_IsFeatureSupported` | 388 | 4 | DisplayCore |
| 47 | `0x1f18` | `DisplayCore_GetFeatureList` | 464 | 3 | DisplayCore |
| 48 | `0x20e8` | `DisplayCore_EncodeMetaData` | 8 | 3 | DisplayCore |
| 49 | `0x20f8` | `DisplayCore_Close` | 124 | 3 | DisplayCore |
| 50 | `0x217c` | `DisplayCore_ConfigureSwFuse` | 368 | 3 | DisplayCore |
| 51 | `0x22ec` | `DisplayCore_GetSwFuseStatus` | 332 | 4 | DisplayCore |
| 52 | `0x2438` | `DisplayCore_GetFeatureName` | 228 | 4 | DisplayCore |
| 53 | `0x2524` | `DisplayCore_Open` | 224 | 2 | DisplayCore |
| 54 | `0x260c` | `IPCore_OpenModule` | 104 | 7 | IPCore |
| 55 | `0x267c` | `IPCore_CloseModule` | 72 | 7 | IPCore |
| 56 | `0x26cc` | `scalbn` | 148 | 3 | libc/math |
| 57 | `0x278c` | `GPParams_ReturnUnsupported` | 8 | 1 | GPParams |
| 58 | `0x279c` | `j_qsee_err_fatal` | 4 | 26 | other |
| 59 | `0x27a0` | `__funcs_on_exit_wrapper` | 40 | 0 | other |
| 60 | `0x27d0` | `GetOptionalFeatureFlag` | 52 | 1 | FE-core |
| 61 | `0x280c` | `CElfFile_invoke` | 732 | 1 | IPCore |
| 62 | `0x2ae8` | `TA_OpenCloseSessionEntryPoint` | 256 | 2 | TA-entry |
| 63 | `0x2be8` | `TA_InvokeCommandEntryPoint` | 648 | 1 | TA-entry |
| 64 | `0x2e70` | `GPParams_InitListNode` | 16 | 1 | GPParams |
| 65 | `0x2e80` | `TA_InvokeCommandEntryPoint_CGP` | 316 | 0 | TA-entry |
| 66 | `0x2fc4` | `GPParams_SessionListClearThunk` | 12 | 1 | GPParams |
| 67 | `0x2fd8` | `GPParams_SessionListClearThunk2` | 12 | 2 | GPParams |
| 68 | `0x2fec` | `ObjectList_lookup_code` | 44 | 2 | ObjectList |
| 69 | `0x3018` | `ObjectList_status_to_code` | 32 | 2 | ObjectList |
| 70 | `0x3040` | `ObjectList_get` | 64 | 3 | ObjectList |
| 71 | `0x3088` | `ObjectList_find` | 92 | 2 | ObjectList |
| 72 | `0x30ec` | `ObjectList_append` | 308 | 1 | ObjectList |
| 73 | `0x3228` | `ObjectList_clear` | 132 | 2 | ObjectList |
| 74 | `0x32b4` | `GPParams_SessionListReset` | 8 | 2 | GPParams |
| 75 | `0x32c4` | `nullsub_2` | 4 | 1 | other |
| 76 | `0x32d0` | `nullsub_3` | 4 | 1 | other |
| 77 | `0x32dc` | `GPParams_ReturnOk` | 8 | 1 | GPParams |
| 78 | `0x32ec` | `GPParams_ReturnErr` | 8 | 1 | GPParams |
| 79 | `0x32fc` | `GPParams_ReturnErr2` | 8 | 1 | GPParams |
| 80 | `0x330c` | `GPParams_allocSession` | 584 | 1 | GPParams |
| 81 | `0x3554` | `GPParams_freeSession` | 512 | 2 | GPParams |
| 82 | `0x3754` | `GPParams_EntryInit` | 56 | 2 | GPParams |
| 83 | `0x3794` | `GPParams_newFromMinkInternal` | 340 | 2 | GPParams |
| 84 | `0x38f0` | `GPParams_EntryDtor` | 124 | 3 | GPParams |
| 85 | `0x3974` | `GPParams_SessionCtor` | 172 | 2 | GPParams |
| 86 | `0x3a28` | `GPParams_SessionDtor` | 120 | 2 | GPParams |
| 87 | `0x3aa8` | `GPParams_Validate` | 64 | 1 | GPParams |
| 88 | `0x3af0` | `CommonUtils_Base64Encode` | 100 | 3 | CommonUtils |
| 89 | `0x3b54` | `CommonUtils_BytesToHex` | 148 | 2 | CommonUtils |
| 90 | `0x3be8` | `CommonUtils_HexToBytes` | 200 | 1 | CommonUtils |
| 91 | `0x3cb0` | `CommonUtils_LoadPair` | 20 | 2 | CommonUtils |
| 92 | `0x3cc4` | `strlcpy` | 112 | 2 | libc/math |
| 93 | `0x3d34` | `QCBOREncode_EncodeBuffer` | 820 | 10 | QCBOR |
| 94 | `0x4068` | `memchr` | 52 | 2 | libc/math |
| 95 | `0x409c` | `memset` | 200 | 10 | libc/math |
| 96 | `0x4164` | `CommonUtils_StrDup` | 92 | 3 | CommonUtils |
| 97 | `0x41c0` | `CommonUtils_StrToU64` | 280 | 5 | CommonUtils |
| 98 | `0x42d8` | `strlen` | 100 | 8 | libc/math |
| 99 | `0x433c` | `strchr` | 52 | 2 | libc/math |
| 100 | `0x4370` | `__stack_chk_guard_ref` | 12 | 1 | other |
| 101 | `0x437c` | `qsee_snprintf_core` | 324 | 1 | QSEE/import |
| 102 | `0x44c0` | `qsee_vsnprintf` | 2508 | 2 | QSEE/import |
| 103 | `0x4e8c` | `QCborUtils_GetHeadByte` | 32 | 7 | QCborUtils |
| 104 | `0x4eac` | `QCborUtils_DecodeTypeHint` | 96 | 2 | QCborUtils |
| 105 | `0x4f0c` | `QCborUtils_IntToText` | 432 | 2 | QCborUtils |
| 106 | `0x50bc` | `QCBOREncode_AddBytes_Internal` | 136 | 3 | QCBOR |
| 107 | `0x5144` | `QCBORDecode_TailCall` | 12 | 2 | QCBOR |
| 108 | `0x5150` | `QCBORDecode_GetNextInternal` | 248 | 1 | QCBOR |
| 109 | `0x5248` | `QCBORDecode_RestoreRegs` | 16 | 2 | QCBOR |
| 110 | `0x5258` | `strnlen` | 184 | 1 | other |
| 111 | `0x5310` | `QCBORDecode_GetTypeHint` | 72 | 1 | QCBOR |
| 112 | `0x5358` | `GPParams_ReturnTrue` | 8 | 1 | GPParams |
| 113 | `0x5360` | `nullsub_1` | 4 | 1 | other |
| 114 | `0x5364` | `QCBOREncode_Init` | 108 | 7 | QCBOR |
| 115 | `0x53d8` | `QCBOREncode_AddBytes_3` | 12 | 10 | QCBOR |
| 116 | `0x53ec` | `QCBOREncode_AddTag` | 204 | 4 | QCBOR |
| 117 | `0x54c0` | `QCBOREncode_AddText_3` | 12 | 1 | QCBOR |
| 118 | `0x54d4` | `QCBOREncode_AddRaw` | 28 | 1 | QCBOR |
| 119 | `0x54f8` | `QCBOREncode_OpenArray_3` | 196 | 3 | QCBOR |
| 120 | `0x55c4` | `QCBOREncode_OpenMap_3` | 32 | 7 | QCBOR |
| 121 | `0x55ec` | `QCBOREncode_CloseArray` | 148 | 9 | QCBOR |
| 122 | `0x5688` | `QCBOREncode_AddUInt64_Internal` | 468 | 6 | QCBOR |
| 123 | `0x585c` | `QCBOREncode_AddUInt64_3` | 12 | 2 | QCBOR |
| 124 | `0x5870` | `QCBOREncode_AddInt64_Internal` | 124 | 3 | QCBOR |
| 125 | `0x58f4` | `QCBOREncode_AddInt64_3` | 12 | 14 | QCBOR |
| 126 | `0x5908` | `QCBOREncode_AddRawSimple_3` | 144 | 1 | QCBOR |
| 127 | `0x59a0` | `QCBOREncode_AddSimple_3` | 44 | 1 | QCBOR |
| 128 | `0x59d4` | `QCBOREncode_AddFloat_3` | 12 | 1 | QCBOR |
| 129 | `0x59e8` | `QCBOREncode_AddDouble_3` | 12 | 1 | QCBOR |
| 130 | `0x59fc` | `QCBOREncode_Finish2` | 128 | 2 | QCBOR |
| 131 | `0x5a7c` | `QCBOREncode_Finish` | 108 | 7 | QCBOR |
| 132 | `0x5ae8` | `QCBOREncode_AddText_Internal` | 192 | 5 | QCBOR |
| 133 | `0x5bb0` | `CommonUtils_MemSCopy` | 120 | 12 | CommonUtils |
| 134 | `0x5c30` | `CommonUtils_GetSocHwVersion` | 252 | 6 | CommonUtils |
| 135 | `0x5d2c` | `CommonUtils_IsUEFIMode` | 108 | 6 | CommonUtils |
| 136 | `0x5d98` | `CommonUtils_IsSecureBoot` | 116 | 2 | CommonUtils |
| 137 | `0x5e0c` | `CommonUtils_CheckAndGetLicenseInfo` | 476 | 3 | CommonUtils |
| 138 | `0x5fe8` | `CommonUtils_SeekPayload` | 284 | 5 | CommonUtils |
| 139 | `0x6104` | `CommonUtils_UpdatePayload` | 324 | 4 | CommonUtils |
| 140 | `0x6248` | `CommonUtils_RemovePayload` | 60 | 4 | CommonUtils |
| 141 | `0x628c` | `CommonUtils_GetPayload` | 104 | 1 | CommonUtils |
| 142 | `0x62fc` | `CommonUtils_IPfmOpen` | 360 | 4 | CommonUtils |
| 143 | `0x6464` | `CommonUtils_IPfmClose` | 28 | 5 | CommonUtils |
| 144 | `0x6488` | `CommonUtils_CheckFeatureIds` | 548 | 4 | CommonUtils |
| 145 | `0x66ac` | `CommonUtils_GetLicenseInfo` | 1328 | 5 | CommonUtils |
| 146 | `0x6bdc` | `HWIOUtils_Open` | 476 | 3 | HWIO |
| 147 | `0x6db8` | `HWIOUtils_Close` | 188 | 4 | HWIO |
| 148 | `0x6e7c` | `HWIOUtils_Read` | 136 | 3 | HWIO |
| 149 | `0x6f0c` | `HWIOUtils_Write` | 112 | 2 | HWIO |
| 150 | `0x6f84` | `QCborUtils_DecodeBytesFromMap` | 300 | 8 | QCborUtils |
| 151 | `0x70b0` | `QCborUtils_DecodeInt64FromMap` | 288 | 11 | QCborUtils |
| 152 | `0x71d0` | `RPMBUtils_BufferCacheSeekStart` | 48 | 5 | RPMB |
| 153 | `0x7208` | `RPMBUtils_BufferCacheSeekNextPayload` | 116 | 3 | RPMB |
| 154 | `0x727c` | `RPMBUtils_BufferCacheGetCurrPayload` | 124 | 7 | RPMB |
| 155 | `0x72f8` | `RPMBUtils_BufferCacheRemoveCurrPayload` | 308 | 2 | RPMB |
| 156 | `0x742c` | `RPMBUtils_BufferCacheAddPayload` | 396 | 2 | RPMB |
| 157 | `0x75b8` | `RPMBUtils_BufferCacheRecreate` | 436 | 2 | RPMB |
| 158 | `0x776c` | `RPMBUtils_BufferCacheReplaceCurrPayload` | 504 | 3 | RPMB |
| 159 | `0x7964` | `RPMBUtils_BufferCacheGetNextPayload` | 104 | 3 | RPMB |
| 160 | `0x79d4` | `RPMBUtils_BufferCacheFlush` | 200 | 4 | RPMB |
| 161 | `0x7a9c` | `RPMBUtils_DeviceCreate` | 696 | 5 | RPMB |
| 162 | `0x7d54` | `RPMBUtils_InitPartition` | 460 | 2 | RPMB |
| 163 | `0x7f20` | `RPMBUtils_DeinitPartition` | 100 | 3 | RPMB |
| 164 | `0x7f8c` | `RPMBUtils_DeviceDestroy` | 176 | 7 | RPMB |
| 165 | `0x803c` | `RPMBUtils_BufferCacheDestroy` | 96 | 2 | RPMB |
| 166 | `0x80a4` | `RPMBUtils_LogClientInfo` | 308 | 2 | RPMB |
| 167 | `0x81e0` | `memcmp` | 48 | 11 | libc/math |
| 168 | `0x8210` | `UsefulBuf_Copy` | 80 | 1 | UsefulBuf |
| 169 | `0x8268` | `UsefulBuf_Compare` | 52 | 2 | UsefulBuf |
| 170 | `0x82a4` | `UsefulBuf_Set` | 16 | 1 | UsefulBuf |
| 171 | `0x82bc` | `UsefulBuf_FindBytes` | 120 | 1 | UsefulBuf |
| 172 | `0x833c` | `UsefulOutBuf_Init` | 44 | 2 | UsefulBuf |
| 173 | `0x8370` | `UsefulOutBuf_InsertUsefulBuf` | 196 | 7 | UsefulBuf |
| 174 | `0x8434` | `UsefulOutBuf_OutUBuf` | 52 | 2 | UsefulBuf |
| 175 | `0x8470` | `UsefulOutBuf_CopyOut` | 124 | 1 | UsefulBuf |
| 176 | `0x84ec` | `UsefulInputBuf_GetBytes` | 104 | 7 | UsefulBuf |
| 177 | `0x855c` | `QCBORDecode_Init` | 88 | 4 | QCBOR |
| 178 | `0x85bc` | `ldexp` | 4 | 1 | libc/math |
| 179 | `0x85c8` | `double_to_i64` | 116 | 2 | other |
| 180 | `0x8644` | `QCBORDecode_GetNext` | 436 | 7 | QCBOR |
| 181 | `0x87f8` | `QCborUtils_DecodeValue` | 1316 | 3 | QCborUtils |
| 182 | `0x8d1c` | `QCBORDecode_Finish` | 52 | 5 | QCBOR |
| 183 | `0x8d58` | `QCborUtils_DecodeLength` | 64 | 3 | QCborUtils |
| 184 | `0x8da0` | `QCborUtils_SkipBytes` | 44 | 1 | QCborUtils |
| 185 | `0x8dd4` | `UsefulBuf_Compare_Internal` | 312 | 1 | UsefulBuf |
| 186 | `0x112e0` | `__imp_qsee_stor_open_partition` | 8 | 3 | QSEE/import |
| 187 | `0x112e8` | `__imp_qsee_stor_write_sectors` | 8 | 3 | QSEE/import |
| 188 | `0x112f0` | `__imp_qsee_get_secure_state` | 8 | 3 | QSEE/import |
| 189 | `0x112f8` | `__imp_qsee_stor_add_partition` | 8 | 3 | QSEE/import |
| 190 | `0x11300` | `__imp_qsee_stor_remove_client` | 8 | 3 | QSEE/import |
| 191 | `0x11308` | `__imp_qsee_malloc` | 8 | 3 | QSEE/import |
| 192 | `0x11310` | `__imp_qsee_open` | 8 | 3 | QSEE/import |
| 193 | `0x11318` | `__imp_qsee_stor_client_get_info` | 8 | 3 | QSEE/import |
| 194 | `0x11320` | `__imp_qsee_stor_device_init` | 8 | 3 | QSEE/import |
| 195 | `0x11328` | `__imp_GPAppLib_handleRequest` | 8 | 3 | QSEE/import |
| 196 | `0x11330` | `__imp_CApp_openSession` | 8 | 3 | QSEE/import |
| 197 | `0x11338` | `__imp_qsee_log` | 8 | 3 | QSEE/import |
| 198 | `0x11340` | `__imp_GPAppLib_init` | 8 | 3 | QSEE/import |
| 199 | `0x11348` | `__imp_cmnlib_release` | 8 | 3 | QSEE/import |
| 200 | `0x11350` | `__imp_cmnlib_init` | 8 | 3 | QSEE/import |
| 201 | `0x11358` | `__imp_GPAppLib_appShutdown` | 8 | 3 | QSEE/import |
| 202 | `0x11360` | `__imp_qsee_free` | 8 | 3 | QSEE/import |
| 203 | `0x11368` | `__imp_qsee_stor_device_get_info` | 8 | 3 | QSEE/import |
| 204 | `0x11370` | `__imp_qsee_is_sw_fuse_blown` | 8 | 3 | QSEE/import |
| 205 | `0x11378` | `__imp_qsee_stor_read_sectors` | 8 | 3 | QSEE/import |
| 206 | `0x11380` | `__imp_GPAppLib_appInit` | 8 | 3 | QSEE/import |
| 207 | `0x11388` | `__imp_qsee_err_fatal` | 8 | 3 | QSEE/import |
| 208 | `0x11398` | `__imp___funcs_on_exit` | 8 | 5 | QSEE/import |

---

## 4. ТОП-15 ФУНКЦИЙ ПО XREF

| Ранг | Адрес | Имя | Xref | Размер |
|------|-------|-----|------|--------|
| 1 | `0x20` | `qsee_log` | 134 | 16 |
| 2 | `0x630` | `jump_x8` | 26 | 8 |
| 3 | `0x279c` | `j_qsee_err_fatal` | 26 | 4 |
| 4 | `0x40` | `qsee_free` | 21 | 16 |
| 5 | `0x0` | `__module_entry_thunk` | 18 | 20 |
| 6 | `0x58f4` | `QCBOREncode_AddInt64_3` | 14 | 12 |
| 7 | `0x30` | `qsee_malloc` | 13 | 16 |
| 8 | `0x5bb0` | `CommonUtils_MemSCopy` | 12 | 120 |
| 9 | `0x70b0` | `QCborUtils_DecodeInt64FromMap` | 11 | 288 |
| 10 | `0x81e0` | `memcmp` | 11 | 48 |
| 11 | `0x640` | `jump_x9` | 10 | 8 |
| 12 | `0x3d34` | `QCBOREncode_EncodeBuffer` | 10 | 820 |
| 13 | `0x409c` | `memset` | 10 | 200 |
| 14 | `0x53d8` | `QCBOREncode_AddBytes_3` | 10 | 12 |
| 15 | `0x55ec` | `QCBOREncode_CloseArray` | 9 | 148 |

---

## 5. ТОП-10 САМЫХ КРУПНЫХ ФУНКЦИЙ

| Ранг | Адрес | Имя | Размер | Xref |
|------|-------|-----|--------|------|
| 1 | `0x44c0` | `qsee_vsnprintf` | 2508 | 2 |
| 2 | `0x66ac` | `CommonUtils_GetLicenseInfo` | 1328 | 5 |
| 3 | `0x87f8` | `QCborUtils_DecodeValue` | 1316 | 3 |
| 4 | `0x190` | `FeatureEnabler_Handler` | 1180 | 3 |
| 5 | `0x14f0` | `LicenseValidator_Handler` | 1156 | 4 |
| 6 | `0x11bc` | `LicenseValidateAndEnable_Handler` | 820 | 4 |
| 7 | `0x3d34` | `QCBOREncode_EncodeBuffer` | 820 | 10 |
| 8 | `0x280c` | `CElfFile_invoke` | 732 | 1 |
| 9 | `0x7a9c` | `RPMBUtils_DeviceCreate` | 696 | 5 |
| 10 | `0x6b8` | `TZ_AppCmdHandler` | 680 | 3 |

---

## 6. ИТОГ

Реестр подтверждает: бинарник `featenabler.img` содержит ровно **208 функций** суммарным объёмом **36080 байт (~35.2 КБ)**. Части 1-4 покрывают их полностью; часть 5 даёт единую точку навигации.
