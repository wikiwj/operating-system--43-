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