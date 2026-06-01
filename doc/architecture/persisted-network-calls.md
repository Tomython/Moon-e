# General Pattern of implementing a persisted network call

 1. do network request
 1. start transaction
 1. write result of network request into transaction store, keeping differences from previous store state in local variables
 1. close transaction
 1. apply differences applied to store to in-memory data
 1. emit events for changes


Перевод

# Общая схема реализации сохраняемого сетевого вызова

 1. выполняем сетевой запрос
 1. запускаем транзакцию
 1. записываем результат сетевого запроса в хранилище транзакций, сохраняя отличия от предыдущего состояния хранилища в локальных переменных
 1. закрываем транзакцию
 1. применяем различия, применяемые для сохранения данных в памяти
 1. генерируйте события для внесения изменений