setupenv:
	( \
       python3 -m venv venv; \
       source venv/bin/activate; \
       pip install -r requirements.txt; \
       wton init; \
       wton config --network testnet; \
    )

updmainnet:
	wget -q https://raw.githubusercontent.com/ton-blockchain/ton-blockchain.github.io/main/global.config.json -O ./mainnet.json 

updtestnet:
	wget -q https://raw.githubusercontent.com/ton-blockchain/ton-blockchain.github.io/main/testnet-global.config.json  -O ./testnet.json

t:
	fift -I ./common/fift/lib -s ./test.fif