# Template for TON smart contracts development

*(We would appreciate any contribution)*


## Setup tools
```code
make setupenv
source venv/bin/activate
```

Packages info: [toncli](https://pypi.org/project/toncli/) | [wton](https://pypi.org/project/wton/) 
```code
toncli -h
wton -h
```

## Tools usage example
To run your tests:
```code
toncli run_tests
```

To get your wallet .addr and .pk use the following command:
```code
wton wallet W_NAME to-addr-pk ./common/
```

## Folders
```
template
├── Makefile            - all common operations (e.g. make updtestnet)
├── common/             - for all files that are used in many smart contracts 
│   ├── func/           - libs
│   ├── fift/           - libs, scripts
│   ├── build/          - deployment wallet addr/pk
├── smartName1/         - for particular smart contract 
│   ├── Makefile        - all smart contract specific operations (e.g. make sccompile)
│   ├── func/           - smart contract code, op-codes, etc
│   ├── fift/           - initial data, messages, etc
│   ├── tests/          - smart contract specific tests
│   ├── build/          - to store all smart contract related compiled files
├── smartName2/
...
```

## Usefull resources
- [FIFT](https://ton-blockchain.github.io/docs/fiftbase.pdf)
- [Blockchain](https://ton-blockchain.github.io/docs/tblkch.pdf)
- [TVM](https://ton-blockchain.github.io/docs/tvm.pdf)
- [TLB](https://github.com/ton-blockchain/ton/blob/master/crypto/block/block.tlb)
- [testnet.tonscan](https://testnet.tonscan.org/address/)
