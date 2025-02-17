# Jellyfin-Plex-Monitoring-in-Zabbix

## Jellyfin & Plex Active playback session monitoring in Zabbix

### 1. Установить zabbix-agent2, если у вас установлен zabbix-agent то замените его на zabbix-agent2
* Zabbix-agent2 может быть установлен на любом хосте, для удобства установим его на сервере где работает Jellyfin
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
* Создадим скрипт для Jellyfin, nano /usr/local/bin/jellyfin_active_sessions_count.sh
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
* chmod +x /usr/local/bin/jellyfin_active_sessions_count.sh
* 
