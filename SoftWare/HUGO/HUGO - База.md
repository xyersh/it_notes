
### Установка HUGO

#### Установить Git-клиент
- **Для Windows** - скачать установщик из офф-сайта [git-scm.com](https://git-scm.com)
- **Для Linux** - команда `sudo apt update && sudo apt install git`

#### Установить HUGO
Всегда устанавливаем Extended-версию - она поддерживает обработку **SASS/SCSS** и современные методы сжатия изображений
- **Для Windows** - команда 
```powershell
winget install Hugo.Hugo.Extended
```
ИЛИ
```powershell
choco install hugo-extended
```

- **Для Linux:**
	- через snap
	```bash
	sudo snap install hugo --channel=extended
	```
	- через apt
	```bash
	sudo apt install hugo
	```

Проверка установки:
```
hugo version
```


### Бизнес-сущности и концепции HUGO

#### Content-Types
Это категории страниц. Например: `posts`, `docs`, `products`, `portfolio`. Каждый тип контента может иметь собственный шаблон отображения.
```
content/
├── posts/          ← тип контента "posts"
│   ├── my-first-post.md
│   └── second-post.md
├── docs/           ← тип контента "docs"
│   ├── getting-started.md
│   └── api-reference.md
└── products/       ← тип контента "products"
    └── widget-pro.md
```


### Команды

- `hugo new site <SITE-NAME>` - создать сайт  **`<SITE-NAME>`** в рабочей директории
- `hugo new site . --force` - создать сайт с именем родительской директории
- `hugo new <путь/к/файлу.md>` - создать файл контента на основе файла-архетипа
- `hugo server` - запускает локальный сервер для разработки с автоперезагрузкой
- `hugo server -D` - Запускает сервер, включая в отображение черновики
- `hugo` - собирает проект в папку **public** для продакшена
- 




### Структура проекта HUGO
- **archetypes** - Шаблоны для создания нового контента (front matter по умолчанию)
- **assets** - Файлы, обрабатываемые конвейером Hugo (SCSS, JS, изображения для ресайза)
- **content** - Весь ваш текстовый контент (Markdown файлы)
- **data** - Файлы данных (YAML, JSON, TOML), доступные в шаблонах
- **i18n** - используется для перевода статических строк интерфейса  на другие языки (имеется в виду не ЯП, а человечьи языки, на которых человеки общаются)
- **layouts** - HTML-шаблоны, определяющие, как выглядит сайт
- **public** - содержит файлы, готовые для продакшена
- **static** -   Статические файлы (CSS, JS, картинки), копируемые "как есть" в финальный сайт
- **themes** - # Папка для установленных тем
- **hugo.toml** - Главный конфигурационный файл сайта (может быть .yaml или .json)