# SMART CONTRACT (SC)
sccompile:
	func -o ./build/contract.fif \
		 -SPA ../common/func/lib/stdlib.fc \
		 ./func/op-codes.fc ./func/contract.fc

scdatacompile:
	fift -I ../common/fift/lib \
		 -s ../common/fift/scripts/fift-to-boc.fif \
		 ./fift/initial-data.fif ./build/boc/initial-data.boc

scinitcompile:
	fift -I ../common/fift/lib \
		 -s ../common/fift/scripts/create-init-message.fif \
		 ./build/contract.fif ./fift/initial-data.fif 0 ./build/boc/init-contract-message.boc ./build/contract.addr ./build/contract-address.txt

scinitsend:
	lite-client -v 3 --timeout 3 -C ../testnet.json -v 2 \
				-c 'sendfile ./build/boc/init-contract-message.boc'

scprocess: sccompile scinitcompile scinitsend

sctest:
	toncli run_tests


# INTERNAL MESSAGE (IM)
imdatacompile:
	fift -I ../common/fift/lib \
		 -s ../common/fift/scripts/fift-to-boc.fif \
		 ./fift/internal-message-body.fif ./build/boc/internal-message-body.boc

imcompile:
	fift -I ../common/fift/lib \
		 -s ../common/fift/scripts/create-internal-message.fif \
		 ../common/deployer.pk ../common/deployer.addr 698983191 1 \
		 kQD6mVG1gaybJWJOb8KoSHTvYQSGlKBZ-Q-8c-KsWsJSdyuU 0.3 \
		 ./build/boc/internal-message.boc \
		 -B ./fift/internal-message-body.fif

imsend:
	lite-client -v 3 --timeout 3 -C ../testnet.json -v 2 -c 'sendfile ./build/boc/internal-message.boc'

improcess: imcompile imsend


# EXTERNAL MESSAGE (EM)
emdatacompile:
	fift -I ../common/fift/lib \
		 -s ../common/fift/scripts/fift-to-boc.fif \
		 ./fift/external-message-body.fif ./build/boc/external-message-body.boc

emcompile:
	fift -I ../common/fift/lib \
		 -s ../common/fift/scripts/create-external-message.fif \
		 ../common/deployer.pk ./build/contract.addr ./fift/external-message-body.fif ./build/boc/external-message.boc

emsend:
	lite-client -v 3 --timeout 3 -C ../testnet.json -v 2 -c 'sendfile ./build/boc/external-message.boc'

emprocess: emcompile emsend


# GET METHODS (G)
gseqno:
	toncli get seqno
