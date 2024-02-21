# Система управления заказами в ресторане
**ВАЖНО:** для тестирования API необходим Postman
## Общие моменты
Всего API управляется 3 контроллерами: 
* UserController - для регистрации, входа, выхода пользователей, а также получения информации о текущем пользователе и списка блюд (достигается по пути `/api/user`)
* AdminController - для выполнения действий от лица администратора (добавление и удаление блюд в меню, а также получение статистики по ресторану (список блюд по убыванию популярности, средняя оценка блюд, количество заказов за определенный период) (достигается по пути `/api/admin`)
* VisitorController - для выполнения действий от лица посетителя (создание, расширение, оплата и отмена заказа, а также возможность оценки выполненного заказа) (достигается по пути `/api/visitor`)

Перед выполнением действия от лица пользователя необходим вход систему по логину, паролю и типу пользователя (admin или visitor). Если выполнить какое-то действие не будучи авторизованным в системе, то приложение выдаст соответствующую ошибку. В рамках одного сеанса допускается только один пользователь в системе   
Для отображения текущего пользователя, авторизованного в системе можно использовать **GET-запрос** по адресу (`localhost:8080/api/user/getCurrentUser`). Также допускается просмотр текущего меню без авторизации в системе посредством **GET-запроса** по адресу (`localhost:8080/api/user/getDishes`)  
## Регистрация, вход и выход
* Регистрация осуществляется посредством **POST-запроса** `localhost:8080/api/user/signUp` с указанием в **Query Params** следующих полей: username - логин (String), password - пароль (String), type - admin или visitor (String)
* Вход осуществляется посредством **GET-запроса** `localhost:8080/api/user/signIn` с указанием в **Query Params** следующих полей: username - логин (String), password - пароль (String), type - admin или visitor (String)
* Выход осуществляется посредством **DELETE-запроса** `localhost:8080/api/user/logOut` с указанием в **Query Params** следующих полей: username - логин (String), password - пароль (String), type - admin или visitor (String)

Каждый из перечисленных методов возвращает строку, содержащую сообщение о статусе выполнения метода. Если произошла ошибка, то в сообщение явно указано, что пошло не так
## Администратор
Авторизованный в системе администратор может выполнить следующие запросы:
* Создание блюда через **POST-запрос** `localhost:8080/api/admin/createDish` с указанием в **Query Params** следующих полей: name - имя блюда (String), description - описание блюда (String), price - цена блюда (BigDecimal), minutesToCook - время приготовления в минутах (Int)
* Удаление блюда через **DELETE-запрос** `localhost:8080/api/admin/deleteDish` с указанием в **Query Params** следующих полей: name - имя блюда (String), description - описание блюда (String), price - цена блюда (BigDecimal), minutesToCook - время приготовления в минутах (Int)
* Получение текущего дохода ресторана через **GET-запрос** `localhost:8080/api/admin/getRevenue`
* Получение списка блюд, отсортированного в порядке убывания популярности через **GET-запрос** `localhost:8080/api/admin/getDishesSortedByPopularity`
* Получение средней оценки заказов через **GET-запрос** `localhost:8080/api/admin/getAverageAssessment`
* Получение всех заказов, обработанных в заданный период времени, через **GET-запрос** `localhost:8080/api/admin/getAllOrdersInPeriod` с указанием в **Query Params** следующих полей: start - начало периода (LocalTime), end - конец периода (LocalTime)

Каждый из перечисленных методов в случае ошибки возвращает строку, содержащую подробную причину возникновения исключительной ситуации

## Посетитель
Авторизованный в системе пользователь может выполнить следующие запросы:
* Создание заказа через **POST-запрос** `localhost:8080/api/visitor/createOrder` с указанием в **Query Params** следующих полей: dishId - id блюда (Int), причём каждое новое блюдо указывается в новом параметре
* Расширение **действующего** заказа через **PUT-запрос** `localhost:8080/api/visitor/expandOrder` с указанием в **Query Params** следующих полей: dishId - id блюда (Int), причём каждое новое блюдо указывается в новом параметре. Предполагается, что у одного пользователя может быть не более одного действующего заказа
* Отмена **действующего** заказа через **DELETE-запрос** `localhost:8080/api/visitor/cancelOrder`
* Оплата **законченного** заказа через **POST-запрос** `localhost:8080/api/visitor/payOrder` c указанием в **Query Params** следующих полей: orderId - id заказа, который пользователь хочет оплатить (Int), sum - сумма оплаты заказа (BigDecimal), причем сумма оплаты может превышать достаточную. В таком случае считается, что остаток идёт в чаевые официантам, но меньшей достаточной сумма быть не может. В таком случае будет выведено в консоль соответствующее сообщение. В случае, если входные данные валидны (сумма как минимум равна сумме заказа и такой заказ в принципе существует), пользователю предлагается подтвердить введенные данные через консольный ввод с возможностью ввести новую сумму оплаты (не меньшую достаточной)
* Дать отзыв **законченному** заказу через **POST-запрос** `localhost:8080/api/visitor/giveFeedback` c указанием в **Query Params** следующих полей: orderId - id заказа, который пользователь хочет оценить (Int), assessment - оценка заказа (от 1 до 5), text - текст отзыва (String). В случае, если заказа с введенным id не существует или введена невалидная оценка, API выведет соответствующее сообщение

Каждый из перечисленных методов в случае ошибки возвращает строку, содержащую подробную причину возникновения исключительной ситуации
## Примечание
* Все строки, вводимые пользователем не должны превышать 255 символов
* Обработка заказов происходит в многопоточном режиме, симулируя процесс приготовления в формате 1 минута = 1 секунда (то есть заказ, заявленный в базе данных как требующий 5 минут для приготовления, в программе будет обрабатываться 5 секунд)