## Warning

Usage of docker is a third-party contribution and not actively tested, used or supported by the main developer(s).

Having said that, you can also use Docker to create a local dev environment or a production deployment.

## Dev environment

In this repository, create a Docker image:

```
docker build -t hydrogen-dev -f Dockerfile-dev .
```

Then start up a container from that image:

```
docker run \
    --name hydrogen-dev \
    --publish 3000:3000 \
    --volume "$PWD":/code \
    --interactive \
    --tty \
    --rm \
    hydrogen-dev
```

Then point your browser to `http://localhost:3000`. You can see the server logs in the terminal where you started the container.

To stop the container, simply hit `ctrl+c`.

## Production deployment

### Build or pull image

In this repository, create a Docker image:

```sh
# Enable BuildKit https://docs.docker.com/develop/develop-images/build_enhancements/
export DOCKER_BUILDKIT=1
docker build -t hydrogen .
```

Or, pull the docker image from GitHub Container Registry:

```
docker pull ghcr.io/element-hq/hydrogen-web
docker tag ghcr.io/element-hq/hydrogen-web hydrogen
```

### Start container image

Then, start up a container from that image:

```
docker run \
    --name hydrogen \
    --publish 8080:8080 \
    hydrogen
```

n.b. the image is now based on the unprivileged nginx base, so the port is now `8080` instead of `80` and you need a writable `/tmp` volume.

You can override the default `config.json` using the `CONFIG_OVERRIDE` environment variable. For example to specify a different Homeserver and :

```
docker run \
    --name hydrogen \
    --publish 8080:8080 \
    --env CONFIG_OVERRIDE='{
  "push": {
    "appId": "io.element.hydrogen.web",
    "gatewayUrl": "https://matrix.org",
    "applicationServerKey": "BC-gpSdVHEXhvHSHS0AzzWrQoukv2BE7KzpoPO_FfPacqOo3l1pdqz7rSgmB04pZCWaHPz7XRe6fjLaC-WPDopM"
  },
  "defaultHomeServer": "https://fosdem.org",
  "themeManifests": [
    "assets/theme-element.json"
  ],
  "defaultTheme": {
    "light": "element-light",
    "dark": "element-dark"
  }
}' \
    hydrogen
```


Перевод

## Предупреждение

Использование docker - это вклад третьей стороны, который не тестировался, не использовался и не поддерживается основными разработчиками.

Тем не менее, вы также можете использовать Docker для создания локальной среды разработки или производственного развертывания.

## Среда разработки

В этом репозитории создайте образ Docker:

```
сборка docker -t hydrogen-dev -f Dockerfile-dev .
```

Затем запустите контейнер из этого изображения:

```
запуск docker \
    --имя hydrogen-dev \
    --опубликовано 3000:3000 \
    --объем "$PWD":/код \
    --интерактивный \
    --tty \
    --rm \
    hydrogen-dev
```

Затем наведите курсор на "http://localhost:3000`. Вы можете просмотреть логи сервера в терминале, где вы запустили контейнер.

Чтобы остановить контейнер, просто нажмите `ctrl+c`.

## Производственное развертывание

### Создайте или извлеките образ

В этом репозитории создайте образ Docker:

``sh
# Включить BuildKit https://docs.docker.com/develop/develop-images/build_enhancements/
экспортировать DOCKER_BUILDKIT=1
docker build -t hydrogen .
```

Или извлеките образ docker из реестра контейнеров GitHub:

```
подключение к докеру ghcr.io/element-hq/hydrogen-web
тег докера ghcr.io/element-hq/hydrogen-web водород
```

### Запустите изображение контейнера

Затем запустите контейнер из этого изображения:

```
запуск docker \
    --присвоить имя hydrogen \
    --опубликовать 8080:8080 \
    водород
```

примечание: образ теперь основан на непривилегированной базе nginx, поэтому порт теперь "8080" вместо "80", и вам нужен доступный для записи том `/tmp`.

Вы можете переопределить значение `config.json` по умолчанию, используя переменную окружения `CONFIG_OVERRIDE`. Например, чтобы указать другой домашний сервер и :

```
запуск docker \
    --присвоить имя hydrogen \
    --опубликовать 8080:8080 \
    --env CONFIG_OVERRIDE='{
  "толчок": {
    "AppID": "io.элемент.водород.сеть",
    "gatewayUrl": "https://matrix.org ",
"Ключ сервера приложений": "BC-gpSdVHEXhvHSHS0AzzWrQoukv2BE7KzpoPO_FfPacqOo3l1pdqz7rSgmB04pZCWaHPz7XRe6fjLaC-WPDopM"
  },
"defaultHomeServer": "https://fosdem.org ",
"themeManifests": [
"активы/элемент темы.json"
  ],
"Тема по умолчанию": {
    "светлый": "стихия-свет",
"темный": "стихия-тьма"
  }
}' \
    водород
```