# Для чего вообще нужны кошельки?

## Взаимодействие со смарт-контрактами

Как мы уже выяснили, все общение внутри TON блокчейна происходит при помощи сообщений (см урок "разбор
сообщений").

В смарт-контрактах на TON блокчейне есть 2 метода обработки сообщений: recv_internal, recv_external.

### recv_internal

Как видно из названия, один метод (recv_internal) отвечает за прием внутренних сообщений,
это те, которые были отправлены внутри сети от одного смарт контракта другому.

Преимущество данного метода в том, что при его получении мы точно знаем, какое количество денег
было отправлено вместе с этим сообщением и адрес смарт-контракта отправителя.

Недостатком является то, что данный метод не сработает для сообщений, которые были присланы извне
блокчейна.
То есть обычный пользователь не сможет взаимодействовать со смарт-контрактом используя данный метод
напрямую.

### recv_external

Данный метод отвечает за сообщения, которые были присланы извне.

Преимущество в том, что любой пользователь может отправить сообщение на смарт-контракт напрямую.

Недостатком в данном случае будет то, что внутри сообщения не передаются TON coin'ы,
поэтому за обработку сообщения смарт-контракт должен платить сам.

### Какой метод предпочтительнее для смарт-контракта?

Конечно, для смарт-контракта более удобным является метод recv_internal,
через который на смарт-контракт поступает не только сообщений, но и TON coin'ы для его обработки.

В случае обработки recv_external, можно успешно устроить DDoS на смарт-контракт и лишить его баланса.

Встает вопрос, **как же сделать так, чтобы обычные пользователи могли общаться с другими
смарт-контрактами?**

## Кошельки

Что нам необходимо для решения данной задачи? Дать пользователю возможность отправлять internal сообщения.

Решение лежит на поверхности: давайте сделаем смарт-контракт, который будет принимать
внешние сообщения методом recv_external и отсылать их дальше internal сообщением!
Так как он является смарт-контрактом, ему доступна такая возможность.

Но в таком случае любой пользователь сможет пользоваться данным смарт-контрактом и отсылать деньги через
него.
Как же быть? Как сделать так, чтобы только владелец смарт-контракта мог отправлять сообщения через него?
Ответ - криптография.
Для подписи сообщений в TON используется алгоритм Ed25519
(подпись эллиптической кривой, с использованием EdDSA и Curve25519).
*Разбор криптографии вне этого урока*. Смысл такого подхода в том, что, зная публичный ключ,
смарт-контракт сможет убедиться, что сообщение было подписано приватным ключом для этого публичного ключа.

Что мы имеем на выходе? У нас есть смарт-контракт, принимающий внешние (external) сообщения,
проверяет подпись публичным ключом, который лежит в его хранилище, и отправляет уже внутренние (internal)
сообщения дальше по указанному адресу. **Данный интерфейс и будет называться кошельком (wallet) в сети
TON**

## Пример подписи на языке fift

```
// формируем сообщение, которое будет отправлять кошелек (схема зависит от того, что вы хотите отправить!)
// https://ton.org/docs/#/smart-contracts/messages?id=sending-messages
// var msg = begin_cell()
//    .store_uint(0x18, 6)
//    .store_slice(addr)
//    .store_coins(amount)
//    .store_uint(0, 1 + 4 + 4 + 64 + 32 + 1 + 1)
//    .store_slice(message_body)
//    .end_cell();
<b b{011000} s, dest_addr Addr, amount Gram+cc, 0 9 64 32 + + u, message_body s, b>

// формируем сообщение для кошелька, которое он примет в recv_external
<b subwallet_id 32 u, now timeout + 32 u, seqno 32 u, send-mode 8 u, swap ref, b>

// берем хэш от полученного сообщения и подписываем его нашим приватным ключом wallet_pk используя ed25519_sign_uint
dup hashu wallet_pk ed25519_sign_uint
// формируем сообщение CommonMsgInfo + init:0 + body: either(0) + body(inplace). 
// Как раз в body кладем нашу подпись и то, что было подписано
<b b{1000100} s, wallet_addr addr, 0 Gram, b{00} s, swap B, swap <s s, b>  
```

## Разберем существующие (10th of September, 2022) смарт-контракты кошельков

- simple wallet (v2-v4)
- highload wallet (v1-v2)
- multisig wallet

### Порядок просмотра
1. wallet2.fc
2. wallet3.fc
3. wallet4.fc
4. wallet4-plugin-example.fc
5. highload-wallet.fc
6. highload-wallet-v2.fc

## Mode отправки сообщения
Sends a raw message contained in msg, which should contain a correctly serialized object Message X, 
with the only exception that the source address is allowed to have dummy value addr_none 
(to be automatically replaced with the current smart contract address), 
and ihr_fee, fwd_fee, created_lt and created_at fields can have arbitrary values 
(to be rewritten with correct values during the action phase of the current transaction).
Integer parameter mode contains the flags. Currently mode = 0 is used for ordinary messages; 
mode = 128 is used for messages that are to carry all the remaining balance of the current smart contract 
(instead of the value originally indicated in the message); mode = 64 is used for messages that carry 
all the remaining value of the inbound message in addition to the value initially indicated in the new 
message (if bit 0 is not set, the gas fees are deducted from this amount); mode' = mode + 1 means that 
the sender wants to pay transfer fees separately; mode' = mode + 2 means that any errors arising while 
processing this message during the action phase should be ignored. Finally, mode' = mode + 32 means that 
the current account must be destroyed if its resulting balance is zero. This flag is usually employed 
together with +128.


0 - обычное сообщение

+1 - отправитель хочет заплатить за gas отдельно (вычитается не из суммы отправленного сообщения, а из баланса смарт контракта)

+2 - если будут ошибки во время ACTION фазы, то они будут проигнорированы (то есть смарт отправит сообщение, даже если другие action были с ошибками)

+32 - удалить текущий аккаунт, если после отправки на нем не останется средств

+64 - отправить весь баланс сообщения, которое пришло, дальше (имеет смысл только в recv_internal) 

+128 - отправка всего баланса текущего смарт контракта

Mode представляется в виде 8 битного числа (uint8)

### Пример (взят у romanovichim)

Пускай на балансе смарт-контракта 100 монет и мы получаем internal message c 60 монетами и отсылаем сообщение с 10, общий fee 3.

mode = 0 - баланс (100+60-10 = 150 монет), отправим(10-3 = 7 монет) 

mode = 1 - баланс (100+60-10-3 = 147 монет), отправим(10 монет) 

mode = 64 - баланс (100-10 = 90 монет), отправим (60+10-3 = 67 монет) 

mode = 65 - баланс (100-10-3=87 монет), отправим (60+10 = 70 монет) 

mode = 128 -баланс (0 монет), отправим (100+60-3 = 157 монет)