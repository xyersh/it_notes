## Что это такое?
**golangci-lint** — это быстрый и эффективный агрегатор линтеров для языка **Go**. . Вместо того, чтобы устанавливать и запускать множество линтеров по отдельности, **golanci-lint** объединяет их в один инструмент, который может выполнять все проверки параллельно. Это значительно экономит время и ресурсы.

## Документация
[https://golangci-lint.run](https://golangci-lint.run)

## Установка
```bash
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
go install github.com/golangci/golangci-lint/v2/cmd/golangci-lint@latest

```
Это установит **go**lanci-lint в ваш `GOPATH/bin` или `GOBIN`. Убедитесь, что эта папка включена в ваш `$PATH`.
Проверка установки:
```bash
golangci-lint --version
```

## Настройка линтера в IDE VS Code

- **Установка расширения Go для VS Code**
	 Если у вас ещё не установлено расширение **Go**, откройте **VS Code**, перейдите в раздел "Extensions" (Расширения) (Ctrl+Shift+X) и найдите **Go**. . Установите его. Это расширение предоставляет все необходимые инструменты для работы с **Go**, включая поддержку линтеров.

 - **Настройка VS Code**
	  После установки расширения вам нужно настроить его для использования **golangci-lint**.

	1. Открываем **VS Code** и перейдите в настройки (File -> Preferences -> Settings или `Ctrl+,`).
	2. Найдем **Go**-настройки, введя **`go`** в строку поиска.
	3. Найдём параметр **`Go: Lint Tool`** и выбираем **`golangci-lint`** из выпадающего списка.
	4. Параметр `Go: Lint on Save` - устанавливаем значение **`workspace`** или **`file`**. Это определит, когда линтер будет запускаться. Рекомендуется использовать **`file`** для быстрой проверки только текущего файла при сохранении.
	5. Если **golangci-lint** не работает сразу, возможно,  придётся указать путь к исполняемому файлу. Для этого найдите параметр **`Go: Lint Flags`** и добавьте флаг **`--path <путь к golangci-lint>`**.
	6. Сохраните настройки. После этого **VS Code** будет автоматически запускать **golangci-lint**   при сохранении файлов.
	
- **Создание файла конфигурации**

Чтобы настроить, какие именно линтеры использовать и как их запускать, нужно создать файл **`.golangci.yml`** в корне вашего проекта. Это позволяет выбрать, какие проверки **golangci-lint** должен проводить. Например, следующий файл включает линтеры **`errcheck`** и `govet`

##  Точки запуска линтера в VS CODE

- **При сохранении файла**: Это самое распространённое событие. Когда вы сохраняете файл (обычно с помощью `Ctrl+S`), **VS Code** запускает линтер, чтобы проверить код на ошибки и предупреждения. Это позволяет быстро получить обратную связь.
    
- **При открытии файла**: Иногда линтер запускается при первом открытии файла в редакторе, чтобы показать существующие проблемы, не дожидаясь, пока вы начнёте его редактировать.
    
- **В фоновом режиме**: Некоторые расширения могут запускать линтер в фоновом режиме, когда вы печатаете, что обеспечивает проверку в реальном времени. Однако это может нагружать систему, поэтому чаще всего используется проверка по событию "сохранения".
    
- **По команде**: Вы также можете запустить линтер вручную, используя специальную команду, предоставляемую расширением.


## Основные команды CLI
После создания файла конфигурации вы можете запустить **golanci-lint** из командной строки в корне вашего проекта.
- Версия установленного golangci
```bash
  golangci-lint version
```
- Для проверки всех файлов текущего проекта:
```bash
golangci-lint run
```
- Для проверки конкретного файла:
```bash
golangci-lint run ./path/to/your_file.go
```
- Для исправления некоторых проблем автоматически (например, форматирование или опечатки):
```bash
golangci-lint run --fix
```
- Показать список всех доступных линтеров (включенных и выключенных).
```bash
golangci-lint linters
```
Признаки линтеров:
- [fast] - анализатор **не требует компиляции всего проекта** и работает только с исходным кодом файла (или пакета)
- [auto-fix] — поддерживает ли автоматическое исправление.
- Более подробная справка по списку доступных линтеров
```bash
golangci-lint help linters
```
- Проверка всех пакетов рекурсивно
```bash
golangci-lint run ./...
```
- очистка кэша (полезно, если линтер ведет себя странно).
```bash
golangci-lint cache clean
```
- Проверить конфиг (.golangci.yml) на ошибки 
```bash
golangci-lint config verify
```
## Конфигурация работы линтера (.golangci.yml)
Для конфигурации работы golangci-lint в рамках определенного проекта используется файл **.golangci.yml** , находящийся в корне проекта

#### Общая структура файла
```yml
# 1. Настройки запуска (run)
run:
  ...

# 2. Выбор и настройка линтеров (linters)
linters:
  ...

# 3. Детальные настройки конкретных линтеров (linters-settings)
linters-settings:
  ...

# 4. Фильтрация и обработка проблем (issues)
issues:
  ...

# 5. Настройки вывода (output)
output:
  ...

# 6. Расширения (extensions)
extensions:
  ...
```

#### run
Определяет общие параметры работы линтера: таймауты, пути для анализа, игнорируемые директории.

```yml
run:
  # Timeout for total work, e.g. 30s, 5m, 5m30s.
  # If the value is lower or equal to 0, the timeout is disabled.
  # Default: 0 (disabled)
  timeout: 5m
  # The mode used to evaluate relative paths.
  # It's used by exclusions, Go plugins, and some linters.
  # The value can be:
  # - `gomod`: the paths will be relative to the directory of the `go.mod` file.
  # - `gitroot`: the paths will be relative to the git root (the parent directory of `.git`).
  # - `cfg`: the paths will be relative to the configuration file.
  # - `wd` (NOT recommended): the paths will be relative to the place where golangci-lint is run.
  # Default: cfg
  relative-path-mode: gomod
  # Exit code when at least one issue was found.
  # Default: 1
  issues-exit-code: 2
  # Include test files or not.
  # Default: true
  tests: false
  # List of build tags, all linters use it.
  # Default: []
  build-tags:
    - mytag
  # If set, we pass it to "go list -mod={option}". From "go help modules":
  # If invoked with -mod=readonly, the go command is disallowed from the implicit
  # automatic updating of go.mod described above. Instead, it fails when any changes
  # to go.mod are needed. This setting is most useful to check that go.mod does
  # not need updates, such as in a continuous integration and testing system.
  # If invoked with -mod=vendor, the go command assumes that the vendor
  # directory holds the correct copies of dependencies and ignores
  # the dependency descriptions in go.mod.
  #
  # Allowed values: readonly|vendor|mod
  # Default: ""
  modules-download-mode: readonly
  # Uses version control information during the loading of packages.
  # Default: false (implies `-buildvcs=false`)
  enable-build-vcs: true
  # Allow multiple parallel golangci-lint instances running.
  # If false, golangci-lint acquires file lock on start.
  # Default: false
  allow-parallel-runners: true
  # Allow multiple golangci-lint instances running, but serialize them around a lock.
  # If false, golangci-lint exits with an error if it fails to acquire file lock on start.
  # Default: false
  allow-serial-runners: true
  # Define the Go version limit.
  # Default: use Go version from the go.mod file, fallback on the env var `GOVERSION`, fallback on 1.22.
  go: '1.23'
  # Number of operating system threads (`GOMAXPROCS`) that can execute golangci-lint simultaneously.
  # Default: 0 (automatically set to match Linux container CPU quota and
  # fall back to the number of logical CPUs in the machine)
  concurrency: 4
```


#### formatters
Раздел для конфигурирования форматтеров
```yml
formatters:
  # Enable specific formatter.
  # Default: [] (uses standard Go formatting)
  enable:
    - gci
    - gofmt
    - gofumpt
    - goimports
    - golines
    - swaggo
  # Formatters settings.
  settings:
    # See the dedicated "formatters.settings" documentation section.
    option: value
  exclusions:
    # Log a warning if an exclusion path is unused.
    # Default: false
    warn-unused: true
    # Mode of the generated files analysis.
    #
    # - `strict`: sources are excluded by strictly following the Go generated file convention.
    #    Source files that have lines matching only the following regular expression will be excluded: `^// Code generated .* DO NOT EDIT\.$`
    #    This line must appear before the first non-comment, non-blank text in the file.
    #    https://go.dev/s/generatedcode
    # - `lax`: sources are excluded if they contain lines like `autogenerated file`, `code generated`, `do not edit`, etc.
    # - `disable`: disable the generated files exclusion.
    #
    # Default: lax
    generated: strict
    # Which file paths to exclude.
    # This option is ignored when using `--stdin` as the path is unknown.
    # Default: []
    paths:
      - ".*\\.my\\.go$"
      - lib/bad.go
```
#### linters
Управляет списком включенных и выключенных линтеров.
```yml
linters:
  # Определяет дефолтный сет линтеров
  # default: none - Отключает все линтеры по умолчанию
  # default: all - Включает все линтеры по умолчанию
  # default: standart - https://golangci-lint.run/docs/linters/#enabled-by-default
  # default: fast - включает только fast-линтеры по умолчанию
  default: none
  
  # приведены правила для исключания файлов из  проверки 
  exclusions:
    # пути к исключаемым директориям И файлам
    paths:
      - src/external_libs
      - lib/bad.go
    # пути к НЕ исключаемым директориям И файлам  
    paths-except:
      - lib/bad.go  
        
	# Устанавливает режимы анализа для сгенерированных файлов
    # generated: lax - файлы исключаются из проверки если содержат `autogenerated file`, `code generated`, `do not edit`
    # generated: disable: отменяет исключение из проверки сгенерированных файлов
    # generated: strict - исключаются файлы которые мэтчатся с реджекспой вида: "^// Code generated .* DO NOT EDIT\.$"
    generated: lax
  
    # Предопреденные правила для исключения проверки
    presets:
      - comments
      - std-error-handling
      - common-false-positives
      - legacy
        

    # Excluding configuration per-path, per-linter, per-text and per-source.    
    rules:
      # Exclude some linters from running on tests files.
      - path: _test\.go
        linters:
          - gocyclo
          - errcheck
          - dupl
          - gosec
      # Run some linter only for test files by excluding its issues for everything else.
      - path-except: _test\.go
        linters:
          - forbidigo
      # Exclude known linters from partially hard-vendored code,
      # which is impossible to exclude via `nolint` comments.
      # `/` will be replaced by the current OS file path separator to properly work on Windows.
      - path: internal/hmac/
        text: "weak cryptographic primitive"
        linters:
          - gosec
      # Exclude some `staticcheck` messages.
      - linters:
          - staticcheck
        text: "SA9003:"
      # Exclude `lll` issues for long lines with `go:generate`.
      - linters:
          - lll
        source: "^//go:generate "        
 
  # Принудительно выключить (приоритет над enable)
  disable:
    - arangolint
    - asasalint
    - asciicheck
    - bidichk
    - bodyclose    
  # Включить конкретные линтеры
  enable:
    - errcheck
    - gosimple
    - govet
    - ineffassign
    - staticcheck
    - unused
    - gofmt
    - goimports
    - revive
    - misspell
    - gocyclo
    - gosec
  
  # Указывает правила для конкретных линтеров точечно. См. документацию
  settings:
  
```


#### issues
	Управляет тем, какие ошибки показывать, а какие скрывать.
```yml
issues:
  # Maximum issues count per one linter.
  # Set to 0 to disable.
  # Default: 50
  max-issues-per-linter: 0
  # Maximum count of issues with the same text.
  # Set to 0 to disable.
  # Default: 3
  max-same-issues: 0
  # Make issues output unique by line.
  # Default: true
  uniq-by-line: false
  # Show only new issues: if there are unstaged changes or untracked files,
  # only those changes are analyzed, else only changes in HEAD~ are analyzed.
  # It's a super-useful option for integration of golangci-lint into existing large codebase.
  # It's not practical to fix all existing issues at the moment of integration:
  # much better don't allow issues in new code.
  #
  # Default: false
  new: true
  # Show only new issues created after the best common ancestor (merge-base against HEAD).
  # Default: ""
  new-from-merge-base: main
  # Show only new issues created after git revision `REV`.
  # Default: ""
  new-from-rev: HEAD
  # Show only new issues created in git patch with set file path.
  # Default: ""
  new-from-patch: path/to/patch/file
  # Show issues in any part of update files (requires new-from-rev or new-from-patch).
  # Default: false
  whole-files: true
  # Apply the fixes detected by the linters and formatters (if it's supported by the linter).
  # Default: false
  fix: true
```

#### output
```yml
output:
  # The formats used to render issues.
  formats:
    # Prints issues in a text format with colors, line number, and linter name.
    # This format is the default format.
    text:
      # Output path can be either `stdout`, `stderr` or path to the file to write to.
      # Default: stdout
      path: ./path/to/output.txt
      # Print linter name in the end of issue text.
      # Default: true
      print-linter-name: false
      # Print lines of code with issue.
      # Default: true
      print-issued-lines: false
      # Use colors.
      # Default: true
      colors: false
    # Prints issues in a JSON representation.
    json:
      # Output path can be either `stdout`, `stderr` or path to the file to write to.
      # Default: stdout
      path: ./path/to/output.json
    # Prints issues in columns representation separated by tabulations.
    tab:
      # Output path can be either `stdout`, `stderr` or path to the file to write to.
      # Default: stdout
      path: ./path/to/output.txt
      # Print linter name in the end of issue text.
      # Default: true
      print-linter-name: true
      # Use colors.
      # Default: true
      colors: false
    # Prints issues in an HTML page.
    # It uses the Cloudflare CDN (cdnjs) and React.
    html:
      # Output path can be either `stdout`, `stderr` or path to the file to write to.
      # Default: stdout
      path: ./path/to/output.html
    # Prints issues in the Checkstyle format.
    checkstyle:
      # Output path can be either `stdout`, `stderr` or path to the file to write to.
      # Default: stdout
      path: ./path/to/output.xml
    # Prints issues in the Code Climate format.
    code-climate:
      # Output path can be either `stdout`, `stderr` or path to the file to write to.
      # Default: stdout
      path: ./path/to/output.json
    # Prints issues in the JUnit XML format.
    junit-xml:
      # Output path can be either `stdout`, `stderr` or path to the file to write to.
      # Default: stdout
      path: ./path/to/output.xml
      # Support extra JUnit XML fields.
      # Default: false
      extended: true
    # Prints issues in the TeamCity format.
    teamcity:
      # Output path can be either `stdout`, `stderr` or path to the file to write to.
      # Default: stdout
      path: ./path/to/output.txt
    # Prints issues in the SARIF format.
    sarif:
      # Output path can be either `stdout`, `stderr` or path to the file to write to.
      # Default: stdout
      path: ./path/to/output.json
  # Add a prefix to the output file references.
  # This option is ignored when using `output.path-mode: abs` mode.
  # Default: ""
  path-prefix: ""
  # By default, the report are related to the path obtained by `run.relative-path-mode`.
  # The mode `abs` allows to show absolute file paths instead of relative file paths.
  # The option `output.path-prefix` is ignored when using `abs` mode.
  # Default: ""
  path-mode: "abs"
  # Order to use when sorting results.
  # Possible values: `file`, `linter`, and `severity`.
  #
  # If the severity values are inside the following list, they are ordered in this order:
  #   1. error
  #   2. warning
  #   3. high
  #   4. medium
  #   5. low
  # Either they are sorted alphabetically.
  #
  # Default: ["linter", "file"]
  sort-order:
    - linter
    - severity
    - file # filepath, line, and column.
  # Show statistics per linter.
  # Default: true
  show-stats: false
```

#### severity
```yml
run:
  # Timeout for total work, e.g. 30s, 5m, 5m30s.
  # If the value is lower or equal to 0, the timeout is disabled.
  # Default: 0 (disabled)
  timeout: 5m
  # The mode used to evaluate relative paths.
  # It's used by exclusions, Go plugins, and some linters.
  # The value can be:
  # - `gomod`: the paths will be relative to the directory of the `go.mod` file.
  # - `gitroot`: the paths will be relative to the git root (the parent directory of `.git`).
  # - `cfg`: the paths will be relative to the configuration file.
  # - `wd` (NOT recommended): the paths will be relative to the place where golangci-lint is run.
  # Default: cfg
  relative-path-mode: gomod
  # Exit code when at least one issue was found.
  # Default: 1
  issues-exit-code: 2
  # Include test files or not.
  # Default: true
  tests: false
  # List of build tags, all linters use it.
  # Default: []
  build-tags:
    - mytag
  # If set, we pass it to "go list -mod={option}". From "go help modules":
  # If invoked with -mod=readonly, the go command is disallowed from the implicit
  # automatic updating of go.mod described above. Instead, it fails when any changes
  # to go.mod are needed. This setting is most useful to check that go.mod does
  # not need updates, such as in a continuous integration and testing system.
  # If invoked with -mod=vendor, the go command assumes that the vendor
  # directory holds the correct copies of dependencies and ignores
  # the dependency descriptions in go.mod.
  #
  # Allowed values: readonly|vendor|mod
  # Default: ""
  modules-download-mode: readonly
  # Uses version control information during the loading of packages.
  # Default: false (implies `-buildvcs=false`)
  enable-build-vcs: true
  # Allow multiple parallel golangci-lint instances running.
  # If false, golangci-lint acquires file lock on start.
  # Default: false
  allow-parallel-runners: true
  # Allow multiple golangci-lint instances running, but serialize them around a lock.
  # If false, golangci-lint exits with an error if it fails to acquire file lock on start.
  # Default: false
  allow-serial-runners: true
  # Define the Go version limit.
  # Default: use Go version from the go.mod file, fallback on the env var `GOVERSION`, fallback on 1.22.
  go: '1.23'
  # Number of operating system threads (`GOMAXPROCS`) that can execute golangci-lint simultaneously.
  # Default: 0 (automatically set to match Linux container CPU quota and
  # fall back to the number of logical CPUs in the machine)
  concurrency: 4
```
#### Полный пример конфигурации
готовый шаблон для современного Go-проекта:
```yml
run:
  timeout: 5m
  go: '1.21'
  tests: true


linters:

  enable:
    # Базовые
    - errcheck
    - gosimple
    - govet
    - ineffassign
    - staticcheck
    - unused
    # Стиль
    - gofmt
    - goimports
    - revive
    - misspell
    - whitespace
    # Сложность
    - gocyclo
    - funlen
    # Безопасность
    - gosec

linters-settings:
  govet:
    check-shadowing: true
  gocyclo:
    min-complexity: 15
  funlen:
    lines: 100
    statements: 50
  goimports:
    local-prefixes: github.com/myorg/myproject
  misspell:
    locale: US
  revive:
    confidence: 0.8
  gosec:
    excludes:
      - G101

issues:
  max-issues-per-linter: 0
  max-same-issues: 0
  exclude-rules:
    - path: _test\.go
      linters:
        - errcheck
        - gocyclo
    - path: internal/mocks/
      linters:
        - all

output:
  format: colored-line-number
  print-issued-lines: true
  print-linter-name: true
  sort-results: true
  show-stats: true
```

- **`run`**: Позволяет настроить общие параметры выполнения, такие как таймаут.
- **`issues`**: Здесь вы можете настроить, какие проблемы исключать. Например, `exclude-dirs` позволяет игнорировать определённые папки.
- **`linters-settings`**: Позволяет тонко настроить каждый линтер. Например, для **`govet`** можно включить проверку на теневые переменные (`check-shadowing`).
- **`linters`**: Самый важный раздел, где вы включаете или отключаете конкретные линтеры. Полный список доступен на [официальном сайте **go**lanci-lint](https://golangci-lint.run/usage/linters/).

