## Promises, async/await and indexedDB

Doesn't indexedDB close your transaction if you don't queue more requests from an idb event handler?
So wouldn't that mean that you can't use promises and async/await when using idb?

It used to be like this, and for IE11 on Win7 (not on Windows 10 strangely enough), it still is like this.
Here we manually flush the promise queue synchronously at the end of an idb event handler.

In modern browsers, indexedDB transactions should only be closed after flushing the microtask queue of the event loop,
which is where promises run.

Keep in mind that indexedDB events, just like any other DOM event, are fired as macro tasks.
Promises queue micro tasks, of which the queue is drained before proceeding to the next macro task.
This also means that if a transaction is completed, you will only receive the event once you are ready to process the next macro tasks.
That doesn't prevent any placed request from throwing TransactionInactiveError though.

## TransactionInactiveError in Safari

Safari doesn't fully follow the rules above, in that if you open a transaction,
you need to "use" (not sure if this means getting a store or actually placing a request) it straight away,
without waiting for any *micro*tasks. See comments about Safari at https://github.com/dfahlander/Dexie.js/issues/317#issue-178349994.

Another failure mode perceived in Hydrogen on Safari is that when the (readonly) prepareTxn in sync wasn't awaited to be completed before opening and using the syncTxn.
I haven't found any documentation online about this at all. Awaiting prepareTxn.complete() fixed the issue below. It's strange though the put does not fail.

## Diagnose of problem

What is happening below is:
 - in the sync loop:
    - we first open a readonly txn on inboundGroupSessions, which we don't use in the example below
    - we then open a readwrite txn on session, ... (does not overlap with first txn)
        - first the first incremental sync on a room (!YxKeAxtNcDZDrGgaMF:matrix.org) it seems to work well
        - on a second incremental sync for that same room, the first get throws TransactionInactiveError for some reason.
        - the put in the second incremental sync somehow did not throw.

So it looks like safari doesn't like (some) transactions still being active while a second one is being openened, even with non-overlapping stores.
For now I haven't awaited every read txn in the app, as this was the only place it fails, but if this pops up again in safari, we might have to do that.

Keep in mind that the `txn ... inactive` logs are only logged when the "complete" or "abort" events are processed,
which happens in a macro task, as opposed to all of our promises, which run in a micro task.
So the transaction is likely to have closed before it appears in the logs.

```
[Log] txn 4504181722375185 active on inboundGroupSessions
[Log] txn 861052256474256 active on session, roomSummary, roomState, roomMembers, timelineEvents, timelineFragments, pendingEvents, userIdentities, groupSessionDecryptions, deviceIdentities, outboundGroupSessions, operations, accountData
[Info] hydrogen_session_5286139994689036.session.put({"key":"sync","value":{"token":"s1572540047_757284957_7660701_602588550_435736037_1567300_101589125_347651623_132704","filterId":"2"}})
[Info] hydrogen_session_5286139994689036.userIdentities.get("@bwindels:matrix.org")
[Log] txn 4504181722375185 inactive
[Log]  * applying sync response to room !YxKeAxtNcDZDrGgaMF:matrix.org ...
[Info] hydrogen_session_5286139994689036.roomMembers.put({"roomId":"!YxKeAxtNcDZDrGgaMF:matrix.org","userId":"@bwindels:matrix.org","membership":"join","avatarUrl":"mxc://matrix.org/aerWVfICBMcyFcEyREcivLuI","displayName":"Bruno","key":"!YxKeAxtNcDZDrGgaMF:matrix.org|@bwindels:matrix.org"})
[Info] hydrogen_session_5286139994689036.roomMembers.get("!YxKeAxtNcDZDrGgaMF:matrix.org|@bwindels:matrix.org")
[Info] hydrogen_session_5286139994689036.timelineEvents.add({"fragmentId":0,"eventIndex":2147483658,"roomId":"!YxKeAxtNcDZDrGgaMF:matrix.org","event":{"content":{"body":"haha","msgtype":"m.text"},"origin_server_ts":1601457573756,"sender":"@bwindels:matrix.org","type":"m.room.message","unsigned":{"age":8360},"event_id":"$eD9z73-lCpXBVby5_fKqzRZzMVHiPzKbE_RSZzqRKx0"},"displayName":"Bruno","avatarUrl":"mxc://matrix.org/aerWVfICBMcyFcEyREcivLuI","key":"!YxKeAxtNcDZDrGgaMF:matrix.org|00000000|8000000a","eventIdKey":"!YxKeAxtNcDZDrGgaMF:matrix.org|$eD9z73-lCpXBVby5_fKqzRZzMVHiPzKbE_RSZzqRKx0"})
[Info] hydrogen_session_5286139994689036.roomSummary.put({"roomId":"!YxKeAxtNcDZDrGgaMF:matrix.org","name":"!!!test8!!!!!!","lastMessageBody":"haha","lastMessageTimestamp":1601457573756,"isUnread":true,"encryption":null,"lastDecryptedEventKey":null,"isDirectMessage":false,"membership":"join","inviteCount":0,"joinCount":2,"heroes":null,"hasFetchedMembers":false,"isTrackingMembers":false,"avatarUrl":null,"notificationCount":5,"highlightCount":0,"tags":{"m.lowpriority":{}}})
[Log] txn 861052256474256 inactive
[Info] syncTxn committed!!

... two more unrelated sync responses ...

[Log] starting sync request with since s1572540191_757284957_7660742_602588567_435736063_1567300_101589126_347651632_132704 ...
[Log] txn 8104296957004707 active on inboundGroupSessions
[Log] txn 2233038992157489 active on session, roomSummary, roomState, roomMembers, timelineEvents, timelineFragments, pendingEvents, userIdentities, groupSessionDecryptions, deviceIdentities, outboundGroupSessions, operations, accountData
[Info] hydrogen_session_5286139994689036.session.put({"key":"sync","value":{"token":"s1572540223_757284957_7660782_602588579_435736078_1567300_101589130_347651633_132704","filterId":"2"}})
[Log]  * applying sync response to room !YxKeAxtNcDZDrGgaMF:matrix.org ...
[Info] hydrogen_session_5286139994689036.roomMembers.get("!YxKeAxtNcDZDrGgaMF:matrix.org|@bwindels:matrix.org")
[Warning] stopping sync because of error
[Error] StorageError: get("!YxKeAxtNcDZDrGgaMF:matrix.org|@bwindels:matrix.org") failed on txn with stores accountData, deviceIdentities, groupSessionDecryptions, operations, outboundGroupSessions, pendingEvents, roomMembers, roomState, roomSummary, session, timelineEvents, timelineFragments, userIdentities on hydrogen_session_5286139994689036.roomMembers: (name: TransactionInactiveError) (code: 0) Failed to execute 'get' on 'IDBObjectStore': The transaction is inactive or finished.
    (anonymous function)
    asyncFunctionResume
    (anonymous function)
    promiseReactionJobWithoutPromise
    promiseReactionJob
[Log] newStatus – "SyncError"
[Log] txn 8104296957004707 inactive
[Log] txn 2233038992157489 inactive
```


Перевод

## Promises, async/await и IndexedDB

Не закрывает ли IndexedDB вашу транзакцию, если вы не ставите в очередь дополнительные запросы от обработчика событий idb?
Не означает ли это, что вы не можете использовать promises и async/await при использовании idb?

Раньше это было так, и для IE11 в Win7 (как ни странно, не в Windows 10) это все еще так.
Здесь мы вручную очищаем очередь promise синхронно в конце обработчика событий idb.

В современных браузерах транзакции IndexedDB должны закрываться только после очистки очереди микрозадач цикла обработки событий,
в котором выполняются обещания.

Имейте в виду, что события IndexedDB, как и любое другое событие DOM, запускаются как макрозадачи.
Обещает поставить в очередь микрозадачи, из которых очередь будет очищена перед переходом к следующей макрозадаче.
Это также означает, что если транзакция завершена, вы получите событие только после того, как будете готовы к обработке следующих макрозадач.
Однако это не мешает любому размещенному запросу выдавать ошибку TransactionInactiveError.

## TransactionInactiveError в Safari

Safari не полностью соответствует приведенным выше правилам в том смысле, что если вы открываете транзакцию,
вам нужно "использовать" (не уверен, означает ли это получение хранилища или фактическое размещение запроса) его сразу же,
не дожидаясь выполнения каких-либо "микро-задач". Смотрите комментарии о Safari по адресу https://github.com/dfahlander/Dexie.js/issues/317#issue-178349994.

Еще один сбой, обнаруженный в Hydrogen в Safari, заключается в том, что перед открытием и использованием syncTxn не было ожидаемого завершения процесса синхронизации (только для чтения) prepareTxn.
Я вообще не нашел в Интернете никакой документации по этому поводу. В ожидании prepareTxn.complete() исправлена проблема, описанная ниже. Странно, но put не завершается ошибкой.

## Диагностика проблемы

Что происходит ниже, так это:
 - в цикле синхронизации:
    - сначала мы открываем txn только для чтения в inboundGroupSessions, который мы не используем в примере ниже
    - затем мы открываем txn для чтения и записи в сеансе, ... (не совпадает с первым txn)
        - сначала выполняется первая инкрементная синхронизация в комнате (!YxKeAxtNcDZDrGgaMF:matrix.org) кажется, это работает хорошо
        - при второй инкрементальной синхронизации для той же комнаты первый get по какой-то причине выдает ошибку TransactionInactiveError.
        - put во второй инкрементальной синхронизации почему-то не выдал ошибку.

Таким образом, похоже, что safari не нравится, что (некоторые) транзакции все еще активны, в то время как открывается вторая, даже с непересекающимися хранилищами.
На данный момент я не стал дожидаться каждого прочитанного txn в приложении, поскольку это было единственное место, где произошел сбой, но если это снова появится в safari, нам, возможно, придется это сделать.

Имейте в виду, что `txn ... журналы неактивных транзакций регистрируются только при обработке событий "завершить" или "прервать",
что происходит в макрозадании, в отличие от всех наших promises, которые выполняются в микрозадании.
Таким образом, транзакция, скорее всего, была закрыта до того, как она появилась в журналах.

```
[Log] тех. номер 4504181722375185 активен во внутренних групповых сессиях
[Log] txn 861052256474256 активен в сеансе, суммах комнат, состоянии комнат, членах комнаты, событиях временной линии, фрагментах временной линии, отложенных событиях, идентификаторах пользователей, групповых сессиях, шифрованиях устройств, внешних групповых сессиях, операциях, данных учетной записи.
[Информация] hydrogen_session_5286139994689036.session.put({"key":"sync","value":{"token":"s1572540047_757284957_7660701_602588550_435736037_1567300_101589125_347651623_132704","filterId":"2"}})
[Информация] hydrogen_session_5286139994689036.Идентификаторы пользователя.получить("@bwindels:matrix.org")
[Log] технический номер 4504181722375185 неактивен
[Log] * применяем синхронизацию к комнате!YxKeAxtNcDZDrGgaMF:matrix.org ...
[Информация] hydrogen_session_5286139994689036.roomMembers.put({"roomId":"!YxKeAxtNcDZDrGgaMF:matrix.org","userId":"@bwindels:matrix.org","membership":"join","avatarUrl":"mxc://matrix.org/aerWVfICBMcyFcEyREcivLuI","displayName":"Bruno","ключ":"!YxKeAxtNcDZDrGgaMF:matrix.org/@bwindels:matrix.org"})
[Информация] hydrogen_session_5286139994689036.roomMembers.get("!YxKeAxtNcDZDrGgaMF:matrix.org/@bwindels:matrix.org")
[Информация] hydrogen_session_5286139994689036.timelineEvents.add({"fragmentId":0,"eventIndex":2147483658,"roomId":"!YxKeAxtNcDZDrGgaMF:matrix.org","event":{"content":{"body":"haha","msgtype":"m.text"},"origin_server_ts":1601457573756,"sender":"@bwindels:matrix.org","type":"m.room.message","unsigned":{"age":8360},"event_id":"$eD9z73-lCpXBVby5_fKqzRZzMVHiPzKbE_RSZzqRKx0"},"displayName":"Bruno","avatarUrl":"mxc://matrix.org/aerWVfICBMcyFcEyREcivLuI","key":"!YxKeAxtNcDZDrGgaMF:matrix.org/00000000/8000000a","eventIdKey":"!YxKeAxtNcDZDrGgaMF:matrix.org|$eD9z73-lCpXBVby5_fKqzRZzMVHiPzKbE_RSZzqRKx0"})
[Информация] hydrogen_session_5286139994689036.roomSummary.put({"roomId":"!YxKeAxtNcDZDrGgaMF:matrix.org","name":"!!!test8!!!!!!","lastMessageBody":"haha","lastMessageTimestamp":1601457573756,"isUnread":true,"encryption":null,"lastDecryptedEventKey":null,"isDirectMessage":false,"membership":"join","inviteCount":0,"joinCount":2,"heroes":null,"hasFetchedMembers":false,"isTrackingMembers":false,"avatarUrl":null,"notificationCount":5,"Количество выделенных объектов":0,"теги":{"m.lowpriority":{}}})
[Log] txn 861052256474256 неактивен
[Информация] syncTxn зафиксирован!!

... еще два несвязанных ответа на синхронизацию ...

[Журнал] запускаю запрос на синхронизацию с s1572540191_757284957_7660742_602588567_435736063_1567300_101589126_347651632_132704 ...
[Log] тех. номер 8104296957004707 активен во внутренних групповых сессиях
[Log] txn 2233038992157489 активен в сеансе, суммах комнат, состоянии комнат, членах комнаты, событиях временной линии, фрагментах временной линии, отложенных событиях, идентификаторах пользователей, групповых сессиях, шифрованиях устройств, внешних групповых сессиях, операциях, данных учетной записи.
[Информация] hydrogen_session_5286139994689036.session.put({"key":"sync","value":{"token":"s1572540223_757284957_7660782_602588579_435736078_1567300_101589130_347651633_132704","filterId":"2"}})
[Журнал] * применяем синхронизацию к комнате!YxKeAxtNcDZDrGgaMF:matrix.org ...
[Информация] hydrogen_session_5286139994689036.roomMembers.get("!YxKeAxtNcDZDrGgaMF:matrix.org/@bwindels:matrix.org")
[Предупреждение] остановка синхронизации из-за ошибки
[Ошибка] Ошибка хранения: get("!YxKeAxtNcDZDrGgaMF:matrix.org/@bwindels:matrix.org ") произошел сбой в txn при хранении данных учетной записи, идентификаторов устройств, групповых сессий, операций, внешних групповых сессий, ожидаемых событий, членов комнаты, состояния комнаты, суммы комнаты, сеанса, временных событий, временных фрагментов, идентификаторов пользователя в hydrogen_session_5286139994689036.roomMembers: (имя: TransactionInactiveError) (код: 0) Не удалось выполнить выполнить "получить" в "IDBObjectStore": Транзакция неактивна или завершена.
    (анонимная функция)
    asyncFunctionResume
    (анонимная функция)
    Обещай действовать без обещания
    promiseReactionJob
[Log] Новый статус – "Синхронизация"
[Log] технический номер 8104296957004707 неактивен
[Логин] технический номер 2233038992157489 неактивен
``