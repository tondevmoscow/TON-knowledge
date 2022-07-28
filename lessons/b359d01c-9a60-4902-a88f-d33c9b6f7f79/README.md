> Внимание: если вы обнаружили в уроке ошибку или неточность, смело пишите об этом в комментариях или автору. Все мы учимся и ошибаемся 🤓

# Введение

[Видео обсуждение урока.](https://youtu.be/vt5fpE0bzSY)

Будем писать смарт контракт для TON.

Стуктура урока:

0. Устанавливаем необходимые инструменты
1. Пишем небольшой смарт контракт на func, тест, начальные данные смарт контракта и сообщение для увеличения counter'а.
2. Слепо используем toncli (simple) для взаимодействия со смарт конрактом (билд, деплой, отправка сообщений)
3. Осознанно используем fift/func/lite-client (advanced) для взаимодействия со смарт контрактом (билд, деплой, отправка сообщений)


# Устанавливаем необходимые инструменты
Подготавливаем свое окружение.

```code
$ mkdir ~/tondevmoscow/lesson1/ & cd ~/tondevmoscow/lesson1/
$ python3 -m venv venv & source venv/bin/activate
$ pip install toncli wton
$ wton init
$ wton config provider.dapp.api_key 8d22e52d-da87-422e-9ace-095539a95706
$ wton keystore new tondev
Password[]:
$ wton config wton.keystore_name tondev
$ wton config --network testnet
```

toncli - работа со смарт контрактами

wton   - управление кошельками, потребуется в advanced уроке (8d22e52d-da87-422e-9ace-095539a95706 - специально выделенный ключ для урока)

Для изучения пакетов [toncli](https://pypi.org/project/toncli/) и [wton](https://pypi.org/project/wton/) можно использовать команду:
```code
$ wton -h
$ toncli -h
```

Вы можете запускать команду -h (--help) для любой подкоманды wton
```code
$ wton wallet -h
$ wton wallet transfer -h
```

Для того, чтобы искать непонятные вещи, можно сразу открыть и использовать (ctrl+f):
- FIFT - https://ton-blockchain.github.io/docs/fiftbase.pdf
- Blockchain - https://ton-blockchain.github.io/docs/tblkch.pdf
- TVM  - https://ton-blockchain.github.io/docs/tvm.pdf
- TLB  - https://github.com/ton-blockchain/ton/blob/master/crypto/block/block.tlb
- testnet.tonscan - https://testnet.tonscan.org/address/


# Пишем смарт контракт
Нам нужна следующая структура для написания смарт контракта:
```code
project
├── project.yaml
├── func
│   ├── *.fc
├── fift
│   ├── *.fif
├── build
│   ├── *
```


Можно воспользоваться командой и удалить ненужные файлы (в целях обучения рекомендуем сделать все своими руками):
```code
$ toncli start wallet
```


- project.yaml - пути до смарт контракта/тестов/данных для инициализации
- func  - в этой директории храним наш func код смарт контрактов
- fift  - в этой директории храним наш fift код (Например, init data cell)
- build - в этой директории будут лежать скомпилированные и другие необходимые(.addr, .pk) файлы 



1️⃣ В каждой из версий сам смарт контракт будет находиться в директории ./func. Напишем каунтер внутренних и внешних сообщений + отслеживаем число сообщений от владельца смарт контракта:
```code
int equal_slices (slice a, slice b) asm "SDEQ";

(int, int, int, int, slice, int) load_data() {
    slice ds = get_data().begin_parse();
    return (ds~load_uint(32), ds~load_uint(32), ds~load_uint(32), ds~load_uint(32), ds~load_msg_addr(), ds~load_uint(256));
}

() store_data(int external_counter, int internal_counter, int owner_external_counter, int owner_internal_counter, slice owner_address, int public_key) impure {
    set_data(
        begin_cell()
            .store_uint(external_counter, 32)
            .store_uint(internal_counter, 32)
            .store_uint(owner_external_counter, 32)
            .store_uint(owner_internal_counter, 32)
            .store_slice(owner_address)
            .store_uint(public_key, 256)
            .end_cell()
    );
}

() recv_internal(int my_balance, int msg_value, cell in_msg_full, slice in_msg_body) impure {
    if (in_msg_body.slice_empty?()) {
        return ();
    }
    slice cs = in_msg_full.begin_parse();
    int flags = cs~load_uint(4);
    if (flags & 1) { ;; ignore all bounced messages
        return ();
    }
    slice sender_address = cs~load_msg_addr();
	throw_if(35, in_msg_body.slice_bits() < 32);
    int n = in_msg_body~load_uint(32);

    var (external_counter, internal_counter, owner_external_counter, owner_internal_counter, owner_address, public_key) = load_data();
    internal_counter += n;
    if (equal_slices(sender_address, owner_address)) {
        owner_internal_counter += n;
    }

    store_data(external_counter, internal_counter, owner_external_counter, owner_internal_counter, owner_address, public_key);
}

() recv_external(slice in_msg) impure {
    int is_owner = 0;  ;; 0 - false; -1 - true
    int n = 0;
    var (external_counter, internal_counter, owner_external_counter, owner_internal_counter, owner_address, public_key) = load_data();

    if (in_msg.slice_bits() == 32) {
        n = in_msg~load_uint(32);
    } else {
        throw_if(35, in_msg.slice_bits() < 512 + 32);
        var signature = in_msg~load_bits(512);
        n = in_msg~load_uint(32);
        int is_owner = check_signature(slice_hash(in_msg), signature, public_key);
    }

    accept_message();

    external_counter += n;
    if (is_owner) {
        owner_external_counter += n;
    }

    store_data(external_counter, internal_counter, owner_external_counter, owner_internal_counter, owner_address, public_key);
}

;; Get methods
(int, int, int, int, slice, int) get_all() method_id {
    var cs = get_data().begin_parse();
    return load_data();
}

int get_external_counter() method_id {
    var cs = get_data().begin_parse();
    return cs~load_uint(32);
}

int get_internal_counter() method_id {
    var cs = get_data().begin_parse();
    cs~load_uint(32);
    return cs~load_uint(32);
}

int get_owner_external_counter() method_id {
    var cs = get_data().begin_parse();
    cs~load_uint(32);
    cs~load_uint(32);
    return cs~load_uint(32);
}

int get_owner_internal_counter() method_id {
    var cs = get_data().begin_parse();
    cs~load_uint(32);
    cs~load_uint(32);
    cs~load_uint(32);
    return cs~load_uint(32);
}
```


2️⃣ Напишем небольшой тест к смарт контракту (его положим в tests/test.fc):
```code
[int, tuple, cell, tuple, int] test_internal_message() method_id(0) {
	int function_selector = 0;

	cell message = begin_cell()
				.store_uint(1618, 32)
				.end_cell();

	tuple stack = unsafe_tuple([message.begin_parse()]);
    
    cell owner_addr = begin_cell()
			.store_uint(1, 2)
			.store_uint(5, 9) 
			.store_uint(8, 5)
            .end_cell();

    cell data = begin_cell()
            .store_uint(0, 32)
            .store_uint(0, 32)
            .store_uint(0, 32)
            .store_uint(0, 32)
            .store_slice(owner_addr.begin_parse())
            .store_uint(1, 256)
            .end_cell();


	return [function_selector, stack, data, get_c7(), null()];
}


_ test_example(int exit_code, cell data, tuple stack, cell actions, int gas) method_id(1) {
	throw_if(100, exit_code != 0);

	var ds = data.begin_parse();
    ds~load_uint(32); ;; skip external_counter

	throw_if(101, ds~load_uint(32) != 10);
	throw_if(102, gas > 1000000); 
}
```



3️⃣ Теперь нам нужно описать стартовые данные смарт контракта. Они будут лежать в папке ./fift и выглядят так:
```code
"TonUtil.fif" include
"Asm.fif" include

// AUTOGENERATED WALLET PATH ON MAC
"/Users/USERNAME/Library/Application Support/toncli/wallet/build/contract.pk" load-keypair // load owner key pair
constant owner_private_key
constant owner_public_key

// AUTOGENERATED WALLET PATH ON MAC
"/Users/USERNAME/Library/Application Support/toncli/wallet/build/contract.addr" load-address // load owner address
2dup 2constant owner_address
."INFO: Owner wallet address = " 2dup 6 .Addr cr


<b
  0 32 u, // external_counter
  0 32 u, // internal_counter
  0 32 u, // owner_external_counter
  0 32 u, // owner_internal_counter
  owner_address addr,
  owner_public_key B,
b>
```


4️⃣ Также нам потребуется описать тело сообщения, которое мы хотим отправлять в наш смарт контракт. Оно тоже будет лежать в папке ./fift. Код тела сообщения (просто кладем в cell любое 32-х разрядное число):
```code
"Asm.fif" include

<b
  1618 32 u,
b>
```


Теперь, когда мы расмотрели используемый код, перейдем на следующий шаг: simple/second_step.md
