First we gonna build our code:
```code
$ toncli build
INFO: 🥌 Build [] successfully, check out ./build
```


Also, we can test it:
```code
$ toncli run_tests
INFO: 🌈 Start tests
INFO: 🥌 Build successfully, check out ./build
[ 3][t 0][2022-07-22 14:21:13.686500][vm.cpp:558]       steps: 49 gas: used=3122, max=9223372036854775807, limit=9223372036854775807, credit=0
[ 3][t 0][2022-07-22 14:21:13.688738][vm.cpp:558]       steps: 5 gas: used=662, max=1000000000, limit=1000000000, credit=0
[ 3][t 0][2022-07-22 14:21:13.689074][vm.cpp:558]       steps: 7 gas: used=540, max=9223372036854775807, limit=9223372036854775807, credit=0
INFO: Test [test_example] status: [FAIL], code: [100] Gas used: [662]
```


Then, when we believe that our code is correct, we can deploy it. Deploying, toncli puts init data inside init message .boc (do not forget to set correct username in ./fift/data.fif file):
```code
$ toncli deploy -n testnet
INFO: 🚀 You want to interact with your contracts ['contract'] in testnet - that's great!
INFO: 🦘 Found existing deploy-wallet [kQDaR5LvGcB4w_omJcwzKOtQmb167sloQ_PtOkkHQRjJWq2R] (Balance: 1.646395055💎, Is inited: True) in /Users/akkireev/Library/Application Support/toncli
INFO: 👻 Your smart contract project [/Users/akkireev/Documents/blockchain/my/dev/lessons/advanced_counter/simple] is now going to be deployed, get ready!
INFO: 🌈 Start building: 
INFO: 🌲 Func compiled
INFO: 🤗 Run tests on ['/Users/akkireev/Documents/blockchain/my/dev/lessons/advanced_counter/simple/fift/data.fif']
Loading private key from file /Users/akkireev/Library/Application Support/toncli/wallet/build/contract.pk
INFO: Owner wallet address = kQDaR5LvGcB4w_omJcwzKOtQmb167sloQ_PtOkkHQRjJWq2R
[ 1][t 0][2022-07-22 14:18:28.413897][Fift.cpp:67]      top: abort
level 1: swap { <continuation 0x7fc157c083d0> } if **HERE** drop 
level 2: <text interpreter continuation>
[ 1][t 0][2022-07-22 14:18:28.413969][fift-main.cpp:204]        Error interpreting file `/var/folders/wk/9gc3hhkd525bl7vy3vlpbtdw0000gn/T/tmp3hsikmww.fif`: tmp3hsikmww.fif:31: test-stack-depth:Not valid stack depth need to be 1 😈
INFO: 🌲 Tests passed
INFO: 🥳 Start contract manipulation
INFO: 🌲 BOC created
INFO: 🌲 Sending TON to new contract [contract] [kQBIp8sJTov02ZXw4hhak-jH3a-j-Kbk57TyfD7DoV3AM22P]
INFO: 🐰 Getting seqno for transaction
ERROR: 😢 Error in lite-client execution: /usr/local/bin/lite-client -v 3 --timeout 3 -C /Users/akkireev/Library/Application Support/toncli/testnet.json -v 0 -c runmethod kQDaR5LvGcB4w_omJcwzKOtQmb167sloQ_PtOkkHQRjJWq2R seqno
INFO: 💾 Will save BOC to /Users/akkireev/Library/Application Support/toncli/wallet/build/boc/usage.boc
INFO: 🍿 Loading fift CLI lib
INFO: 👀 Source wallet address = kQDaR5LvGcB4w_omJcwzKOtQmb167sloQ_PtOkkHQRjJWq2R
Loading private key from file build/contract.pk
INFO: 👋 Send 0QBIp8sJTov02ZXw4hhak-jH3a-j-Kbk57TyfD7DoV3AMzBK = subwallet_id=0x0 seqno=0x7 bounce=0 
INFO: 🧟 Body of transfer message is x{}
INFO: 🥰  Save boc
INFO: 💾 (Saved to file /Users/akkireev/Library/Application Support/toncli/wallet/build/boc/usage.boc)
[ 1][t 1][2022-07-22 14:18:36.043376][lite-client.h:362][!testnode]     conn ready
[ 2][t 1][2022-07-22 14:18:36.118948][lite-client.cpp:363][!testnode]   server version is 1.1, capabilities 7
[ 1][t 2][2022-07-22 14:18:36.265301][lite-client.cpp:1150][!testnode]  sending query from file /Users/akkireev/Library/Application Support/toncli/wallet/build/boc/usage.boc
 [🌔] [contract] [kQBIp8sJTov02ZXw4hhak-jH3a-j-Kbk57TyfD7DoV3AM22P] 0.05💎 / Inited: False
INFO: 🙀 All contracts now with non-zero balance
[ 1][t 1][2022-07-22 14:18:50.610416][lite-client.h:362][!testnode]     conn ready
[ 2][t 1][2022-07-22 14:18:50.669067][lite-client.cpp:363][!testnode]   server version is 1.1, capabilities 7
[ 1][t 1][2022-07-22 14:18:50.771847][lite-client.cpp:1150][!testnode]  sending query from file /Users/akkireev/Documents/blockchain/my/dev/lessons/advanced_counter/simple/build/boc/contract.boc
INFO: 💥 Deployed successfully!
INFO: 🚀 It may take some time to get is_inited to True
 [🌔] [contract] [kQBIp8sJTov02ZXw4hhak-jH3a-j-Kbk57TyfD7DoV3AM22P] 0.043818999💎 / Inited: True
INFO: 🙀 All contracts successfully deployed!
```

Now we can interact with the smart contract using get methods and send message to it (to send message you should copy the address from ./build/contract_address file):
```code
$ toncli get get_internal_counter
INFO: 🚀 You want to interact with your contracts ['contract'] in testnet - that's great!
INFO: 🦘 Found existing deploy-wallet [kQDaR5LvGcB4w_omJcwzKOtQmb167sloQ_PtOkkHQRjJWq2R] (Balance: 1.590868942💎, Is inited: True) in /Users/akkireev/Library/Application Support/toncli
INFO: 👯 [contract] [kQBIp8sJTov02ZXw4hhak-jH3a-j-Kbk57TyfD7DoV3AM22P] runmethod ['get_internal_counter']
INFO: 🧐 Output: [ 0  ]
ERROR: 🧐 Can't auto parse string

$ toncli send -n testnet -a 0.03 --address kQBIp8sJTov02ZXw4hhak-jH3a-j-Kbk57TyfD7DoV3AM22P --body ./fift/msg_body.fif
INFO: 🦘 Found existing deploy-wallet [kQDaR5LvGcB4w_omJcwzKOtQmb167sloQ_PtOkkHQRjJWq2R] (Balance: 1.590868942💎, Is inited: True) in /Users/akkireev/Library/Application Support/toncli
INFO: 🤔 You want to send internal message to [my-cool-smc] from deploy-wallet with amount [0.03]
INFO: 🐰 Getting seqno for transaction
INFO: Run command: /Users/akkireev/Library/Application Support/toncli/wallet/fift/usage.fif build/contract kQBIp8sJTov02ZXw4hhak-jH3a-j-Kbk57TyfD7DoV3AM22P 0 8 0.03 -B /Users/akkireev/Documents/blockchain/my/dev/lessons/advanced_counter/simple/./fift/msg_body.fif
INFO: 💾 Will save BOC to /Users/akkireev/Library/Application Support/toncli/wallet/build/boc/usage.boc
INFO: 🍿 Loading fift CLI lib
INFO: 👀 Source wallet address = kQDaR5LvGcB4w_omJcwzKOtQmb167sloQ_PtOkkHQRjJWq2R
Loading private key from file build/contract.pk
INFO: 👋 Send kQBIp8sJTov02ZXw4hhak-jH3a-j-Kbk57TyfD7DoV3AM22P = subwallet_id=0x0 seqno=0x8 bounce=-1 
INFO: 🧟 Body of transfer message is x{00000652}
INFO: 🥰  Save boc
INFO: 💾 (Saved to file /Users/akkireev/Library/Application Support/toncli/wallet/build/boc/usage.boc)
[ 1][t 1][2022-07-22 14:28:38.270347][lite-client.h:362][!testnode]     conn ready
[ 2][t 1][2022-07-22 14:28:38.342471][lite-client.cpp:363][!testnode]   server version is 1.1, capabilities 7
[ 1][t 1][2022-07-22 14:28:38.488370][lite-client.cpp:1150][!testnode]  sending query from file /Users/akkireev/Library/Application Support/toncli/wallet/build/boc/usage.boc

$ toncli get get_internal_counter
INFO: 🚀 You want to interact with your contracts ['contract'] in testnet - that's great!
INFO: 🦘 Found existing deploy-wallet [kQDaR5LvGcB4w_omJcwzKOtQmb167sloQ_PtOkkHQRjJWq2R] (Balance: 1.555318863💎, Is inited: True) in /Users/akkireev/Library/Application Support/toncli
INFO: 👯 [contract] [kQBIp8sJTov02ZXw4hhak-jH3a-j-Kbk57TyfD7DoV3AM22P] runmethod ['get_internal_counter']
INFO: 🧐 Output: [ 1618  ]
ERROR: 🧐 Can't auto parse string
```
