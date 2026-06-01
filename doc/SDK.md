# Hydrogen View SDK


This SDK allows developers to integrate parts of the Hydrogen application into the UI of their own application. Hydrogen is written with the MVVM pattern, so to construct a view, you'd first construct a view model, which you then pass into the view. For most view models, you will first need a running client.

## Changelog
[See CHANGELOG.md](./CHANGELOG.md)  

## Tutorial

The Hydrogen SDK requires some assets to be shipped along with your app for things like downloading attachments, and end-to-end encryption. A convenient way to make this happen is provided by the SDK (importing `hydrogen-view-sdk/paths/vite`) but depends on your build system. Currently, only [vite](https://vitejs.dev/) is supported, so that's what we'll be using in the example below.

You can create a vite project using the following commands:

```sh
# you can pick "vanilla-ts" here for project type if you're not using react or vue
yarn create vite
cd <your-project-name>
yarn
yarn add hydrogen-view-sdk
yarn add https://gitlab.matrix.org/api/v4/projects/27/packages/npm/@matrix-org/olm/-/@matrix-org/olm-3.2.14.tgz
```
#### Including the service worker
In addition to the assets mentioned above, you will also need to supply a serviceworker. This is needed for supporting authenticated media ([MSC3916](https://github.com/matrix-org/matrix-spec-proposals/pull/3916), [blog post](https://matrix.org/blog/2024/06/26/sunsetting-unauthenticated-media/)).  
This is how you can do that:  
1. create a `public` directory in your project root.
2. In your `vite.config.js` file, configure [publicDir](https://vitejs.dev/config/shared-options.html#publicdir) option to point to the directory you just created.
3. Symlink `node_modules/hydrogen-view-sdk/lib-build/sw.js` to `public/sw.js`:
    ```bash
    cd public
    ln -s ../node_modules/hydrogen-view-sdk/lib-build/sw.js sw.js
    ``` 
Now `sw.js` will be in the root of your dev server/ build root.

#### Rendering the app

You should see a `index.html` in the project root directory, containing an element with `id="app"`. Add the attribute `class="hydrogen"` to this element, as the CSS we'll include from the SDK assumes for now that the app is rendered in an element with this classname.

If you go into the `src` directory, you should see a `main.ts` file. If you put this code in there, you should see a basic timeline after login and initial sync have finished (might take a while before you see anything on the screen actually).

You'll need to provide the username and password of a user that is already in the [#element-dev:matrix.org](https://matrix.to/#/#element-dev:matrix.org) room (or change the room id).

```ts
import {
    Platform,
    Client,
    LoadStatus,
    createNavigation,
    createRouter,
    RoomViewModel,
    TimelineView,
    viewClassForTile,
    FeatureSet
} from "hydrogen-view-sdk";
import downloadSandboxPath from 'hydrogen-view-sdk/download-sandbox.html?url';
import workerPath from 'hydrogen-view-sdk/main.js?url';
import olmWasmPath from '@matrix-org/olm/olm.wasm?url';
import olmJsPath from '@matrix-org/olm/olm.js?url';
import olmLegacyJsPath from '@matrix-org/olm/olm_legacy.js?url';
const assetPaths = {
    downloadSandbox: downloadSandboxPath,
    worker: workerPath,
    olm: {
        wasm: olmWasmPath,
        legacyBundle: olmLegacyJsPath,
        wasmBundle: olmJsPath
    },
    serviceWorker: "sw.js",
};
import "hydrogen-view-sdk/assets/theme-element-light.css";
// OR import "hydrogen-view-sdk/assets/theme-element-dark.css";

async function main() {
    const app = document.querySelector<HTMLDivElement>('#app')!
    const config = {};
    const platform = new Platform({container: app, assetPaths, config, options: { development: import.meta.env.DEV }});
    const navigation = createNavigation();
    platform.setNavigation(navigation);
    await platform.init();
    const urlRouter = createRouter({
        navigation: navigation,
        history: platform.history
    });
    urlRouter.attach();
    const client = new Client(platform);

    const loginOptions = await client.queryLogin("matrix.org").result;
    client.startWithLogin(loginOptions.password("username", "password"));

    await client.loadStatus.waitFor((status: string) => {
        return status === LoadStatus.Ready ||
            status === LoadStatus.Error ||
            status === LoadStatus.LoginFailed;
    }).promise;

    if (client.loginFailure) {
        alert("login failed: " + client.loginFailure);
    } else if (client.loadError) {
        alert("load failed: " + client.loadError.message);
    } else {
        const {session} = client;
        // looks for room corresponding to #element-dev:matrix.org, assuming it is already joined
        const room = session.rooms.get("!bEWtlqtDwCLFIAKAcv:matrix.org");
        const features = await FeatureSet.load(platform.settingsStorage);
        const vm = new RoomViewModel({
            room,
            ownUserId: session.userId,
            platform,
            urlRouter: urlRouter,
            navigation,
            features,
        });
        await vm.load();
        const view = new TimelineView(vm.timelineViewModel, viewClassForTile);
        app.appendChild(view.mount());
    }
}

main();
```

## Typescript support

Typescript support is not yet available while we're converting the Hydrogen codebase to Typescript.
In your `src` directory, you'll need to add a `.d.ts` (can be called anything, e.g. `deps.d.ts`)
containing this snippet to make Typescript not complain that `hydrogen-view-sdk` doesn't have types:

```ts
declare module "hydrogen-view-sdk";
```

## API Stability

This library follows semantic versioning; there is no API stability promised as long as the major version is still 0. Once 1.0.0 is released, breaking changes will be released with a change in major versioning.

## Third-party licenses

This package bundles the bs58 package ([license](https://github.com/cryptocoinjs/bs58/blob/master/LICENSE)), and the Inter font ([license](https://github.com/rsms/inter/blob/master/LICENSE.txt)).


Перевод

# Hydrogen View SDK


Этот SDK позволяет разработчикам интегрировать части приложения Hydrogen в пользовательский интерфейс своего собственного приложения. Hydrogen написан с использованием шаблона MVVM, поэтому для создания представления сначала необходимо создать модель представления, которую затем передать в представление. Для большинства моделей просмотра вам сначала понадобится запущенный клиент.

## Список изменений
[Смотрите CHANGELOG.md](./CHANGELOG.md)  

## Руководство

Hydrogen SDK требует, чтобы вместе с вашим приложением поставлялись некоторые ресурсы для таких задач, как загрузка вложений и сквозное шифрование. Удобный способ сделать это предоставляется SDK (импорт "hydrogen-view-sdk/paths/vite"), но зависит от вашей системы сборки. В настоящее время поддерживается только [vite](https://vitejs.dev/), поэтому в приведенном ниже примере мы будем использовать именно его.

Вы можете создать проект vite, используя следующие команды:

``sh
# вы можете выбрать "vanilla-ts" здесь в качестве типа проекта, если вы не используете react или
vue, и создать vite
cd <название вашего проекта>
пряжа
добавление yarn hydrogen-просмотр-sdk
добавление yarn https://gitlab.matrix.org/api/v4/projects/27/packages/npm/@matrix-org/olm/-/@matrix-org/olm-3.2.14.tgz
```
#### Включая сервисного работника
В дополнение к перечисленным выше ресурсам вам также потребуется предоставить сервисного работника. Это необходимо для поддержки аутентифицированных носителей ([MSC3916](https://github.com/matrix-org/matrix-spec-proposals/pull/3916), [сообщение в блоге](https://matrix.org/blog/2024/06/26/sunsetting-unauthenticated-media/)).  
Вот как это можно сделать:  
1. создайте "общедоступный" каталог в корневом каталоге вашего проекта.
2. В вашем файле "vite.config.js` настройте параметр [publicDir](https://vitejs.dev/config/shared-options.html#publicdir) так, чтобы он указывал на только что созданный вами каталог.
3. Символическая ссылка `node_modules/hydrogen-view-sdk/lib-build/sw.js" на `public/sw.js`:
    `"bash
cd public
ln -s ../node_modules/hydrogen-view-sdk/lib-build/sw.js sw.js
    ``` 
Теперь `sw.js` будет находиться в корневом каталоге вашего сервера разработки/ сборки.

#### Рендеринг приложения

Вы должны увидеть "index.html` в корневом каталоге проекта, содержащий элемент с `id="app"`. Добавьте атрибут `class="hydrogen" к этому элементу, поскольку CSS, который мы включим из SDK, на данный момент предполагает, что приложение отображается в элементе с таким именем класса.

Если вы зайдете в каталог "src", вы увидите файл "main.ts". Если вы введете туда этот код, вы увидите основную временную шкалу после завершения входа в систему и начальной синхронизации (может пройти некоторое время, прежде чем вы увидите что-либо на экране).

Вам нужно будет ввести имя пользователя и пароль пользователя, который уже находится в [#element-dev:matrix.org](https://matrix.to/#/#element-dev:matrix.org) комнате (или изменить идентификатор комнаты).

``ts
import {
    Платформа,
    Клиент,
    LoadStatus,
createNavigation,
createRouter загружается автоматически.,
    RoomViewModel,
    TimelineView,
viewClassForTile,
    Набор функций
} из "hydrogen-view-sdk";
импортировать downloadSandboxPath из "hydrogen-view-sdk/download-sandbox.html?url";
импортировать рабочий путь из 'hydrogen-view-sdk/main.js?url';
импортировать olmWasmPath из '@matrix-org/olm/olm.wasm?url';
импорт olmJsPath из '@matrix-org/olm/olm.js?url';
импорт olmLegacyJsPath из '@matrix-org/olm/olm_legacy.js?url';
постоянные пути к активам = {
    downloadSandbox: downloadSandboxPath,
рабочий: workerPath,
olm: {
        wasm: olmWasmPath,
legacyBundle: olmLegacyJsPath,
wasmBundle: olmJsPath
    },
ServiceWorker: "sw.js",
};
импортировать "hydrogen-view-sdk/assets/theme-element-light.css";
// ИЛИ импортировать "hydrogen-view-sdk/assets/theme-element-dark.css";

асинхронная функция main()
_BOS_ const приложение = документ.querySelector<HTMLDivElement>('#приложение')!
    const config = {} постоянная конфигурация = _BOS_};
    const platform = новая платформа({контейнер: приложение, пути к ресурсам, конфигурация, параметры: { разработка: импорт.meta.env.DEV }});
    const navigation = createNavigation();
    platform.setNavigation(навигация);
    ждем platform.init();
    const urlRouter = createRouter({
        навигация: навигация,
история: platform.history
    });
    urlRouter.attach();
    постоянный клиент = новый клиент(платформа);

    const loginOptions = ожидание клиента.queryLogin("matrix.org").результат;
    client.startWithLogin(loginOptions.password("имя пользователя", "пароль"));

    ожидающий клиент.LoadStatus.waitFor((статус: строка) => {
        возвращаемый статус === LoadStatus.Готов ||
            статус === LoadStatus.Ошибка ||
            статус === LoadStatus.Ошибка входа в систему;
    }).обещаю;

    если (ошибка входа в систему клиента) {
        предупреждение("ошибка входа в систему: " + client.loginFailure);
    } иначе, если (client.LoadError) {
        предупреждение("ошибка загрузки: " + client.LoadError.message);
    } еще {
        const {сессия} = клиент;
        // ищет комнату, соответствующую #element-dev:matrix.org , предполагая, что он уже присоединен
        const room = session.rooms.get("!bEWtlqtDwCLFIAKAcv:matrix.org");
        const features = ожидание набора функций.load(платформа.Хранилище настроек);
        const vm = новая модель просмотра комнаты({
            комната,
собственный идентификатор пользователя: session.userId,
            платформа,
            urlRouter: urlRouter,
навигация,
функции,
});
        ожидание загрузки виртуальной машины();
        const view = новый TimelineView(vm.timelineViewModel, viewClassForTile);
        app.appendChild(view.mount());
    }
}

main();
```

## Поддержка машинописи

Поддержка Typescript пока недоступна, пока мы переводим кодовую базу Hydrogen в Typescript.
В вашем каталоге "src" вам нужно добавить ".d.ts" (может называться как угодно, например "deps.d.ts").
содержащий этот фрагмент, чтобы Typescript не жаловался на то, что в "hydrogen-view-sdk" нет типов:

"`ts
объявляет модуль "hydrogen-view-sdk";
```

## Стабильность API

Эта библиотека поддерживает семантическое управление версиями; стабильность API не обещана, пока основная версия по-прежнему равна 0. Как только будет выпущена версия 1.0.0, будут выпущены основные изменения с изменением основных версий.

## Лицензии сторонних производителей

Этот пакет включает в себя пакет bs58 ([лицензия](https://github.com/cryptocoinjs/bs58/blob/master/LICENSE)) и шрифт Inter ([лицензия](https://github.com/rsms/inter/blob/master/LICENSE.txt)).