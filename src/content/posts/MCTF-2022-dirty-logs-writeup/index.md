---
title: "Разбор таска Dirty logs с M*CTF 2022 или как желание пихнуть кавычку помогает решать CTF"
date: 2022-12-11T12:05:14+03:00
tags:
- writeup
- CTF
- WEB
- MCTF
---

Прошел почти год с того момента, как я написал свою первую [сатью](https://habr.com/ru/post/589433/) на Хабр. Начал этот путь именно с разбора задания MCTF 2021. Решил продолжить традицию в этом году и написать writeup на интересный таск с MCTF 2022.

![MCTF_Logo](https://habrastorage.org/r/w1560/getpro/habr/upload_files/eef/2b5/855/eef2b585560dc8fa4252deef7e4f5bce.png)

## Обзор

Для начала перейдем на сайт с заданием и посмотрим его структуру.Из кликабельных элементов пока заметна только кнопка "Логин". Нажмем на нее и посмотрим, что будет.

![http://mctf.ru:7777/](https://habrastorage.org/r/w1560/getpro/habr/upload_files/b3c/90d/ea7/b3c90dea7826757e501a0bea54512a0b.png)

Открывается страница авторизации. Можно пробовать проверить на XSS- и SQL-инъекции, но результата это не даст (еще не время для кавычек). Нужно искать другой путь.

![http://mctf.ru:7777/auth](https://habrastorage.org/r/w1560/getpro/habr/upload_files/144/b88/560/144b8856055b0764bcb9da50f3bb1001.png)

Вспоминаем про название таска - "Dirty logs".

Попробуем перейти по адресу http://mctf.ru:7777/logs.

![http://mctf.ru:7777/logs](https://habrastorage.org/r/w1560/getpro/habr/upload_files/032/4de/eea/0324deeead6a1b64cd3c3f93e9223024.png)

Странно, но это сработало, изучим логи и подумаем, что делать дальше...

## Проникаем в систему

Похоже на результат журналирования событий нашей страницы авторизации. Набор данных содержит логины пользователей и пароли, при неуспешной попытки аутентификации. Мне сразу бросился в глаза товарищ "v_ozerov", по попыткам которого можно сделать предположение, что он просто не может правильно написать свою парольную фразу.

![Похоже, что товарищ Озеров не может правильно написать фразу HeadOfTheCity
](https://habrastorage.org/r/w1560/getpro/habr/upload_files/c8b/00f/58c/c8b00f58c51359109c8d300e5d3a7b21.png)

Попробуем ввести его возможный пароль и пройти аутентификацию с кредами v_ozerov/HeadOfTheCity. Удалось попасть в "яблочко" с первого раза.Креды для входа оказались верными - идем дальше! 

А дальше мы попадаем на страницу http://mctf.ru:7777/dashboard.

![http://mctf.ru:7777/dashboard](https://habrastorage.org/r/w1560/getpro/habr/upload_files/fdc/dca/bc1/fdcdcabc1462037360762e298d84a881.png)


## Изучаем систему

Теперь, когда мы в системе, давайте посмотрим, что тут вообще происходит. Есть какие-то "События", "Аварии" и "Загрязнение".

По нажатию кнопки "Вывести данные" - выводятся данные (очень логично). Для пользовательского ввода доступно только поле "Дата" в формате даты (и снова логично). То есть ничего кроме цифр даты мы ввести туда не можем.

![Пользовательский ввод веб-интерфейса](https://habrastorage.org/r/w1560/getpro/habr/upload_files/3e6/8ac/759/3e68ac759942d3b404f026a36e09b634.png)

Обычно такие ограничения реализованы только на стороне клиента, а не сервера, если сайт написан на коленке. Чтобы это проверить, нужно как-то перехватить запрос пользователя, модифицировать его и отправить на сервер - запускаем Burp Suite!

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/dc4/1b7/128/dc41b712896cf9a74259d1857d3bfcef.png)

## Пробуем модифицировать запросы

Заранее оговорю тот момент, что подробно рассказывать о том, что такое Burp Suite и как им пользоваться я не буду. Это тема целой статьи, да и мануалов в Интернете достаточно. Итак, приступим...В Burp Suite переходим в раздел "Proxy" и нажимаем "Open Browser".

![Раздел "Proxy" в Burp Suite](https://habrastorage.org/r/w1560/getpro/habr/upload_files/146/849/bc5/146849bc5ecb28e0396121b238309a1c.png)

Переходим в открывшемся браузере на страницу с дашбордом http://mctf.ru:7777/dashboard (перед этим необходимо пройти авторизацию или просто использовать токен из прошлой сессии).

Затем, когда мы попали на дашборд - начинается самое интересное. 

Нажимаем в Burp Suite "Intercept is off", чтобы сменить режим для перехвата запросов, и в браузере жмем "Вывести данные" в любом из разделов.

Снова открываем Burp Suite и видим в главном окне содержимое нашего перехваченного, еще не отправленного на сервер, запроса.

![Запрос отправлен на сервер, но ответа еще нет, так как мы перехватили запрос и не доставили на сервер](https://habrastorage.org/r/w1560/getpro/habr/upload_files/b9e/8f5/d0d/b9e8f5d0de9f4af7e97e5675c6e76cec.png)

Жмем в любой из строке тела запроса ПКМ и выбираем "Send to Repeater".

![Отправляем перехваченный запрос в "Репитер"
](https://habrastorage.org/r/w1560/getpro/habr/upload_files/df1/b4d/965/df1b4d9651acc7e009e9bc38a09c8125.png)

После этого переходим на вкладку "Repeater" и нажимаем "Send".

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/9bc/33c/6bd/9bc33c6bde1262eff20532b3d50d7926.png)

Запрос успешно выполнился, результат в правой части окна. Данная вкладка позволяет изменять содержимое запроса и отправлять его снова и снова (опять логично, это же Repeater).

Как видно из запроса, нам действительно доступен только один параметр - date (дата, которую мы можем выбрать в веб-интерфейсе). Мы можем проверить логику приложения и передать в этот параметр дату, но мы сразу приступим к активной фазе - передадим в параметре date обычную "кавычку".

Вводим в запрос `date='` и нажимаем "Send".

![Пихнули кавычку и получили желаемый результат!](https://habrastorage.org/r/w1560/getpro/habr/upload_files/f1a/e06/149/f1ae06149be7321bdc35ae0410e31349.png)

Результат выполнения запроса выдает нам ошибку обработки команды и полностью подтверждает нашу теорию о том, что пользовательский ввод ограничен только на стороне клиента. Но почему именно кавычку? Пентестеры же любят "пихать" кавычки, а если серьезно, до ввода кавычки у меня не получилось заставить сервер хоть что-то внятное мне ответить. Не стал долго думать и "пихнул" - считаю оправданно!

## Готовим payload

Как видно из ошибки, параметр запроса date передается на вход Linux-команде (интерпретатора bash) grep на сервере, которая ищет соответствующий паттерн в определенном файле и выводит результат выполнения Linux-командой echo. 

При штатном поведении пользователя, при выборе в веб-интерфейсе даты, она передается аргументом команды grep и пользователю выдается список найденых по дате событий.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/53a/7fc/7a5/53a7fc7a5cbd33d4c596f05c4bbfef89.png)

На лицо атака типа Command Injection. Проблема заключается в том, что аргумент команды grep, куда попадает наш параметр, экранируется с двух сторон одинарными "кавычками". Необходимо подготовить payload имея следующий паттерн:

```bash
grep '<payload_from_date_argument>' /home/server/data/events.txt || echo ""
```

Из того, что пришло мне в голову - надо как-то выйти за пределы "кавычек" и выполнить произвольную команду. Немного пофантазировав и поигравшись с параметрами, удалось скрафтить следующую конструкцию:

```bash
grep '.' *;grep '.' /home/server/data/events.txt || echo ""
```

где, `.' *;grep '.` - и есть наш payload.Следуя логике полученной конструкции, в результате успешного выполнения запроса, нам должно показать содержимое всех файлов в текущей директории и содержимое файла events.txt. Проверяем...В Burp Suite в тело запроса пишем `date=.' *;grep '.` , нажимаем "Send".

[Пробуем наш payload](https://habrastorage.org/r/w1560/getpro/habr/upload_files/2fb/b55/623/2fbb556234169d6d13636b99a93220f5.png)

О чудо! Нам вывели содержимое всех файлов в текущей директории.

Продвигаемся дальше, давайте посмотрим список пользователей в домашней директории /home, для чего в теле запроса пишем `date=.' /home/*;grep '.` 

![Программа выполнилась с ошибкой, но дала нам полезную информацию о названии директорий](https://habrastorage.org/r/w1560/getpro/habr/upload_files/e8b/109/6ec/e8b1096ec7f2de514ad8eed009c23471.png)

Получаем localuser и server.

Посмотрим что есть у пользователя localuser - `date=.' /home/localuser/*;grep '.`

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/c4e/815/9a1/c4e8159a16eb56057efc5b9baa0c81eb.png)

Видим еще две директории: Classified и programs.

Посмотрим что есть в первой директории Classified - `date=.' /home/localuser/Classified/*;grep '.`

![Получаем заветный флагПолучаем заветный флаг](https://habrastorage.org/r/w1560/getpro/habr/upload_files/3c1/1b5/c17/3c11b5c174037f0d56554e108864913b.png)

И получаем флаг - `MCTF{D@RkZ3r5K-iS_w@1ch1ng}`

## Заключение

Таким образом, наш боевой payload выглядит следующим образом: `date=.' /home/localuser/Classified/*;grep '.`, благодаря которому на стороне сервера выполняется команда:

```bash
grep '.' /home/localuser/Classified/*;grep '.' /home/server/data/events.txt || echo ""
```

Надеюсь, что читателям понравится данный writeup. Как и для всех заданий в CTF, не исключаю возможности более быстрого и лаконичного решения. Данная статья - это лишь описание хода моих мыслей при решении.

Тем, кто не смог решить данный таск, желаю не расстраиваться, а взять технику "пихать кавычку везде,где только можно" себе на вооружение!
