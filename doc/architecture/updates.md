# Updates

How updates flow from the model to the view model to the UI.

## EventEmitter, single values

When interested in updates from a single object, chances are it inherits from `EventEmitter` and it supports a `change` event.

`ViewModel` by default follows this pattern, but it can be overwritten, see Collections below.

### Parameters

Often a `parameters` or `params` argument is passed with the name of the field who's value has now changed. This parameter is currently only sometimes used, e.g. when it is too complicated or costly to check every possible field. An example of this is `TilesListView.onUpdate` to see if the `shape` property of a tile changed and hence the view needs to be recreated. Other than that, bindings in the web UI just reevaluate all bindings when receiving an update. This is a soft convention that could probably be more standardized, and it's not always clear what to pass (e.g. when multiple fields are being updated).

Another reason to keep this convention around is that if one day we decide to add support for a different platform with a different UI, it may not be feasible to reevaluate all data-bindings in the UI for a given view model when receiving an update.

## Collections

As an optimization, Hydrogen uses a pattern to let updates flow over an observable collection where this makes sense. There is an `update` event for this in both `ObservableMap` and `ObservableList`. This prevents having to listen for updates on each individual item in large collections. The `update` event uses the same `params` argument as explained above.

Some values like `BaseRoom` emit both with a `change` event on the event emitter and also over the collection. This way consumers can use what fits best for their case: the left panel can listen for updates on the room over the collection to power the room list, and the room view model can listen to the event emitter to get updates from the current room only.

### MappedMap and mapping models to `ViewModel`s

This can get a little complicated when using `MappedMap`, e.g. when mapping a model from `matrix/`
to a view model in `domain/`. Often, view models will want to emit updates _spontanously_,
e.g. without a prior update being sent from the lower-lying model. An example would be to change the value of a field after the view has called a method on the view model.
To support this pattern while having updates still flow over the collection requires some extra work;
`ViewModel` has a `emitChange` option which you can pass in to override
what `ViewModel.emitChange` does (by default it emits the `change` event on the view model).
`MappedMap` passes a callback to emit an update over the collection to the mapper function.
You can pass this callback as the `emitChange` option and updates will now flow over the collection.

`MappedMap` also accepts an updater function, which you can use to make the view model respond to updates
from the lower-lying model.

Here is an example:

```ts
const viewModels = someCollection.mapValues(
	    (model, emitChange) => new SomeViewModel(this.childOptions({
	        model,
	        // will make ViewModel.emitChange go over
	        // the collection rather than emit a "change" event
	        emitChange,
	    })),
	    // an update came in from the model, let the vm know
	    (vm: SomeViewModel) => vm.onUpdate(),
	);
```

### `ListView` & the `parentProvidesUpdates` flag.

`ObservableList` is always rendered in the UI using `ListView`. When receiving an update over the collection, it will find the child view for the given index and call `update(params)` on it. Views will typically need to be told whether they should listen to the `change` event in their view model or rather wait for their `update()` method to be called by their parent view, `ListView`. That's why the `mount(args)` method on a view supports a `parentProvidesUpdates` flag. If `true`, the view should not subscribe to its view model, but rather updates the DOM when its `update()` method is called. Also see `BaseUpdateView` and `TemplateView` for how this is implemented in the child view.

## `ObservableValue`

When some method wants to return an object that can be updated, often an `ObservableValue` is used rather than an `EventEmitter`. It's not 100% clear cut when to use the former or the latter, but `ObservableValue` is often used when the returned value in it's entirety will change rather than just a property on it.  `ObservableValue` also has some nice facilities like lazy evaluation when subscribed to and the `waitFor` method to work with promises.


Перевод

# Обновления

Как обновления передаются из модели в модель представления и в пользовательский интерфейс.

## EventEmitter, отдельные значения

Если вас интересуют обновления из одного объекта, скорее всего, он наследуется от EventEmitter и поддерживает событие `изменение`.

`ViewModel" по умолчанию соответствует этому шаблону, но его можно перезаписать, смотрите коллекции ниже.

### Параметры

Часто в качестве аргумента `parameters` или `params` передается название поля, значение которого в данный момент изменилось. В настоящее время этот параметр используется только иногда, например, когда проверка каждого возможного поля является слишком сложной или дорогостоящей. Примером этого является `TilesListView.onUpdate`, чтобы увидеть, изменилось ли свойство `shape` плитки, и, следовательно, необходимо воссоздать представление. Кроме этого, привязки в веб-интерфейсе пользователя просто переоценивают все привязки при получении обновления. Это мягкое соглашение, которое, вероятно, можно было бы более стандартизировать, и не всегда ясно, что передавать (например, при обновлении нескольких полей).

Еще одна причина сохранить это соглашение заключается в том, что если однажды мы решим добавить поддержку для другой платформы с другим пользовательским интерфейсом, может оказаться невозможным повторно оценить все привязки данных в пользовательском интерфейсе для данной модели представления при получении обновления.

## Коллекции

В качестве оптимизации Hydrogen использует шаблон, позволяющий обновлять наблюдаемую коллекцию там, где это имеет смысл. Для этого есть событие "update" как в "ObservableMap", так и в "ObservableList`. Это избавляет от необходимости отслеживать обновления для каждого отдельного элемента в больших коллекциях. Событие `update` использует тот же аргумент `params`, что и описанный выше.

Некоторые значения, такие как `BaseRoom`, генерируются как при событии `change` в источнике событий, так и в коллекции. Таким образом, пользователи могут использовать то, что лучше всего подходит для их случая: левая панель может прослушивать обновления по комнате поверх коллекции, чтобы активировать список комнат, а модель просмотра комнаты может прослушивать источник событий, чтобы получать обновления только для текущей комнаты.

### MappedMap и отображение моделей в ViewModel'ы

Это может немного усложниться при использовании MappedMap, например, при отображении модели из matrix/
к модели представления в `домене/`. Часто модели представления хотят отправлять обновления автоматически,
например, без предварительного обновления, отправляемого из нижележащей модели. Примером может служить изменение значения поля после того, как представление вызвало метод в модели представления.
Для поддержки этого шаблона при сохранении потока обновлений в коллекции требуется дополнительная работа;
В "ViewModel" есть параметр "emitChange", который вы можете передать, чтобы переопределить
то, что делает "ViewModel.emitChange" (по умолчанию он генерирует событие "change" в модели представления).
`MappedMap` передает обратный вызов для отправки обновления коллекции в функцию mapper.
Вы можете передать этот обратный вызов как параметр "emitChange", и теперь обновления будут передаваться по коллекции.

"MappedMap" также поддерживает функцию обновления, которую вы можете использовать, чтобы заставить модель представления реагировать на обновления
из нижележащей модели.

Вот пример:

``ts
const ViewModels = SomeCollection.mapValues(
	    (model, emitChange) => создать SomeViewModel(this.childOptions({
	        model,
// заставит ViewModel.emitChange перейти на
	        // коллекцию, а не генерировать событие "change"
	        Выполните изменение,
	    })),
// из модели поступило обновление, сообщите об этом виртуальной машине
	    (vm: SomeViewModel) => vm.onUpdate(),
);
```

### `ListView` и флаг `parentProvidesUpdates`.

`ObservableList` всегда отображается в пользовательском интерфейсе с помощью "ListView". При получении обновления по коллекции он найдет дочернее представление для данного индекса и вызовет для него `update(params)`. Представлениям, как правило, нужно сообщить, следует ли им прослушивать событие "change" в своей модели представления или лучше дождаться, пока их метод "update()" будет вызван их родительским представлением `ListView`. Вот почему метод `mount(args)` в представлении поддерживает флаг `parentProvidesUpdates`. Если значение `true`, представление не должно подписываться на свою модель представления, а скорее обновлять DOM при вызове метода `update()`. Также смотрите "BaseUpdateView" и "TemplateView", чтобы узнать, как это реализовано в дочернем представлении.

## `ObservableValue`

Когда какой-либо метод хочет вернуть объект, который может быть обновлен, часто используется "ObservableValue", а не "EventEmitter`. На 100% неясно, когда использовать первое или второе, но "ObservableValue" часто используется, когда изменяется возвращаемое значение целиком, а не только его свойство.  В "ObservableValue" также есть несколько полезных функций, таких как отложенная оценка при подписке и метод "waitFor" для работы с обещаниями.