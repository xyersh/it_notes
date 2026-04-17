
## Проверка установленного ssh-клиента
Windows 10 включает **нативный OpenSSH-клиент** (с версии 1809), поэтому работать с ключами можно прямо в PowerShell или CMD — без сторонних программ.
```powershell
# Проверка версии клиента
ssh -V

# Если ошибка "ssh is not recognized" — установите OpenSSH:
# Настройки → Приложения → Дополнительные компоненты → Добавить компонент → "OpenSSH Client"
# Или через PowerShell (от администратора):
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```


## Генерация SSH-ключа в Windows 10
-  Открыть PowerShell или CMD
-  Создайте пару ключей (рекомендуется Ed25519)
```powershell
ssh-keygen -t ed25519 -C "your_email@example.com"
```
-  Указать путь и пароль (по желанию)
```powershell
# Пример вывода:
Generating public/private ed25519 key pair.
Enter file in which to save the key (C:\Users\YourName\.ssh\id_ed25519): [Enter]
Enter passphrase (empty for no passphrase): [введите надёжный пароль или оставьте пустым]
Enter same passphrase again: [повторите]
```
- Ключи сохранятся в (Путь `~` в Windows = `C:\Users\ВашеИмя\`): 
	- ~/.ssh/id_ed25519      # приватный (никому не показывать!)
	- ~/.ssh/id_ed25519.pub  # публичный (загружать на GitHub)

## Запуск и настройка ssh-agent в Windows
Windows имеет встроенную службу `ssh-agent`, но по умолчанию она **отключена**.
```powershell
# Запустите от имени Администратора!

# Включите автозапуск службы
Set-Service -Name ssh-agent -StartupType Automatic

# Запустите службу
Start-Service ssh-agent

# Добавьте ключ (уже можно от обычного пользователя)
ssh-add ~/.ssh/id_ed25519
```


## Добавление публичного ключа на GitHub

-  Скопируйте содержимое публичного ключа
-  Добавьте ключ в GitHub:
	- GitHub → **Settings** → **SSH and GPG keys** → **New SSH key**
	- Вставьте ключ, дайте имя (например, "Windows 10 Laptop"), сохраните
-  Проверьте подключение
```powershell
ssh -T git@github.com
```
при первом подключении к любому SSH-серверу, включая `github.com` возникает стандартное предупреждение вида `# The authenticity of host can't be established....`
В этом случае: 
-	Сервер показывает свой **публичный ключ-отпечаток** (fingerprint), который можно найти на сайте гуглением
- SSH-клиент спрашивает: _"Вы уверены, что это настоящий сервер?"_ 
- После подтверждения ключ сохраняется в `~/.ssh/known_hosts`
- 
- При следующих подключениях проверка будет автоматической
```
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
                                                    ↑
                              Этот отпечаток нужно сверить с официальным
```



## Клонирование репозитория по SSH
Убедитесь, что используете **SSH-URL** (начинается с `git@github.com:`), а не HTTPS.
```powershell
git clone git@github.com:username/repository.git
```