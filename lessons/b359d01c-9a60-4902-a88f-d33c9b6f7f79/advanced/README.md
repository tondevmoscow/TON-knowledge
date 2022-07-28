# Создаем кошельки owner'а и guest'а
Создаем кошельки для владельца контракта и для "гостя"
```code
$ wton wallet create owner --save-to-whitelist my_owner_wallet
$ wton wallet create guest --save-to-whitelist my_guest_wallet
```

После этого нужно отправить тестовые токены на любой из кошельков (лучше на owner), используя либо старые кошельки, либо @testgiver_ton_bot. Можно проверять баланс используя команду
```code
$ wton wallet list -v
```

Теперь переведем немного денег с owner кошелька на guest (чтобы оплачивать внутренние транзакции)
```code
$ wton wallet transfer owner my_guest_wallet 0.4
```

wton wallet init owner

Для того, чтобы подписывать транзакции, нам нужен будет .pk от нашего кошелька, а также .addr, для указания адреса (предполагается, что вы находитесь в директории advanced_counter/advanced/.)
```code
$ wton wallet to-addr-pk owner ./build/
Password[]:
$ wton wallet to-addr-pk guest ./build/
Password[]:
```

Данные кошельки будут использованы для подписи транзакций.

# Компилируем в fift
## Как это было в simple версии
```code
$ toncli build
INFO: 🥌 Build [] successfully, check out ./build
```

## Как это выглядит в advanced
Что запускается в toncli
```code
$ /usr/local/bin/func \
    -o "/Users/USERNAME/PATH/advanced_counter/build/contract.fif" \
    -SPA \
    "/Users/USERNAME/Library/Application Support/toncli/func-libs/stdlib.func" \
    "/Users/USERNAME/PATH/advanced_counter/func/advanced_counter.fc"
```

stdlib.fc - подключаемая библиотека. При компиляции все предыдущие файлы инклудятся в последующие, поэтому мы можем использовать такие функции, как get_data, set_data, etc.

Изучим флаги
```code
$ func -h
...
-o<fift-output-filename>        Writes generated code into specified file instead of stdout
-A      Prefix code with `"Asm.fif" include` preamble
-P      Envelope code into PROGRAM{ ... }END>c
-S      Include stack layout comments in the output code
...
```

Поэтому, чтобы скомпилировать, нам нужно запустить (make build-func)
```code
$ func -o ./build/advanced_counter.fif -SPA ./func/lib/stdlib.fc ./func/advanced_counter.fc
```

# Деплоим 

## Как это было в simple версии
```code
$ toncli deploy
```

## Как это выглядит в advanced
Изначально toncli генерирует временный скрипт для создания init data .boc файла /var/folders/wk/9gc3hhkd525bl7vy3vlpbtdw0000gn/T/tmp48m0d210.boc:
```code
"/Users/USERNAME/PATH/advanced_counter/fift/init_data_cell_of_advanced_counter.fif" include
2 boc+>B
"/var/folders/wk/9gc3hhkd525bl7vy3vlpbtdw0000gn/T/tmp48m0d210.boc" tuck B>file
```

Затем запускает этот скрипт и получает .boc файл
```code
$ /usr/local/bin/fift -I '/Users/USERNAME/Library/Application Support/toncli/fift-libs' -s /var/folders/wk/9gc3hhkd525bl7vy3vlpbtdw0000gn/T/tmp48m0d210.boc
```

После этого запускает скрипт на fift, который генерирует external message и кладет его в .boc файл
```code
$ /usr/local/bin/fift -I '/Users/USERNAME/Library/Application Support/toncli/fift-libs' -s /Users/USERNAME/PATH/venv/lib/python3.9/site-packages/toncli/modules/fift/contract_manipulation.fif /Users/USERNAME/PATH/advanced_counter/build/contract.fif /var/folders/wk/9gc3hhkd525bl7vy3vlpbtdw0000gn/T/tmp48m0d210.boc 0 /Users/USERNAME/PATH/advanced_counter/build/boc/contract.boc /Users/USERNAME/PATH/advanced_counter/build/contract_address
```

Чтобы понять, что тут происходит, давайте откроем сам скрипт contract_manipulation.fif:
```code
#!/usr/bin/fift -s
// ok, it's hard to understand but samples are here:
// https://gist.github.com/tvorogme/fdb174ac0740b6a52d1dbdf85f4ddc63
"TonUtil.fif" include
"Asm.fif" include
"Color.fif" include

// define usage comment
{ ."usage: " $0 type ." code-path data-path workchain boc-path address-path [lib-path]" cr
."Generate init message and and calculate wallet address" cr 1 halt } : usage

// if less then 5 arguments print usage message
$# 5 < ' usage if

// load at least 2 command line arguments
5 :$1..n

$1 constant code-path           // <-  compiled fif code of advanced_counter.fc (/Users/USERNAME/PATH/advanced_counter/build/contract.fif)
$2 constant data-path           // <-  compiled boc of data.fif (/var/folders/wk/9gc3hhkd525bl7vy3vlpbtdw0000gn/T/tmp48m0d210.boc)
$3 parse-workchain-id =: wc     // <-  workchain (0)
$4 constant output_boc          // <-  output .boc file (/Users/USERNAME/PATH/advanced_counter/build/boc/contract.boc)
$5 constant output_address      // <-  output address file

// It's trick from https://t.me/tonsc_chat/949
<{ SETCP0 ACCEPT
   code-path include PUSHREF SETCODE
}>c constant code

data-path file>B B>boc constant data

// 0011 why??? Shows what data is presented in the StateInit
// split_depth - field is present and non-zero only in instances of large smart contracts 
// special(ticktock) -  may be present only in the masterchain—and within the masterchain, only in some fundamental smart contracts required for the whole system to function.
// code - code cell
// data - data cell
// library - library cell
<b b{0011} s, code ref, data ref, null dict, b> constant StateInit


StateInit hashu wc swap 2dup 2constant contract_addr
// { -rot 256 u>B swap 32 i>B B+ swap B>file } : save-address
2dup "build/advanced_counter.addr" save-address

// Create raw address string
2dup (dump) swap (dump) ":" $+ swap $+ constant raw_address

// Create bounceable_address ()
// smca>$ - (x y z – S)  where z is: +1 for non-bounceable addresses, 2 for testnet-only addresses, and +4 for base64url output instead of base64
2dup 6 smca>$ constant bounceable_address

// Create Non-bounceable address
2dup 7 smca>$ constant nonbounceable_address

^magenta ."INFO: 🦄 Raw address: " raw_address type cr
^green ."INFO: 🦝 Bounceable address: " bounceable_address type cr
^yellow  ."INFO: 🐏 Non-bounceable address: " nonbounceable_address type cr
^reset

// Save all addresses to text
raw_address " " $+
bounceable_address " " $+ $+
nonbounceable_address " " $+ $+
$>B
output_address B>file
^green "🖊  Save all addresses to " output_address $+ type cr
^reset


// External message: 
// message$_ {X:Type} info:CommonMsgInfo
//   init:(Maybe (Either StateInit ^StateInit))
//   body:(Either X ^X) = Message X;
//
// CommonMsgInfo:
// ext_in_msg_info$10 src:MsgAddressExt dest:MsgAddressInt 
//   import_fee:Grams = CommonMsgInfo;
//
// addr_none$00 = MsgAddressExt;
// 
// addr_std$10 anycast:(Maybe Anycast) 
//   workchain_id:int8 address:bits256  = MsgAddressInt;
// 
// import fee sets to nothing in our case: 0
//
// So:
// b{1000100} s,         <- Common message info for external
// contract_addr addr,   <- addr_std: address of our smart contract
// b{000011} s,          <- ?
// StateInit ref,        <- StateInit
// b{0} s,               <- body
<b b{1000100} s, contract_addr addr, b{000011} s, StateInit ref, b{0} s, b>
2 boc+>B

output_boc tuck B>file

^green "🖊  Save boc_file to " output_boc $+ type cr
^reset
```


Теперь у нас есть адрес нашего нового контракта. Следующим этапом мы должны отправить туда N коинов, чтобы external message смог проинициализировать контракт. toncli делает это автоматически кошельком, который был создан во вспомогательной директории. Мы же можем сделать это своими руками (после запуска contract_manipulation.fif, адрес будущего смарт контракта можно найти в build/advanced_counter_address файле):
```code
$ wton whitelist add advanced_counter kQC0FVqbg33FgNYbBsFpGmC4KXaJmR8-Sj20zmqWfABrrLsi
$ wton wallet transfer owner advanced_counter 0.1 
$ wton whitelist list -v
```
**Не важно, с какого кошелька отправлять токены! Это никак не влияет на владение смарт контракта**

После того, как мы убедились, что на адресе будущего смарт контракта есть монеты, toncli отправляет наше внешнее сообщение:
```code
$ /usr/local/bin/lite-client -v 3 --timeout 3 -C '/Users/USERNAME/Library/Application Support/toncli/testnet.json' -v 2 -c 'sendfile /Users/USERNAME/PATH/advanced_counter/build/boc/contract.boc'
```

В нашем случае
```code
$ lite-client -v 3 --timeout 3 -C ./testnet.json -v 2 -c 'sendfile ./build/boc/init_contract_message.boc'
```

Те же команды, используя Makefile
```code
$ make init-data-to-boc
$ make build-init-contract-message
$ make send-init-contract-message
```

# Internal messages

## Как это было в simple версии

Для отправки авто сгенерированным кошельком toncli можно воспользоваться командой
```code
$ toncli send -n testnet -a 0.03 --address kQBIp8sJTov02ZXw4hhak-jH3a-j-Kbk57TyfD7DoV3AM22P --body ./fift/msg_body.fif
```

## Как это выглядит в advanced

Что запускает toncli
```code
$ /usr/local/bin/fift -I '/Users/USERNAME/Library/Application Support/toncli/fift-libs' \ 
    -L '/Users/USERNAME/Library/Application Support/toncli/wallet/build/cli.fif' \ 
    -s '/Users/USERNAME/Library/Application Support/toncli/wallet/fift/usage.fif' \
    build/contract 'адрес вашего контракта' 0 7 0.03 \ 
    -B /Users/USERNAME/PATH/advanced_counter/fift/internal_msg_body_to_increase_counter.fif

$ /usr/local/bin/lite-client -v 3 --timeout 3 -C '/Users/USERNAME/Library/Application Support/toncli/testnet.json' -v 2 -c 'sendfile /Users/USERNAME/Library/Application Support/toncli/wallet/build/boc/usage.boc'
```

Изучим флаги fift:
```code
$ fift -h
...
-I<source-search-path>  Sets colon-separated library source include path. If not indicated, $FIFTPATH is used instead.
-L<library-fif-file>    Pre-loads a library source file
-s                      Script mode: use first argument as a fift source file and import remaining arguments as $n)
...
```

Все внутренние сообщения в toncli отправляются через кошелек, который сгенерирован автоматически. Если мы хотим отправлять сообщения с другого кошелька, нам нужно добавить свой скрипт в директорию advanced_counter/fift/create_internal_message.fif:

```code
#!/usr/bin/fift -s
"TonUtil.fif" include
"GetOpt.fif" include

{ show-options-help 1 halt } : usage

"" =: comment  // comment for simple transfers
true =: allow-bounce
false =: force-bounce
3 =: send-mode  // mode for SENDRAWMSG: +1 - sender pays fees, +2 - ignore errors
60 =: timeout   // external message expires in 60 seconds
variable extra-currencies
{ extra-currencies @ cc+ extra-currencies ! } : extra-cc+!

begin-options
     " <filename-base> <dest-addr> <subwallet-id> <seqno> <amount> [-x <extra-amount>*<extra-currency-id>] [-n|-b] [-t<timeout>] [-B <body-boc>] [-C <comment>] [<savefile>]" +cr +tab
    +"Creates a request to advanced wallet created by new-wallet-v3.fif, with private key loaded from file <filename-base>.pk "
    +"and address from <filename-base>.addr, and saves it into <savefile>.boc ('wallet-query.boc' by default)"
    disable-digit-options generic-help-setopt
  "n" "--no-bounce" { false =: allow-bounce } short-long-option
    "Clears bounce flag" option-help
  "b" "--force-bounce" { true =: force-bounce } short-long-option
    "Forces bounce flag" option-help
  "x" "--extra" { $>xcc extra-cc+! } short-long-option-arg
    "Indicates the amount of extra currencies to be transfered" option-help
  "t" "--timeout" { parse-int =: timeout } short-long-option-arg
    "Sets expiration timeout in seconds (" timeout (.) $+ +" by default)" option-help
  "B" "--body" { =: body-fift-file } short-long-option-arg
    "Sets the payload of the transfer message" option-help
  "C" "--comment" { =: comment } short-long-option-arg
    "Sets the comment to be sent in the transfer message" option-help
  "m" "--mode" { parse-int =: send-mode } short-long-option-arg
    "Sets transfer mode (0..255) for SENDRAWMSG (" send-mode (.) $+ +" by default)"
    option-help
  "h" "--help" { usage } short-long-option
    "Shows a help message" option-help
parse-options

$# dup 5 < swap 6 > or ' usage if
6 :$1..n

true constant bounce
$1 =: file-base     // <- build/contract
$2 bounce parse-load-address force-bounce or allow-bounce and =: bounce 2=: dest_addr  // <- 'адрес вашего контракта'
$3 parse-int =: subwallet_id  // <- 698983191 (стандартное число для wallet_id для кошельков TON, оно также используется во wton. BTW, toncli использует 0.)
$4 parse-int =: seqno         // <- seqno of the wallet that will send internal message (ВНИМАНИЕ, С КАЖДОЙ СЛЕДУЮЩЕЙ ТРАНЗАКЦИЕЙ ЕГО НУЖНО МЕНЯТЬ, чтобы его узнать, можно запустить 'wton contract -w owner seqno')
$5 $>cc extra-cc+! extra-currencies @ 2=: amount  // <- 0.03 toncoins
$6 "wallet-query" replace-if-null =: savefile
subwallet_id (.) 1 ' $+ does : +subwallet

"./build/owner.addr"
load-address
2dup 2constant wallet_addr
."INFO: 👀 Owner wallet address = " 2dup 6 .Addr cr
"./build/owner.pk" load-keypair nip constant wallet_pk

def? body-fift-file { @' body-fift-file include } { comment simple-transfer-body } cond  // <- инклудим ./fift/internal_msg_body_to_increase_counter.fif
constant body-cell

."INFO: 👋 Send " dest_addr 2dup bounce 7 + .Addr ." = "
."subwallet_id=0x" subwallet_id x.
."seqno=0x" seqno x. ."bounce=" bounce . cr
."INFO: 🧟 Body of transfer message is " body-cell <s csr.

// create a message
// https://ton.org/docs/#/smart-contracts/messages?id=sending-messages
// var msg = begin_cell()
//    .store_uint(0x18, 6)
//    .store_slice(addr)
//    .store_coins(amount)
//    .store_uint(0, 1 + 4 + 4 + 64 + 32 + 1 + 1)
//    .store_slice(message_body)
//    .end_cell();
<b b{01} s, bounce 1 i, b{000} s, dest_addr Addr, amount Gram+cc, 0 9 64 32 + + u,
  body-cell <s 2dup 1 s-fits-with? not rot over 1 i, -rot { drop body-cell ref, } { s, } cond
b>

<b subwallet_id 32 u, now timeout + 32 u, seqno 32 u, send-mode 8 u, swap ref, b>

dup hashu wallet_pk ed25519_sign_uint
<b b{1000100} s, wallet_addr addr, 0 Gram, b{00} s,
   swap B, swap <s s, b>

2 boc+>B

saveboc // it is defined in build/cli.fif (or /tmp if in script mode)
// it is project-specific lib from tlcli thats specify needed locations
```

Теперь мы готовы создать .boc файл нашего сообщения и отправить его в блокчейн TON (ВНИМАНИЕ, НЕ ЗАБУДЬТЕ ИЗМЕНИТЬ АДРЕС И SEQNO!)
```code
$ fift -I ./fift/lib/fift-libs -L ./fift/lib/cli.fif -s ./fift/create_internal_message.fif build/contract kQA3esZyDjmwHo6eewGLoDjm4kYpc9mFNI_tVYDje101-_nG 698983191 4 0.1 -B ./fift/internal_msg_body_to_increase_counter.fif
$ lite-client -v 3 --timeout 3 -C ./testnet.json -v 2 -c 'sendfile ./build/boc/internal_message.boc'
```

Используя Makefile (ВНИМАНИЕ, НЕ ЗАБУДЬТЕ ИЗМЕНИТЬ АДРЕС И SEQNO!)
```code
$ make build-internal-message
$ make send-internal-message
```


# External messages

## Как это было в simple версии
Как?

## Как это выглядит в advanced
Теперь мы должны создать external сообщение и отправить его в сеть. Код для создания create_external_message.fif
```code
#!/usr/bin/fift -s
"TonUtil.fif" include
"Asm.fif" include


"./build/owner.pk" load-keypair
constant owner_private_key
constant owner_public_key

"./build/advanced_counter.addr" load-address
2dup 2constant advanced_counter_addr

<b 1618 32 u, b>  // тело сообщения
dup hashu owner_private_key ed25519_sign_uint
<b b{1000100} s, advanced_counter_addr addr, 0 Gram, b{00} s, swap B, swap <s s, b>
2 boc+>B
"./build/boc/external_message.boc" tuck B>file
```

Компилируем и отправляем
```code
$ fift -I ./fift/lib/fift-libs -L ./fift/lib/cli.fif -s ./fift/create_external_message.fif
$ lite-client -v 3 --timeout 3 -C ./testnet.json -v 2 -c 'sendfile ./build/boc/external_message.boc'
```

Используя Makefile
```code
$ make build-external-message
$ make send-external-message
```

# Get methods

## Как это было в simple версии
```code
$ toncli get get_external_counter
```

## Как это выглядит в advanced

Также просто
```code
$ lite-client -v 3 --timeout 3 -C ./testnet.json -v 0 -c 'runmethod kQA3esZyDjmwHo6eewGLoDjm4kYpc9mFNI_tVYDje101-_nG get_external_counter'
```

Используя Makefile
```code
$ make get-external
```
