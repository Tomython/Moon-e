The matrix layer consists of a `Session`, which represents a logged in user session. It's the root object you can get rooms off. It can persist and load itself from storage, at which point it's ready to be displayed. It doesn't sync it's own though, and you need to create and start a Sync object for updates to be pushed and persisted to the session. `Sync` is the thing (although not the only thing) that mutates the `Session`, with `Session` being unaware of `Sync`.

The matrix layer assumes a transaction-based storage layer, modelled much to how IndexedDB works. The idea is that any logical operation like process sync response, send a message, ... runs completely in a transaction that gets aborted if anything goes wrong. This helps the storage to always be in a consistent state. For this reason you'll often see transactions (txn) being passed in the code. Also, the idea is to not emit any events until readwrite transactions have been committed. 

 - Reduce the chance that errors (in the event handlers) abort the transaction. You *could* catch & rethrow but it can get messy.
 - Try to keep transactions as short-lived as possible, to not block other transactions.

For this reason a `Room` processes a sync response in two phases: `persistSync` & `emitSync`, with the return value of the former being passed into the latter to avoid double processing.

## Timeline, fragments & event indices.

A room in matrix is a DAG (directed, acyclic graph) of events, also known as the timeline. brawl is only aware of fragments of this graph, and can be unaware how these fragments relate to each other until a common event is found while paginating a fragment. After doing an initial sync, you start with one fragment. When looking up an event with the `/context` endpoint (for fetching a replied to message, or navigating to a given event id, e.g. through a permalink), a new, unconnected, fragment is created. Also, when receiving a limited sync response during incremental sync, a new fragment is created. Here, the relationship is clear, so they are immediately linked up at creation. Events in brawl are identified within a room by `[fragment_id, event_index]`. The `event_index` is an unique number within a fragment to sort events in chronological order in the timeline. `fragment_id` cannot be directly compared for sorting (as the relationship may be unknown), but with help of the `FragmentIndex`, one can attempt to sort events by their `FragmentIndex([fragment_id, event_index])`.

A fragment is the following data structure:
```
let fragment := {
    roomId: string
    id: number
    previousId: number?
    nextId: number?
    previousToken: string?
    nextToken: string?
}
```

## Observing the session

`Room`s on the `Session` are exposed as an `ObservableMap` collection, which is like an ordinary `Map` but emits events when it is modified (here when a room is added, removed, or the properties of a room change). `ObservableMap` can have different operators applied to it like `mapValues()`, `filterValues()` each returning a new `ObservableMap`-like object, and also `sortValues()` returning an `ObservableList` (emitting events when a room at an index is added, removed, moved or changes properties).

So for example, the room list, `Room` objects from `Session.rooms` are mapped to a `RoomTileViewModel` and then sorted. This gives us fine-grained events at the end of the collection chain that can be easily and efficiently rendered by the `ListView` component.

On that note, view components are just a simple convention, having these methods:

    - `mount()` - prepare to become part of the document and interactive, ensure `root()` returns a valid DOM node.
    - `root()` - the room DOM node for the component. Only valid to be called between `mount()` and `unmount()`.
    - `update(attributes)` (to be renamed to `setAttributes(attributes)`) - update the attributes for this component. Not all components support all attributes to be updated. For example most components expect a viewModel, but if you want a component with a different view model, you'd just create a new one.
    - `unmount()` - tear down after having been removed from the document.

The initial attributes are usually received by the constructor in the first argument. Other arguments are usually freeform, `ListView` accepting a closure to create a child component from a collection value.

Templating and one-way databinding are neccesary improvements, but not assumed by the component contract.

Updates from view models can come in two ways. View models emit a change event, that can be listened to from a view. This usually includes the name of the property that changed. This is the mechanism used to update the room name in the room header of the currently active room for example.

For view models part of an observable collection (and to be rendered by a ListView), updates can also propagate through the collection and delivered by the ListView to the view in question. This avoids every child component in a ListView having to attach a listener to it's viewModel. This is the mechanism to update the room name in a RoomTile in the room list for example.

TODO: specify how the collection based updates work. (not specified yet, we'd need a way to derive a key from a value to emit an update from within a collection, but haven't found a nice way of specifying that in an api)


Перевод

Матричный слой состоит из "сеанса", который представляет собой сеанс пользователя, вошедшего в систему. Это корневой объект, с которого вы можете снимать номера. Он может сохраняться и загружаться из хранилища, после чего он готов к отображению. Однако он не синхронизируется сам по себе, и вам нужно создать и запустить объект синхронизации, чтобы обновления отправлялись и сохранялись в сеансе. `Синхронизация" - это то (хотя и не единственное), что изменяет "сессию", причем "Сессия" не знает о "синхронизации`.

Матричный уровень предполагает уровень хранения, основанный на транзакциях, который во многом схож с тем, как работает IndexedDB. Идея заключается в том, что любая логическая операция, такая как обработка ответа синхронизации, отправка сообщения и т.д., полностью выполняется в транзакции, которая прерывается, если что-то идет не так. Это помогает хранилищу всегда находиться в согласованном состоянии. По этой причине вы часто будете видеть, что в коде передаются транзакции (txn). Кроме того, идея состоит в том, чтобы не генерировать никаких событий до тех пор, пока не будут зафиксированы транзакции чтения-записи. 

 - Уменьшите вероятность того, что ошибки (в обработчиках событий) прервут транзакцию. Вы могли бы перехватить и повторно обработать транзакцию, но это может привести к путанице.
 - Старайтесь, чтобы транзакции были как можно более короткими, чтобы не блокировать другие транзакции.

По этой причине "Комната" обрабатывает ответ синхронизации в два этапа: "persistSync" и "emitSync", причем возвращаемое значение первого передается во второй, чтобы избежать двойной обработки.

## Временная шкала, фрагменты и индексы событий.

Комната в matrix представляет собой DAG (направленный ациклический график) событий, также известный как временная шкала. brawl знает только о фрагментах этого графика и может не знать, как эти фрагменты связаны друг с другом, пока при разбиении фрагмента на страницы не будет найдено общее событие. Выполнив первоначальную синхронизацию, вы начинаете с одного фрагмента. При поиске события с помощью конечной точки `/context` (для получения ответа на сообщение или перехода к заданному идентификатору события, например, через постоянную ссылку) создается новый, несвязанный фрагмент. Кроме того, при получении ограниченного ответа синхронизации во время инкрементальной синхронизации создается новый фрагмент. Здесь взаимосвязь очевидна, поэтому они сразу же объединяются при создании. События в brawl идентифицируются в комнате по "[fragment_id, event_index]`. "event_index" - это уникальный номер во фрагменте для сортировки событий в хронологическом порядке на временной шкале. `fragment_id` нельзя напрямую сравнить для сортировки (поскольку связь может быть неизвестна), но с помощью `FragmentIndex` можно попытаться отсортировать события по их `FragmentIndex([fragment_id, event_index])`.

Фрагмент представляет собой следующую структуру данных:
```
пусть фрагмент := {
    roomId: строка
    id: номер
    previousId: номер?
    NextID: номер?
    previousToken: строка?
    nextToken: строка?
}
```

## Наблюдение за сеансом

Комнаты в "Сеансе" отображаются как коллекция "ObservableMap", которая похожа на обычную "Карту", но генерирует события при ее изменении (в данном случае, когда комната добавляется, удаляется или меняются свойства комнаты). К "ObservableMap" могут применяться разные операторы, такие как "mapValues()", "filterValues()", каждый из которых возвращает новый объект, подобный "ObservableMap", а также "sortValues()", возвращающий "ObservableList" (генерирующий события, когда добавляется, удаляется, перемещается комната с индексом или изменяет свойства).

Так, например, список комнат, объекты `Room` из `Session.rooms` сопоставляются с `RoomTileViewModel` и затем сортируются. Это дает нам детализированные события в конце цепочки сбора, которые могут быть легко и эффективно отображены компонентом "ListView".

На этом этапе компоненты представления - это просто соглашение, в котором используются следующие методы:

    - `mount()` - подготовьтесь к тому, чтобы стать частью документа и взаимодействовать с ним, убедитесь, что `root()` возвращает допустимый DOM-узел.
    - `root()` - DOM-узел помещения для компонента. Допустимо только для вызова между `mount()` и `unmount()`.
    - `обновить(атрибуты)` (будет переименовано в `Установить атрибуты(attributes)`) - обновить атрибуты для этого компонента. Не все компоненты поддерживают все атрибуты, которые необходимо обновить. Например, для большинства компонентов требуется ViewModel, но если вам нужен компонент с другой моделью представления, вам нужно просто создать новый.
    - `unmount()` - отключение после удаления из документа.

Исходные атрибуты обычно передаются конструктором в первом аргументе. Другие аргументы обычно имеют произвольную форму, `ListView` принимает замыкание для создания дочернего компонента из значения коллекции.

Шаблонизация и односторонняя привязка к базе данных являются необходимыми улучшениями, но не предусмотрены контрактом компонента.

Обновления из моделей представлений могут поступать двумя способами. Модели представлений генерируют событие изменения, которое можно прослушать из представления. Обычно это включает в себя название измененного объекта недвижимости. Это механизм, используемый, например, для обновления названия комнаты в заголовке активной в данный момент комнаты.

Для моделей представлений, входящих в наблюдаемую коллекцию (и отображаемых с помощью ListView), обновления также могут распространяться по коллекции и доставляться с помощью ListView в рассматриваемое представление. Это позволяет избежать необходимости подключать прослушиватель к каждому дочернему компоненту в ListView к его ViewModel. Это, например, механизм обновления названия комнаты в RoomTile в списке комнат.

Задача: укажите, как работают обновления на основе коллекции. (пока не указано, нам нужен способ получить ключ из значения, чтобы отправить обновление из коллекции, но мы не нашли подходящего способа указать это в api)