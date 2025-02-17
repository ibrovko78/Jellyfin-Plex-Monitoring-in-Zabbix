# Jellyfin-Plex-Monitoring-in-Zabbix

## Jellyfin & Plex Active playback session monitoring in Zabbix

### 1. Установить zabbix-agent2, если у вас установлен zabbix-agent то замените его на zabbix-agent2
* Zabbix-agent2 может быть установлен на любом хосте, для удобства установим его на сервере где работает Jellyfin и Plex
### 2. Отредактируйте файл конфигурации zabbix-agent2, nano /etc/zabbix/zabbix_agent2.conf
* В конец файла конфигурации добавить UserParameter для запуска скриптов
```js
UserParameter=j_active_sessions_count, /usr/local/bin/./jellyfin_active_sessions_count.sh
UserParameter=p_active_sessions_count, /usr/local/bin/./plex_active_sessions_count.sh
```
### 3. Создадать API ключ в web интефейсе jellyfin 
* Панель -> API Ключи -> + (Добавить), теперь все что вам нужно это ввести имя токена, оно может быть любое
* Пусть вас не смущает что в русско язычном варианте написано "Название приложения"
* Нажать кнопку ОК, Api ключ создан
* Скопировать API ключ, далее его необходимо вставить в скрипт для Jellyfin

### 4. Создать скрипты
#### Создадим скрипт для Jellyfin
* nano /usr/local/bin/jellyfin_active_sessions_count.sh
* Наполним файл содержимым, замените localhost:8096 на ваш IP с jellyfin, вставьте ваш API_KEY=
```js
#!/bin/bash

# Jellyfin server details
JELLYFIN_URL="http://localhost:8096"
API_KEY="ваш API ключ"

# Fetch active sessions
ACTIVE_SESSIONS=$(curl -s -H "X-Emby-Token: $API_KEY" "$JELLYFIN_URL/Sessions")

# Count active playback sessions
ACTIVE_COUNT=$(echo "$ACTIVE_SESSIONS" | jq -r '[.[] | select(.NowPlayingItem)] | length')

# Display the total count of active playback sessions
echo "$ACTIVE_COUNT"
```
* chmod 770 +x /usr/local/bin/jellyfin_active_sessions_count.sh
* chown zabbix:zabbix /usr/local/bin/jellyfin_active_sessions_count.sh

#### Создадим скрипт для Plex
* Для начала необходимо получить Plex Token
* Вам потребуется аккаунт Plex, хватит даже акаунта без PlexPass
* Создайте аккаунт Plex если у вас его нет
* Войдите на сервере с Plex media server в аккаунт Plex
* Установите Python
* Создать скрипт python, nano /usr/local/bin/get_plex_token.py
* Наполните файл содержимым, замените "username" и "password" на вашии учетные данные аккаунта Plex
```js
import requests

# Plex credentials
PLEX_USERNAME = "username"
PLEX_PASSWORD = "password"

# Authenticate and get token
auth_url = "https://plex.tv/users/sign_in.json"
headers = {
    "X-Plex-Client-Identifier": "YourClientIdentifier",
    "X-Plex-Product": "YourProductName",
    "X-Plex-Version": "1.0"
}
data = {
    "user[login]": PLEX_USERNAME,
    "user[password]": PLEX_PASSWORD
}

response = requests.post(auth_url, headers=headers, data=data)
if response.status_code == 201:
    print("Plex Token:", response.json()["user"]["authToken"])
else:
    print("Failed to retrieve token. Check your credentials.")
```
* Запустить скрипт Python, python3 /usr/local/bin/get_plex_token.py
* Plex Token будет выведен в консоль
---
* Установить xmllint
* GitHub не позволяет разместить команды скрипта, скачайте его по ссылке ![скачать скрипт](script/plex_active_sessions_count.rar)
* Скачать скрипт для Plex, и поместить его в /usr/local/bin/ plex_active_sessions_count.sh
* chmod +x /usr/local/bin/ plex_active_sessions_count.sh
* В скрипте замените "ваш Plex Token" на Plex Token, замените "http://localhost:32400" на ваш IP c Plex
* Создайте свой шаблон zabbix или скачайте шаблон по ссылке ![скачать шаблон](script/jellyfin_and_Plex.yaml)
