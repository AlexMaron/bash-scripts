## Samba script repo.

### Samba Server
#### Описание конфигурации Samba сервера.

Здесь храняться файлы конфигурации, скрипты для администрирования Samba и пояснения к ним.  
Samba настроена на использование ACL (Access Control List) позволяющее более гибко управлять политикой доступа к общим ресурсам.  
Настроено логирование где можно просматривать кто, когда и что делал с файлами.  
  
## Конфигурация
#### Путь к файлу конфигурации:
/etc/samba/smb.conf  

## Логи
#### Путь к логам samba:
/var/log/samba/  
Логи чтения и записи файлов пишутся в файл audit.log  
Логи авторизации и ACL пишутся в файлы m%.log  
m% - NetBIOS name of the client machine  
  
## Администрирование
https://$IP:9090  
#### Добавление UNIX пользователя:
* useradd -M -g smbuser -s /sbin/nologin $USERNAME \
* passwd $USERNAME

#### Добавление Samba пользователя:
smbpasswd -a $USERNAME

#### Добавление UNIX пользователя в дополнительную группу:
usermod -aG $GROUPNAME $USERNAME

#### Выдача ACL прав для папок
**Просмотр прав для папки/файла:**  
getfacl [path]  
**Пример:** getfacl /srv/
  
**Папка без прав будет иметь такой вывод:**  
 \# file: srv/samba/
 \# owner: root  
 \# group: root  
 *user::rwx  
 *group::rwx  
 *other::rwx  
  
Когда выдаются права добавляется дополнительная строка где будет указана группа или пользователь и права.  
  
**Добавление прав для группы:**  
setfacl -m g:$GROUPNAME:$PERMISSIONS $PATH  
**Пример:** setfacl -m g:smbadm:rwx /srv/  
**Добавление прав для отдельного пользователя:**  
setfacl -m u:$USERNAME:$PERMISSINS $PATH  
**Удаление прав:**  
setfacl -x g:$GROUPNAME  
setfacl -x $USERNAME  
  
Каждая папка/файл может иметь отдельные права для доступа  

#### Скрипты
Скрипты можно запускать напрямую из shell.
Название скрипта | Опции | Описание
:-----   | :---- | :-------:
${basename} | --- | [-vdr] [-n USERNAME] [-g GROUP,GROUP...] [-c COMMENT] [-a ACL PATH] [-p PERMISSION] [-s PASSWORD MODE].<br/>Автоматический комбаин который позволяет добавлять unix и samba пользователя, добавлять к ним автоматически генирируемый пароль, добавлять индивидуальный доступ пользователю к определенному ресурсу, удалять unix и samba пользователей.
--- | -v | Verbose mode. Выводит в shell информацию о пользователе, группах, пароле и индивидуальному ACL доступу к папке/файлу.
--- | -d | Dry run mode. Проверочный запуск скрипта только отображающий информацию о том как будет работать скрипт, без запуска функциональных команд.
--- | -n USERNAME | Добавляет переменную имени пользователя в скрипт. Поcле опции -n пишется имя пользователя которого необходимо добавить.
--- | -g GROUP,GROUP... | Добавляет переменную группы пользователя в скрип. После опции -g пишется имена групп пользователя в которых он должен состоять. Несколько групп обозначаются в скобках, пример -g "group1,group2,group3".
--- | -c COMMENT | Добавляет переменную комментария к пользователю. Пример -c "IP 192.168.0.156, Стас Борецкий, продюссер".
--- | -a ACL PATH | Определяет путь к папке для индивидуального доступа через ACL. Пример -a /srv/samba/scanner.
--- | -p PERMISSION | Добавляет переменную ключей ACL доступа, которые добавляются к ресурсу обозначенному с помощью опции '-a'. Пример -p rwx. Где r - read, w - write, x - execute. Если необходимо дать доступ, например, только на чтение, ключи wx опускаются и на их место ставится знак минуса: -p r--
--- | -P PASSWORD MODE | Добавляет переменную запускающую установку пароля пользователю. Есть два режима работы, same и random. Same устанавливает легкий пароль который идентичен имени пользователя, random генерирует восьмизначный пароль разных регистров со спецсимволом.
--- | -r | Опция которая запускает режим удаления пользователя. Опция -n USERNAME обязательна к использованию с этим ключем, опционально -a ACL PATH. Удаляет unix пользователя, samba пользователя и очищает ACL доступ по заданному пути если тот был ранее установлен.

Скрипт пишет лог по адресу /var/log/${basename}.log  

03.03.2021.  
Добавлен скрипт инкрементального бэкапа по адресу /srv/smb_backup.sh  
Скрипт добавлен в Crontab и выполняется раз в сутки в 3:00  
