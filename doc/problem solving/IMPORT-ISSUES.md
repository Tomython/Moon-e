## How to import common-js dependency using ES6 syntax
---
Until [#6632](https://github.com/vitejs/vite/issues/6632) is fixed, such imports should be done as follows:

```ts
import * as pkg from "off-color";
// @ts-ignore 
const offColor = pkg.offColor ?? pkg.default.offColor;
```

This way build, dev server and unit tests should all work.


Перевод

## Как импортировать зависимость common-js с использованием синтаксиса ES6
---
До тех пор, пока [#6632](https://github.com/vitejs/vite/issues/6632) не будет исправлено, такой импорт должен выполняться следующим образом:

``ts
import * as pkg from "off-color";
// @ts-игнорировать
константу offColor = pkg.offColor ?? pkg.default.offColor;
```

Таким образом, сборка, сервер разработки и модульные тесты - все это должно работать.