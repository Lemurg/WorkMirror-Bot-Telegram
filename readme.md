# ﻿**WorkMirror Bot**

## **WorkMirror Bot** — это Telegram-бот для опросов и анализа отзывов сотрудников. Он позволяет:

1. **Добавлять вопросы** (в т.ч. разных типов: да/нет, числовые, открытый текст),
1. **Отвечать** на них,
1. **Анализировать** результаты с помощью Yandex GPT,
1. **Выгружать** результаты анализа в PDF/DOCX/TXT.

## **1. Краткое описание**

- **Язык:** Python 3.
- **Библиотеки:**
  - [python-telegram-bot](https://python-telegram-bot.org/) для работы с Telegram API,
  - [SQLAlchemy](https://www.sqlalchemy.org/) для ORM и работы с БД,
  - [ReportLab](https://www.reportlab.com/) и [python-docx](https://python-docx.readthedocs.io/en/latest/) для генерации файлов,
  - [bcrypt](https://pypi.org/project/bcrypt/) для хеширования кодов доступа,
  - [requests](https://pypi.org/project/requests/) для HTTP-запросов к Yandex GPT,
  - [dotenv](https://pypi.org/project/python-dotenv/) для подгрузки переменных окружения.
- **База данных:** SQLite (файл WorkMirror\_bot.db по умолчанию).
-----
## **2. Требования**

- **Python 3.9+** (желательно, но может работать и на 3.7+).
- Установленные библиотеки из **requirements.txt.**

**Примечание.** Версии библиотек указываются в вашем requirements.txt, который вы генерируете с помощью pip freeze > requirements.txt.

-----
## **3. Установка и запуск**

### **3.1 Клонирование репозитория**

bash

git clone https://github.com/ваш\_аккаунт/WorkMirror\_bot.git

cd WorkMirror\_bot

### **3.2 Создание виртуального окружения (рекомендуется)**

bash

python -m venv venv

source venv/bin/activate  # Linux/Mac

venv\Scripts\activate     # Windows

### **3.3 Установка зависимостей**

bash

pip install -r requirements.txt

### **3.4 Настройка переменных окружения**

В корне проекта должен лежать файл **.env** со следующими переменными:

dotenv

TELEGRAM\_BOT\_TOKEN=<Ваш\_телеграм\_токен>

YANDEX\_OAUTH\_TOKEN=<Ваш\_Yandex\_OAuth\_токен>

YANDEX\_FOLDER\_ID=<Ваш\_FolderID\_от\_Yandex>

- **TELEGRAM\_BOT\_TOKEN** — токен Telegram-бота (можно получить у @BotFather).
- **YANDEX\_OAUTH\_TOKEN** — OAuth-токен, используемый для получения IAM-токена в Яндекс.
- **YANDEX\_FOLDER\_ID** — идентификатор каталога в Яндекс.Cloud.

### **3.5 Запуск бота**

python -m bot.main

Если всё в порядке, бот запустится и начнёт опрашивать Telegram API. В консоли будет отображаться информация о запуске.

-----
## **4. Структура проекта**

Проект разделён на пакеты, чтобы упростить поддержку и масштабирование.

markdown

WorkMirror\_bot/

├─ bot/

│   ├─ main.py

│   ├─ handlers.py

│   ├─ callbacks.py

│   ├─ conversation.py

│   └─ states.py

├─ database/

│   ├─ db.py

│   └─ models.py

├─ services/

│   ├─ file\_generator.py

│   ├─ gpt\_service.py

│   └─ validators.py

├─ config.py

├─ .env

├─ requirements.txt

└─ README.md 

### **4.1 config.py**

- Содержит **константы** и **переменные окружения** (читаются через dotenv).
- Пример:

  import os

  from dotenv import load\_dotenv

  load\_dotenv()

  TELEGRAM\_BOT\_TOKEN = os.getenv("TELEGRAM\_BOT\_TOKEN")

  YANDEX\_OAUTH\_TOKEN = os.getenv("YANDEX\_OAUTH\_TOKEN")

  YANDEX\_FOLDER\_ID   = os.getenv("YANDEX\_FOLDER\_ID")

  DB\_PATH = "sqlite:///WorkMirror\_bot.db"

  MAX\_TRIES = 3

  YES\_NO\_VALID = ["да", "нет", "yes", "no"]

  DEFAULT\_MAX\_LENGTH = 9999

  YANDEX\_GPT\_API\_ENDPOINT = "https://llm.api.cloud.yandex.net/foundationModels/v1/completion"

  YANDEX\_GPT\_TEMPERATURE = 0.8

  YANDEX\_GPT\_MAX\_TOKENS  = 1000

### **4.2 Папка database/**

1. **models.py** — объявление моделей SQLAlchemy: Company, AccessCode, Question, Answer, AnalysisLog.
1. **db.py** — создание движка (engine), инициализация таблиц, SessionLocal.

### **4.3 Папка services/**

- **gpt\_service.py** — функции для работы с Яндекс GPT (get\_iam\_token, request\_yandex\_gpt).
- **file\_generator.py** — генерация файлов PDF/DOCX/TXT (generate\_file).
- **validators.py** — функции валидации ответов, проверки кода доступа, JSON-констрейнтов.

### **4.4 Папка bot/**

- **main.py** — точка входа, где создаётся Application, регистрируется ConversationHandler и вызывается run\_polling().
- **conversation.py** — конфигурация ConversationHandler, привязка состояний к хендлерам.
- **handlers.py** — функции обработки текстовых сообщений (например, /start, ввод кода компании, вопросы и т.д.).
- **callbacks.py** — обработка inline-кнопок (CallbackQueryHandler).
- **states.py** — набор констант состояний, чтобы избежать циклических импортов.
-----
## **5. Основные сущности (ORM-модели)**

### **5.1 Company**

    class Company(Base):
    `    `\_\_tablename\_\_ = "companies"
    
    `    `id = Column(Integer, primary\_key=True)
    
    `    `company\_code = Column(String, unique=True, nullable=False)
    
    `    `company\_name = Column(String, nullable=True)
    
    `    `questions = relationship("Question", back\_populates="company")
    
    `    `access\_codes = relationship("AccessCode", back\_populates="company")

- **company\_code** — уникальный код компании, вводимый пользователем.
- **company\_name** — название компании (необязательно).

### **5.2 AccessCode**

    class AccessCode(Base):
    
    `    `\_\_tablename\_\_ = "access\_codes"
    
    `    `id = Column(Integer, primary\_key=True)
    
    `    `company\_id = Column(Integer, ForeignKey("companies.id"), nullable=False)
    
    `    `access\_code = Column(String, nullable=False)  # хранится в хешированном виде

- **access\_code** — код доступа, **захешированный** через bcrypt.

### **5.3 Question**

    class Question(Base):
    
    `    `\_\_tablename\_\_ = "questions"
    
    `    `id = Column(Integer, primary\_key=True)
    
    `    `company\_id = Column(Integer, ForeignKey("companies.id"), nullable=False)
    
    `    `question\_text = Column(Text, nullable=False)
    
    `    `question\_type = Column(String, nullable=True)
    
    `    `constraints   = Column(Text, nullable=True)  # JSON
    
    `    `company = relationship("Company", back\_populates="questions")

### **5.4 Answer**

    class Answer(Base):
    
    `    `\_\_tablename\_\_ = "answers"
    
    `    `id = Column(Integer, primary\_key=True)
    
    `    `company\_id = Column(Integer, ForeignKey("companies.id"), nullable=False)
    
    `    `question\_id = Column(Integer, ForeignKey("questions.id"), nullable=False)
    
    `    `user\_id = Column(String, nullable=True)
    
    `    `answer\_text = Column(Text, nullable=False)
    
    `    `created\_at = Column(DateTime, default=datetime.datetime.now)

### **5.5 AnalysisLog**

    class AnalysisLog(Base):
    
    `    `\_\_tablename\_\_ = "analysis\_logs"
    
    `    `id = Column(Integer, primary\_key=True)
    
    `    `company\_id = Column(Integer, ForeignKey("companies.id"), nullable=False)
    
    `    `created\_at = Column(DateTime, default=datetime.datetime.now)
    
    `    `analysis\_result = Column(Text, nullable=True)  # JSON-ответ от YandexGPT

-----
## **6. Основные процессы**

### **6.1 Сценарий «Ответить на вопросы»**

1. Пользователь выбирает «Ответить на вопросы».
1. Бот запрашивает **код компании** (company\_code).
1. Если компания найдена в БД — бот начинает задавать вопросы, сохраняет ответы (Answer).

### **6.2 Сценарий «Результаты»**

1. Пользователь выбирает «Результаты».
1. Бот просит ввести **код компании**, затем **код доступа**.
1. Если проверка прошла, бот собирает все ответы из БД, формирует строку отзывов и отправляет в **request\_yandex\_gpt**.
1. Отображается результат анализа, предлагается выгрузка в документ (PDF/DOCX/TXT).

### **6.3 Сценарий «Управление вопросами»**

1. Пользователь выбирает «Управление вопросами».
1. Вводит код компании, код доступа (хеш в AccessCode).
1. Меню добавления/удаления/списка вопросов.
   1. **Добавить вопрос**: бот спрашивает тип вопроса (да/нет, numeric, открытый текст), констрейнты (min/max, длину), текст вопроса.
   1. **Удалить**: бот выводит список, пользователь выбирает номер вопроса, идёт удаление.
-----
## **7. Взаимодействие с Yandex GPT**

1. **Получение IAM-токена** (get\_iam\_token):

       def get\_iam\_token():
    
       `    `response = requests.post(
    
       `        `"https://iam.api.cloud.yandex.net/iam/v1/tokens",
    
       `        `json={"yandexPassportOauthToken": YANDEX\_OAUTH\_TOKEN}
    
       `    `)
    
       `    `response.raise\_for\_status()
    
       `    `return response.json()["iamToken"]

1. **Отправка запроса** (request\_yandex\_gpt):

       def request\_yandex\_gpt(user\_text: str) -> dict:
    
       `    `token = get\_iam\_token()
    
       `    `headers = {"Authorization": f"Bearer {token}", ...}
    
       `    `data = {...}  # Формируем payload
    
       `    `response = requests.post(YANDEX\_GPT\_API\_ENDPOINT, headers=headers, json=data)
    
       `    `return response.json()

1. **Интеграция:** при вводе «Результаты» бот формирует общий текст отзывов, отправляет их в Я.GPT и получает «анализ», который выводит пользователю.
-----
## **8. Генерация файлов PDF/DOCX/TXT**

Модуль file\_generator.py обеспечивает функцию generate\_file(text, fmt):

- При **fmt="pdf"**:
  - Использует **ReportLab** (reportlab.pdfbase, SimpleDocTemplate и т.д.).
  - Важно наличие шрифта (например, DejaVuSans.ttf) и правильный путь к нему.
- При **fmt="docx"**:
  - Использует **python-docx**.
- При **fmt="txt"**:
  - Создаёт обычный текстовый файл.

После генерации бот отправляет полученный файл пользователю.

-----
## **9. Настройка шрифта для PDF**

ReportLab требует наличия шрифта, например, **DejaVuSans.ttf**. Поместите его в папку services/ или другую директорию и укажите верный путь:

font\_path = os.path.join(os.path.dirname(\_\_file\_\_), "DejaVuSans.ttf")

pdfmetrics.registerFont(TTFont("DejaVuSans", font\_path))

-----
## **10. Стартовые данные и добавление компаний**

- Чтобы бот нашёл **код компании**, в таблице companies должна быть соответствующая запись.
- Если в базе нет записей, добавьте тестовую компанию вручную или через скрипт:

from database.db import SessionLocal

from database.models import Company

session = SessionLocal()

new\_company = Company(company\_code="test123", company\_name="Test Company")

session.add(new\_company)

session.commit()

session.close()

Также добавьте AccessCode, предварительно захешируйте код c помощью: 

import bcrypt

plain\_code = "123"

hashed = bcrypt.hashpw(plain\_code.encode('utf-8'), bcrypt.gensalt())

hashed\_str = hashed.decode('utf-8')

print(hashed\_str)bcrypt.hashpw).

-----
## **11. Запуск и использование**

1. **Запустить бота:** python -m bot.main.
1. **В Telegram** открыть ваш бот по @username и отправить /start.
1. Выбрать одно из меню:
   1. ` `«Ответить на вопросы» — ввод кода компании -> отвечаем на вопросы.
   1. «Результаты» — ввод кода компании -> ввод access code -> анализ от Я.GPT.
   1. «Управление вопросами» — код компании -> access code -> меню добавления/удаления.
-----
## **12. Тестирование и отладка**

- **Логи**: основной вывод происходит в консоль. Если бот молчит, проверьте, нет ли ошибок при запуске.
- **Дублирование кодов**: если company\_code уже есть в БД и поле unique=True, при попытке добавить новую компанию с тем же кодом выбросится ошибка.
- **Установка логгера**: для более детальной отладки можно настроить logging в Python (например, logging.basicConfig(level=logging.INFO)).
-----
## **13. Часто встречающиеся проблемы**

1. **Компания не найдена**
   1. Убедитесь, что компания есть в БД.
   1. Проверьте ввод пользователя: лишние пробелы, регистр символов.
   1. По возможности используйте func.lower(Company.company\_code) == company\_code.lower().
1. **Неверный код доступа**
   1. Проверяйте, что в access\_codes хранится **захешированное** значение.
   1. При проверке используйте bcrypt.checkpw.
1. **Не генерируется PDF**
   1. Убедитесь, что у вас есть файл шрифта.
   1. Проверьте путь (os.path.exists(font\_path)).
   1. При необходимости используйте системный путь (например, C:/Windows/Fonts/DejaVuSans.ttf).
1. **Ошибка при анализе (Yandex GPT)**
   1. Проверьте валидность YANDEX\_OAUTH\_TOKEN.
   1. Убедитесь, что проект Яндекс.Cloud активен и YANDEX\_FOLDER\_ID верен.
   1. Убедитесь, что в запросе modelUri корректен: f"gpt://{YANDEX\_FOLDER\_ID}/yandexgpt".
-----
## **14. Расширение и кастомизация**

- **Дополнительные типы вопросов**: вы можете расширить функцию validate\_answer или добавить новые поля в Question.
- **Дополнительная логика анализа**: например, обрабатывать результаты Yandex GPT, выделять ключевые слова и т.д.
- **Поддержка нескольких ботов**: добавьте дополнительные переменные окружения или конфигурации.
-----
## **15. Заключение**

**WorkMirror Bot** упрощает сбор и анализ отзывов. Архитектура бота (с разделением на модули bot/, services/, database/, config.py) облегчает сопровождение и дальнейшее развитие. При возникновении вопросов или проблем смотрите логи, проверяйте корректность БД и путей к шрифтам.

Если у вас есть идеи по улучшению, обращайтесь по контактным данным.

-----
## **Контакты**

- **Разработчик**: Цепелев Кирилл (Telegram: @feuillenoire)
- **Разработчик:** Головко Александр (Telegram: @g0_al3)

Спасибо за использование WorkMirror Bot!

