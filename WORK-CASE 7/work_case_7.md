# Work-case 7. Планування завдань в ОС: Cron та його альтернативи

**Середовище:** віртуальна машина VirtualBox, Ubuntu (Linux), користувач `user`, домашній каталог `/home/user`.

# Гапонюк Назар

## Словник термінів (Glossary)

| Term | Пояснення |
|---|---|
| **scheduler** | планувальник завдань |
| **job / task** | завдання, яке запускається планувальником |
| **cron** | демон-планувальник періодичних завдань у Unix/Linux |
| **crontab** (*cron table*) | таблиця з розкладом завдань; також команда для її редагування |
| **daemon** | фоновий процес-служба |
| **trigger** | подія або час, що запускає завдання |
| **timer unit / service unit** | одиниці systemd: таймер (коли) і сервіс (що виконати) |
| **anacron** | планувальник, що виконує пропущені завдання після увімкнення машини |
| **at** | команда для разового відкладеного запуску |
| **@reboot** | спеціальний запис cron: виконати при старті системи |
| **backup** | резервна копія |
| **log** | журнал подій |

---

## Частина 1. Теоретичні питання

### 1.1. Основні функції планувальника завдань в ОС. Порівняння Windows та Linux

**Основні функції будь-якого планувальника завдань:**

* запуск програм та скриптів у **заданий час** (одноразово);
* **періодичний запуск** (щохвилини, щогодини, щодня, щотижня, щомісяця, щороку);
* запуск **за подією**: при старті системи, при вході користувача, при простої (idle), за записом у журналі подій;
* запуск від імені **певного користувача** з потрібними правами;
* **керування умовами**: наявність мережі, живлення від мережі, відсутність інших екземплярів завдання;
* **повторний запуск при невдачі** та виконання пропущених завдань;
* **логування** результатів виконання та сповіщення (наприклад, листом);
* автоматизація рутинних адміністративних задач: backup, архівування, очищення каталогів, оновлення, перевірка стану системи.

**Порівняння Windows та Linux:**

| Критерій | Windows (**Task Scheduler**) | Linux (**cron**, **systemd timers**) |
|---|---|---|
| Основний інструмент | Task Scheduler (`taskschd.msc`), `schtasks`, PowerShell (`Register-ScheduledTask`) | `cron` / `crontab`, `at`, `anacron`, `systemd timers` |
| Інтерфейс | графічна оболонка + CLI | переважно CLI та текстові конфігураційні файли |
| Формат розкладу | майстер у GUI, XML-опис завдання | рядок із 5 полів у crontab; `OnCalendar=` у systemd |
| Типи тригерів | за часом, при вході, при запуску, при простої, за подією журналу, при підключенні до сесії, з пробудженням комп'ютера | час, `@reboot`, у systemd - також залежності від інших юнітів, `OnBootSec`, `OnUnitActiveSec` |
| Пропущені завдання | опція «Run task as soon as possible after a scheduled start is missed» | cron їх не виконує; для цього `anacron` або `Persistent=true` у systemd |
| Умови (мережа, живлення) | налаштовуються вбудовано | потрібно перевіряти в скрипті або через залежності systemd |
| Журнал | вкладка History, Event Viewer | `/var/log/syslog`, `journalctl` |
| Конфігурація | зберігається в `C:\Windows\System32\Tasks` та реєстрі | `/var/spool/cron/crontabs`, `/etc/crontab`, `/etc/cron.d`, `/etc/systemd/system` |

Висновок: Windows Task Scheduler зручніший для початківця (графічний інтерфейс і багато умов «з коробки»), а Linux-планувальники простіші, легші, скриптуються текстовими файлами й легко автоматизуються та переносяться між машинами.

### 1.2. Принципи роботи з Cron, налаштування, альтернативи

**Принцип роботи.** Демон `cron` (`crond`) запускається при старті системи, щохвилини перевіряє таблиці розкладу й запускає завдання, час яких настав. Перевірка служби:

```bash
sudo systemctl status cron
```

**Де зберігаються розклади:**

* `crontab -e` - персональна таблиця користувача (файли у `/var/spool/cron/crontabs/`);
* `/etc/crontab` - системна таблиця (має додаткове поле **користувача**);
* `/etc/cron.d/` - окремі системні файли розкладів;
* каталоги `/etc/cron.hourly`, `cron.daily`, `cron.weekly`, `cron.monthly` - скрипти, що запускаються з відповідною періодичністю;
* `/etc/cron.allow` та `/etc/cron.deny` - кому дозволено користуватися cron.

**Команди керування crontab:**

| Команда | Дія |
|---|---|
| `crontab -e` | редагувати свою таблицю |
| `crontab -l` | показати свою таблицю |
| `crontab -r` | видалити свою таблицю повністю |
| `sudo crontab -u user -l` | переглянути таблицю іншого користувача |

**Формат запису:**

```
┌───────── хвилина (0–59)
│ ┌─────── година (0–23)
│ │ ┌───── день місяця (1–31)
│ │ │ ┌─── місяць (1–12)
│ │ │ │ ┌─ день тижня (0–7, 0 і 7 - неділя)
│ │ │ │ │
* * * * *  команда
```

Спеціальні символи: `*` - будь-яке значення; `,` - перелік (`9,21`); `-` - діапазон (`8-18`); `/` - крок (`*/15`). Є скорочення: `@reboot`, `@hourly`, `@daily`, `@weekly`, `@monthly`, `@yearly`.

**Важливі правила налаштування:**

* у cron мінімальне середовище (`PATH` обмежений), тому команди слід вказувати **з повним шляхом**;
* символ `%` у командах треба екранувати як `\%`;
* вивід команди краще перенаправляти у файл (`>> file.log 2>&1`), інакше cron спробує надіслати його поштою (`MAILTO`);
* скрипти мають бути виконуваними (`chmod +x`).

**Альтернативи cron:**

| Планувальник | Характеристика |
|---|---|
| **anacron** | виконує завдання з періодичністю в днях, **навіть якщо машина була вимкнена** в запланований час; підходить для ноутбуків і ПК, які працюють не постійно; не вміє планувати на конкретну годину |
| **at / batch** | **одноразовий** запуск у вказаний час (`at 18:30`), `batch` - коли навантаження системи низьке |
| **systemd timers** | вбудований у сучасні дистрибутиви; пов'язується з `.service`, підтримує залежності, `Persistent=true` (виконання пропущених запусків), журнал `journalctl`, випадкову затримку, `OnBootSec` |
| **fcron** | поєднує можливості cron і anacron, підтримує умови та виконання пропущених завдань; потребує окремого встановлення й заміщує cron |

---
# Пальгуй Богдан

## Частина 2. Планування завдань через Cron

### 2.0. Підготовка

Створюємо каталоги, тестові файли та три скрипти.

```bash
mkdir -p ~/scripts ~/backups ~/cron_logs ~/test_data ~/Downloads
echo "test file" > ~/test_data/file1.txt
```

**`~/scripts/log.sh`** - записує повідомлення з міткою часу:

```bash
#!/bin/bash
echo "$(date '+%Y-%m-%d %H:%M:%S') | $1" >> /home/user/cron_logs/tasks.log
```

**`~/scripts/backup.sh`** - створює архів каталогу `test_data` (архівування та backup):

```bash
#!/bin/bash
tar -czf /home/user/backups/test_data_$(date +%F_%H%M).tar.gz /home/user/test_data 2>/dev/null
echo "$(date '+%Y-%m-%d %H:%M:%S') | backup created" >> /home/user/cron_logs/tasks.log
```

**`~/scripts/cleanup.sh`** - видаляє файли в `Downloads` старші за 30 днів та старі архіви (очищення каталогів):

```bash
#!/bin/bash
find /home/user/Downloads -type f -mtime +30 -delete
find /home/user/backups -type f -name "*.tar.gz" -mtime +7 -delete
echo "$(date '+%Y-%m-%d %H:%M:%S') | cleanup done" >> /home/user/cron_logs/tasks.log
```

Робимо їх виконуваними та перевіряємо вручну:

```bash
chmod +x ~/scripts/*.sh
~/scripts/backup.sh && cat ~/cron_logs/tasks.log
```

```
2026-11-12 10:05:41 | backup created
```

Відкриваємо редактор розкладу:

```bash
crontab -e
```

### 2.1. Завдання в чітко визначений час

```cron
# Щодня о 08:00
0 8 * * * /home/user/scripts/log.sh "morning task 08:00"
# Щодня о 18:30
30 18 * * * /home/user/scripts/log.sh "evening task 18:30"
```

Поля: `0 8` - хвилина 0, година 8; `30 18` - хвилина 30, година 18; решта `*` - кожного дня, місяця, дня тижня.

### 2.2. Одна й та сама задача двічі на день

```cron
# Backup о 09:00 та 21:00
0 9,21 * * * /home/user/scripts/backup.sh
```

Кома в полі години створює перелік значень (9 і 21).

### 2.3. Тільки в будні / тільки у вихідні, у проміжку 8–18 годин

```cron
# Щогодини з 08:00 до 18:00, лише у будні (пн–пт)
0 8-18 * * 1-5 /home/user/scripts/log.sh "weekday working hours"
# Кожні 30 хвилин з 08:00 до 17:30, лише у будні
*/30 8-17 * * 1-5 /home/user/scripts/log.sh "weekday every 30 min"
# Лише у вихідні (сб, нд) о 10:00
0 10 * * 6,0 /home/user/scripts/cleanup.sh
```

`1-5` - понеділок–п'ятниця; `6,0` - субота та неділя; `8-18` - діапазон годин.

### 2.4. Раз на рік, місяць, день, годину та при вмиканні

```cron
# Раз на рік (1 січня о 00:00)
@yearly   /home/user/scripts/log.sh "yearly task"
# Раз на місяць (1 числа о 00:00)
@monthly  /home/user/scripts/backup.sh
# Раз на день (щодня о 00:00)
@daily    /home/user/scripts/cleanup.sh
# Щогодини (на початку години)
@hourly   /home/user/scripts/log.sh "hourly task"
# При вмиканні / перезавантаженні
@reboot   /home/user/scripts/log.sh "system started"
```

Те саме в повній формі: `@yearly` = `0 0 1 1 *`, `@monthly` = `0 0 1 * *`, `@daily` = `0 0 * * *`, `@hourly` = `0 * * * *`.

### 2.5. Перевірка роботи

```bash
crontab -l                      # переглянути поточний розклад
sudo systemctl status cron      # служба має бути active (running)
grep CRON /var/log/syslog | tail -5
cat ~/cron_logs/tasks.log
```

Приклад виводу:

```
Nov 12 18:30:01 ubuntu CRON[2214]: (user) CMD (/home/user/scripts/log.sh "evening task 18:30")
2026-11-12 18:30:01 | evening task 18:30
```

Для швидкої перевірки тимчасово ставимо `* * * * *` (щохвилини), чекаємо запису в `tasks.log`, після чого повертаємо потрібний розклад.

---
# Єршов Олексій

## Частина 3. Альтернативний планувальник - systemd timers

**Вибір.** Як альтернативу cron використано **systemd timers**: це сучасний планувальник, що входить до складу Ubuntu, не конфліктує з cron, підтримує всі необхідні типи розкладів, має журнал і виконання пропущених запусків. Перевірка наявності:

```bash
systemctl --version | head -1
systemctl list-timers --all | head
```

Окремого встановлення він не вимагає (на відміну від fcron, який замінює cron). Якщо потрібно, можна додатково встановити `anacron` / `at`: `sudo apt install anacron at`.

### 3.1. Структура

Кожне завдання складається з двох файлів:

* `name.service` - **що** виконати;
* `name.timer` - **коли** виконати.

Розміщуємо їх у каталозі користувача `~/.config/systemd/user/` і керуємо через `systemctl --user`.

```bash
mkdir -p ~/.config/systemd/user
```

**`~/.config/systemd/user/backup.service`:**

```ini
[Unit]
Description=Backup of test_data

[Service]
Type=oneshot
ExecStart=/home/user/scripts/backup.sh
```

**`~/.config/systemd/user/backup.timer`** (двічі на день, 09:00 та 21:00):

```ini
[Unit]
Description=Run backup twice a day

[Timer]
OnCalendar=*-*-* 09,21:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

Активація:

```bash
systemctl --user daemon-reload
systemctl --user enable --now backup.timer
systemctl --user list-timers
```

```
NEXT                         LEFT      LAST PASSED UNIT          ACTIVATES
Thu 2026-11-12 21:00:00 EET  10h left  -    -      backup.timer  backup.service
```

### 3.2. Відповідність завдань із частини 2 (параметр `OnCalendar`)

Кожен із наведених розкладів оформлюється як окремий `.timer` з відповідним `.service` (за зразком вище), змінюється лише рядок у секції `[Timer]`:

| Завдання | Cron | systemd timer |
|---|---|---|
| Щодня о 08:00 | `0 8 * * *` | `OnCalendar=*-*-* 08:00:00` |
| Щодня о 18:30 | `30 18 * * *` | `OnCalendar=*-*-* 18:30:00` |
| Двічі на день (09:00 і 21:00) | `0 9,21 * * *` | `OnCalendar=*-*-* 09,21:00:00` |
| Щогодини у будні з 8 до 18 | `0 8-18 * * 1-5` | `OnCalendar=Mon..Fri *-*-* 08..18:00:00` |
| Тільки у вихідні о 10:00 | `0 10 * * 6,0` | `OnCalendar=Sat,Sun *-*-* 10:00:00` |
| Раз на рік | `@yearly` | `OnCalendar=yearly` |
| Раз на місяць | `@monthly` | `OnCalendar=monthly` |
| Раз на день | `@daily` | `OnCalendar=daily` |
| Щогодини | `@hourly` | `OnCalendar=hourly` |
| При вмиканні | `@reboot` | `OnBootSec=1min` (запуск через хвилину після старту) |

Приклад таймера «при вмиканні»:

```ini
[Timer]
OnBootSec=1min
Unit=backup.service
```

### 3.3. Перевірка виразів календаря та журнал

Синтаксис `OnCalendar` можна перевірити до запуску:

```bash
systemd-analyze calendar "Mon..Fri *-*-* 08..18:00:00"
```

```
  Original form: Mon..Fri *-*-* 08..18:00:00
Normalized form: Mon..Fri *-*-* 08..18:00:00
    Next elapse: Fri 2026-11-13 08:00:00 EET
```

Журнал виконання:

```bash
journalctl --user -u backup.service -n 5
systemctl --user list-timers --all
```

Щоб користувацькі таймери працювали і без активного входу в систему (наприклад, після перезавантаження), вмикаємо:

```bash
sudo loginctl enable-linger user
```

### 3.4. Порівняння з cron

| Критерій | cron | systemd timers |
|---|---|---|
| Конфігурація | один рядок у crontab | два файли (`.service` + `.timer`) |
| Пропущені запуски | не виконуються | виконуються при `Persistent=true` |
| Журнал | syslog | `journalctl`, окремий для кожної служби |
| Залежності | немає | є (`After=`, `Requires=`) |
| Точність / випадкова затримка | до хвилини | є `AccuracySec`, `RandomizedDelaySec` |
| Простота | простіший для коротких задач | гнучкіший для складних |

---

## Conclusions

During this work we studied how task schedulers work and compared Windows Task Scheduler with Linux tools. Windows offers a convenient graphical interface and many built-in conditions, while Linux relies on text-based configuration, which is lighter and easier to automate. We learned the structure of the crontab format (five time fields plus a command), the main `crontab` options and the special strings `@reboot`, `@hourly`, `@daily`, `@monthly` and `@yearly`.

In the practical part we created scripts for logging, backup and cleanup and scheduled them in cron: at a fixed time, twice a day, on weekdays during working hours (8–18), on weekends only, and with yearly, monthly, daily, hourly and at-boot periodicity. As an alternative scheduler we used systemd timers and showed that every cron schedule can be expressed with `OnCalendar` or `OnBootSec`, with extra advantages such as `Persistent=true` for missed runs and logging through `journalctl`.

The main practical lessons: always use absolute paths in cron jobs, redirect output to a log file, check the result in logs and with `systemd-analyze calendar`, and choose the scheduler according to the task: cron for simple periodic jobs, anacron or systemd timers for machines that are not always powered on, `at` for one-time jobs.
