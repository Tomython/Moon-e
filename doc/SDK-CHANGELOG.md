# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [v0.3.1] - 2024-10-21

### Fixed    

-  Authenticated media failed to load due to an issue with how the access-token was retrieved.

### Security 

= See https://github.com/element-hq/element-web/security/advisories/GHSA-3jm3-x98c-r34x; we only share the access-token after verifying that the request is going to the homeserver.

## [v0.3.0] - 2024-08-20

### Added

-  You can now use the service-worker in sdk
-  Support for authenticated media endpoints


## [v0.2.0] - 2024-06-18

### Added

-   Add ability to adjust the token, see https://github.com/element-hq/hydrogen-web/pull/1153.

## [v0.1.8] - 2023-11-08

### Changed

-   Pass `sendReadReceipt: false` to RoomViewModel options to disable sending read receipts, see https://github.com/vector-im/hydrogen-web/pull/1150

### Fixed

-   Switch over to olm from npm registry, fixes https://github.com/vector-im/hydrogen-web/issues/1146

## [v0.1.7] - 2023-10-08

### Added

-   Export many view classes, see https://github.com/vector-im/hydrogen-web/pull/1124.

### Fixed

-   Fixed an issue where some UI components were missing in AccountSetupView.

## [v0.1.6] - 2023-08-22

### Changed

-   Pass `isReadonly: true` in `Client.startWithAuthData` to disable uploading OTKs.

## [v0.1.5] - 2023-08-10

### Added

-   Export classes MemberList, MemberListView, MemberListViewModel and avatar functions.

### Fixed

-   Fixed an issue where `FilteredMap` was not emitting when setting a new filter.

## [v0.1.4] - 2023-07-20

### Added

-   Export more classes for the SDK

### Fixed

-   Fixed `RoomViewModel.load` not awaiting promise

## [v0.1.3] - 2023-05-11

### Added

-   Added `Copy matrix.to permalink` option to message action

### Fixed

-   Fix an issue where keys for encrypted messages sent from Hydrogen was not shared leading to UTDs.
-   Fix documentation which failed to mention that `FeatureSet` needs to be passed into view models.
-   Mention how to add `libolm` as dependency in the tutorial.
-   Fix an issue where large log files were generated for long lived calls.
-   Fix an issue which prevented the SDK from being used without encryption.

### Changed

-   Long dates in sticky date headers are now rendered in a single line


Перевод

# Журнал изменений

Все заметные изменения в этом проекте будут задокументированы в этом файле.

Формат основан на [Вести журнал изменений] (https://keepachangelog.com/en/1.1.0/),
а этот проект придерживается [семантического управления версиями](https://semver.org/spec/v2.0.0.html).

## [версия 0.3.1] - 2024-10-21

### Исправлено    

- Не удалось загрузить аутентифицированный носитель из-за проблемы с получением токена доступа.

### Безопасность 

= Видеть https://github.com/element-hq/element-web/security/advisories/GHSA-3jm3-x98c-r34x ; мы предоставляем токен доступа только после проверки того, что запрос направляется на домашний сервер.

## [версия 0.3.0] - 2024-08-20

### Добавлено

- Теперь вы можете использовать service-worker в sdk
- Поддержка аутентифицированных конечных точек мультимедиа


## [версия 0.2.0] - 2024-06-18

### Добавлено

- Добавлена возможность корректировать токен, см. https://github.com/element-hq/hydrogen-web/pull/1153.

## [версия 0.1.8] - 2023-11-08

### Изменено

- Передайте `sendReadReceipt: false` в параметры RoomViewModel, чтобы отключить отправку квитанций о прочтении, см. https://github.com/vector-im/hydrogen-web/pull/1150

### Исправлено

- Переключитесь на olm из реестра npm, исправлены https://github.com/vector-im/hydrogen-web/issues/1146

## [версия 0.1.7] - 2023-10-08

### Добавлено

- Экспорт многих классов представлений, см. https://github.com/vector-im/hydrogen-web/pull/1124.

### Исправлено

- Исправлена ошибка, из-за которой в AccountSetupView отсутствовали некоторые компоненты пользовательского интерфейса.

## [версия 0.1.6] - 2023-08-22

### Изменено

- Введите "IsReadOnly: true" в "Client.startWithAuthData", чтобы отключить загрузку OTKS.

## [версия 0.1.5] - 2023-08-10

### Добавлено

- Экспорт классов MemberList, MemberListView, MemberListViewModel и функций аватара.

### Исправлено

- Исправлена ошибка, из-за которой "Фильтрованная карта" не отображалась при установке нового фильтра.

## [версия 0.1.4] - 2023-07-20

### Добавлено

- Добавлен экспорт дополнительных классов для SDK

### Исправлено

- Исправлено, что `RoomViewModel.load` не загружался в ожидании обещанного

## [версия 0.1.3] - 2023-05-11

### Добавлено

- Добавлена "Копия" matrix.to опция постоянной ссылки на действие сообщения

### Исправлено

- Исправлена ошибка, из-за которой ключи для зашифрованных сообщений, отправленных с Hydrogen, не передавались, что приводило к UTDs.
- Исправлена документация, в которой не упоминалось, что "FeatureSet" должен быть передан в view models.
- В руководстве указано, как добавить "libolm" в качестве зависимости.
- Исправлена ошибка, из-за которой для длительных вызовов создавались большие файлы журнала.
- Исправлена ошибка, из-за которой SDK не мог использоваться без шифрования.

### Изменено

- Длинные даты в заголовках с привязкой к дате теперь отображаются в одну строку