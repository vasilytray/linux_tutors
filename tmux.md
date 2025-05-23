# Менеджер терминалов Tmux
Краткая шпоргалка

**tmux** — это менеджер терминалов, к которому удобно подключаться и отключаться, не теряя при этом процессы и историю. 
Как screen, только лучше (в первую очередь потому, что использует модель клиент—сервер).

Вашему вниманию предлагается минималистическая шпаргалка, позволяющая быстро начать использовать tmux, а уж тонкую настройку, продвинутые команды и бесконечные хот-кеи желающие найдут, набрав man tmux.

### Установка tmux
```sh
sudo apt-get install tmux
```

### Руководство по tmux

```sh
man tmux
```

### Запуск tmux
Хороший вариант запустить tmux:
```sh
tmux attach || tmux new
```
сперва пытаемся подключиться к уже существующему серверу tmux, если он существует; 
если такого ещё нет — создаёте новый.
Основное функциональное сочетание клавиш в tmux
**Ctrl+b** - нажав его  потом нажимаете нужную горячую кнопку (не все сразу!)

### Дальше просто хоткеи

Ctrl+b d — отключиться.

#### Работа с окнами

| горячая клавиша | функция |
|---------|------------------|
| Ctrl+b  c | — создать окошко; |
|Ctrl+b  0...9 | — перейти в такое-то окошко; |
|Ctrl+b  p | — перейти в предыдущее окошко;|
|Ctrl+b  n |— перейти в следующее окошко;|
|Ctrl+b  l |— перейти в предыдущее активное окошко (из которого вы переключились в текущее);|
|Ctrl+b  & |— закрыть вкладку (а можно просто набрать exit в терминале).|
|Ctrl+b  , | - показать название окна;|

#### В одном окошке может быть много панелей
| горячая клавиша | функция |
|---------|------------------|
|Ctrl+b  % |— разделить текущую панель на две, по вертикали;|
|Ctrl+b  " |— разделить текущую панель на две, по горизонтали (это кавычка, которая около Enter, а не Shift+2);|
|Ctrl+b  →←↑↓ |— переходить между панелями;|
|Ctrl+b  o |-  поменять панели местами;|
|Ctrl+b  ; |- перейти к послежней панели;|
|Ctrl+b  q |- показать номера панелей;|
|Ctrl+b  x |— закрыть панель (а можно просто набрать exit в терминале).|

#### Скроллинг
| горячая клавиша | функция |
|---------|------------------|
|Ctrl+b PgUp 
или Ctrl+b [ |— вход в «режим копирования», после чего:|
|PgUp, PgDown |— скроллинг;|
|q |— выход из «режима копирования».|

#### Дополнительные

| горячая клавиша | функция |
|---------|------------------|
|Ctrl+b  d	|отключиться от текущего tmux	|
|Ctrl+b  t	|показать большие часы	|
|Ctrl+b  ?	|показать список ярлыков	|
|Ctrl+b  :	|prompt|
|Ctrl+b  i	|показать сообщение|
