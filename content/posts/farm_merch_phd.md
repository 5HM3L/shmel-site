---

title: "farm_merch_phd"
date: 2022-05-25T10:00:00+03:00
tags:
- PHD11
- telebot
- Telegram
---

# Фарм мерча от Positive Technologies на PHD11

![img](https://github.com/eeenvik1/shmel-site/blob/main/content/posts/farm_merch_phd.assets/logo.png)

C 18 по 19 мая 2022 года проходил всем известный международный форум по практической безопасности [Positive Hack Days](https://phdays.com/). 
На мероприятии как всегда были представлены различные вендоры из сферы IT, каждый из которых раздавал свои именные товары. 
Самыми интересными на мой взгляд были [Positive Technologies](http://ptsecurity.com/). 

И так, для получения мерча, вендором было предложено успешно пройти квиз (набрать 10 очков из 10 или 100% правильных ответов) по различным темам, а именно:
- Анализ защищенности web приложений
- Безопасность мобильных приложений
- Вопросы от SOC
- Пентест
- АСУТП
- Linux

Реализация самого квиза - это телеграмм бот [@PHD_QuizBot](t.me/PHD_QuizBot), который имеет имя `phd_quiz_bot`.  
Немного потыкав самого бота и поняв логику его работы, а также, внимательно посмотрев на имя бота понимаем что буква *Q* очень сильно похожа на букву *O*. 
Сразу же появилась идея создать копию данного бота используя атаку *тайпсквоттинг*. *Тайпсквоттинг - это вид атак социальной инженерии, 
нацеленный на пользователей интернета, допустивших опечатку при вводе веб-адреса в браузере и не использующих поисковую систему. 
(в данном случае мы подменим букву Q на букву O).*

Итак, приступим:
1. Через [@BotFather](t.me/BotFather) создадим своего бота @PHD_OuizBot с именем `phd_quiz_bot`.

Оригинальный бот:

![img](https://github.com/eeenvik1/shmel-site/blob/main/content/posts/farm_merch_phd.assets/Screenshot_1.png)

Созданный нами ложный бот:

![img](https://github.com/eeenvik1/shmel-site/blob/main/content/posts/farm_merch_phd.assets/Screenshot_2.png)

2. Начинаем проходить квиз у оригинального бота, для того, чтобы вытащить по 10 вопросов в каждой категории 
(одну из категорий придется решить на 10/10, чтобы получить валидный ответ от бота на получение мерча).

Пример вопроса:

![img](https://github.com/eeenvik1/shmel-site/blob/main/content/posts/farm_merch_phd.assets/Screenshot_3.png)

Пример валидного ответа от бота:

![img](https://github.com/eeenvik1/shmel-site/blob/main/content/posts/farm_merch_phd.assets/Screenshot_4.png)

3. После того, как мы всё прорешали, у нас получился список из 60 вопросов и 6 ответов от бота по каждой категории. Переходим к кодингу.
4. Реализовывать ложного бота будем через python и библиотеку `telebot`:
```python
import telebot 
from telebot.types import InlineKeyboardMarkup, InlineKeyboardButton 
bot = telebot.TeleBot(<'bot_API_key'>) 
CHAT_ID = <'Your chat id'>
```
Для того, чтобы узнать `chat_id` нужно написать этому боту [@getmyid_bot](t.me/getmyid_bot). 
Ответ будет выглядеть так:
```python
Your user ID: 666******
Current chat ID: 666******
```

Теперь пропишем каждый вопрос и ответ на него:
```python
message_SOC_1 = '''Какой прикладной протокол затрагивает уязвимость BlueKeep? 
1  SMB 
2  SMTP 
3  RDP 
4  FTP''' 
def gen_markup_SOC_1(): 
    markup = InlineKeyboardMarkup() 
    markup.row_width = 4 
    markup.add(InlineKeyboardButton("1✔️", callback_data="cb_yes"), 
               InlineKeyboardButton("2", callback_data="cb_no"), 
               InlineKeyboardButton("3", callback_data="cb_no"), 
               InlineKeyboardButton("4", callback_data="cb_no")) 
    markup.row_width = 8 
    markup.add(InlineKeyboardButton("Следующий вопрос", callback_data="cb_yes")) 
    return markup
...
```

Также пропишем валидный ответ от бота по каждой категории (незабываем про markdown разметку):
```python
message_result_1 = '''Раздел:ЭСК 
*Вопросы от SOC* 
Правильных ответов 10 из 10 
Ты такой молодец! Набрал прекрасное количество баллов. 
Покажи этот экран одному из организаторов на стойке и выбирай подарок.'''
...
```

Пропишем функцию ответа бота при нажатии на кнопки:
```python
def callback_query(call):  
    if call.data == "rez_1": 
        bot.send_message(CHAT_ID, message_result_1, parse_mode='markdown') 
        bot.send_message(CHAT_ID, message_main_menu, reply_markup=gen_markup_menu())
...
```

Не стоит забывать про ответа бота при нажатии на другие кнопки:

![img](https://github.com/eeenvik1/shmel-site/blob/main/content/posts/farm_merch_phd.assets/Screenshot_5.png)

```python
elif call.data == "error_1": 
        bot.send_message(CHAT_ID, "Ты что, хакер?")
```

Как итог мы имеем аналогичного бота с "правильно решенными" вопросами:

![img](https://github.com/eeenvik1/shmel-site/blob/main/content/posts/farm_merch_phd.assets/Screenshot_6.png)

P.S. автор честно прорешал 3 категории на 10 баллов (правда с разных аккаунтов) и получил только 3 приза.
