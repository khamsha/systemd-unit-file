# systemd-unit-file
Сервис, который мониторит файл с логами на предмет наличия ключевого слова (параметры задаются в /etc/sysconfig).\
Также в репозитории настраивается конфигурация unit-файлов на примере сервисов spawn-fcgi и apache httpd

## Задание
1. Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/sysconfig или в /etc/default)
1. Установить spawn-fcgi и переписать init-скрипт на unit-файл (имя service должно называться так же: spawn-fcgi)
1. Дополнить unit-файл httpd (он же apache2) возможностью запустить несколько инстансов сервера с разными конфигурационными файлами

## Реализация
Создан [Vagrantfile](Vagrantfile) с встроенным shell скриптом, выполняющим действия, необходимые для реализации задания.\
Пояснения к скрипту указаны в комментариях

## Заметки по скрипту
[Правила использования EOF / Here Documents](https://www.gnu.org/software/bash/manual/bash.html#Here-Documents) c кавычками и без:\
If any part of *word* is quoted, the *delimiter* is the result of quote removal on *word*, and the lines in the here-document are not expanded.\
If *word* is unquoted, all lines of the here-document are subjected to parameter expansion, command substitution, and arithmetic expansion, the character sequence `\newline` is ignored, and ‘\’ must be used to quote the characters ‘\’, ‘$’, and '\`'.

Обратить внимание на правила регулярных выражений и экранирование специальных символов в `sed`, особенно при выполнении команды append (*a*)
