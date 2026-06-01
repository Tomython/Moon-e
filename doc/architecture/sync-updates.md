# persistance vs model update of a room

## persist first, return update object, update model with update object
 - we went with this
## update model first, return update object, persist with update object
 - not all models exist at all times (timeline only when room is "open"),
 	so model to create timeline update object might not exist for persistence need

## persist, update, each only based on sync data (independent of each other)
 - possible inconsistency between syncing and loading from storage as they are different code paths
 + storage code remains very simple and focussed

## updating model and persisting in one go
 - if updating model needs to do anything async, it needs to postpone it or the txn will be closed

## persist first, read from storage to update model
 + guaranteed consistency between what is on screen and in storage
 - slower as we need to reread what was just synced every time (big accounts with frequent updates)


Перевод

# упорство против обновления модели комнаты

## сначала сохраняем, возвращаем объект обновления, обновляем модель с помощью объекта обновления
 - мы пошли по этому пути
## сначала обновляем модель, возвращаем объект обновления, сохраняем объект обновления
 - не все модели существуют постоянно (временная шкала только тогда, когда комната "открыта"),
поэтому модель для создания объекта обновления временной шкалы может не существовать из-за необходимости сохранения

## сохранение, обновление, каждое из которых основано только на данных синхронизации (независимо друг от друга)
 - возможная несогласованность между синхронизацией и загрузкой из хранилища, поскольку это разные пути к коду
+ код хранилища остается очень простым и сфокусированным

## обновление модели и сохранение за один раз
- если для обновления модели требуется выполнить что-либо асинхронное, это нужно отложить, иначе txn будет закрыт

## сначала сохраните, прочитав из хранилища, чтобы обновить модель
 + гарантируется согласованность между тем, что отображается на экране, и тем, что находится в хранилище
 - медленнее, так как нам нужно каждый раз перечитывать то, что только что было синхронизировано (большие аккаунты с частыми обновлениями)