# Bitcoin Cold Wallet

This proof of concept is part of our "crytocurrency ETF" (exchange-traded fund) hackathon project that won third place at the CIBC DisruptConstruct hackathon in 2017. A realistic cryptocurrency ETF means that the cryptocurrency is secured against network attacks by being stored in an offline or "cold storage" wallet. Below is the overview and Bitcoin Command-Line Interface (`bitcoin-cli`) implementation of how to retrieve your coins from the cold wallet, tested on the Bitcoin test network, `testnet`.

## Overview
1. Generate private key + Bitcoin address (public key + network details) pair offline.  This is your cold storage wallet.
2. Provide cold storage wallet address to the Authorized Participant (AP).
3. AP sends bitcoins to the cold storage wallet address by executing the transaction on the blockchain.  The cold storage wallet does not need to be connected to the blockchain for it to receive bitcoins, because the public ledger keeps track of each bitcoin’s owner.
4. Create an online watch wallet with the imported cold storage wallet address to track the wallet’s contents.
5. The watch wallet creates a raw, unsigned transaction to send bitcoins from the cold storage wallet to the AP.
6. Transfer the raw, unsigned transaction to an offline, air-gapped computer that has the cold storage wallet, including the private key.
7. The offline, cold storage wallet signs the raw transaction using its private key.
8. Online, broadcast the signed, raw transaction onto a Bitcoin transaction broadcast site.  The blockchain will execute the transaction, and the AP will receive their bitcoins.

## bitcoin-cli implementation
### Watch wallet

Online, import the Bitcoin address of the cold wallet, view its unspent coins, and create a raw, unsigned transaction on behalf of the cold wallet to send the coins to the new address.
```
watchwallet@itcortex:~/.bitcoin/testnet3$ bitcoin-cli -testnet importaddress mmbtqLpNzDhuEwdkcgwRHwp191Kot93uEG
watchwallet@itcortex:~/.bitcoin/testnet3$ bitcoin-cli -testnet listunspent
[
  {
    "txid": "26492a2dbd352e767c79704d4cbab42657dac6d0817eaee9ed72c04d093ce945",
    "vout": 0,
    "address": "mmbtqLpNzDhuEwdkcgwRHwp191Kot93uEG",
    "account": "",
    "scriptPubKey": "76a91442bff21fa29af71ba21dddb57090c7d2eef08c2788ac",
    "amount": 0.00001000,
    "confirmations": 285,
    "spendable": false,
    "solvable": false
  }, 
  {
    "txid": "76a31f947082d6a5d24c840de8d1b4f4dc138d7a8cf00f8977c9f2248a5813e9",
    "vout": 0,
    "address": "mmbtqLpNzDhuEwdkcgwRHwp191Kot93uEG",
    "account": "",
    "scriptPubKey": "76a91442bff21fa29af71ba21dddb57090c7d2eef08c2788ac",
    "amount": 0.10000000,
    "confirmations": 290,
    "spendable": false,
    "solvable": false
  }
]
watchwallet@itcortex:~/.bitcoin/testnet3$ UTXO_TXID=76a31f947082d6a5d24c840de8d1b4f4dc138d7a8cf00f8977c9f2248a5813e9
watchwallet@itcortex:~/.bitcoin/testnet3$ UTXO_VOUT=0
watchwallet@itcortex:~/.bitcoin/testnet3$ UTXO_OUTPUT_SCRIPT=76a91442bff21fa29af71ba21dddb57090c7d2eef08c2788ac
watchwallet@itcortex:~/.bitcoin/testnet3$ NEW_ADDRESS=mwLVPtn5eWKeUhTuvWv429WM8t2xciwatb
watchwallet@itcortex:~/.bitcoin/testnet3$ bitcoin-cli -testnet createrawtransaction '''
>     [
>       {
>         "txid": "'$UTXO_TXID'",
>         "vout": '$UTXO_VOUT'
>       }
>     ]
>     ''' '''
>     {
>       "'$NEW_ADDRESS'": 0.0999
>     }'''
0200000001e913588a24f2c977890ff08c7a8d13dcf4b4d1e80d844cd2a5d68270941fa3760000000000ffffffff01706f9800000000001976a914ad87660439a1a70e43b20bcaf51029c4dfddf20688ac00000000
watchwallet@itcortex:~/.bitcoin/testnet3$ RAW_TX=0200000001e913588a24f2c977890ff08c7a8d13dcf4b4d1e80d844cd2a5d68270941fa3760000000000ffffffff01706f9800000000001976a914ad87660439a1a70e43b20bcaf51029c4dfddf20688ac00000000
```
### Cold wallet

Offline, import the private key of your cold wallet and sign the raw transaction you created from the watch wallet.
```
coldwallet@itcortex:~/.bitcoin/testnet3$ bitcoin-cli -testnet importprivkey cNos9jFnDXWatn7TXVor1X8nJJPm7CDxxb9QnXAmHiC8kCCUsf85
coldwallet@itcortex:~/.bitcoin/testnet3$ RAW_TX=0200000001e913588a24f2c977890ff08c7a8d13dcf4b4d1e80d844cd2a5d68270941fa3760000000000ffffffff01706f9800000000001976a914ad87660439a1a70e43b20bcaf51029c4dfddf20688ac00000000
coldwallet@itcortex:~/.bitcoin/testnet3$ bitcoin-cli -testnet signrawtransaction $RAW_TX '''
>     [
>       {
>         "txid": "'$UTXO_TXID'", 
>         "vout": '$UTXO_VOUT', 
>         "scriptPubKey": "'$UTXO_OUTPUT_SCRIPT'"
>       }
>     ]'''
{
  "hex": "0200000001e913588a24f2c977890ff08c7a8d13dcf4b4d1e80d844cd2a5d68270941fa376000000006b483045022100d49d930a515bf3364a400081e9b1893439e5e42f9f7142e227f062ce66a20aa8022022aee2c3de857d60e61eeaca886a7632c3930a9dfa1b14eeb289ca68d9d2163a012102280bf9b163635a303b70eee2e188401e5c861c898cc08759a8b7029a14865ba2ffffffff01706f9800000000001976a914ad87660439a1a70e43b20bcaf51029c4dfddf20688ac00000000",
  "complete": true
}
coldwallet@itcortex:~/.bitcoin/testnet3$ 
```
### Broadcast signed transaction

Broadcast your signed transaction on the blockchain by posting it on public Bitcoin transaction site like blockcypher.com.
