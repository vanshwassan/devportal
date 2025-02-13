---
menu_order: 300
menu_title: Local Network with Incentives
layout: rsk
title: Running a local Swarm network on Rootstock (RSK)
tags: rif, rif-storage, ipfs, swarm, storage, node, sdk, libraries, infrastructure, protocols, mvp, design, rbtc, defi, decentralized, quick-start, guides, tutorial, networks, dapps, tools, rootstock, rsk, ethereum, smart-contracts, install, get-started, how-to, mainnet, testnet, contracts, wallets, web3, crypto
---

This guide sets up 2 Swarm nodes in a local private network. Each of the nodes is loaded into a specific directory; i.e. the folders `./s1` and `./s2`.

The _Swarm Accounting Protocol_ (**SWAP**) is required in order to form an <a href="/rif/storage/providers/swarm/incentives/">incentivized network</a> in Swarm. This requires chequebook Smart Contracts which keep track of the accounting done between Swarm nodes, and therefore there is need for a blockchain.

If you need these nodes to run without incentivization, please refer to the much simpler <a href="/rif/storage/providers/swarm/guides/local-network/">Standard version</a> of this guide.

## Table of Contents

1. [Requirements](#requirements)
2. [Run the network](#run-the-network)
3. [Interact with the network](#interact-with-the-network)
4. [Restart the network](#restart-the-network)
5. [Add more nodes to the network](#add-more-nodes-to-the-network)
6. [Known issues](#known-issues)

---

# Requirements

## 1. RSKj

[RSKj](https://github.com/rsksmart/rskj) will be the node used to start the local Rootstock (RSK) blockchain.

Download it from:

- [Google Drive](https://drive.google.com/open?id=1zdYpL6kFaOtyZjhbssWkNbYzdQwBA36j)
- [Swarm](https://swarm-gateways.net/bzz:/52d5a1a3a18cbb2b163288554159eabb2c1a531883df23fba5ce329a18285279) (make sure to rename the downloaded file to `rskj-core-unformatted-log-all.jar`)

## 2. `websocat` (optional)

`websocat` is a command-line web socket client, used to query the nodes running in the private network.

If you plan to [query the nodes](#query-the-nodes), follow the instructions [here](https://github.com/vi/websocat/) to install it.

---

# Run the Network

# 1. Start the blockchain

## 1.1. Start RSKj

Start the RSK node through:

```bash
rm -rf datadir # clean up previous directory
java -jar rskj-core-unformatted-log-all.jar co.rsk.Start --regtest -base-path datadir
```

This starts the blockchain with the pre-mined address of `0xcd2a3d9f938e13cd947ec05abc7fe734df8dd826`.

## 1.2. Deploy a SWAP Factory Smart Contract

A `SimpleSwapFactory` Smart Contract will need to be deployed to the blockchain next.

This is a security measure to verify chequebook contracts deployed by the nodes in the network.

Execute:

```bash
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_sendTransaction","params":[{ "from": "0xcd2a3d9f938e13cd947ec05abc7fe734df8dd826", "gas": "0x2dc6c0", "data": "0x608060405234801561001057600080fd5b50612228806100206000396000f3fe6080604052600436106100295760003560e01c8063576d72711461002e578063c70242ad146100bc575b600080fd5b61007a6004803603604081101561004457600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff16906020019092919080359060200190929190505050610125565b604051808273ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200191505060405180910390f35b3480156100c857600080fd5b5061010b600480360360208110156100df57600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff169060200190929190505050610258565b604051808215151515815260200191505060405180910390f35b60008034848460405161013790610278565b808373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001828152602001925050506040518091039082f08015801561018f573d6000803e3d6000fd5b509050905060016000808373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002060006101000a81548160ff0219169083151502179055507fc0ffc525a1c7689549d7f79b49eca900e61ac49b43d977f680bcc3b36224c00481604051808273ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200191505060405180910390a18091505092915050565b60006020528060005260406000206000915054906101000a900460ff1681565b611f6e806102868339019056fe6080604052604051611f6e380380611f6e8339818101604052604081101561002657600080fd5b8101908080519060200190929190805190602001909291905050508060008190555081600460006101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff16021790555060003411156100fe577fe1fffcc4923d04b559f4d29a8bfc6cda04eb5b0d3c460751c2402c5c5cc9109c3334604051808373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020018281526020019250505060405180910390a15b5050611e5f8061010f6000396000f3fe6080604052600436106100dd5760003560e01c806381f03fcb1161007f578063b777035011610059578063b7770350146106a6578063b7ec1a3314610701578063df3243801461072c578063e0bcf13a1461081e576100dd565b806381f03fcb14610576578063946f46a2146105db578063b6343b0d1461062c576100dd565b80631d143848116100bb5780631d1438481461045e5780632e1a7d4d146104b5578063338f3fed146104f05780635eb541601461054b576100dd565b8063065c804f146101545780630d5f2659146101b95780631633fb1d146102ab575b6000341115610152577fe1fffcc4923d04b559f4d29a8bfc6cda04eb5b0d3c460751c2402c5c5cc9109c3334604051808373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020018281526020019250505060405180910390a15b005b34801561016057600080fd5b506101a36004803603602081101561017757600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff169060200190929190505050610849565b6040518082815260200191505060405180910390f35b3480156101c557600080fd5b506102a9600480360360608110156101dc57600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff169060200190929190803590602001909291908035906020019064010000000081111561022357600080fd5b82018360208201111561023557600080fd5b8035906020019184600183028401116401000000008311171561025757600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600081840152601f19601f8201169050808301925050505050505091929192905050506108ae565b005b3480156102b757600080fd5b5061045c600480360360c08110156102ce57600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff169060200190929190803573ffffffffffffffffffffffffffffffffffffffff169060200190929190803590602001909291908035906020019064010000000081111561033557600080fd5b82018360208201111561034757600080fd5b8035906020019184600183028401116401000000008311171561036957600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600081840152601f19601f82011690508083019250505050505050919291929080359060200190929190803590602001906401000000008111156103d657600080fd5b8201836020820111156103e857600080fd5b8035906020019184600183028401116401000000008311171561040a57600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600081840152601f19601f8201169050808301925050505050505091929192905050506108c1565b005b34801561046a57600080fd5b5061047361096f565b604051808273ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200191505060405180910390f35b3480156104c157600080fd5b506104ee600480360360208110156104d857600080fd5b8101908080359060200190929190505050610995565b005b3480156104fc57600080fd5b506105496004803603604081101561051357600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff16906020019092919080359060200190929190505050610b5b565b005b34801561055757600080fd5b50610560610d80565b6040518082815260200191505060405180910390f35b34801561058257600080fd5b506105c56004803603602081101561059957600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff169060200190929190505050610d86565b6040518082815260200191505060405180910390f35b3480156105e757600080fd5b5061062a600480360360208110156105fe57600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff169060200190929190505050610d9e565b005b34801561063857600080fd5b5061067b6004803603602081101561064f57600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff169060200190929190505050610ef1565b6040518085815260200184815260200183815260200182815260200194505050505060405180910390f35b3480156106b257600080fd5b506106ff600480360360408110156106c957600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff16906020019092919080359060200190929190505050610f21565b005b34801561070d57600080fd5b50610716611109565b6040518082815260200191505060405180910390f35b34801561073857600080fd5b5061081c6004803603606081101561074f57600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff169060200190929190803590602001909291908035906020019064010000000081111561079657600080fd5b8201836020820111156107a857600080fd5b803590602001918460018302840111640100000000831117156107ca57600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600081840152601f19601f82011690508083019250505050505050919291929050505061113c565b005b34801561082a57600080fd5b50610833611372565b6040518082815260200191505060405180910390f35b60006108a7600260008473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002060000154610899611109565b61137890919063ffffffff16565b9050919050565b6108bc338484600085611400565b505050565b6108d76108d13033878987611925565b84611a06565b73ffffffffffffffffffffffffffffffffffffffff168673ffffffffffffffffffffffffffffffffffffffff161461095a576040517f08c379a0000000000000000000000000000000000000000000000000000000008152600401808060200182810382526022815260200180611d956022913960400191505060405180910390fd5b6109678686868585611400565b505050505050565b600460009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1681565b600460009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff1614610a58576040517f08c379a00000000000000000000000000000000000000000000000000000000081526004018080602001828103825260168152602001807f53696d706c65537761703a206e6f74206973737565720000000000000000000081525060200191505060405180910390fd5b610a60611109565b811115610ab8576040517f08c379a0000000000000000000000000000000000000000000000000000000008152600401808060200182810382526028815260200180611e036028913960400191505060405180910390fd5b600460009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff166108fc829081150290604051600060405180830381858888f19350505050158015610b20573d6000803e3d6000fd5b507f5b6b431d4476a211bb7d41c20d1aab9ae2321deee0d20be3d9fc9b1093fa6e3d816040518082815260200191505060405180910390a150565b600460009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff1614610c1e576040517f08c379a00000000000000000000000000000000000000000000000000000000081526004018080602001828103825260168152602001807f53696d706c65537761703a206e6f74206973737565720000000000000000000081525060200191505060405180910390fd5b3073ffffffffffffffffffffffffffffffffffffffff1631610c4b8260035461137890919063ffffffff16565b1115610ca2576040517f08c379a0000000000000000000000000000000000000000000000000000000008152600401808060200182810382526034815260200180611d616034913960400191505060405180910390fd5b6000600260008473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000209050610cfc82826000015461137890919063ffffffff16565b8160000181905550610d198260035461137890919063ffffffff16565b600381905550600081600301819055508273ffffffffffffffffffffffffffffffffffffffff167f2506c43272ded05d095b91dbba876e66e46888157d3e078db5691496e96c5fad82600001546040518082815260200191505060405180910390a2505050565b60005481565b60016020528060005260406000206000915090505481565b6000600260008373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020905080600301544210158015610dfa57506000816003015414155b610e4f576040517f08c379a0000000000000000000000000000000000000000000000000000000008152600401808060200182810382526025815260200180611db76025913960400191505060405180910390fd5b610e6a81600101548260000154611a2290919063ffffffff16565b816000018190555060008160030181905550610e958160010154600354611a2290919063ffffffff16565b6003819055508173ffffffffffffffffffffffffffffffffffffffff167f2506c43272ded05d095b91dbba876e66e46888157d3e078db5691496e96c5fad82600001546040518082815260200191505060405180910390a25050565b60026020528060005260406000206000915090508060000154908060010154908060020154908060030154905084565b600460009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff1614610fe4576040517f08c379a00000000000000000000000000000000000000000000000000000000081526004018080602001828103825260168152602001807f53696d706c65537761703a206e6f74206973737565720000000000000000000081525060200191505060405180910390fd5b6000600260008473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002090508060000154821115611084576040517f08c379a0000000000000000000000000000000000000000000000000000000008152600401808060200182810382526027815260200180611ddc6027913960400191505060405180910390fd5b60008082600201541461109b57816002015461109f565b6000545b905080420182600301819055508282600101819055508373ffffffffffffffffffffffffffffffffffffffff167fc8305077b495025ec4c1d977b176a762c350bb18cad4666ce1ee85c32b78698a846040518082815260200191505060405180910390a250505050565b60006111376003543073ffffffffffffffffffffffffffffffffffffffff1631611a2290919063ffffffff16565b905090565b600460009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff16146111ff576040517f08c379a00000000000000000000000000000000000000000000000000000000081526004018080602001828103825260168152602001807f53696d706c65537761703a206e6f74206973737565720000000000000000000081525060200191505060405180910390fd5b61121361120d308585611aab565b82611a06565b73ffffffffffffffffffffffffffffffffffffffff168373ffffffffffffffffffffffffffffffffffffffff1614611296576040517f08c379a0000000000000000000000000000000000000000000000000000000008152600401808060200182810382526022815260200180611d956022913960400191505060405180910390fd5b81600260008573ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020600201819055508273ffffffffffffffffffffffffffffffffffffffff167f86b5d1492f68620b7cc58d71bd1380193d46a46d90553b73e919e0c6f319fe1f600260008673ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020600201546040518082815260200191505060405180910390a2505050565b60035481565b6000808284019050838110156113f6576040517f08c379a000000000000000000000000000000000000000000000000000000000815260040180806020018281038252601b8152602001807f536166654d6174683a206164646974696f6e206f766572666c6f77000000000081525060200191505060405180910390fd5b8091505092915050565b600460009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff161461152c57611469611463308786611b4b565b82611a06565b73ffffffffffffffffffffffffffffffffffffffff16600460009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff161461152b576040517f08c379a000000000000000000000000000000000000000000000000000000000815260040180806020018281038252601d8152602001807f53696d706c65537761703a20696e76616c69642069737375657253696700000081525060200191505060405180910390fd5b5b6000611580600160008873ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000205485611a2290919063ffffffff16565b905060006115968261159189610849565b611beb565b905060006115e682600260008b73ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002060000154611beb565b90508482101561165e576040517f08c379a000000000000000000000000000000000000000000000000000000000815260040180806020018281038252601d8152602001807f53696d706c65537761703a2063616e6e6f74207061792063616c6c657200000081525060200191505060405180910390fd5b6000811461171d576116bb81600260008b73ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002060000154611a2290919063ffffffff16565b600260008a73ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000206000018190555061171681600354611a2290919063ffffffff16565b6003819055505b61176f82600160008b73ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000205461137890919063ffffffff16565b600160008a73ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020819055508673ffffffffffffffffffffffffffffffffffffffff166108fc6117df8785611a2290919063ffffffff16565b9081150290604051600060405180830381858888f1935050505015801561180a573d6000803e3d6000fd5b506000851461185b573373ffffffffffffffffffffffffffffffffffffffff166108fc869081150290604051600060405180830381858888f19350505050158015611859573d6000803e3d6000fd5b505b3373ffffffffffffffffffffffffffffffffffffffff168773ffffffffffffffffffffffffffffffffffffffff168973ffffffffffffffffffffffffffffffffffffffff167f950494fc3642fae5221b6c32e0e45765c95ebb382a04a71b160db0843e74c99f858a8a60405180848152602001838152602001828152602001935050505060405180910390a481831461191b577f3f4449c047e11092ec54dc0751b6b4817a9162745de856c893a26e611d18ffc460405160405180910390a15b5050505050505050565b60008585858585604051602001808673ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1660601b81526014018573ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1660601b81526014018481526020018373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1660601b81526014018281526020019550505050505060405160208183030381529060405280519060200120905095945050505050565b6000611a1a611a1484611c04565b83611c5c565b905092915050565b600082821115611a9a576040517f08c379a000000000000000000000000000000000000000000000000000000000815260040180806020018281038252601e8152602001807f536166654d6174683a207375627472616374696f6e206f766572666c6f77000081525060200191505060405180910390fd5b600082840390508091505092915050565b6000838383604051602001808473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1660601b81526014018373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1660601b815260140182815260200193505050506040516020818303038152906040528051906020012090509392505050565b6000838383604051602001808473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1660601b81526014018373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1660601b815260140182815260200193505050506040516020818303038152906040528051906020012090509392505050565b6000818310611bfa5781611bfc565b825b905092915050565b60008160405160200180807f19457468657265756d205369676e6564204d6573736167653a0a333200000000815250601c01828152602001915050604051602081830303815290604052805190602001209050919050565b60006041825114611c705760009050611d5a565b60008060006020850151925060408501519150606085015160001a90507f7fffffffffffffffffffffffffffffff5d576e7357a4501ddfe92f46681b20a08260001c1115611cc45760009350505050611d5a565b601b8160ff1614158015611cdc5750601c8160ff1614155b15611ced5760009350505050611d5a565b60018682858560405160008152602001604052604051808581526020018460ff1660ff1681526020018381526020018281526020019450505050506020604051602081039080840390855afa158015611d4a573d6000803e3d6000fd5b5050506020604051035193505050505b9291505056fe53696d706c65537761703a2068617264206465706f7369742063616e6e6f74206265206d6f7265207468616e2062616c616e636553696d706c65537761703a20696e76616c69642062656e656669636961727953696753696d706c65537761703a206465706f736974206e6f74207965742074696d6564206f757453696d706c65537761703a2068617264206465706f736974206e6f742073756666696369656e7453696d706c65537761703a206c697175696442616c616e6365206e6f742073756666696369656e74a265627a7a7231582089af678b6e43eebcf058524d8177d9f06f37f73f03c7dec537f3652623c5ea4f64736f6c634300050b0032a265627a7a72315820ebd4cb32a61da1ceb5a10453df5bff99698d82ce29addf07fe22d33246f941de64736f6c634300050b0032"}], "id":1 }' http://localhost:4444
```

The factory will be deployed at address `0x77045e71a7a2c50903d88e564cd72fab11e82051`.

## 1.3. Pre-fund the nodes

Execute:

```bash
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_sendTransaction","params":[{ "from": "0xcd2a3d9f938e13cd947ec05abc7fe734df8dd826", "to": "0xbf4f9637C281DDFb1Fbd3be5a1daE6531D408F11", "value":"0xde0b6b3a7640000" }], "id":1 }' http://localhost:4444 &&
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_sendTransaction","params":[{ "from": "0xcd2a3d9f938e13cd947ec05abc7fe734df8dd826", "to": "0xc45D64d8F9642a604Db93C59FD38492b262391CA", "value":"0xde0b6b3a7640000" }], "id":1 }' http://localhost:4444
```

Unlike in Ethereum, transactions don't confirm instantly here. Wait a few seconds after this step!

# 2. Start the nodes

## 2.1. Choose a directory

Start a terminal and run `cd` to move to a directory where the files for the nodes will be created.

## 2.2. Start each node

Make sure the `swarm` command boots up Swarm correctly before starting the nodes.

Start the first node through:

```bash
DATADIR1="s1" &&
rm -rf "$DATADIR1" && mkdir "$DATADIR1/" &&
swarm --bzznetworkid 5 --datadir "$DATADIR1" --swap --swap-backend-url http://localhost:4444 --ws --wsaddr=0.0.0.0 --wsapi=accounting,bzz,swap --wsorigins='*' --no-sync --bzzkeyhex 40b3e576b606d4580ad3c875e9fda07ba3e4d99a40534c5bf1bc72226451adb1 --nodekeyhex 2eae3526db799cb5f1ab6ab64255ba8182cdaeb4f773a0ae1244f4ca59978dc2 --swap-initial-deposit 500000000000 --swap-chequebook-factory 0x77045e71a7a2c50903d88e564cd72fab11e82051
```

Start the second node through:

```bash
DATADIR2="s2" &&
rm -rf "$DATADIR2" && mkdir "$DATADIR2" &&
swarm --bzznetworkid 5 --datadir "$DATADIR2" --port 40400 --swap --swap-backend-url http://localhost:4444 --bzzport 9100 --bootnodes "enode://9b7571c26d50bed78f614be5bf3b2d661176fdfeb546f100b84dd03545f4bc98e42e640286ac92fe110ec5f4995141743e47d8f642aa49ac05bd5f2cab2e881a@127.0.0.1:30399" --ws --wsaddr=0.0.0.0 --wsapi=accounting,bzz,swap --wsport 8556 --wsorigins='*' --no-sync --bzzkeyhex 73f0f0e024f09059acb267f836ba7924e0a02fe9dd8089293e7ca3b7a1c14a67 --swap-initial-deposit 500000000000 --swap-chequebook-factory 0x77045e71a7a2c50903d88e564cd72fab11e82051
```

This will populate the directories with all of the files needed for each of the Swarm nodes.

Note that:

- The node accounts will already be pre-funded in Ganache; this is achieved through the use of a specific seed and hard-coded node keys.
- The Swarm network id `5` must be used for incentives to be enabled. This value can be verified by inspecting the `AllowedNetworkID` variable set in the [codebase](https://github.com/ethersphere/swarm).
- Sync is disabled by default in order to be able to [trigger a cheque manually](#trigger-cheques) later.
- A private key is specified in the first node so that the second one can find it through the `bootnodes` parameter.
- The first node uses default values for parameters such as ports, but the second one needs them explicitely specified to avoid clashes.
- Both nodes have web socket endpoints enabled through the `ws` (and related) parameters. The APIs enabled in this case are `accounting`, `bzz`, and `swap`. You can omit this parameter if you don't plan to [query the nodes](#query-the-nodes).

---

# Interact with the Network

You can interact with the nodes in the network through the following means.

## Query the nodes

`websocat` can be used to call Swarm functions exposed through [RPC](https://www.tutorialspoint.com/remote-procedure-call-rpc).

For example, to query all balances for the node listening on port `8546`, execute the following:

```bash
echo swap_balances | websocat "ws://127.0.0.1:8546" --origin localhost --jsonrpc -n --one-message &&
```

Note that this particular example requires SWAP to be enabled for this node.

Other calls might only be available depending on the `wsapi` [configuration parameter](https://swarm-guide.readthedocs.io/en/latest/node_operator.html#general-configuration-parameters).

The Swarm documentation might not be up-to-date in terms of including all exposed functions. Search for the `rpc.API` string in the Swarm [codebase](https://github.com/ethersphere/swarm) to figure out which calls are available.

## Web interface

A Swarm local web server endpoint for each node should be accessible through your browser.

The web interface will allow you to upload and download files.

By default, the server will be located at `http://localhost:8500`.

You can find which port is used for each node by taking a look at the `bzzport` flag used in each case.

## CLI

By using the `swarm` binary you can execute operations in the standard manner, such as `up`, `down`, etc., for the node that uses the default parameters (such as port `8500`).

You can find a list of commands [here](https://swarm-guide.readthedocs.io/en/latest/dapp_developer/upload_cli.html#reference-table).

## Trigger cheques

It's possible to generate a random file and upload it to the network in order to trigger a cheque to be sent from one node to the other.

The following command generates a 1.6 MB file, pushes it to one node and then retrieves it on the other.

The file should be large enough to immediately trigger a cheque.

```bash
dd if=/dev/urandom of=output bs=1600k count=1 &&
swarm --bzzapi http://localhost:8500 down \
"bzz-raw://$(swarm --bzzapi http://localhost:9100 up output)"
```

If this doesn't work, check that:

- The endpoints (specified in `bzzapi`) match the parameters for starting the nodes.
- The size of the file is enough to trigger a cheque to be sent. This can be verified through the value of the `SwapPaymentThreshold` variable set in the [codebase](https://github.com/ethersphere/swarm).

---

# Restart the Network

If you want to start from scratch, simply execute the entire code again.

If you want the to maintain state when restarting the network, only repeat the `swarm` command for each node (make sure the `DATADIR` variables are defined) found in the [Start the nodes section](#2-start-the-nodes).

---

# Add more nodes to the network

To start up more Swarm nodes, repeat the previous instructions with as many directories (`./s1`, `./s2`, `./s3`, ..., `./sn`) as you wish.

Make sure that:

- You use different ports for each node.
- All nodes except the first one have the same `bootnodes` parameter value.
- Nodes use node IDs (`bzzkeyhex`) which are prefunded by the Ganache seed.
  - Alternatively, use node IDs outside of this seed to see how nodes without funding behave in an incentivized network.

---

# Known issues

## Failure to suggest Gas price

### Cashing cheques does not work

```bash
ERROR[11-26|16:29:40.620] error cashing cheque
swaplog=* base=d79b7346431a6112 err="failed to suggest gas price: Post http://localhost:4444: EOF"
```

### Chequebook deploying requires multiple tries

```bash
INFO [11-26|16:32:48.728] Deploying new swap                       swaplog=* base=d79b7346431a6112 owner=0xc45D64d8F9642a604Db93C59FD38492b262391CA deposit=500000000000
WARN [11-26|16:32:48.733] chequebook deploy error, retrying...     swaplog=* base=d79b7346431a6112 try=0 error="failed to suggest gas price: Post http://localhost:4444: EOF"
INFO [11-26|16:32:51.780] Deployed chequebook                      swaplog=* base=d79b7346431a6112 contract address=0x8e9a2a73BCA088852de1b6b510D9d9673cc09e90 deposit=500000000000 owner=0xc45D64d8F9642a604Db93C59FD38492b262391CA
```

#### Linux

On a Linux VM it works less often than on MAC.

```bash
INFO [11-26|17:09:11.769] Deploying new swap                       swaplog=* base=1b02472d87b178b0 owner=0xbf4f9637C281DDFb1Fbd3be5a1daE6531D408F11 deposit=500000000000
WARN [11-26|17:09:11.777] chequebook deploy error, retrying...     swaplog=* base=1b02472d87b178b0 try=0 error="Post http://localhost:4444: read tcp 127.0.0.1:40528->127.0.0.1:4444: read: connection reset by peer"
WARN [11-26|17:09:12.784] chequebook deploy error, retrying...     swaplog=* base=1b02472d87b178b0 try=1 error="failed to suggest gas price: Post http://localhost:4444: read tcp 127.0.0.1:40532->127.0.0.1:4444: read: connection reset by peer"
WARN [11-26|17:09:13.790] chequebook deploy error, retrying...     swaplog=* base=1b02472d87b178b0 try=2 error="failed to suggest gas price: Post http://localhost:4444: EOF"
WARN [11-26|17:09:14.796] chequebook deploy error, retrying...     swaplog=* base=1b02472d87b178b0 try=3 error="failed to suggest gas price: Post http://localhost:4444: read tcp 127.0.0.1:40540->127.0.0.1:4444: read: connection reset by peer"
WARN [11-26|17:09:15.801] chequebook deploy error, retrying...     swaplog=* base=1b02472d87b178b0 try=4 error="failed to suggest gas price: Post http://localhost:4444: read tcp 127.0.0.1:40544->127.0.0.1:4444: read: connection reset by peer"
Fatal: Error starting protocol stack: failed to deploy chequebook: <nil>
```

Under the hood Gas Price Suggestion uses `eth_gasPrice` for RPC connections.

This call works on RSKj as verified with `curl`:

```bash
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_gasPrice","params":[], "id":1 }' http://localhost:4444
```

It is unknown if this problem is due to SWAP or the RSK node.

---

_Guide based on [Swap Rootstock (RSK) Guide](https://hackmd.io/ijxzDYaySrqZ06LsR36OsA?view) by Ralph Pichler._
