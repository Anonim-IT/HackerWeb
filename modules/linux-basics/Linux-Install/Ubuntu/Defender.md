# Обезопась себя в Linux

Руководство по базовой безопасности для пользователей Linux (на примере Ubuntu и производных дистрибутивов).

---

## 📋 Содержание

1. [Базовые принципы](#базовые-принципы)
2. [Настройка системы](#настройка-системы)
3. [Брандмауэр (UFW)](#брандмауэр-ufw)
4. [Безопасность SSH](#безопасность-ssh)
5. [Антивирусы и сканеры](#антивирусы-и-сканеры)
6. [Шифрование](#шифрование)
7. [Безопасность браузера](#безопасность-браузера)
8. [Резервное копирование](#резервное-копирование)
9. [Ежедневные привычки](#ежедневные-привычки)
10. [Чек-лист безопасности](#чек-лист-безопасности)

---

## 🛡 Базовые принципы

1. **Не работай от root** — используй `sudo` только когда необходимо.
2. **Минимализм** — удаляй неиспользуемые пакеты и сервисы.
3. **Регулярные обновления** — критически важны для безопасности.
4. **Принцип наименьших привилегий** — давай программам минимум прав.

---

## ⚙️ Настройка системы

### 1. Обновление системы

Регулярно обновляйте систему:

```bash
# Обновление списка пакетов и установка обновлений
sudo apt update && sudo apt upgrade -y

# Обновление всех пакетов, включая ядро
sudo apt full-upgrade -y

# Автоматическое удаление ненужных пакетов
sudo apt autoremove -y
```

### 2. Настройка автоматических обновлений

```bash
# Установка инструмента для автоматических обновлений
sudo apt install unattended-upgrades

# Настройка автоматических обновлений
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

### 3. Отключение ненужных сервисов

Проверьте и отключите неиспользуемые сервисы:

```bash
# Просмотр активных сервисов
systemctl list-units --type=service --state=running

# Отключение ненужного сервиса (пример)
sudo systemctl disable bluetooth.service
sudo systemctl stop bluetooth.service
```

---

## 🔥 Брандмауэр (UFW)

### Включение и базовая настройка

```bash
# Включение UFW
sudo ufw enable

# Проверка статуса
sudo ufw status verbose

# Разрешение SSH (важно, если подключены удаленно!)
sudo ufw allow ssh
# или для конкретного порта
sudo ufw allow 22/tcp
```

### Типовые правила

```bash
# Разрешить HTTP/HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Разрешить только локальную сеть
sudo ufw allow from 192.168.1.0/24

# Запретить все входящие, разрешить все исходящие (по умолчанию)
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Ограничение количества подключений (защита от DoS)
sudo ufw limit ssh/tcp
```

### GUI-интерфейс (опционально)

```bash
# Установка графической оболочки для UFW
sudo apt install gufw
```

---

## 🔐 Безопасность SSH

Если вы используете SSH-сервер:

### 1. Изменение порта и настройки

Отредактируйте файл `/etc/ssh/sshd_config`:

```bash
sudo nano /etc/ssh/sshd_config
```

Рекомендуемые настройки:
```bash
# Измените порт (защита от сканирования)
Port 2222

# Запретить вход root-пользователем
PermitRootLogin no

# Использовать только ключи (запретить пароли)
PasswordAuthentication no
PubkeyAuthentication yes

# Ограничить пользователей, которые могут заходить
AllowUsers username

# Отключить пустые пароли
PermitEmptyPasswords no
```

### 2. Перезапуск SSH после изменений

```bash
sudo systemctl restart sshd
```

### 3. Использование SSH-ключей

```bash
# Генерация ключа (лучше ed25519)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Копирование ключа на сервер
ssh-copy-id -p 2222 user@server_ip

# Настройка агента SSH для удобства
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

---

## 🦠 Антивирусы и сканеры

### ClamAV (антивирус)

```bash
# Установка ClamAV
sudo apt install clamav clamav-daemon

# Обновление баз
sudo freshclam

# Сканирование домашней папки
clamscan -r ~/

# Сканирование всей системы (может долго)
sudo clamscan -r / --exclude-dir=/sys --exclude-dir=/proc
```

### Rkhunter (проверка руткитов)

```bash
# Установка
sudo apt install rkhunter

# Обновление базы
sudo rkhunter --update

# Проверка системы
sudo rkhunter --check --skip-keypress
```

### Lynis (аудит безопасности)

```bash
# Установка
sudo apt install lynis

# Запуск аудита
sudo lynis audit system

# Просмотр отчета
sudo cat /var/log/lynis.log
```

---

## 🔒 Шифрование

### 1. Шифрование диска (LUKS)

При установке системы выберите опцию **"Encrypt the new Ubuntu installation for security"**.

Для уже установленной системы:

```bash
# Установка cryptsetup
sudo apt install cryptsetup

# Создание зашифрованного контейнера
sudo cryptsetup luksFormat /dev/sdX
sudo cryptsetup open /dev/sdX encrypted_volume
```

### 2. Шифрование отдельных папок (ecryptfs)

```bash
# Установка
sudo apt install ecryptfs-utils

# Шифрование домашней папки
sudo ecryptfs-migrate-home -u username
```

### 3. Шифрование файлов (GnuPG)

```bash
# Шифрование файла
gpg -c secret_file.txt

# Расшифровка
gpg secret_file.txt.gpg
```

---

## 🌐 Безопасность браузера

### Firefox (предустановлен)

1. Настройки приватности:
   - `about:preferences#privacy`
   - Включите "Отправлять сайтам сигнал "Не отслеживать меня"
   - Выберите "Строгий" режим защиты от отслеживания

2. Рекомендуемые расширения:
   - **uBlock Origin** — блокировка рекламы и трекеров
   - **HTTPS Everywhere** — принудительное HTTPS
   - **Bitwarden** — менеджер паролей
   - **NoScript** — контроль скриптов (для продвинутых)

### Установка расширений через терминал (Firefox)

```bash
# Установка uBlock Origin
firefox https://addons.mozilla.org/firefox/downloads/file/4160558/ublock_origin-latest.xpi
```

---

## 💾 Резервное копирование

### Deja Dup (простой бэкап)

```bash
# Установка
sudo apt install deja-dup

# Запуск
deja-dup
```

### Rsync (продвинутый бэкап)

```bash
# Резервное копирование домашней папки на внешний диск
rsync -avz --progress ~/ /mnt/backup/home_backup/

# С синхронизацией (удаляет лишнее на приемнике)
rsync -avz --delete ~/ /mnt/backup/home_backup/
```

### Автоматизация бэкапа (cron)

```bash
# Редактирование cron-задач
crontab -e

# Добавить строку для ежедневного бэкапа в 2 часа ночи
0 2 * * * rsync -avz --delete /home/username/ /mnt/backup/home_backup/
```

---

## 👨‍💻 Ежедневные привычки

### 1. Проверка логов

```bash
# Просмотр последних записей в auth.log (попытки входа)
sudo tail -f /var/log/auth.log

# Поиск неудачных попыток входа
sudo grep "Failed password" /var/log/auth.log

# Проверка системных ошибок
sudo journalctl -p err -b
```

### 2. Мониторинг процессов и сетевых подключений

```bash
# Активные сетевые подключения
sudo netstat -tulpn

# Просмотр процессов
htop

# Проверка открытых портов
sudo ss -tulwn
```

### 3. Проверка прав SUID (потенциально опасные файлы)

```bash
# Поиск файлов с SUID битом
find / -perm -4000 -type f 2>/dev/null
```

### 4. Использование `sudo` с умом

```bash
# Никогда не используйте sudo для программ, которым не доверяете
# Не запускайте скрипты из интернета с sudo без проверки

# Проверка истории sudo
sudo cat /var/log/auth.log | grep sudo
```

---

## ✅ Чек-лист безопасности

После установки системы выполните следующие действия:

- [ ] **Обновлена система** (`sudo apt update && sudo apt upgrade`)
- [ ] **Включен UFW** (`sudo ufw enable`)
- [ ] **Настроены автоматические обновления**
- [ ] **Создан обычный пользователь** (если используется root)
- [ ] **Отключены ненужные сервисы**
- [ ] **Настроен SSH** (если используется):
  - [ ] Изменен порт
  - [ ] Запрещен root login
  - [ ] Используются ключи вместо паролей
- [ ] **Установлены расширения браузера** для безопасности
- [ ] **Настроено шифрование** (диска или важных папок)
- [ ] **Настроено резервное копирование**
- [ ] **Установлен и настроен менеджер паролей** (Bitwarden, KeePassXC)
- [ ] **Проверка системы** (`lynis audit system`)

---

## 📚 Полезные команды быстрого доступа

```bash
# Сохраните в файл ~/.bash_aliases и выполните source ~/.bashrc

alias update='sudo apt update && sudo apt upgrade -y'
alias firewall='sudo ufw status verbose'
alias logs='sudo tail -f /var/log/auth.log'
alias security='sudo lynis audit system'
alias ports='sudo netstat -tulpn'
alias scan-home='clamscan -r ~/'
```

---

## 🚨 Что делать при подозрении на взлом

1. **Отключите компьютер от сети** (выдерните кабель/отключите Wi-Fi)
2. **Сделайте копию логов** для анализа:
   ```bash
   sudo cp /var/log/auth.log ~/Desktop/
   sudo cp /var/log/syslog ~/Desktop/
   ```
3. **Проверьте запущенные процессы**:
   ```bash
   ps aux | grep -v root
   ```
4. **Проверьте автозагрузку**:
   ```bash
   ls -la ~/.config/autostart/
   crontab -l
   sudo crontab -l
   ```
5. **Используйте rkhunter и chkrootkit** для поиска руткитов
6. **При серьезном взломе** — переустановка системы с последующим восстановлением данных из бэкапа

---

## 📖 Дополнительные ресурсы

- [Ubuntu Security Documentation](https://ubuntu.com/security)
- [CIS Benchmarks for Ubuntu](https://www.cisecurity.org/benchmark/ubuntu_linux/)
- [Arch Linux Security Wiki](https://wiki.archlinux.org/title/Security)
- [OWASP Linux Security](https://owasp.org/www-community/Linux_Security)

---

*Помните: безопасность — это не разовое действие, а постоянный процесс. Регулярно обновляйте систему и проверяйте логи.*