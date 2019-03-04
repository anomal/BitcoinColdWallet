# Bitcoin Cold Wallet

I developed this proof of concept as a part of our "crytocurrency ETF" (exchange-traded fund) hackathon project that won third place at the CIBC DisruptConstruct hackathon in 2017. A realistic cryptocurrency ETF means that the cryptocurrency is secured against network attacks by being stored in an offline or "cold storage" wallet. Below is the overview and Bitcoin Command-Line Interface (`bitcoin-cli`) implementation of how to retrieve your coins from the cold wallet, tested on the Bitcoin test network, `testnet`.

## Overview

### Receiving Bitcoins from the AP into a Cold Storage Wallet
1. Download open source code for generating an offline Bitcoin wallet, e.g., “Paper wallet” JavaScript code from bitaddress.org.  Have developers review the code.  This is a one-time activity.
2. Using a USB thumb drive, copy the open source program to a newly imaged, clean computer that is not and has never been connected to the Internet.  Run the program using custom entropy as input to generate an offline, cold storage wallet consisting of a private key and bitcoin address.  The private key should never leave this computer.  If the private key is exposed, someone can steal all the bitcoins from the corresponding bitcoin wallet.
3. Copy the bitcoin address only to a USB thumb drive and move it to a computer connected to the Internet. 
4.  Provide the bitcoin address to the Authorized Participant (AP), who can deposit bitcoins into the CIBC bitcoin address by broadcasting the signed transaction to the nodes.  Review the public blockchain to confirm that the correct number of bitcoins have been transferred to the CIBC bitcoin address. Once this is done, deliver the Bitcoin ETF creation unit to the Authorized Participant. 

### Withdrawing Bitcoins from the Cold Storage Wallet
1. Authorized Participant delivers the Bitcoin ETF creation unit to CIBC.
2. Create an unsigned transaction to send bitcoins from the CIBC
bitcoin address to the AP's bitcoin address. You must spend almost all the bitcoins in the set, as the remainder would go to the miners as the transaction fee. Using a USB thumb drive, copy the unsigned transaction to the offline computer that has the cold storage wallet.
3. On the offline computer with the cold storage wallet, sign the transaction using the private key.  Copy the signed transaction to the USB thumb drive and transfer it to an online computer.
4. Broadcast the raw signed transaction to the nodes using a website for broadcasting Bitcoin transactions.
5. Wait for the transaction to be mined, and confirm it by viewing the public Bitcoin blockchain.

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

Broadcast your signed transaction on the blockchain by posting it on a public Bitcoin transaction site like blockcypher.com.
