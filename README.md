# menu-processing

## Задача
При помощи любой доступной Вам LLM (open-source, проприетарной) создать прототип решения, которое автоматизировало бы:

- Шаг 1. Дополнение QR-меню ресторана информацией с описанием блюд/напитков, которое будет cгенерировано LLM. По каждому блюду/напитку к уже имеющимся данным о названии и цене должен быть добавлен пункт «description», содержащий короткое описание данного блюда/напитка (не более 350 символов);

- Шаг 2. Перевод QR-меню ресторана на английский язык.

Дано: JSON-файл, содержащий меню ресторана на испанском языке. Блюда и напитки в меню разбиты по категориям.

Результат: валидный JSON-файл, содержащий меню в исходной разбивке по категориям блюд, и при этом дополненный описаниями блюд и переведенный на английский язык.


Результат необходимо сопроводить следующей информацией:
- Описание выполненной работы (какая конфигурация решения использована: LLM, библиотеки, др.);
- Текст использованных промптов;
- Cопутствующая аналитика: что получилось, что нет, какие есть пути повышения качества решения задачи.

## TL;DR
### Реализованные подоходы:
1. Baseline (Solution 1)
    - С обычными промптами сходить в chat gpt для написания описаний позиций
    - Перевести все нужные поля через переводчик
2. Улучшенная версия (Solution 2)
    - Поумнее промпты с few-shot
    - Контроль длины сгенерированного описания
    - Перевод наименования позиции и ее описания через LLM
    - Перевод разделов меню через переводчик
    - Защита от галюцинаций на неоднозначных позициях

### Конфигурация решения: 
- LLM: 4o-mini (задача несложная, в чем-то умнее потребности нет, можно использовать и более простые/OS модельки)
- библиотеки: requests, openai, pydantic-settings, tqdm (полный список можно увидеть [тут](pyproject.toml))
- разработка в jupyter ноутбуке, установка окружения через poetry

### Текст промптов
Промпты довольно большие, для каждого из решений они приведены в [ноубуке](notebook.ipynb) в разделах Solution 1 и Solution 2.

Промпт генерации описаний (Solution 2):
>    You are a helpful culinary assistant. You will be provided with the name of a dish from the menu and the name of the dish category. 
>    Your task is to respond with a concise description of the dish in Spanish. The description must align with a family restaurant style and must not exceed 350 characters in total.
>
>    If the name of the dish does not provide enough meaningful information to generate a description or is unclear, respond with: {"description": "Unpredictable"}.
>
>    Return the result in a valid JSON format: {"description": <GENERATED_DESCRIPTION>}.
>
>    Here are some examples to guide you:
>
>    Category: "Postres"<br>
>    Name: "Pastel Tres Leches"<br>
>    Description: {"description": "Un esponjoso pastel bañado en tres tipos de leche, coronado con crema batida, perfecto para los amantes de los postres dulces y cremosos."}
>
>    Category: "Entradas"<br>
>    Name: "Guacamole con Totopos"<br>
>    Description: {"description": "Guacamole fresco hecho con aguacates maduros, jugo de limón y un toque de cilantro, acompañado de crujientes totopos de maíz."}
>
>    Category: "Sopas"<br>
>    Name: "Sopa de Tortilla"<br>
>    Description: {"description": "Tradicional sopa mexicana con tiras de tortilla crujiente, aguacate, queso fresco y un toque de chile pasilla."}
>
>    Category: "A cualquier hora"<br>
>    Name: "Flambe Bambe"<br>
>    Description: {"description": "Unpredictable"}"
>
>    Category: "Ensaladas"<br>
>    Name: "Ensalada César"<br>
>    Description: {"description": "Crujientes hojas de lechuga romana con aderezo César, crotones y queso parmesano fresco."}
>
>    Category: "Tacos"<br>
>    Name: "Tacos de Barbacoa"<br>
>    Description: {"description": "Tacos suaves con carne de barbacoa cocida lentamente, acompañados de cebolla y salsa verde."}
>
>    Category: "Smoothies"<br>
>    Name: "Caribe (plátano, coco, chocolate)"<br>
>    Description: {"description": "Una mezcla exótica y dulce de plátano, coco y chocolate, perfecta para consentirte."}
>
>    Category: "Café"<br>
>    Name: "Mocha"<br>
>    Description: {"description": "Una deliciosa combinación de café espresso, chocolate y leche cremosa, perfecta para los amantes del chocolate"}
>
>    Category: "Desayunos"<br>
>    Name: "Huevos Motuleños (2 huevos fritos sobre tortilla, salsa roja, jamón y plátano frito)"<br>
>    Description: {"description": "Un clásico desayuno yucateco con huevos fritos sobre tortilla, salsa roja, jamón y un toque dulce de plátano frito."}
>
>    Category: "Café"<br>
>    Name: "Bloom"<br>
>    Description: {"description": "Unpredictable"}

Промпт перевода позиций (Solution 2):
>    You are a helpful culinary assistant. You will be provided with name, and description of a dish from a menu in Spanish. 
>    Your task is to translate the name and description into English, preserving the original meaning, style, and tone.
>
>    Return the result in a valid JSON format: {"name": <TRANSLATED_NAME>, "description": <TRANSLATED_DESCRIPTION>}.
>
>    Here are some examples to guide you:
>
>    Name: "Pastel Tres Leches"<br>
>    Description: {"description": "Un esponjoso pastel bañado en tres tipos de leche, coronado con crema batida, perfecto para los amantes de los postres dulces y cremosos."}<br>
>    Translation: {"name": "Three Milk Cake", "description": "A fluffy cake soaked in three types of milk, topped with whipped cream, perfect for lovers of sweet and creamy desserts."}
>
>    Name: "Guacamole con Totopos"<br>
>    Description: {"description": "Guacamole fresco hecho con aguacates maduros, jugo de limón y un toque de cilantro, acompañado de crujientes totopos de maíz."}<br>
>    Translation: {"name": "Guacamole with Tortilla Chips", "description": "Fresh guacamole made with ripe avocados, lime juice, and a touch of cilantro, served with crispy tortilla chips."}
>
>    Name: "Sopa de Tortilla"<br>
>    Description: {"description": "Tradicional sopa mexicana con tiras de tortilla crujiente, aguacate, queso fresco y un toque de chile pasilla."}<br>
>    Translation: {"name": "Tortilla Soup", "description": "Traditional Mexican soup with crispy tortilla strips, avocado, fresh cheese, and a hint of pasilla chili."}
>
>    Name: "Ensalada César"<br>
>    Description: {"description": "Crujientes hojas de lechuga romana con aderezo César, crotones y queso parmesano fresco."}<br>
>    Translation: {"name": "Caesar Salad", "description": "Crisp romaine lettuce with Caesar dressing, croutons, and fresh Parmesan cheese."}
>
>    Name: "Tacos de Barbacoa"<br>
>    Description: {"description": "Tacos suaves con carne de barbacoa cocida lentamente, acompañados de cebolla y salsa verde."}<br>
>    Translation: {"name": "Barbacoa Tacos", "description": "Soft tacos filled with slow-cooked barbacoa meat, served with onions and green salsa."}
>
>    Name: "Caribe (plátano, coco, chocolate)"<br>
>    Description: {"description": "Una mezcla exótica y dulce de plátano, coco y chocolate, perfecta para consentirte."}<br>
>    Translation: {"name": "Caribbean (banana, coconut, chocolate)", "description": "An exotic and sweet blend of banana, coconut, and chocolate, perfect for indulging yourself."}
>
>    Name: "Mocha"<br>
>    Description: {"description": "Una deliciosa combinación de café espresso, chocolate y leche cremosa, perfecta para los amantes del chocolate"}<br>
>    Translation: {"name": "Mocha", "description": "A delightful combination of espresso coffee, chocolate, and creamy milk, perfect for chocolate lovers."}
>
>    Name: "Huevos Motuleños (2 huevos fritos sobre tortilla, salsa roja, jamón y plátano frito)"<br>
>    Description: {"description": "Un clásico desayuno yucateco con huevos fritos sobre tortilla, salsa roja, jamón y un toque dulce de plátano frito."}<br>
>    Translation: {"name": "Motuleños Eggs (2 fried eggs on tortilla, red sauce, ham, and fried plantain)", "description": "A classic Yucatecan breakfast with fried eggs on tortilla, red sauce, ham, and a sweet touch of fried plantain."}

### Результат
Сохранил файлы для двух подходов
- [первый](data/solution_1_translation.json)
- [второй](data/solution_2_translation.json)

### Аналитика
Если кратко, few-shot промптинг и перевод наименований+описаний позиций одновременно показали себя лучше бейзлайна, получилось неплохо.

### Baseline обладал следующими недостатками:
1. **Слабая предсказуемость результатов**:
    - Различные непредсказуемые стили описания блюд.
    - Нет гарантии, что модель ответит в соответствии с заданными ограничениями (≤350 символов), так как она оперирует токенами 🍓.

2. **Галлюцинации**:  
   Если наименование блюда слишком творческое или недостаточно информативное, модель может "словить галлюцинации" и выдать некорректное описание.

3. **Проблемы перевода**:  
   При ручном отсмотре итогового файла было выявлено три однотипные ошибки.  
   При раздельном переводе названия позиции и её описания, название может быть переведено корректно в соответствующем поле, однако оригинал названия остаётся в описании (или наоборот).  
   Примеры ошибок:
   - `{"name": "Papas mix (French fries, Egg, Salad)", "description": "Mix Potatoes are a ..."}`  
   - `{"name": "Chilaquiles with Chicken", "description": "Chilaquiles con pollo are a ..."}`  
   - `{"name": "Chilaquiles with Egg", "description": "Chilaquiles con huevo are a ..."}`

---

### Для исправления недостатков во втором подходе использовано следующее:
1. **Для получения более предсказуемых результатов**:
    - Используем **few-shot** подход для генерации описаний (возможно, у ресторана семейная тематика, и владелец хочет сохранить этот стиль в описаниях блюд).
    - Добавляем программный контроль количества символов в ответе LLM.

2. **Для предотвращения галлюцинаций**:
    - Добавляем в промпт чёткие инструкции и примеры в **few-shot**, что значительно снижает вероятность ошибок.

3. **Для сохранения консистентности названий**:
    - Используем одновременный перевод всех текстовых полей позиции меню с помощью LLM, чтобы названия и описания были согласованными.
    - Перевод названий секций меню можно оставить через автоматический переводчик, так как при тестировании проблем с их переводом выявлено не было (и не должно возникнуть в будущем).
    
К недостаткам второго подхода можно отнести наличие позиций, для которых невозможно сгенерировать однозначное описание при использовании относительно безопасного подхода.

В деталях все расписано в [ноубуке](notebook.ipynb), если код не особо интересен, можно скрыть разделы Implementation

## Содержание репозитория

- [notebook](notebook.ipynb) - ноутбук с реализацией всей задачи и детальной аналитикой/промптами
- [.env.example](.env.example) - пример .env файла
- [data](data) - директория с исходными данными и результатами выполнения шагов для всех решений

## Разработка

Рекомендуемые ОС для разработки: Linux, macOS, WSL v2

### Установка окружения

- Устанавливаем `python 3.10.*`. Рекомендуемая версия: `3.11.7`
    - Рекомендуемый метод установки: pyenv

        - Устанавливаем pyenv через [официальные инструкции](https://github.com/pyenv/pyenv)

        - Устанавливаем `python 3.11.7` и указываем локальную версию в корне проекта
        ```bash
        pyenv install 3.11.7
        pyenv local 3.11.7
        ```
        - В папке проекта должен появиться файл `.python-version`, который указывает на используемую версию python


    - Windows

        Устанавливаем через [официальный установщик](https://www.python.org/downloads/)

    - Linux

        ```bash
        sudo apt install python3.10-dev
        ```

- Устанавливаем [poetry](https://python-poetry.org/docs/#installing-with-the-official-installer)
    - Linux, macOS, Windows (WSL)

        ```bash
        curl -sSL https://install.python-poetry.org | python3 -
        ```

    - Windows `powershell`

        ```powershell
        (Invoke-WebRequest -Uri https://install.python-poetry.org -UseBasicParsing).Content | py -
        ```

- Делаем так, чтобы виртуальные окружения `poetry` ставились внутрь проекта
    ```bash
    poetry config virtualenvs.in-project true
    ```

- Устанавливаем зависимости
    ```
    poetry install
    ```

- Активируем виртуальное окружение
    ```
    poetry shell
    ```
