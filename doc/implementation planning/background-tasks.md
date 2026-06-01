we make the current session status bar float and display generally short messages for all background tasks like:
    "Waiting Xs to reconnect... [try now]"
    "Reconnecting..."
    "Sending message 1 of 10..."

As it is floating, it doesn't pop they layout and mess up the scroll offset of the timeline.
Need to find a good place to float it though. Preferably on top for visibility, but it could occlude the room header. Perhaps bottom left?

If more than 1 background thing is going on at the same time we display (1/x).
If you click the button status bar anywhere, it takes you to a page adjacent to the room view (and e.g. in the future the settings) and you get an overview of all running background tasks.


Перевод

мы настраиваем плавающую строку состояния текущего сеанса и обычно отображаем короткие сообщения для всех фоновых задач, например::
    "Ожидание Xs для повторного подключения... [попробуйте сейчас]"
    "Повторное подключение..."
    "Отправлено сообщение 1 из 10..."

Поскольку он плавающий, он не выскакивает из макета и не изменяет смещение прокрутки временной шкалы.
Однако нужно найти подходящее место для его размещения. Желательно сверху для наглядности, но это может закрыть заголовок комнаты. Возможно, внизу слева?

Если одновременно происходит более 1 фонового события, мы показываем (1/x).
Если вы нажмете на кнопку в строке состояния в любом месте, вы перейдете на страницу, расположенную рядом с обзором комнаты (и, например, в будущем - с настройками), и получите обзор всех запущенных фоновых задач.