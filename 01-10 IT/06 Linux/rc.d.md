rc - run command.

В linux существуют 7 уровней загрузки операционной системы. В нашем случае нулевой уровень — это режим восстановления, первый — это запуск в одиночном режиме под root. 2-5 загрузка в мульти пользовательском режиме (т.е. обычный режим). Отличаются они лишь набором стартовых скриптов. 6 уровень это перезагрузка. Скрипты берутся из директорий, которые расписаны в inittab. Наша система по умолчанию загружается на 5 уровне, посмотрим что-же находится в директории /etc/rc5.d

![[Pasted image 20230817214228.png]]

Скрипты в этом каталоге выполняются каждый раз при старте системы. А если быть точнее это лишь символьные ссылки на сами скрипты. Первая буква означает S(start) K(kill или stop) для изменения порядка скриптов меняется цифра, т.е. запуск скриптов выполняется по возрастанию.

Как мы уже поняли в каталогах /etc/rc[0-6].d/* лежат символьные ссылки на скрипты. Где цифры от [0-6] это уровень загрузки у inittab или systemd. Мы можем менять руками порядок запуска, убирать и добавлять. По сути systemd пробежится по всем файлам и попытается их инициализировать при старте системы.