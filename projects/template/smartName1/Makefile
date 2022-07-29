# SMART CONTRACT (SC)
sccompile:
	func -o ./build/contract.fif \
		 -SPA ../common/func/lib/stdlib.fc \
		 ./func/op-codes.fc ./func/contract.fc

scdatacompile:
	fift -I ../common/fift/lib \
		 -s ../common/fift/scripts/create-init-data.fif \
		 ./fift/initial-data.fif ./build/boc/initial-data.boc

scinitcompile:
	fift -I ../common/fift/lib \
		 -s ../common/fift/scripts/create-init-message.fif \
		 ./build/contract.fif ./fift/initial-data.fif 0 ./build/boc/init-contract-message.boc ./build/contract.addr ./build/contract-address.txt

scinitsend:
	lite-client -v 3 --timeout 3 -C ./testnet.json -v 2 \
				-c 'sendfile ./build/boc/init_contract_message.boc'

scproccess: sccompile scdatacompile scinitcompile scinitsend

sctest:
	toncli run_tests


# INTERNAL MESSAGE (IM)
imcompile:
	fift -I ../common/fift/lib \
		 -L ../common/fift/scripts/cli.fif \
		 -s ../common/fift/scripts/create-internal-message.fif \
		 build/contract kQA3esZyDjmwHo6eewGLoDjm4kYpc9mFNI_tVYDje101-_nG 698983191 4 0.1 \
		 -B ./fift/internal-message-body.fif

imsend:
	lite-client -v 3 --timeout 3 -C ../testnet.json -v 2 -c 'sendfile ./build/boc/internal_message.boc'

improcess: imcompile imsend


# EXTERNAL MESSAGE (EM)
emcompile:
	fift -I ../common/fift/lib \
		 -s ../common/fift/scripts/create-external-message.fif \
		 ../common/deployer.pk ./build/contract.addr ./fift/external-message-body.fif ./build/boc/external-message.boc

emsend:
	lite-client -v 3 --timeout 3 -C ../testnet.json -v 2 -c 'sendfile ./build/boc/external-message.boc'

extmsgprocess: emcompile emsend


# GET METHODS
get-seqno:
	toncli get seqno
