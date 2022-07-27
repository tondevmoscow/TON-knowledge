# SMART CONTRACT
build-func:
	func -o ./build/advanced_counter.fif -SPA ./func/lib/stdlib.fc ./func/advanced_counter.fc

init-data-to-boc:
	fift -I ./fift/lib/fift-libs -s ./fift/create_init_data_boc.fif

build-init-contract-message:
	fift -I ./fift/lib/fift-libs -s ./fift/lib/contract_manipulation.fif ./build/advanced_counter.fif ./build/boc/init_data_cell.boc 0 ./build/boc/init_contract_message.boc ./build/advanced_counter_address

send-init-contract-message:
	lite-client -v 3 --timeout 3 -C ./testnet.json -v 2 -c 'sendfile ./build/boc/init_contract_message.boc'


# INTERNAL MESSAGE
build-internal-message:
	fift -I ./fift/lib/fift-libs -L ./fift/lib/cli.fif -s ./fift/create_internal_message.fif build/contract kQA3esZyDjmwHo6eewGLoDjm4kYpc9mFNI_tVYDje101-_nG 698983191 4 0.1 -B ./fift/internal_msg_body_to_increase_counter.fif

send-internal-message:
	lite-client -v 3 --timeout 3 -C ./testnet.json -v 2 -c 'sendfile ./build/boc/internal_message.boc'


# EXTERNAL MESSAGE
build-external-message:
	fift -I ./fift/lib/fift-libs -L ./fift/lib/cli.fif -s ./fift/create_external_message.fif

send-external-message:
	lite-client -v 3 --timeout 3 -C ./testnet.json -v 2 -c 'sendfile ./build/boc/external_message.boc'


# GET METHODS
get-all:
	lite-client -v 3 --timeout 3 -C ./testnet.json -v 0 -c 'runmethod kQA3esZyDjmwHo6eewGLoDjm4kYpc9mFNI_tVYDje101-_nG get_all'

get-external:
	lite-client -v 3 --timeout 3 -C ./testnet.json -v 0 -c 'runmethod kQA3esZyDjmwHo6eewGLoDjm4kYpc9mFNI_tVYDje101-_nG get_external_counter'

get-owner-external:
	lite-client -v 3 --timeout 3 -C ./testnet.json -v 0 -c 'runmethod kQA3esZyDjmwHo6eewGLoDjm4kYpc9mFNI_tVYDje101-_nG get_owner_external_counter'

get-internal:
	lite-client -v 3 --timeout 3 -C ./testnet.json -v 0 -c 'runmethod kQA3esZyDjmwHo6eewGLoDjm4kYpc9mFNI_tVYDje101-_nG get_internal_counter'

get-owner-internal:
	lite-client -v 3 --timeout 3 -C ./testnet.json -v 0 -c 'runmethod kQA3esZyDjmwHo6eewGLoDjm4kYpc9mFNI_tVYDje101-_nG get_owner_internal_counter'
