
# Integrate payments for Exchanges, Shops, etc

Для интеграции вам нужно всего две команды: послать актив и получить данные о приходе актива на заданный счет

Вам нужно все входящие транзакции (полученные через API или  RPC) сохранять в промежуточной таблице. Так чтобы когда приходит новый блок проверять сколько они уже получили подтверждений и если какие-то транзакции получили достаточно подтверждений то пытаемся найти ее в блокчейн по подписи - если ответ положительный, удаляем ее из промежуточной таблицы "Ожидание Подтверждения" и вносим в "Обработанные" и производим нужную нам обработку полученного платежа.

При этом если у вас не особо важные платеже, и вы не боитесь что они могут не подтвердиться то обработку можно запускать мгновенно, как только неподтвержденная транзакция появилась на ноде.

Отсылка в принципе так же происходит, хотя можно и не заморачиваться, подразумевая что ваши транзакции всегда подтвердятся.

### Способы взаимодействия с сетью блокчейн
Послать транзакцию можно двумя способами:
1. Через RPC - это самый простой способ, так как нода сама поставит текущее время, проверит баланс, доступ к кошельку и подпишет транзакцию. При этом конечно же нода должна стоять на этой же вычислительной машине, чтобы доступ к ней был по локальной сети - для безопасности секретных ключей в кошельке ноды.   
2. Через API - это более сложный способ, но зато нода вообще не нужна. Транзакция собирается самостоятельно на стороне клиента и подписывается так же на стороне клиента. И уже в готовом виде посылается через любой узел сети на которой открыт порт API (работает блокэксплорер). Для облегчения сборки транзакции есть библиотеки на PHP и примеры на JS (см. ниже)

### Транзакции выплаты актива

#### RPC
GET r_send/{creator}/{recipient}?feePow={feePow}&assetKey={assetKey}&amount={amount}&title={title}&message={message}&encoding={encoding}&encrypt=true&password={password}

    /**  
     * send and broadcast GET
     * @param creatorStr address in wallet  
     * @param recipientStr recipient  
     * @param feePowStr fee  
     * @param assetKey assetKey  
     * @param amount amount  
     * @param title title or head  
     * @param message message  
     * @param encoding code if exist is text (not required field)  
     * @param encrypt bool value encrypt (not required field)  
     * @param password password  
     * @return JSON row  
     * * <h2>Example request</h2>  
      * GET r_send/7GvWSpPr4Jbv683KFB5WtrCCJJa6M36QEP/79MXsjo9DEaxzu6kSvJUauLhmQrB4WogsH?message={\"msg\":\"123\"}&amp;encrypt=false&amp;password=123456789  
     * <h2>Example response</h2>  
      * {"type_name":"LETTER", "creator":"7GvWSpPr4Jbv683KFB5WtrCCJJa6M36QEP",  
     * "message":"{"msg":"123"}","signature":"Vf8qtG3tiYV6LwvCrrTPf6zq3ikUVNZWfgwkrLU1tckvEQ2Dx8qB1qLEGkX8Wqj4WVKDYZRYfJyGb3dZCTR3asz", * "fee":"0.00010624", "publickey":"5sD1mTM2tB8aiQdUzKtXiNtesmJQCvHjXMrcCZWQca37", "type":31, * "confirmations":0, "version":0, "record_type":"LETTER", "property2":0, "property1":128, "size":166, * "encrypted":false, "recipient":"79MXsjo9DEaxzu6kSvJUauLhmQrB4WogsH", "sub_type_name":"", "isText":true, "timestamp":1529585877655} */

POST r_send {\"creator\": \"<creator>\", \"recipient\": \"<recipient>\", \"asset\":\"<assetKey>\", \"amount\":\"<amount>\", \"title\": \"<title>\", \"message\": \"<message>\", \"encoding\": <encoding>, \"encrypt\": <true/false>, \"password\": \"<password>\"}

    /**  
     * send and broadcast POST r_send * @param x JSON row with all parameter  
     * @return  
      * <h2>Example request</h2>  
      * POST r_send {"creator":"79WA9ypHx1iyDJn45VUXE5gebHTVrZi2iy","recipient":"79MXsjo9DEaxzu6kSvJUauLhmQrB4WogsH",  
     * "feePow":"1","assetKey":"643","amount":"1","title":"123","message":"{"msg":"1223"}", * "encoding":"0","encrypt":"false","password":"123456789"} * * <h2>Example response</h2>  
      * {  
     *   "type_name":"SEND", "creator":"79WA9ypHx1iyDJn45VUXE5gebHTVrZi2iy", "amount":"1", "message":"{", *   "signature":"5QASfQZ8kp8VWdBh9tNJkvdZqNynPki2fMSyaRh3oWPiV8bGw49v66cAjYp1dCC5LKWUirkE9kqWm7kUanNChxsi", *   "fee":"0.00035424", "publickey":"krksTcZunJmmnXQtUoNVQhwWAXFfQ4LbCJw3Qg8THo8", "type":31, *   "confirmations":0, "version":0, "record_type":"SEND", "property2":0, "action_key":1, "head":"123", *   "property1":0, "size":169, "encrypted":false, "action_name":"PROPERTY", "recipient":"79MXsjo9DEaxzu6kSvJUauLhmQrB4WogsH", *   "backward":false, "asset":643, "sub_type_name":"PROPERTY", "isText":true, "timestamp":1529586600467 } */

### Запросы получения данных на входящие транзакции

## Другие приемы
### Активация ноды при приходе транзакций
Можно настроить так что нода сама начнет посылать внешние команды на заданный URL если в новом блоке есть транзакции присланные на счета кошелька ноды. Нода будет дергать заданную ссылку либо каждые ХХ блоков (для обновления например состояния обработки - последний обработанный блок), либо если есть входящая транзакция связанная с нашим кошельком (в том числе и исходящая). Для этого задайте в файле настроек:
+ `notify_incoming_url` - URL  
+ `notify_incoming_confirmations` - если 0 то отключает вызовы с ноды данного URL

Пример строк в файле настроек settngs.json:

    ...
    "notify_incoming_url": "http://127.0.0.1:8000/exhange/era/income",
    "notify_incoming_confirmations": 1,
    ...

Замечание: имя "localhost" нельзя использовать.


#### API
see https://app.swaggerhub.com/apis-docs/Erachain/era-api/

or SDK - [https://github.com/erachain/sdk-php](https://github.com/erachain/sdk-php)


Пример общения на языке Питон

[https://github.com/icreator/7pay_in/blob/master/modules/rpc_erachain.py](https://github.com/icreator/7pay_in/blob/master/modules/rpc_erachain.py)


> Written with [StackEdit](https://stackedit.io/).
