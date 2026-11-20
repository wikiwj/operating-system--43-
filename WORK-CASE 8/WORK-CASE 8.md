# Work-case 8

## Тема: Робота в Linux без графічної оболонки: термінальні програми, редактори, системні монітори та «пасхалки»

**Середовище:** Ubuntu (Linux), термінал Bash. Усі пакети встановлюються через `apt`, тому перед початком оновлюємо індекс пакетів:

```bash
sudo apt update
```
# Гапонюк Назар
---

## Glossary (словник англійських термінів)

| Term | Переклад / пояснення |
|---|---|
| GUI (Graphical User Interface) | графічний інтерфейс користувача |
| CLI / TUI (Text User Interface) | консольний / текстовий інтерфейс у терміналі |
| terminal emulator | програма-емулятор термінала |
| shell | командна оболонка (Bash) |
| package manager | менеджер пакетів (`apt`) |
| repository | репозиторій, сховище пакетів |
| file manager | файловий менеджер |
| web browser | веб-браузер |
| mail client (MUA) | поштовий клієнт |
| IMAP / SMTP | протоколи отримання / надсилання пошти |
| torrent client | клієнт для завантаження через BitTorrent |
| magnet link | magnet-посилання на торрент |
| calendar / reminder | календар / нагадування |
| text editor | текстовий редактор |
| process | процес |
| resource monitor | монітор системних ресурсів |
| CPU load / memory usage | навантаження процесора / використання пам'яті |
| daemon | служба, що працює у фоні |
| framebuffer | область пам'яті відеокарти для виводу зображення без GUI |
| ASCII art | малюнок, складений із символів |
| easter egg | «пасхалка», прихована жартівлива функція |
| pipe (`\|`) | конвеєр: передача виводу однієї команди на вхід іншої |
| headless server | сервер без монітора і графічної оболонки |

---

## 1. Робота без графічної оболонки: відповіді на питання

Для сервера або слабкого комп'ютера графічну оболонку зазвичай не ставлять: вона споживає пам'ять, процесорний час і розширює поверхню атаки. Проте майже всі звичні задачі мають текстові (TUI) аналоги.

### 1.1. Файловий менеджер у терміналі

Основна програма - **Midnight Commander** (`mc`), двопанельний файловий менеджер у стилі Norton Commander.

```bash
sudo apt install mc
mc
```

Основні клавіші: `Tab` - перемикання між панелями, `F5` - копіювати, `F6` - перемістити, `F7` - створити каталог, `F8` - видалити, `F3` - переглянути файл, `F4` - редагувати, `F10` - вихід.

Альтернативи: **ranger** (`sudo apt install ranger`, керування в стилі Vim, показує попередній перегляд файлів), **nnn** (дуже швидкий мінімалістичний менеджер).

### 1.2. Веб-браузер у терміналі

Текстові браузери відображають сторінки без зображень і скриптів: **lynx**, **w3m**, **links2**, **elinks**.

```bash
sudo apt install lynx
lynx https://www.wikipedia.org
```

Керування в lynx: стрілки `↑/↓` - перехід між посиланнями, `→` - відкрити посилання, `←` - назад, `g` - ввести адресу, `/` - пошук на сторінці, `q` - вихід. `w3m` (`sudo apt install w3m`) трохи зручніший у таблицях і може показувати зображення в сумісних терміналах.

### 1.3. Електронна пошта в терміналі

Поштові клієнти: **mutt** / **neomutt** (найпопулярніші), **alpine**. Для читання пошти з Gmail або іншого сервера потрібні дані IMAP/SMTP.

```bash
sudo apt install neomutt
neomutt
```

Мінімальна конфігурація `~/.neomuttrc` (замінити значення на власні, пароль - лише пароль застосунку, не основний):

```
set imap_user = "user@example.com"
set imap_pass = "APP_PASSWORD"
set folder = "imaps://imap.gmail.com:993"
set spoolfile = "+INBOX"
set smtp_url = "smtps://user@example.com@smtp.gmail.com:465/"
set from = "user@example.com"
set realname = "User Name"
```

Основні клавіші: `Enter` - відкрити лист, `m` - новий лист, `r` - відповісти, `d` - видалити, `q` - назад/вихід. Швидка відправка з командного рядка: `echo "Текст" | mail -s "Тема" user@example.com` (пакет `mailutils`).

# Пальгуй Богдан:

### 1.4. Музика в терміналі

Програми: **cmus** (повноцінний TUI-плеєр), **mpg123** (прості MP3), **MOC** (`mocp`), **mpd + ncmpcpp** (серверний підхід).

```bash
sudo apt install cmus
cmus
```

У cmus: `:add ~/Music` - додати каталог, `5` - вкладка Library (бібліотека), `Enter` - відтворити, `c` - пауза, `b` / `z` - наступний / попередній трек, `+` / `-` - гучність, `q` - вихід. Швидкий варіант для одного файлу: `mpg123 song.mp3`.

### 1.5. Торренти в терміналі

Клієнти: **rtorrent** (ncurses-інтерфейс), **transmission-cli**, **aria2**.

```bash
sudo apt install rtorrent
rtorrent file.torrent
```

Клавіші rtorrent: `Ctrl+S` - старт, `Ctrl+D` - зупинка (двічі - видалення), `Ctrl+Q` - вихід. Альтернативи:

```bash
sudo apt install transmission-cli
transmission-cli file.torrent
sudo apt install aria2
aria2c "magnet:?xt=urn:btih:..."
```

### 1.6. Планування в календарі та нагадування

Програми: **calcurse** (календар + список завдань + нагадування), **remind**, а також системні `cal`, `at`.

```bash
sudo apt install calcurse
calcurse
```

У calcurse: `a` - додати подію (appointment), `t` - додати завдання (todo), `Tab` - перемикання між панелями, `s` - зберегти, `q` - вихід. Нагадування: `calcurse --notify` / параметр `notification-daemon` в `~/.config/calcurse/conf`.

Альтернатива **remind**: файл `~/.reminders` із рядком

```
REM 25 Nov 2026 AT 10:00 +30 MSG Здача лабораторної роботи
```

і команда `remind ~/.reminders`. Просте одноразове нагадування: `echo "notify-send 'Нагадування' 'Пора на пару'" | at 10:00`.

### 1.7. Перегляд зображень у терміналі

Програми: **chafa**, **timg**, **catimg**, **viu** (малюють зображення блоками Unicode чи ASCII), **fim** (перегляд у framebuffer прямо з консолі без X11).

```bash
sudo apt install chafa
chafa picture.png
sudo apt install fim
fim picture.jpg
```

`fim` у консолі (TTY) показує зображення повноцінно; у графічному терміналі краще працюють `chafa` та `timg`.

---

## 2. Класичні інструменти адміністратора

### 2.1. Текстові редактори (введення, редагування, видалення тексту)

| Редактор | Особливості | Основні клавіші |
|---|---|---|
| **nano** | найпростіший, підказки внизу екрана | `Ctrl+O` зберегти, `Ctrl+X` вихід, `Ctrl+K` вирізати рядок, `Ctrl+U` вставити, `Ctrl+W` пошук |
| **vim** | дуже потужний, режими (normal/insert/command) | `i` режим вставки, `Esc` назад, `dd` видалити рядок, `x` видалити символ, `:wq` зберегти і вийти, `:q!` вийти без збереження, `/слово` пошук |
| **micro** | сучасний, зрозумілі гарячі клавіші, як у GUI | `Ctrl+S` зберегти, `Ctrl+Q` вийти, `Ctrl+C/V` копіювати/вставити |
| **emacs -nw** | розширюваний, багатофункціональний | `Ctrl+X Ctrl+S` зберегти, `Ctrl+X Ctrl+C` вийти |
| **ed / sed** | рядкові редактори для автоматизації | `sed -i 's/old/new/g' file` |

Установлення та демонстрація (**nano**):

```bash
sudo apt install nano
nano --version
nano test.txt          # ввести текст, відредагувати, видалити рядок Ctrl+K, зберегти Ctrl+O, вийти Ctrl+X
cat test.txt
```

Для порівняння встановлюється `vim`: `sudo apt install vim`, запуск `vim test.txt`.

# Єршов Олексій

### 2.2. Моніторинг процесів та ресурсів системи

Це аналог «Диспетчера завдань» (Task Manager) у Windows або «Системного монітора» в GNOME.

| Програма | Призначення |
|---|---|
| `top` | вбудований монітор процесів, завжди доступний |
| **htop** | кольоровий інтерактивний аналог `top`, зручне сортування і завершення процесів |
| **btop** | сучасний красивий монітор із графіками CPU, RAM, диска й мережі |
| `atop` | історія навантаження, облік ресурсів |
| `glances` | зведений огляд системи (Python), у тому числі через веб |
| `iotop` | навантаження на диск за процесами |
| `free -h`, `df -h`, `vmstat`, `uptime` | швидка перевірка пам'яті, дисків, загальне навантаження |
| `ps aux`, `pgrep`, `kill`, `pkill` | перегляд та керування процесами одноразово |

Установлення та запуск (**htop**):

```bash
sudo apt install htop
htop
```

Клавіші htop: `F1` - довідка, `F3` - пошук, `F4` - фільтр, `F5` - дерево процесів, `F6` - сортування (за CPU, MEM, PID), `F9` - завершити процес (`kill`), `F10` - вихід. У верхній частині відображаються завантаження ядер процесора, використання RAM і swap, а також load average.

---

## 3. «Пасхалки» (easter eggs)

### 3.1. Паровоз у терміналі - `sl`

Програма `sl` (від «Steam Locomotive») - жартівливий «покарання» за помилку в команді `ls`: показує локомотив із вагоном.

```bash
sudo apt install sl
sl
```

Корисні параметри: `sl -a` - локомотив «у біді» (люди кличуть на допомогу), `sl -l` - менший варіант, `sl -F` - локомотив «летить» угору, `sl -e` - дозволяє перервати `Ctrl+C`.

### 3.2. «Зоряні війни» (Star Wars) - ASCII-фільм

Класичний спосіб - подивитися першу частину в ASCII-графіці по протоколу Telnet:

```bash
sudo apt install telnet
telnet towel.blinkenlights.nl
```

Завершення - `Ctrl+]`, потім `quit`. Якщо сервер недоступний (його підтримка не гарантована) або Telnet заблоковано, можна скористатися запасним способом: `nc towel.blinkenlights.nl 23` (пакет `netcat`). Методика виконання завдання: з'єднання зі сторінкою за адресою з портом 23, прийом ASCII-кадрів і їх відображення терміналом із затримкою.

### 3.3. Діалог із коровою - `cowsay`

```bash
sudo apt install cowsay
cowsay "Hello, Linux!"
cowthink "Hmm..."
cowsay -f tux "I love Linux"
cowsay -l                 # список доступних «персонажів»
```

Параметр `-f` змінює «персонажа» (`dragon`, `tux`, `stegosaurus`, `sheep` тощо), параметри `-b`, `-d`, `-g` змінюють вираз «очей» корови.

### 3.4. Власні інтерактиви та жарти

```bash
sudo apt install fortune-mod lolcat figlet cmatrix nyancat asciiquarium bsdgames
```

| Команда | Що робить |
|---|---|
| `fortune` | виводить випадковий афоризм |
| `fortune \| cowsay \| lolcat` | корова каже афоризм райдужними кольорами (конвеєр із трьох команд) |
| `figlet "Linux"` | великий напис із символів (ASCII art) |
| `cmatrix` | «цифровий дощ» у стилі фільму «Матриця» (вихід - `q`) |
| `nyancat` | райдужний кіт у космосі (вихід - `Ctrl+C`) |
| `asciiquarium` | анімований акваріум із рибками |
| `curl wttr.in/Kyiv` | погода у вигляді ASCII-графіки |
| `curl parrot.live` | танцюючий папуга |
| `telnet mapscii.me` | інтерактивна карта світу в терміналі |
| `bsdgames`: `worm`, `snake`, `robots`, `hangman`, `arithmetic` | класичні консольні ігри, що запускаються як окремі команди |
| `nethack`, `ninvaders` | консольна RPG та клон Space Invaders |

> Деякі пасхалки залежать від дистрибутива, доступності пакетів у репозиторіях та сторонніх серверів, тому на інших системах результат може відрізнятися.

---

## Conclusions

In this work-case we discovered that the absence of a graphical shell does not limit an Ubuntu Linux user: every common GUI task has a reliable text-based equivalent. We used or described Midnight Commander and ranger for file management, lynx and w3m for browsing the web, neomutt for e-mail, cmus for music, rtorrent and transmission-cli for torrents, calcurse and remind for calendars and reminders, and chafa and fim for viewing images directly in the terminal. All of these programs are installed with a single `apt install` command and consume very little memory, which makes them ideal for headless servers and low-resource machines.

We also compared the main text editors (nano, vim, micro, emacs) and resource monitors (top, htop, btop, atop, glances). For beginners, nano and htop are the most convenient: nano shows hints for all shortcuts at the bottom of the screen, while htop offers an interactive process list with sorting and the ability to terminate processes with the F9 key, replacing the Task Manager of a graphical desktop. Vim, in turn, remains the most powerful editor available on almost every server.

Finally, the easter eggs (`sl`, `cowsay`, `fortune`, `cmatrix`, `nyancat` and the ASCII Star Wars movie) showed that the terminal can also be entertaining, and they demonstrated a very important Linux concept: the pipe (`|`), which allows several simple programs to be combined into one command, for example `fortune | cowsay | lolcat`. As a result, we became more confident in using the command line, the package manager and TUI applications, which is an essential skill for a future system administrator or software developer.
