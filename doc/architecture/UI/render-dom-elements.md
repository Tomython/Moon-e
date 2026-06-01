tldr; Use `tag` from `ui/general/html.js` to quickly create DOM elements.

## Syntax
---
The general syntax is as follows:
```js
tag.tag_name({attribute1: value, attribute2: value, ...}, [child_elements]);
```
**tag_name** can be any one of the following:
```
    br, a, ol, ul, li, div, h1, h2, h3, h4, h5, h6,
    p, strong, em, span, img, section, main, article, aside,
    pre, button, time, input, textarea, label, form, progress, output, video
```

<br />

eg:
Here is an example HTML segment followed with the code to create it in Hydrogen.
```html
<section class="main-section">
    <h1>Demo</h1>
    <button class="btn_cool">Click me</button>
</section>
```
```js
tag.section({className: "main-section"},[
    tag.h1("Demo"), 
    tag.button({className:"btn_cool"}, "Click me")
    ]);
```
<br />

**Note:** In views based on `TemplateView`, you will see `t` used instead of `tag`.  
`t` is is `TemplateBuilder` object passed to the render function in `TemplateView`.
Although syntactically similar, they are not functionally equivalent.  
Primarily `t` **supports** bindings and event handlers while `tag` **does not**.

```js
    // The onClick here wont work!!
    tag.button({className:"awesome-btn", onClick: () => this.foo()});

    render(t, vm){
        // The onClick works here.
        t.button({className:"awesome-btn", onClick: () => this.foo()});
    }
```


Перевод

tldr; Используйте `tag` из `ui/general/html.js` для быстрого создания элементов DOM.

## Синтаксис
---
Общий синтаксис следующий:
``
тег js.имя_тега({атрибут1: значение, атрибут2: значение, ...}, [дочерние_элементы]);
```
**имя_тега** может быть любым из следующих значений:
```
    br, a, ol, ul, li, div, h1, h2, h3, h4, h5, h6,
p, strong, em, span, img, раздел, основной, статья, в стороне,
предварительно, кнопка, время, ввод, текстовое поле, метка, форма, ход выполнения, вывод, видео
```

<br />

напр.:
Вот пример HTML-фрагмента, за которым следует код для его создания в Hydrogen.
``html
<класс раздела="main-section">
    <h1>Демонстрация</h1>
    <класс кнопки="btn_cool">Нажмите на меня</кнопка>
</раздел>
```
``js
tag.section({имя класса: "основной раздел"},[
tag.h1("Демонстрация"),
tag.button({имя класса:"btn_cool"}, "Нажмите на меня")
    ]);
```
<br />

**Примечание:** В представлениях, основанных на "TemplateView", вместо "tag" будет использоваться "t".  
"t" - это объект "TemplateBuilder", передаваемый функции рендеринга в `TemplateView`.
Хотя синтаксически они похожи, функционально они не эквивалентны.  
В первую очередь `t` ** поддерживает ** привязки и обработчики событий, в то время как `tag` ** не поддерживает **.

``js
    // onClick здесь не сработает!!
    tag.button({Имя класса:"awesome-btn", onClick: () => this.foo()});

    рендеринг(t, vm){
        // Здесь работает onClick.
        t.кнопка({Имя класса:"awesome-btn", onClick: () => this.foo()});
    }
```