Программное обеспечение `Git` представляет собой распределённую систему для контроля версий, которая позволяет управлять исходным кодом. Изначально, `Git` был разработан  `Линусом Торвальдсом` для удобной разработки `Linux Kernel`.

Система контроля версий представляет собой программное обеспечение, которое помогает разработчикам работать в команде и сохранять историю разработки проекта.
Они могут классифицироваться на централизованные и децентрализованные.

Централизованные системы используют центральный сервер для хранения всех файлов. Но данный тип систем подвержен большой опасности – выходу из строя центрального сервера. Если с сервером что-то происходит, то работа прекращается.

Клиент децентрализованной системы не только проверяет крайние изменения базовой директории, но также является и резервным хранилищем проекта. Если сервер выходит из строя, базовая версия может быть восстановлена с помощью клиентской машины.

---

Основное отличие `Git` от любой другой системы контроля версий это подход к работе со своими данными. Концептуально, большинство других систем хранят информацию в виде списка изменений в файлах. Эти системы представляют хранимую информацию в виде набора файлов и изменений, сделанных в каждом файле, по времени.

В свою очередь `Git` не хранит и не обрабатывает данные таким способом. Вместо этого, каждый раз, когда вы делаете коммит, система создает снимок текущего состояния репозитория, и сохраняет ссылку на этот снимок. Для эффективности, если файлы не были изменены, `Git` не запоминает эти файлы вновь, а только создаёт ссылку на предыдущую версию идентичного файла, который уже сохранён.

В `Git` для всего вычисляется хеш-сумма, и только потом происходит сохранение. В дальнейшем обращение к сохранённым объектам происходит по этой хеш-сумме. Это значит, что невозможно изменить файл так, чтобы система `Git` не узнала об этом. Это встроено в `Git` на низком уровне и является неотъемлемой частью его философии. 

Хеш представляет строку длиной в 40 шестнадцатеричных символов, она вычисляется на основе содержимого файла или структуры каталога. На самом деле, Git сохраняет все объекты в свою базу данных не по имени, а по хеш-сумме содержимого объекта.

У `Git` есть три основных состояния, в которых могут находиться внутренние         файлы: `modified(изменён)`,  `staged(индексирован)`, `committed (зафиксирован)`:

К изменённым относятся файлы, которые поменялись, но ещё не были зафиксированы.
К индексированным измененные файлы, отмеченные для включения в коммит.
К зафиксированным файлы, которые уже сохранены в вашей локальной базе.

![[Pasted image 20240302142052.png]]

---

В состав `Git` входит утилита `git config`, которая позволяет настраивать параметры, контролирующие все аспекты работы `Git`, а также его внешний вид. Эти параметры для настройки могут быть сохранены в трех местах на трех различных уровнях:

Файл `[path]/etc/gitconfig` содержит глобальные значения, общие для всех пользователей системы и для всех их репозиториев. Если при запуске `git config` указать параметр `--system`, то параметры будут читаться и сохраняться именно в него.

Файл `~/.gitconfig` или `~/.config/git/config` хранит настройки конкретного пользователя. Этот файл используется при указании параметра `--global` и применяется ко всем репозиториям, с которыми вы работаете системе.

Файл `config` в каталоге `Git` репозитория, который вы используете в данный момент, хранит настройки конкретного репозитория. Вы можете заставить `Git` читать и писать в этот файл с помощью параметра `--local` по умолчанию. Неудивительно, что вам нужно находиться где-то в репозитории `Git`, чтобы эта опция работала правильно.

Чтобы посмотреть все установленные настройки и узнать где именно они заданы, используйте команду: `git config --list --show-origin`. Также вы можете проверить значение конкретного ключа, выполнив `git config <key>`:

```sh
git config --list --show-origin
git config user.name
```

Первое, что вам следует сделать после установки `Git` - указать ваше имя и адрес электронной почты. Это важно, потому что каждый коммит в `Git` содержит эту информацию, и она включена в коммиты, передаваемые вами.

```sh
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```

Когда вы инициализируете репозиторий командой `git init`, создается ветка с именем `master` по умолчанию. Вы можете задать другое имя для создания ветки по умолчанию.

```sh
git config --global init.defaultBranch main
```

---

**==Init==**. Эта команда создаёт в текущем каталоге новый подкаталог с именем `.git`, содержащий все необходимые файлы репозитория структуру `Git` репозитория. На этом этапе ваш проект ещё не находится под версионным контролем.

```sh
git init
```