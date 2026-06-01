## IView components

The [interface](https://github.com/vector-im/hydrogen-web/blob/master/src/platform/web/ui/general/types.ts) adopted by view components is agnostic of how they are rendered to the DOM. This has several benefits:
 - it allows Hydrogen to not ship a [heavy view framework](https://bundlephobia.com/package/react-dom@18.2.0) that may or may not be used by its SDK users, and also keep bundle size of the app down.
 - Given the interface is quite simple, is should be easy to integrate this interface into the render lifecycle of other frameworks.
 - The main implementations used in Hydrogen are [`ListView`](https://github.com/vector-im/hydrogen-web/blob/master/src/platform/web/ui/general/ListView.ts) (rendering [`ObservableList`](https://github.com/vector-im/hydrogen-web/blob/master/src/observable/list/BaseObservableList.ts)s) and [`TemplateView`](https://github.com/vector-im/hydrogen-web/blob/master/src/platform/web/ui/general/TemplateView.ts) (templating and one-way databinding), each only a few 100 lines of code and tailored towards their specific use-case. They work straight with the DOM API and have no other dependencies.
 - a common inteface allows us to mix and match between these different implementations (and gradually shift if need be in the future) with the code.

## Templates

### Template language

Templates use a mini-DSL language in pure javascript to express declarative templates. This is basically a very thin wrapper around `document.createElement`, `document.createTextNode`, `node.setAttribute` and `node.appendChild` to quickly create DOM trees. The general syntax is as follows:
```js
t.tag_name({attribute1: value, attribute2: value, ...}, [child_elements]);
t.tag_name(child_element);
t.tag_name([child_elements]);
```
**tag_name** can be [most HTML or SVG tags](https://github.com/vector-im/hydrogen-web/blob/master/src/platform/web/ui/general/html.ts#L102-L110).

eg:
Here is an example HTML segment followed with the code to create it in Hydrogen.
```html
<section class="main-section">
    <h1>Demo</h1>
    <button class="btn_cool">Click me</button>
</section>
```
```js
t.section({className: "main-section"},[
    t.h1("Demo"), 
    t.button({className:"btn_cool"}, "Click me")
]);
```

All these functions return DOM element nodes, e.g. the result of `document.createElement`.

### TemplateView

`TemplateView` builds on top of templating by adopting the IView component model and adding event handling attributes, sub views and one-way databinding.
In views based on `TemplateView`, you will see a render method with a `t` argument.  
`t` is `TemplateBuilder` object passed to the render function in `TemplateView`. It also takes a data object to render and bind to, often called `vm`, short for view model from the MVVM pattern Hydrogen uses.

You either subclass `TemplateView` and override the `render` method:
```js
class MyView extends TemplateView {
    render(t, vm) {
        return t.div(...);
    }
}
```

Or you pass a render function to `InlineTemplateView`:
```js
new InlineTemplateView(vm, (t, vm) => {
    return t.div(...);
});
```

**Note:** the render function is only called once to build the initial DOM tree and setup bindings, etc ... Any subsequent updates to the DOM of a component happens through bindings.

#### Event handlers

Any attribute starting with `on` and having a function as a value will be attached as an event listener on the given node. The event handler will be removed during unmounting.

```js
t.button({onClick: evt => {
    vm.doSomething(evt.target.value);
}}, "Click me");
```

#### Subviews

`t.view(instance)` will mount the sub view (can be any IView) and return its root node so it can be attached in the DOM tree.
All subviews will be unmounted when the parent view gets unmounted.

```js
t.div({className: "Container"}, t.view(new ChildView(vm.childViewModel)));
```

#### One-way data-binding

A binding couples a part of the DOM to a value on the view model. The view model emits an update when any of its properties change, to which the view can subscribe. When an update is received by the view, it will reevaluate all the bindings, and update the DOM accordingly.

A binding can appear in many places where a static value can usually be used in the template tree.
To create a binding, you pass a function that maps the view value to a static value.

##### Text binding

```js
t.p(["I've got ", vm => vm.counter, " beans"])
```

##### Attribute binding

```js
t.button({disabled: vm => vm.isBusy}, "Submit");
```

##### Class-name binding
```js
t.div({className: {
    button: true,
    active: vm => vm.isActive
}})
```
##### Subview binding

So far, all the bindings can only change node values within our tree, but don't change the structure of the DOM. A sub view binding allows you to conditionally add a subview based on the result of a binding function.

All sub view bindings return a DOM (element or comment) node and can be directly added to the DOM tree by including them in your template.

###### map

`t.mapView` allows you to choose a view based on the result of the binding function:

```js
t.mapView(vm => vm.count, count => {
    return count > 5 ? new LargeView(count) : new SmallView(count);
});
```

Every time the first or binding function returns a different value, the second function is run to create a new view to replace the previous view.

You can also return `null` or `undefined` from the second function to indicate a view should not be rendered. In this case a comment node will be used as a placeholder.

There is also a `t.map` which will create a new template view (with the same value) and you directly provide a render function for it:

```js
t.map(vm => vm.shape, (shape, t, vm) => {
    switch (shape) {
        case "rect": return t.rect();
        case "circle": return t.circle();
    }
})
```

###### if

`t.ifView` will render the subview if the binding returns a truthy value:

```js
t.ifView(vm => vm.isActive, vm => new View(vm.someValue));
```

You equally have `t.if`, which creates a `TemplateView` and passes you the `TemplateBuilder`:

```js
t.if(vm => vm.isActive, (t, vm) => t.div("active!"));
```

##### Side-effects

Sometimes you want to imperatively modify your DOM tree based on the value of a binding.
`mapSideEffect` makes this easy to do:

```js
let node = t.div();
t.mapSideEffect(vm => vm.color, (color, oldColor) => node.style.background = color);
return node;
```

**Note:** you shouldn't add any bindings, subviews or event handlers from the side-effect callback,
the safest is to not use the `t` argument at all.
If you do, they will be added every time the callback is run and only cleaned up when the view is unmounted.

#### `tag` vs `t`

If you don't need a view component with data-binding, sub views and event handler attributes, the template language also is available in `ui/general/html.js` without any of these bells and whistles, exported as `tag`. As opposed to static templates with `tag`, you always use
`TemplateView` as an instance of a class, as there is some extra state to keep track (bindings, event handlers and subviews).

Although syntactically similar, `TemplateBuilder` and `tag` are not functionally equivalent.  
Primarily `t` **supports** bindings and event handlers while `tag` **does not**. This is because to remove event listeners, we need to keep track of them, and thus we need to keep this state somewhere which
we can't do with a simple function call but we can insite the TemplateView class.

```js
    // The onClick here wont work!!
    tag.button({className:"awesome-btn", onClick: () => this.foo()});

class MyView extends TemplateView {
    render(t, vm){
        // The onClick works here.
        t.button({className:"awesome-btn", onClick: () => this.foo()});
    }
}
```

## ListView

A view component that renders and updates a list of sub views for every item in a `ObservableList`.

```js
const list = new ListView({
    list: someObservableList
}, listValue => return new ChildView(listValue))
```

As items are added, removed, moved (change position) and updated, the DOM will be kept in sync.

There is also a `LazyListView` that only renders items in and around the current viewport, with the restriction that all items in the list must be rendered with the same height.

### Sub view updates

Unless the `parentProvidesUpdates` option in the constructor is set to `false`, the ListView will call the `update` method on the child `IView` component when it receives an update event for one of the items in the `ObservableList`.

This way, not every sub view has to have an individual listener on it's view model (a value from the observable list), and all updates go from the observable list to the list view, who then notifies the correct sub view.


Перевод

## Компоненты IView

Принцип [interface](https://github.com/vector-im/hydrogen-web/blob/master/src/platform/web/ui/general/types.ts), используемый компонентами view, не зависит от того, как они отображаются в DOM. У этого есть несколько преимуществ:
 - это позволяет Hydrogen не поставлять [тяжелый фреймворк для просмотра] (https://bundlephobia.com/package/react-dom@18.2.0), который может использоваться или не использоваться пользователями SDK, а также уменьшить размер пакета приложений.
 - Учитывая, что интерфейс довольно прост, интегрировать этот интерфейс в жизненный цикл рендеринга других фреймворков должно быть несложно.
 - Основными реализациями, используемыми в Hydrogen, являются [`ListView`](https://github.com/vector-im/hydrogen-web/blob/master/src/platform/web/ui/general/ListView.ts) (рендеринг [`ObservableList`](https://github.com/vector-im/hydrogen-web/blob/master/src/observable/list/BaseObservableList.ts)s) и [`TemplateView`](https://github.com/vector-im/hydrogen-web/blob/master/src/platform/web/ui/general/TemplateView.ts) (шаблонизация и односторонняя привязка к базе данных), каждая из которых состоит всего из нескольких 100 строк кода и адаптирована к конкретному случаю их использования-case. Они работают напрямую с DOM API и не имеют никаких других зависимостей.
 - общий интерфейс позволяет нам смешивать и сопоставлять эти различные реализации (и постепенно изменять их, если потребуется в будущем) с помощью кода.

## Шаблоны

### Язык шаблонов

Шаблоны используют мини-язык DSL на чистом javascript для выражения декларативных шаблонов. По сути, это очень тонкая оболочка для `document.createElement`, `document.createTextNode`, `node.setAttribute` и `node.appendChild`, позволяющая быстро создавать DOM-деревья. Общий синтаксис следующий:
``js
t.имя_тега({атрибут1: значение, атрибут2: значение, ...}, [дочерние_элементы]);
t.имя_тега(дочерний_элемент);
t.имя_тега([дочерние_элементы]);
```
**имя_тега** может быть [большинство HTML или SVG tags](https://github.com/vector-im/hydrogen-web/blob/master/src/platform/web/ui/general/html.ts#L102-L110).

напр.:
Вот пример HTML-фрагмента, за которым следует код для его создания в Hydrogen.
``html
<класс раздела="основной раздел">
    <h1>Демонстрация</h1>
    <класс кнопки="btn_cool">Нажмите на меня</кнопка>
</раздел>
```
``js
t.section({имя класса: "основной раздел"},[
t.h1("Демонстрация"),
t.button({имя класса:"btn_cool"}, "Нажмите на меня")
]);
```

Все эти функции возвращают узлы элементов DOM, например, результат `document.createElement`.

### TemplateView

"TemplateView" строится поверх шаблонизатора, используя компонентную модель IView и добавляя атрибуты обработки событий, вложенные представления и одностороннюю привязку к базе данных.
В представлениях, основанных на `TemplateView`, вы увидите метод рендеринга с аргументом `t`.  
`t` - это объект `TemplateBuilder`, передаваемый функции рендеринга в `TemplateView`. Для визуализации и привязки к нему также требуется объект данных, часто называемый "vm", сокращение от view model из шаблона MVVM, который использует Hydrogen.

Вы либо создаете подкласс "TemplateView", либо переопределяете метод `render`:
``js
-класс MyView расширяет TemplateView {
    рендеринг(t, vm) {
        вернуть t.div(...);
    }
}
```

Или вы передаете функцию рендеринга в `InlineTemplateView`:
``js
new InlineTemplateView(vm, (t, vm) => {
    вернуть t.div(...);
});
```

**Примечание:** функция рендеринга вызывается только один раз для построения исходного дерева DOM и настройки привязок и т.д... Все последующие обновления DOM компонента выполняются с помощью привязок.

#### Обработчики событий

Любой атрибут, начинающийся с `on` и имеющий функцию в качестве значения, будет подключен в качестве прослушивателя событий на данном узле. Обработчик событий будет удален во время размонтирования.

``js
t.button({onClick: evt => {
    vm.doSomething(evt.target.value);
}}, "Нажмите на меня");
```

#### Подзаголовки

`t.view(instance)` подключит дополнительный вид (это может быть любой IView) и вернет его корневой узел, чтобы его можно было прикрепить к дереву DOM.
Все дополнительные виды будут отключены, когда родительский вид будет отключен.

``js
t.div({имя класса: "Контейнер"}, t.view(новый дочерний вид(vm.childViewModel)));
```

#### Односторонняя привязка данных

Привязка связывает часть DOM со значением в модели представления. Модель представления выдает обновление при изменении любого из своих свойств, на которое может подписаться представление. Когда представление получает обновление, оно переоценивает все привязки и соответствующим образом обновляет DOM.

Привязка может появиться во многих местах дерева шаблонов, где обычно можно использовать статическое значение.
Чтобы создать привязку, вы передаете функцию, которая преобразует значение представления в статическое значение.

##### Текстовая привязка

``js
t.p(["У меня есть ", vm => vm.counter, "beans"])
```

##### Привязка атрибута

``js
нажмите кнопку({отключено: vm => vm.IsBusy}, "Отправить");
```

##### Привязка к имени класса
``js
t.div({Имя класса: {
    кнопка: true,
активна: vm => vm.isActive
}})
```
##### Привязка подвида

До сих пор все привязки могли изменять только значения узлов в нашем дереве, но не изменяли структуру DOM. Привязка к вспомогательному виду позволяет вам условно добавлять вспомогательный вид на основе результата функции привязки.

Все привязки к вспомогательному виду возвращают узел DOM (элемент или комментарий) и могут быть непосредственно добавлены в дерево DOM путем включения их в ваш шаблон.

###### map

`t.MapView` позволяет выбрать вид на основе результата работы функции привязки:

``js
t.MapView(vm => vm.count, count => {
    возвращает значение count > 5 ? новый LargeView(количество) : новый SmallView(количество);
});
```

Каждый раз, когда первая функция или функция привязки возвращает другое значение, запускается вторая функция, чтобы создать новое представление взамен предыдущего.

Вы также можете вернуть "null" или "undefined" из второй функции, чтобы указать, что представление не должно отображаться. В этом случае в качестве заполнителя будет использоваться узел комментария.

Существует также `t.map`, который создаст новый шаблонный вид (с тем же значением), и вы напрямую предоставите для него функцию рендеринга:

``js
t.map(vm => vm.shape, (shape, t, vm) => {
    переключатель (форма) {
        регистр "rect": возвращает t.rect();
        регистр "circle": возвращает значение t.circle();
    }
})
```

###### if

`t.ifView` отобразит подвид, если привязка возвращает истинное значение:

``js
t.ifView(vm => vm.isActive, vm => новый вид(vm.someValue));
```

У вас также есть `t.if`, который создает `TemplateView` и передает вам `TemplateBuilder`:

``js
t.если(vm => vm.isActive, (t, vm) => t.div("активен!"));
```

##### Побочные эффекты

Иногда требуется в обязательном порядке изменить дерево DOM на основе значения привязки.
"mapSideEffect" позволяет легко это сделать:

``js
let node = t.div();
t.mapSideEffect(vm => vm.color, (color, oldColor) => node.style.background = цвет);
обратный узел;
```

** Примечание:** вам не следует добавлять какие-либо привязки, подпросмотры или обработчики событий из обратного вызова с побочным эффектом,
самое безопасное - вообще не использовать аргумент `t`.
Если вы это сделаете, они будут добавляться при каждом запуске обратного вызова и очищаться только после того, как представление будет размонтировано.

#### "tag" против "t"

Если вам не нужен компонент представления с привязкой к данным, вспомогательными представлениями и атрибутами обработчика событий, язык шаблонов также доступен в "ui/general/html.js" без каких-либо из этих наворотов, экспортируемый как `tag`. В отличие от статических шаблонов с "тегом", вы всегда используете
"TemplateView" в качестве экземпляра класса, поскольку необходимо отслеживать некоторые дополнительные состояния (привязки, обработчики событий и подвиды).

Хотя синтаксически они схожи, "TemplateBuilder" и "tag" функционально не эквивалентны.  
В первую очередь `t` ** поддерживает ** привязки и обработчики событий, в то время как `tag` ** этого не делает **. Это связано с тем, что для удаления прослушивателей событий нам нужно отслеживать их, и, следовательно, нам нужно где-то сохранить это состояние, чего
мы не можем сделать простым вызовом функции, но мы можем создать класс TemplateView.

``js
    // Нажатие здесь не сработает!!
    tag.button({Имя класса:"awesome-btn", onClick: () => this.foo()});

класс MyView расширяет TemplateView {
    рендеринг(t, vm){
        // Здесь работает onClick.
        t.button({Имя класса:"awesome-btn", onClick: () => this.foo()});
    }
}
```

## Просмотр списка

Компонент представления, который отображает и обновляет список вложенных представлений для каждого элемента в "ObservableList".

``js
const list = новый ListView({
    список: someObservableList
}, listValue => возвращает новый дочерний вид(listValue))
```

По мере добавления, удаления, перемещения (изменения позиции) и обновления элементов DOM будет синхронизироваться.

Существует также "LazyListView", который отображает элементы только в текущем окне просмотра и вокруг него, с ограничением, что все элементы в списке должны отображаться с одинаковой высотой.

### Обновления дополнительного представления

Если для параметра "parentProvidesUpdates" в конструкторе не установлено значение "false", ListView вызовет метод "update" дочернего компонента "IView", когда получит событие обновления для одного из элементов в "ObservableList".

Таким образом, не каждое дополнительное представление должно иметь отдельного слушателя в своей модели представления (значение из наблюдаемого списка), и все обновления переходят из наблюдаемого списка в представление списка, которое затем уведомляет правильное дополнительное представление.