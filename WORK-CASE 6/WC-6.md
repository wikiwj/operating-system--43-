# Work-case 6 

**Дисципліна:** Операційні системи
**Група:** КСМ-43а
**Тема:** Командні інтерпретатори (shells), користувачі та групи в Linux
**Середовище:** Ubuntu (Linux), доступ через `sudo`
**Студенти:** Гапонюк Назар, Пальгуй Богдан, Єршов Олексій


---
# Гапонюк Назар
## Зміст

1. [Мета роботи](#мета-роботи)
2. [Словник англійських термінів (Glossary)](#словник-англійських-термінів-glossary)
3. [Завдання 1. Встановлення командних інтерпретаторів](#завдання-1-встановлення-командних-інтерпретаторів)
4. [Завдання 2. Створення груп і користувачів](#завдання-2-створення-груп-і-користувачів)
5. [Завдання 3. Призначення shell за замовчуванням](#завдання-3-призначення-shell-за-замовчуванням)
6. [Завдання 4. Демонстрація роботи груп](#завдання-4-демонстрація-роботи-груп)
7. [Очищення системи (Cleanup)](#очищення-системи-cleanup)
8. [Висновки](#висновки)
9. [Conclusions (English)](#conclusions-english)

---

## Мета роботи

Навчитися встановлювати та порівнювати командні інтерпретатори (shells), створювати облікові записи (user accounts) і групи (groups), призначати login shell та обмежувати інтерактивний доступ до системи за допомогою `nologin`.

---

## Словник англійських термінів (Glossary)

| Term | Переклад / пояснення |
|---|---|
| shell | командний інтерпретатор, програма, що виконує команди користувача |
| login shell | shell, який запускається при вході користувача в систему |
| default shell | shell за замовчуванням, записаний у `/etc/passwd` |
| user account | обліковий запис користувача |
| group | група користувачів, спосіб розподілу прав доступу |
| primary group | основна група користувача (поле GID у `/etc/passwd`) |
| supplementary group | додаткова група користувача |
| UID / GID | ідентифікатор користувача / групи |
| home directory | домашній каталог користувача (`/home/<user>`) |
| package manager | менеджер пакетів (`apt`) |
| repository | репозиторій, сховище пакетів |
| `/etc/passwd` | файл з обліковими записами користувачів |
| `/etc/group` | файл з описом груп |
| `/etc/shells` | список дозволених login shells |
| `nologin` | заглушка, що забороняє інтерактивний вхід |
| privilege escalation | підвищення привілеїв (`sudo`) |
| environment variable | змінна середовища (`$SHELL`, `$HOME`) |
| working directory | поточний робочий каталог |
| system date | системна дата й час |
| hardware configuration | апаратна конфігурація |
| script | скрипт, файл зі сценарієм команд |
| autosuggestion | автопідказка команд під час набору |
| tab completion | автодоповнення за клавішею Tab |

---

## Завдання 1. Встановлення командних інтерпретаторів

Окрім **bash** встановлюємо ще два shells:

- **zsh** (Z shell), далі Interpreter 1;
- **fish** (Friendly Interactive Shell), далі Interpreter 2.

### 1.1. Перевірка поточного стану

```bash
echo $SHELL
cat /etc/shells
bash --version | head -n 1
```

**Output (приклад):**

```text
/bin/bash
# /etc/shells: valid login shells
/bin/sh
/bin/bash
/usr/bin/bash
/bin/rbash
/usr/bin/rbash
/usr/bin/sh
/bin/dash
/usr/bin/dash
GNU bash, version 5.2.21(1)-release (x86_64-pc-linux-gnu)
```

### 1.2. Встановлення

```bash
sudo apt update
sudo apt install -y zsh fish
```

Пояснення:

- `apt update` оновлює локальний індекс пакетів із repositories;
- `apt install -y` встановлює пакети без додаткових запитань;
- при встановленні пакети автоматично реєструють свої shells у `/etc/shells`.

### 1.3. Перевірка встановлення

```bash
which zsh fish
zsh --version
fish --version
cat /etc/shells
```

**Output (приклад):**

```text
/usr/bin/zsh
/usr/bin/fish
zsh 5.9 (x86_64-ubuntu-linux-gnu)
fish, version 3.7.0
# /etc/shells: valid login shells
/bin/sh
/bin/bash
/usr/bin/bash
/bin/rbash
/usr/bin/rbash
/usr/bin/sh
/bin/dash
/usr/bin/dash
/bin/zsh
/usr/bin/zsh
/usr/bin/fish
```

> Якщо shell не з'явився у `/etc/shells`, його додають вручну:
> `echo "$(which fish)" | sudo tee -a /etc/shells`.
> Без запису в цьому файлі `chsh` може відмовити в зміні shell.

### 1.4. Інші shells (для довідки)

Інші інтерпретатори встановлюються тим самим способом:

```bash
sudo apt install -y ksh tcsh dash
```

### 1.5. Короткий опис можливостей

#### bash (Bourne Again Shell)

- shell за замовчуванням у Ubuntu для інтерактивної роботи;
- сумісний з POSIX `sh`, підтримує змінні, цикли, функції, масиви;
- історія команд (`history`), tab completion, aliases;
- найпоширеніший для написання shell scripts, тому більшість інструкцій у мережі написана саме під нього.

#### zsh (Z shell)

- синтаксично близький до bash, але має розширені можливості;
- потужний tab completion з меню вибору та виправленням помилок;
- розширений globbing (наприклад, `ls **/*.txt` для рекурсивного пошуку);
- підтримка тем і плагінів через фреймворки (Oh My Zsh);
- спільна історія між сесіями, правка слів у каталогах (`cd /u/l/b` розкривається в `/usr/local/bin`);
- підходить розробникам, які багато працюють у терміналі.

#### fish (Friendly Interactive Shell)

- орієнтований на зручність "з коробки" без налаштування;
- autosuggestions на основі історії та syntax highlighting під час набору;
- власний, чистіший синтаксис (`set VAR value`, `end` замість `fi`/`done`);
- веб-конфігуратор `fish_config`;
- **не сумісний з POSIX**: скрипти для bash/sh у ньому можуть не працювати без змін.

### 1.6. Порівняльна таблиця

| Можливість | bash | zsh | fish |
|---|---|---|---|
| POSIX-сумісність | так | переважно | ні |
| Autosuggestions | ні | через плагін | вбудовано |
| Syntax highlighting | ні | через плагін | вбудовано |
| Tab completion | базовий | розширений | розширений |
| Налаштування "з коробки" | мінімальні | потребує налаштування | зручні одразу |
| Поширеність у скриптах | найвища | висока | низька |

---

## Завдання 2. Створення груп і користувачів

### 2.1. Створення груп

```bash
sudo groupadd tech_support
sudo groupadd developers
sudo groupadd financiers
sudo groupadd founders
sudo groupadd guests
```

Перевірка:

```bash
getent group tech_support developers financiers founders guests
```

**Output (приклад):**

```text
tech_support:x:1001:
developers:x:1002:
financiers:x:1003:
founders:x:1004:
guests:x:1005:
```

# Єршов Олексій
### 2.2. Розподіл 10 користувачів по групах

| № | Користувач (username) | Група (group) | Роль |
|---|---|---|---|
| 1 | `ts_oleh` | tech_support | системний адміністратор |
| 2 | `ts_iryna` | tech_support | технічна підтримка |
| 3 | `dev_andriy` | developers | backend-розробник |
| 4 | `dev_marta` | developers | frontend-розробниця |
| 5 | `fin_olena` | financiers | бухгалтер |
| 6 | `fin_taras` | financiers | економіст |
| 7 | `fnd_viktor` | founders | CEO |
| 8 | `fnd_sofiia` | founders | співзасновниця |
| 9 | `gst_max` | guests | гість |
| 10 | `gst_anna` | guests | гостя |

### 2.3. Створення користувачів

Користувачів створюємо одразу з потрібною primary group (`-g`), home directory (`-m`), shell (`-s`) і коментарем (`-c`). Shells призначаються згідно із Завданням 3.

```bash
# Technical support: bash
sudo useradd -m -g tech_support -s /bin/bash -c "Oleh, system administrator"  ts_oleh
sudo useradd -m -g tech_support -s /bin/bash -c "Iryna, technical support"    ts_iryna

# Developers: zsh
sudo useradd -m -g developers -s /usr/bin/zsh -c "Andriy, backend developer"  dev_andriy
sudo useradd -m -g developers -s /usr/bin/zsh -c "Marta, frontend developer"  dev_marta

# Financiers: access to shells denied
sudo useradd -m -g financiers -s /usr/sbin/nologin -c "Olena, accountant"     fin_olena
sudo useradd -m -g financiers -s /usr/sbin/nologin -c "Taras, economist"      fin_taras

# Founders: fish
sudo useradd -m -g founders -s /usr/bin/fish -c "Viktor, CEO"                 fnd_viktor
sudo useradd -m -g founders -s /usr/bin/fish -c "Sofiia, co-founder"          fnd_sofiia

# Guests: access to shells denied
sudo useradd -m -g guests -s /usr/sbin/nologin -c "Guest Max"                 gst_max
sudo useradd -m -g guests -s /usr/sbin/nologin -c "Guest Anna"                gst_anna
```

### 2.4. Встановлення паролів

Паролі потрібні, щоб користувачі могли входити через `login`/`ssh`. Для демонстрації можна задати тимчасові:

```bash
echo "ts_oleh:Passw0rd#1" | sudo chpasswd
echo "ts_iryna:Passw0rd#2" | sudo chpasswd
echo "dev_andriy:Passw0rd#3" | sudo chpasswd
echo "dev_marta:Passw0rd#4" | sudo chpasswd
echo "fnd_viktor:Passw0rd#7" | sudo chpasswd
echo "fnd_sofiia:Passw0rd#8" | sudo chpasswd
```

> Для фінансистів і гостей паролі не задаємо, бо вхід у shell їм заборонений. Паролі вище є навчальними і після роботи облікові записи видаляються (див. розділ Cleanup).

### 2.5. Додаткові групи (опціонально)

Якщо потрібно, щоб адміністратори мали права `sudo`, а розробники доступ до `docker`:

```bash
sudo usermod -aG sudo ts_oleh
sudo usermod -aG sudo ts_iryna
```

### 2.6. Перевірка створених облікових записів

```bash
grep -E "ts_|dev_|fin_|fnd_|gst_" /etc/passwd
```

**Output (приклад):**

```text
ts_oleh:x:1001:1001:Oleh, system administrator:/home/ts_oleh:/bin/bash
ts_iryna:x:1002:1001:Iryna, technical support:/home/ts_iryna:/bin/bash
dev_andriy:x:1003:1002:Andriy, backend developer:/home/dev_andriy:/usr/bin/zsh
dev_marta:x:1004:1002:Marta, frontend developer:/home/dev_marta:/usr/bin/zsh
fin_olena:x:1005:1003:Olena, accountant:/home/fin_olena:/usr/sbin/nologin
fin_taras:x:1006:1003:Taras, economist:/home/fin_taras:/usr/sbin/nologin
fnd_viktor:x:1007:1004:Viktor, CEO:/home/fnd_viktor:/usr/bin/fish
fnd_sofiia:x:1008:1004:Sofiia, co-founder:/home/fnd_sofiia:/usr/bin/fish
gst_max:x:1009:1005:Guest Max:/home/gst_max:/usr/sbin/nologin
gst_anna:x:1010:1005:Guest Anna:/home/gst_anna:/usr/sbin/nologin
```

Перевірка членства в групах:

```bash
id ts_oleh
id dev_marta
getent group tech_support developers founders
```

**Output (приклад):**

```text
uid=1001(ts_oleh) gid=1001(tech_support) groups=1001(tech_support),27(sudo)
uid=1004(dev_marta) gid=1002(developers) groups=1002(developers)
tech_support:x:1001:
developers:x:1002:
founders:x:1004:
```

> У `/etc/group` primary group користувача не відображається у списку учасників, бо вона визначається полем GID у `/etc/passwd`. Тому поле учасників порожнє. Це нормальна поведінка.

---

## Завдання 3. Призначення shell за замовчуванням

| Група | Default shell | Шлях | Спосіб обмеження |
|---|---|---|---|
| Technical support | bash | `/bin/bash` | повний доступ |
| Developers | Interpreter 1: zsh | `/usr/bin/zsh` | повний доступ |
| Financiers | немає | `/usr/sbin/nologin` | вхід заборонено |
| Founders | Interpreter 2: fish | `/usr/bin/fish` | повний доступ |
| Guests | немає | `/usr/sbin/nologin` | вхід заборонено |

Shell задано при створенні користувачів параметром `-s` (див. п. 2.3). Для зміни вже існуючого користувача використовуються:

```bash
# Варіант 1: usermod (від root)
sudo usermod -s /usr/bin/zsh dev_andriy

# Варіант 2: chsh
sudo chsh -s /usr/bin/fish fnd_viktor

# Заборона доступу
sudo usermod -s /usr/sbin/nologin fin_olena
```

### Перевірка shell кожного користувача

```bash
for u in ts_oleh ts_iryna dev_andriy dev_marta fin_olena fin_taras fnd_viktor fnd_sofiia gst_max gst_anna; do
  printf "%-12s %s\n" "$u" "$(getent passwd "$u" | cut -d: -f7)"
done
```

**Output (приклад):**

```text
ts_oleh      /bin/bash
ts_iryna     /bin/bash
dev_andriy   /usr/bin/zsh
dev_marta    /usr/bin/zsh
fin_olena    /usr/sbin/nologin
fin_taras    /usr/sbin/nologin
fnd_viktor   /usr/bin/fish
fnd_sofiia   /usr/bin/fish
gst_max      /usr/sbin/nologin
gst_anna     /usr/sbin/nologin
```

### Як працює `nologin`

`/usr/sbin/nologin` не є повноцінним shell: при спробі інтерактивного входу він виводить повідомлення й одразу завершується. Службові сервісні облікові записи (`www-data`, `daemon` тощо) налаштовані так само. Альтернатива: `/bin/false`, який завершується мовчки, без повідомлення.

---

## Завдання 4. Демонстрація роботи груп

Для переходу в сесію користувача використовуємо `su -` (login shell) із правами `sudo`:

```bash
sudo su - <username>
```

Параметр `-` завантажує повне середовище користувача: shell, змінні, home directory.

### 4.1. Technical support (bash)

```bash
sudo su - ts_oleh
```

```bash
whoami
id
echo $SHELL
pwd
date
hostname
uname -a
lsb_release -d
uptime
free -h
df -h /
ls -la ~
exit
```

**Output (приклад):**

```text
ts_oleh
uid=1001(ts_oleh) gid=1001(tech_support) groups=1001(tech_support),27(sudo)
/bin/bash
/home/ts_oleh
Fri Nov  6 14:32:10 EET 2026
ubuntu-vm
Linux ubuntu-vm 6.8.0-48-generic #48-Ubuntu SMP PREEMPT_DYNAMIC x86_64 x86_64 x86_64 GNU/Linux
Description:    Ubuntu 24.04.1 LTS
 14:32:10 up  2:15,  1 user,  load average: 0.12, 0.08, 0.05
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       812Mi       1.9Gi        14Mi       1.3Gi       2.8Gi
Swap:          2.0Gi          0B       2.0Gi
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        25G  9.4G   14G  41% /
total 20
drwxr-x--- 2 ts_oleh tech_support 4096 Nov  6 14:30 .
drwxr-xr-x 12 root   root         4096 Nov  6 14:30 ..
-rw-r--r-- 1 ts_oleh tech_support  220 Nov  6 14:30 .bash_logout
-rw-r--r-- 1 ts_oleh tech_support 3771 Nov  6 14:30 .bashrc
-rw-r--r-- 1 ts_oleh tech_support  807 Nov  6 14:30 .profile
```

Адміністративні задачі (завдяки групі `sudo`):

```bash
sudo whoami
sudo systemctl status ssh --no-pager | head -n 3
sudo journalctl -n 3 --no-pager
```

**Output (приклад):**

```text
root
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-11-06 12:17:02 EET; 2h 15min ago
```
