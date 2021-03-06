# Шаблон для разработки навыков на JAICF для разработчиков JAICP

Этот шаблон JAICF проекта для разработчиков, ранее разабатывающих на JAICP DSL. Из коробки предоставляется:

* HttpClient для внешних запросов
* Классы для работы с JSON
* YAML конфигурации

> Если вы никогда не работали с JAICF - советуем почитать
[документацию для миграции с JAICP на JAICF](https://github.com/Denire/jaicf-template-for-jaicp-developers/wiki).

## Запуск в два клика:

1. Вставить ваш токен проекта в `src/main/resources/application.yml`
2. Запустить файл TemplateBot.kt

![Run Kotlin File](https://github.com/Denire/jaicf-template-for-jaicp-developers/raw/master/static/Run%20Kotlin%20File.png)

## Структура проекта

```
src
├── main
│   ├── kotlin
│   │   └── com
│   │       └── justai
│   │           └── jaicf
│   │               └── template
│   │                   ├── TemplateBot.kt // файл с запуском бота
│   │                   ├── configuration 
│   │                   │   └── MainConfiguration.kt // файл с kotlin-конфигурациями
│   │                   ├── extensions
│   │                   │   └── BotEngineRunners.kt // утилитарные расширеня для удобного запуска
│   │                   ├── http
│   │                   │   └── Http.kt // настроенный HttpClent
│   │                   ├── scenario
│   │                   │   ├── ExampleBitcoinScenario.kt // пример сценария с внешней интеграцией
│   │                   │   └── MainScenario.kt // пример сценария
│   │                   └── serializers
│   │                       ├── Jackson.kt // Jackson-серилазатор для работы с JSON
│   │                       └── Kotlinx.kt // KotlinX-сериализатор для работы с JSON
│   └── resources
│       ├── application-local.example.yml // пример локальной конфигурации проекта
│       └── application.yml // конфигурация проекта
└── test
    └── kotlin
        └── MainScenarioTest.kt // Пример тесты
```

## Файлы для сборки через gradle

Файл сборки называется build.gradle.kts, он лежит в корне проекта

```kotlin
plugins {
    // подключение плагинов. 
}

group = "com.justai.jaicf"
version = "1.0.0"

val jaicf = "1.1.3" // версия библиотеки JAICF

// Main class to run application on heroku. Either JaicpPollerKt, or JaicpServerKt. Will propagate to .jar main class.
application {
    mainClassName =
        "com.justai.jaicf.template.TemplateBotKt" // имя класса, с которым будет запускаться исполняемый .jar файл
}

repositories {
    // список репозиториев, с которых будут выкачиваться зависимости. Обычно не меняется.
}

dependencies {
    // список зависимостей, используемых в проекте. 
    // Вы можете находить нужные вам зависимости в интернете и добавлять их сюда, чтобы использовать в проекте.
}

tasks {
    // список задач. В том числе задачи для тестирования
}

tasks.create("stage") {
    dependsOn("shadowJar") // задача для сборки проекта для Heroku
}

tasks.withType<com.justai.jaicf.plugins.jaicp.build.JaicpBuild> {
    mainClassName.set(application.mainClassName) // установка имени класса для деплоя в JAICP Cloud
}

```

## Подключение

В шаблоне предусмотрены варианты подключения через webhook или polling-соединения.

* `polling` - тип соединения, когда наш бот сам обращается к серверам JustAI для получения новых запросов. Этот тип
  подключения проще и рекомендуется для локальной разработки. Пример запуска:
    1. Установить `connection.mode: polling` в файле `application-local.yml`
    ```yaml
      connection:
        mode: polling
        accessToken: <Your Access Token from JAICP Application Console>
    ```
    2. Запустить приложение
       ```kotlin
          fun main() {
              templateBot.run(ChatWidgetChannel, ChatApiChannel, TelephonyChannel)
          }
       ```

* `webhook` - тип соединения, когда сервера JustAI ходят на наше приложение через внешний URL. В этом случае мы должны
  обеспечить доступность приложения по внешнему адресу, например, с помощью утилиты ngrok.

  Пример запуска:
    1. Установить `connection.mode: webhook` в файле `application-local.yml`
    ```yaml
      connection:
        mode: webhook
    ```
    2. Запустить приложение
       ```kotlin
          fun main() {
              templateBot.run(ChatWidgetChannel, ChatApiChannel, TelephonyChannel)
          }
       ```
    3. Получить внешний адрес через ngrok, скопировать его.
    ```shell
    $ ngrok http 8080
    ```
    4. Поставить тип соединения `webhook`  в настройках JAICF проекта в админке, указать скопированный URL.

Для локальной разработки (для запуска бота на своём копмьютере) рекомендуется использовать polling-соединение.

## Конфигурирование

Для конфигурирования бота предусмотрены .yml файлы в `/src/main/resources` и обьект `Configuration` в коде. Используется
библиотека [hoplite](https://github.com/sksamuel/hoplite) для загрузки конфигурации.

Пример конфигурации лежит в файле `MainConfiguration.kt`. В нем обьявляется формат для `yml` конфига.

```kotlin
data class MainConfiguration(
    val caila: CailaNLUSettings,
    val connection: ConnectionsConfiguration,
    val bot: BotConfiguration,
)

// ...
data class BotConfiguration(
    val onErrorReply: String,
)
```

```yaml
caila:
  accessToken: ${JAICP_API_TOKEN}

connection:
  mode: webhook
  accessToken: ${JAICP_API_TOKEN}

bot:
  onErrorReply: "Извините, у нас все сломалось"
```

#### Как добавить новое поле в конфигурацию?

1. Необходимо добавить значение в `.yml` файл.

```yaml
bot:
  onErrorReply: "Извините, у нас все сломалось"
  randomImagesOnError:
    - "sad_cat.jpg"
    - "crying_dog.jpg" 
```

2. Необходимо добавть чтение значение в котлин-файл конфигурации.

```kotlin
data class BotConfiguration(
    val onErrorReply: String,
    val randomImagesOnError: List<String>
)
```

3. После чего использовать значение в любом необходимом месте. Например, в сценарии:

```kotlin
handle<AnyErrorHook> {
    reactions.say(Configuration.bot.onErrorReply)
    reactions.image(Configuration.bot.randomImagesOnError.random())
    logger.error(exception)
}
```

## Работа с JSON

В шаблоне подключены две библиотеки для работы с JSON:

* Jackson
* Kotlinx Serialization

Они доступны из любого места в приложении и решают задачи приведения обьекта в строковый вид и обратно. Рекомендуем
использовать Jackson, так как на нем легче нагуглить нужную информацию.

#### Пример работы с Jackson:

Сериализация - приведение обьекта к строковому виду в JSON:

```kotlin
data class MyDataClass(
    val foo: String,
    var bar: Int,
)

fun main(args: Array<String>) {
    val example = MyDataClass("foo!", 1)
    val stringify = Jackson.stringify(example)
    println(stringify)
    // {"foo":"foo!","bar":1}
}
```

Десериализация - чтение обьекта из строкового (JSON) вида:

```kotlin
data class MyDataClass(
    val foo: String,
    var bar: Int,
)

fun main(args: Array<String>) {
    val json = """{"foo":"foo!","bar":1}"""
    val example = Jackson.parse<MyDataClass>(json)
    println(example)
    // MyDataClass(foo=foo!, bar=1)
}
```

## Работа с HTTP

У нас используется HTTP-клиент из библиотеки [KTOR](https://ktor.io/docs/client.html). Его синтаксис достаточно похож на
синтаксис сервиса `$http` в JAICP DSL. Посмотрим на пример сценария с внешней интеграцией.

```kotlin
data class BitcoinToUSD(val USD: Record) {
    data class Record(val buy: Double, val sell: Double)
}

val ExampleBitcoinScenario = Scenario {
    state("getBitcoinPrice") {
        action {
            val data = runBlocking {
                httpClient.get<BitcoinToUSD>("https://blockchain.info/ticker")
            }
            reactions.say("You can buy BitCoin for ${data.USD.buy}$")
        }
    }
}
```

> (!) KTOR заточен под асинхронную работу с http вызовами в Kotlin. Для того, чтобы сделать вызов синхронным, мы оборачиваем его в `runBlocking`.

В данном примере у нас есть `data class BitcoinToUSD`, и мы говорим, что формат ответа внешнего сервиса должен
привестись к формату данного класса. Это позволяет нам не ковыряться в JSON руками, доставая элементы и волнуясь, вдруг
какой-то элемент придет пустым и мы получим ошибку вида `cannot get property ... from undefined`. Это также позволяет
писать более компактный код в теле `action` блока в сценарии.

#### Более сложные запросы

В примере выше использовался простой GET запрос без параметров. Запросы могут быть сколько угодно сложными.

```kotlin
data class RequestData(
    val amount: Int,
    val name: String,
)

data class ResponseData(
    val status: Int,
    val response: String,
)

fun main(args: Array<String>) {

    val response = runBlocking {
        httpClient.post<ResponseData>("https://example.com") {
            body = RequestData(1, "JustAI")
            parameter("parameter-name", "parameterValue")
            contentType(ContentType.Application.Json)
            header("header-name", "header-value")
        }
    }

}
```

В примере выше используется тело, парметры, тип контента и хэдеры. Также можно добавить таймауты, кэширование, и
остальное, см. документацию Ktor-Client.
