[English](/README_en_EN.md) | [Русский](/README.md)

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="./media/logo-dark.png">
    <img alt="Project Logo" src="./media/logo-light.png" width="512" height="auto">
  </picture>
</p>

---

<div align="center">

[![GitHub](https://img.shields.io/badge/GitHub-blue?style=flat&logo=github)](https://github.com/AnikBeris)
[![License](https://img.shields.io/badge/License-purple?style=flat&logo=github)](https://github.com/AnikBeris/AutoRoleChannelBot/blob/main/LICENSE)
[![GitHub Stars](https://img.shields.io/github/stars/your-repo?style=flat&logo=github&label=Звёзды&color=orange)](https://github.com/AnikBeris)

</div>

> **Отказ от ответственности:** Этот проект предназначен только для личного обучения.

**Если этот проект оказался полезным для вас, вы можете оценить его, поставив звёздочку.**:star2:

<p align="left">
  <a href="https://pay.cloudtips.ru/p/7249ba98" target="_blank">
    <img src="./media/buymeacoffe.png" alt="Image">
  </a>
</p>

Пожертвования горячо приветствуются, какими бы маленькими они ни были, и большое спасибо. 😌

| | |
|-------------:|:-------------|
| **Bitcoin (BTC)** |`1Dbwq9EP8YpF3SrLgag2EQwGASMSGLADbh`|
| **Ethereum (ERC20)** | `0x22258ea591966e830199d27dea7c542f31ed5dc5`|
| **Binance Smart Chain (BEP20)** | `0x22258ea591966e830199d27dea7c542f31ed5dc5`|
| **Solana (SOL)** | `yYYXsiVTzsvfvsMnBxfxSZEWTGytjAViE2ojf3hbLeF`|
| **Cloud tips** | [cloudtips](https://pay.cloudtips.ru/p/7249ba98) |
---



![//](./media/CI-CD.png)
Принципы CI/CD

# 🚀 Установка и настройка CI/CD для Unreal Engine 4 | Unreal Engine 5 с Gitea и self-hosted runners



Этот гайд поможет вам развернуть **Gitea** на сервере, настроить раннеры и автоматизировать сборки проекта **Unreal Engine** или любой другой игры. 

---

Шаг 1: 📌 Установка Gitea через Docker Compose
---

Создаём новую директорию для Gitea и переходим в неё:

```bash
mkdir -p ~/gitea/{data,config,logs} && cd ~/gitea
```

Создаём `docker-compose.yml`:

```bash
touch docker-compose.yml && nano docker-compose.yml
```
заполняем фаил `docker-compose.yml`:

```yaml
services:
  gitea:
    image: gitea/gitea:latest
    container_name: gitea
    restart: always
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - GITEA__server__ROOT_URL=http://192.168.1.40:3000
      - GITEA__database__DB_TYPE=sqlite3
      - GITEA__database__PATH=/data/gitea.db
    volumes:
      - ./data:/data
      - ./config:/config
      - ./logs:/logs
    ports:
      - "3000:3000"
      - "222:22"
    networks:
      - gitea

networks:
  gitea:
    driver: bridge
```
После вставки нажми `Ctrl+D`, чтобы завершить ввод. 🚀


⚠️ Важно: Gitea и Gitea Runner должны находиться в одной локальной сети.
```yaml
    networks:
      - gitea
networks:
  gitea:
    driver: bridge
```


После этого запустим Gitea командой:

```sh
docker-compose up -d
```

Теперь Gitea доступна по адресу: `http://<адрес_сервера_на_котором_запущен_Gitea>:3000` 🎉
пример -> `http://192.168.1.1:3000`

---

### Установка вручную
<details>
    <summary>Установка </summary>

Если не хотите использовать Docker, можно установить Gitea вручную:


1. **Создаём папку для Gitea**
   ```sh
   mkdir -p /srv/gitea/{custom,data,log}
   chmod 750 /srv/gitea/{custom,data,log}
   ```

2. **Скачиваем Gitea**
   ```sh
   wget -O /usr/local/bin/gitea https://dl.gitea.io/gitea/latest/gitea-linux-amd64
   chmod +x /usr/local/bin/gitea
   ```

3. **Создаём файл службы systemd**
   ```sh
   nano /etc/systemd/system/gitea.service
   ```
   Вставляем:
   ```ini
   [Unit]
   Description=Gitea Service
   After=syslog.target network.target
   
   [Service]
   Type=simple
   User=git
   Group=git
   WorkingDirectory=/srv/gitea
   ExecStart=/usr/local/bin/gitea web
   Restart=always
   
   [Install]
   WantedBy=multi-user.target
   ```

4. **Запускаем Gitea**
   ```sh
   systemctl enable --now gitea
   ```
   Теперь Gitea работает! 🎉

</details>

---

Шаг 2: 🏃 Установка и настройка Gitea Runner
---

Gitea Runner нужен для выполнения CI/CD задач на вашей локальной машине.

### 🔹 Способ 1: Использование Docker

Создаём `docker-compose-runner.yml` в той же папке `~/gitea`:

```yaml
services:
  runner:
    image: gitea/act_runner:nightly
    environment:
      GITEA_INSTANCE_URL: "http://192.168.1.40:3000"
      GITEA_RUNNER_REGISTRATION_TOKEN: "ВАШ_ТОКЕН"
      GITEA_RUNNER_NAME: "Gitea Runner"
    volumes:
      - ./runner:/data
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - gitea

networks:
  gitea:
    driver: bridge
```

Запускаем:

```bash
docker-compose up -d
```


---

## 🛠 Способ 2: Ручная установка на Windows

### 1. **С указанием точного пути**

```bash 
mkdir C:\_gitea_runner && cd C:\_gitea_runner
```

### 2. Скачивание последней версии `act_runner` и меняем название на `gitea-act-runner.exe`
```
Invoke-WebRequest -Uri "https://dl.gitea.io/act_runner/latest/gitea-act-runner-windows-amd64.exe" -OutFile "gitea-act-runner.exe"
```

### 3.Запусаем и регестрируем `gitea-act-runner.exe` в `Gitea`
❗ **важно, что запуск должен происходить в `PowerShell` с правами администратора** примеры, как это сделать приведено вконце статьи
```
.\gitea-act-runner.exe register
```

---

## Во время регистрации укажи:

- **Gitea instance URL:** `http://192.168.1.40:3000/` <- это ip сервера Gitea
- **Token:** (смотри в Gitea: `Settings -> Actions -> Generate Token`) <- тут его можно получить
- **Runner name:** `self-hosted`
- **Labels:** `self-hosted, windows`



Теперь раннер готов к работе! 🚀
---

<details>
    <summary> варианты установок </summary>

### 🔹 Способ 1
1. **Создаём папку для раннера**
   ```sh
   mkdir -p /srv/gitea-runner
   cd /srv/gitea-runner
   ```



2. **Скачиваем Gitea Runner**
   ```sh
   wget -O gitea-act-runner https://dl.gitea.io/act_runner/latest/gitea-act-runner-linux-amd64
   chmod +x gitea-act-runner
   ```

3. **Запускаем и регистрируем раннер**
   ```sh
   ./gitea-act-runner register --url http://<gitea-server>:3000/ --token <ВАШ_ТОКЕН>
   ```

4. **Запускаем раннер**
   ```sh
   ./gitea-act-runner daemon
   ```
   
---

### 🔹 Способ 2: Использование Docker (рекомендуемый)

Создаём `docker-compose-runner.yml` в той же папке `~/gitea`:

```yaml
services:
  runner:
    image: gitea/act_runner:nightly
    environment:
      GITEA_INSTANCE_URL: "http://192.168.1.40:3000"
      GITEA_RUNNER_REGISTRATION_TOKEN: "ВАШ_ТОКЕН"
      GITEA_RUNNER_NAME: "Gitea Runner"
    volumes:
      - ./runner:/data
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - gitea

networks:
  gitea:
    driver: bridge
```

⚠️ Важно: Gitea Runner и Gitea должны находиться в одной локальной сети.

```yaml
    networks:
      - gitea
networks:
  gitea:
    driver: bridge
```

Запускаем:

```bash
docker-compose-runner up -d
```


</details>


---

2. **Запусти workflow**
Теперь, когда твой `self-hosted` runner активен, просто **запушь изменения** в `master`, и Gitea сам запустит `main.yml`.
```
git add .
```
```
git commit -m "Запуск CI/CD"
```
```
git push origin master
```
Gitea должен автоматически запустить `Package Unreal Engine Game`. Проверь в Gitea (`Settings -> Actions`).

---





Шаг 3: 🚀 Настройка CI/CD для сборки Unreal Engine 5
---

Создаём папки в репозитории с проектом `.github/workflows` и файл `main.yml` в папке `workflows` должно получиться так `.github/workflows/main.yml`:

Вот пример **YAML**-файла для автоматической сборки проекта на **Unreal Engine 5** :

```yaml
name: Package Unreal Engine Project 

on:
  push:
    branches:
      - master # <- Ветка, при изменение которой будет запускаться компиляция проекта. Можите создать специальную ветку для билда проекта
  workflow_dispatch:  # <- Добавляет возможность ручного запуска

jobs:
  build:
    runs-on: self-hosted
    name: Package Unreal Engine Game
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Package Project
        env: 
          UE_PATH: 'F:\Epic Games\UE_5.4' # <- Путь к вашей установке Unreal Engine
          PROJECT_PATH: 'C:\Documents\GitHub\Test_Games_ci-cd\Games_CI_CD\Games_CI_CD.uproject' # <- Путь к файлу запуска вышей игры-проекта .uproject
          OUTPUT_PATH: 'C:\Documents\GitHub\Test_Games_ci-cd\BUILD' # <- Путь куда будет сохранён проект билда
        run: |
          & $env:UE_PATH\Engine\Build\BatchFiles\RunUAT.bat BuildCookRun -project="$env:PROJECT_PATH" -noP4 -platform=Win64 -clientconfig=Development -serverconfig=Development -cook -allmaps -build -stage -pak -archive -archivedirectory="$env:OUTPUT_PATH"
```

---

Шаг 4: ✅ Запуск Workflow в Gitea
---

1. Заходим в Gitea.
2. Открываем репозиторий с workflow.
3. Переходим в `Actions` → `Package Unreal Engine Project`.
4. Нажимаем `Run workflow`.

После успешного выполнения, билд появится в папке `BUILD` или в той который вы указали в файле `main.yml` в строке OUTPUT_PATH: `'C:\Documents\GitHub\Test_Games_ci-cd\BUILD`!

---

🎯 **Теперь у вас настроен мощный CI/CD для Unreal Engine 5 с Gitea и self-hosted Runner!** 🚀



---

# решение ошибок

❗ Запуск **PowerShell от имени администратора.**

- Нажми `Win + X` → выбери: 
 ```
Windows PowerShell (Admin)
 ```

Ошибка **`PSSecurityException`** связана с тем, что **`PowerShell`** запрещает выполнение скриптов из-за политики безопасности.

## 🔧 Как исправить?
1. Проверь текущую политику:

```
Get-ExecutionPolicy -List

```
Важно, чтобы **CurrentUser** имел RemoteSigned или Unrestricted.


2. Разрешить выполнение скриптов в PowerShell
**Открой PowerShell с правами администратора.**

- Нажми `Win + X` → выбери: 
 ```sh 
Windows Terminal (Admin) 
 ``` 
или 
 ```
PowerShell (Admin)
 ```

**`Теперь выполни команду:`**

 ```
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

 ```

**💡 Что делает команда?**

`RemoteSigned` — разрешает локальные скрипты без подписи, но требует подпись для загруженных из интернета.
`CurrentUser` — изменяет политику только для текущего пользователя.

---

❗ Почему act не работает?
---
- `act` не поддерживает `self-hosted`, так как он не может эмулировать кастомные runner'ы.
- `act` нужен только для локального тестирования workflows

Если тебе нужен тестовый запуск, измени `runs-on` в `main.yml` на ubuntu-latest, но тогда он не сможет работать с Windows-командами (`RunUAT.bat`) который запускает билд проекта и находится в деректории движка 
`Epic Games\UE_5.4\Engine\Build\BatchFiles\`RunUAT.bat

---


❗ Чтобы `gitea-act-runner.exe` всегда работал на `Windows`, можно использовать несколько вариантов:
---
### 1.Запуск через Планировщик задач (Task Scheduler)

Этот способ позволяет автоматически запускать `gitea-act-runner.exe` при загрузке Windows.

**Шаги:**

1. Вызываем Планировщик задач
Нажать `Win + R` и введите
```
taskschd.msc.
```
2. Выбрать **Создать задачу**...
3. Вкладка **Общие**:
- **Имя:** Gitea Act Runner
- **Запускать независимо от пользователя** (если требуется)
- **Использовать наивысшие права**
4. Вкладка **Триггеры → Создать...**
- При запуске системы
5. Вкладка **Действия → Создать...**
- **Программа или сценарий:** `C:\путь_к_gitea-act-runner.exe`
- **Аргументы (если нужно):** `daemon`
6. **ОК** и запустить задачу.

---

### 2.Запуск через автозапуск (Registry или Startup Folder)

Если не нужен запуск с правами администратора, можно добавить в автозагрузку.

**Вариант 1: Startup Folder**
Нажать `Win + R`, и введите 
```
%appdata%\Microsoft\Windows\Start Menu\Programs\Startup
``` 
нажать **Enter**.
2. Создать ярлык на `gitea-act-runner.exe`.
3. Готово – теперь **runner** будет запускаться при входе в систему.

### Вариант 2: Реестр
Нажать `Win + R`, и ведите 
```sh
regedit
```
 нажать **Enter**.
2. Перейти в `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run`.
3. Правой кнопкой **→ Создать → Строковый параметр** → назвать 
```sh
GiteaActRunner
```
3. Ввести путь к `gitea-act-runner.exe`, например:

```sh
"C:\путь_к_gitea-act-runner.exe" daemon
```
5. Закрыть редактор реестра.


---

## 3. Запуск как Windows Service

Если **runner** должен работать в фоне даже без входа пользователя в систему, можно запустить его как службу (**service**).

### Шаги:

1. Откройте **cmd** от имени администратора.
2. Установите службу с помощью `sc.exe`:
    ```sh
    sc create GiteaActRunner binPath= "C:\путь_к_gitea-act-runner.exe daemon" start= auto
    ```
3. Запустите службу:
    ```sh
    sc start GiteaActRunner
    ```
4. Чтобы удалить службу:
    ```sh
    sc delete GiteaActRunner
    ```

---

## 4. Использование NSSM (Non-Sucking Service Manager)

Если `sc.exe` не работает, можно использовать **NSSM** (удобный инструмент для управления службами).

### Шаги:

1. [Скачать NSSM](https://nssm.cc/download).
2. Распаковать в `C:\nssm`.
3. В **cmd** от имени администратора выполнить:
    ```sh
    C:\nssm\nssm.exe install GiteaActRunner
    ```
4. В появившемся окне:
    - **Application → Path**: `C:\путь_к_gitea-act-runner.exe`
    - **Arguments**: `daemon`
    - **Startup type**: `Automatic`
    - Нажать **Install service**.
5. Запустить службу:
    ```sh
    net start GiteaActRunner
    ```



## Лицензия
Этот проект распространяется по [MIT License](https://github.com/your-repo/blob/main/LICENSE).

---

Для детальной документации ознакомьтесь с [Английским README](/README.md) или [Русским README](/README.ru_RU.md).

