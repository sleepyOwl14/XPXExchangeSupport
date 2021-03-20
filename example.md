## About XPX

- The default network currency for Sirius Chain
- Represent with asset we call `mosaic`
- Alias to `prx.xpx` namespace, any transaction can directly use this namespace to transfer
- Have divisibility of 6
- MosaicID will be different in different chain, so might be a good idea to cache it in hex format

&nbsp;

## Check incoming transaction

With REST API: 

Query Parameters (Optional):
- pageSize: [10-100]
- id: \<transaction id to start from>
- ordering: '-id' | 'id'

\<REST API URL>/account/{publicKey | address}/transactions/incoming

[Incoming Transaction Endpoint Reference](https://bcdocs.xpxsirius.io/endpoints/#operation/incomingTransactions)

With SDK:
```js
// tsjs-xpx-chain-sdk

// you can use ES6 import or require
const sdk = require('tsjs-xpx-chain-sdk'); 

let API_URL = "https://arcturus.xpxsirius.io";
const accountHttp = new sdk.AccountHttp(API_URL);

const address = sdk.Address.createFromRawAddress(<raw_address>);

let pageSize = 10, // min 10, max 100
        lastId = '', // the transaction ID to start query from
        order = sdk.Order.ASC;
let qp = new sdk.QueryParams(pageSize, lastId, order)

accountHttp.incomingTransactions(address, qp).subscribe((transactions)=>{
    
	// loop through each transactions
    for(let tx of transactions){

        // get transaction id with - tx.transactionInfo.id

        // your code
    }
});


```

&nbsp;

## Check outgoing transaction

With REST API: 

Query Parameters (Optional):
- pageSize: [10-100]
- id: \<transaction id to start from>
- ordering: '-id' | 'id'

\<REST API URL>/account/{publicKey}/transactions/outgoing

[Outgoing Transaction Endpoint Reference](https://bcdocs.xpxsirius.io/endpoints/#operation/outgoingTransactions)

With SDK:
```js
// tsjs-xpx-chain-sdk

// you can use ES6 import or require
const sdk = require('tsjs-xpx-chain-sdk'); 

let API_URL = "https://arcturus.xpxsirius.io";

// please use the correct network type for different blockchain network
let network = sdk.NetworkType.MAIN_NET; 
const accountHttp = new sdk.AccountHttp(API_URL);

const publicAccount = sdk.PublicAccount.createFromPublicKey(<publicKey>, sdk.NetworkType.MAIN_NET);

let pageSize = 10, // min 10, max 100
        lastId = '', // the transaction ID to start query from
        order = sdk.Order.ASC;
let qp = new sdk.QueryParams(pageSize, lastId, order)

accountHttp.outgoingTransactions(publicAccount, qp).subscribe((transactions)=>{
    
	// loop through each transactions
    for(let tx of transactions){

        // get transaction id with - tx.transactionInfo.id

        // your code
    }
});
```

&nbsp;

## Send transaction

With SDK:
```js
// tsjs-xpx-chain-sdk

const sdk = require('tsjs-xpx-chain-sdk');

var API_URL = "https://arcturus.xpxsirius.io";
const transactionHttp = new sdk.TransactionHttp(API_URL);

// please use the correct network type for different blockchain network
let network = sdk.NetworkType.MAIN_NET; 

// please get it from the API at block 1
// eg. https://arcturus.xpxsirius.io/block/1
const generationHash = '10540AD3A1BF46B1A05D8B1CF0252BC9FB2E0B53CFD748262B0CE341CEAFEB6B'; 

const account = sdk.Account.createFromPrivateKey(<privateKey>, network)
const recipientAddress = sdk.Address.createFromRawAddress(<raw_address>);

const tx = sdk.TransferTransaction.create(
    sdk.Deadline.create(),
    recipientAddress,
    [
       sdk.NetworkCurrencyMosaic.createRelative(1) // Sending 1 XPX 
    ],
    sdk.PlainMessage.create('Type message here or left empty'),
    network
);

// sign the transaction with account
const signedTransaction = account.sign(tx, generationHash);

// **please track the annnounced transaction by transasction hash, signedTransaction.hash

// announce the transaction, it will always return message received
transactionHttp.announce(signedTransaction);


```

&nbsp;

## Check transaction

With REST API: 

\<REST API URL>/transaction/{transactionHash}

** If response is of status 200, this transaction is confirmed.

With SDK:
```js
// tsjs-xpx-chain-sdk

const sdk = require('tsjs-xpx-chain-sdk');

var API_URL = "https://arcturus.xpxsirius.io";
const transactionHttp = new sdk.TransactionHttp(API_URL);

// will throw error if not found
transactionHttp.getTransaction(<transactionHash>).subscribe((transaction)=>{

    // your code...
});
```

[Endpoint reference](https://bcdocs.xpxsirius.io/endpoints/#operation/getTransaction)

&nbsp;

## Check transaction status

With REST API: 

\<REST API URL>/transaction/{transactionHash}/status

[Endpoint reference](https://bcdocs.xpxsirius.io/endpoints/#operation/getTransactionStatus)

** Transaction with error will not be synced across different nodes (except AggregateBondedTransaction), so make sure to connect to the same API node to get the error message with the transaction hash


&nbsp;

## Create a new HD account for each user instead of using message.

HD - 
Hierarchical Deterministic

You can use your own logic to create master and child account.
Examples:

```
eg 1:
master
master.child1
master.child2
master.child.grandchild1
<string above> -> Crypto function -> 64 hex string as privateKey

eg 2:
<master privateKey> // auto generated
<master privateKey>+'1'-> Crypto function -> 64 hex string as privateKey
<master privateKey>+'2'-> Crypto function -> 64 hex string as privateKey
```

** If your encryption implementation does not have random character, user input or highly allow guessable keywords, it is best to keep encryption implementation as secret.


With SDK Example:
```js
// tsjs-xpx-chain-sdk

var sdk = require('tsjs-xpx-chain-sdk');
```

&nbsp;

## Check balance

With SDK:
```js
// tsjs-xpx-chain-sdk

const sdk = require('tsjs-xpx-chain-sdk');

var API_URL = "https://arcturus.xpxsirius.io";

const accountHttp = new sdk.AccountHttp(API_URL);
const namespaceHttp = new sdk.NamespaceHttp(API_URL);

const address = sdk.Address.createFromRawAddress(<raw_address>);

const xpxNamespace = new sdk.NamespaceId('prx.xpx');
let xpxAmount = 0;

namespaceHttp.getLinkedMosaicId(xpxNamespace).subscribe((xpxMosaicId)=>{

    accountHttp.getAccountInfo(address).subscribe((accInfo)=>{

        for(const mosaic of accInfo.mosaics){
            if(mosaic.id.toHex() === xpxMosaicId.toHex() ){
                xpxAmount = mosaic.amount.compact() / Math.pow(10, sdk.XpxMosaicProperties.MOSAIC_PROPERTIES.divisibility);
            }
        }
    });
});
```

&nbsp;

## Create Multisig Account

Please make sure the account that converting to multisig account have the sufficient amount, else the lock fund of 10 XPX will be forfeited.

With SDK:
```js
// tsjs-xpx-chain-sdk

const sdk = require('tsjs-xpx-chain-sdk');

var API_URL = "https://arcturus.xpxsirius.io";

const transactionHttp = new sdk.TransactionHttp(API_URL);

// please use the correct network type for different blockchain network
let network = sdk.NetworkType.MAIN_NET; 

// please get it from the API at block 1
// https://arcturus.xpxsirius.io/block/1
const generationHash = '10540AD3A1BF46B1A05D8B1CF0252BC9FB2E0B53CFD748262B0CE341CEAFEB6B'; 

const accountToConvert = sdk.Account.createFromPrivateKey(<privateKey>, network);
const cosigner1 = sdk.PublicAccount.createFromPublicKey(<publicKey>, network);
const cosigner2 = sdk.PublicAccount.createFromPublicKey(<publicKey>, network);

const convertIntoMultisigTransaction = sdk.ModifyMultisigAccountTransaction.create(
    sdk.Deadline.create(),
    2, // min approval
    2, // min removal
    [
        new sdk.MultisigCosignatoryModification(
            sdk.MultisigCosignatoryModificationType.Add,
            cosigner1
        ),
        new sdk.MultisigCosignatoryModification(
            sdk.MultisigCosignatoryModificationType.Add,
            cosigner2
        ),
    ],
    network
);

const aggregateTransaction = sdk.AggregateTransaction.createBonded(
    sdk.Deadline.create(),
    [
        convertIntoMultisigTransaction.toAggregate(accountToConvert.publicAccount)
    ],
    network
);

const signedAggregateBoundedTransaction = accountToConvert.sign(aggregateTransaction, generationHash);

const lockFundsTransaction = sdk.LockFundsTransaction.create(
    sdk.Deadline.create(),
    sdk.NetworkCurrencyMosaic.createRelative(10), // locking 10 XPX
    sdk.UInt64.fromUint(1000), // duration - in block
    signedAggregateBoundedTransaction,
    network);

const lockFundsTransactionSigned = accountToConvert.sign(lockFundsTransaction, generationHash);

// the steps is announce lockFundsTransactionSigned and wait for confirmation
// then announce the signedAggregateBoundedTransaction
/*
transactionHttp.announce(lockFundsTransactionSigned)

// wait for lockFundsTransactionSigned hash confirmation, should confirm within 15 seconds 

transactionHttp.announceAggregateBonded(signedAggregateBoundedTransaction)

================================================================
Here I will use listener to get confirmation
*/

(async ()=>{

    try {
        const confirmedTx = await announceLockfundAndWaitForConfirmation(API_URL, accountToConvert.address, lockFundsTransactionSigned, lockFundsTransactionSigned.hash);

        var aggregateTx = await announceAggregateBonded(API_URL, accountToConvert.address, signedAggregateBoundedTransaction, signedAggregateBoundedTransaction.hash )

        console.log("Done");

    } catch (error) {
        console.log(error);
    }
    
})();

async function announceLockfundAndWaitForConfirmation(nodeUrl, senderAddress, signedLockFundsTransaction, lockHash){
    return new Promise((resolve, reject) => {
      const listener = new sdk.Listener(nodeUrl);
      listener.open().then(() => {
        const txStatus = listener.status(senderAddress).subscribe(txStatusError => {
          if (txStatusError.hash === lockHash) {
            console.error('Lockfund status (' + lockHash + ') error: ' + txStatusError.status);
            if (txConfirmed) {
              txConfirmed.unsubscribe();
            }
            if (txStatus) {
              txStatus.unsubscribe();
            }
            listener.close();
            reject(txStatusError.status);
          }
        });

        const txConfirmed = listener.confirmed(senderAddress).subscribe(confirmedTx => {
          if (confirmedTx && confirmedTx.transactionInfo && confirmedTx.transactionInfo.hash === lockHash) {
            console.log('Lockfund (' + lockHash + ') confirmed at height: ' + confirmedTx.transactionInfo.height.compact());
            if (txConfirmed) {
              txConfirmed.unsubscribe();
            }
            if (txStatus) {
              txStatus.unsubscribe();
            }
            listener.close();
            resolve(confirmedTx);
          }
        }, error => {
          console.error('Lockfund confirmation (' + lockHash + ') subscription failed: ' + error);
          if (txConfirmed) {
            txConfirmed.unsubscribe();
          }
          if (txStatus) {
            txStatus.unsubscribe();
          }
          listener.close();
          reject(error);
        });

        transactionHttp.announce(signedLockFundsTransaction).subscribe(
            (message)=>{
                console.log('Lockfund transaction (' + lockHash + ') announced');
            }, 
            (error)=>{
                console.log(error);
                if (txConfirmed) {
                    txConfirmed.unsubscribe();
                }
                if (txStatus) {
                    txStatus.unsubscribe();
                }
                reject(error);
            }
        );

      }).catch(reason => {
        console.error('Lockfund transaction (' + lockHash + ') listener exception caught: ' + reason);
        listener.terminate();
        reject(reason);
      });
    });
  }

async function announceAggregateBonded(nodeUrl, senderAddress, aggBondTx, aggBondHash){
    return new Promise((resolve, reject) => {
      const listener = new sdk.Listener(nodeUrl);
      listener.open().then(() => {
        const txStatus = listener.status(senderAddress).subscribe(txStatusError => {
          if (txStatusError.hash === aggBondHash) {
            console.error('Aggregate status (' + aggBondHash + ') error: ' + txStatusError.status);
            if (partialAdded) {
              partialAdded.unsubscribe();
            }
            if (txStatus) {
              txStatus.unsubscribe();
            }
            listener.close();
            reject(txStatusError.status);
          }
        });

        const partialAdded = listener.aggregateBondedAdded(senderAddress).subscribe(aggTransaction => {
          if (aggTransaction && aggTransaction.transactionInfo && aggTransaction.transactionInfo.hash === aggBondHash) {
            console.log('Aggregate bonded transaction (' + aggBondHash + ') added');
            if (partialAdded) {
              partialAdded.unsubscribe();
            }
            if (txStatus) {
              txStatus.unsubscribe();
            }
            listener.close();
            resolve(aggTransaction);
          }
        }, error => {
          console.error('Aggregate bonded transaction (' + aggBondHash + ') subscription failed: ' + error);
          if (partialAdded) {
            partialAdded.unsubscribe();
          }
          if (txStatus) {
            txStatus.unsubscribe();
          }
          listener.close();
          reject(error);
        });

        transactionHttp.announceAggregateBonded(aggBondTx).subscribe(
            (message)=>{
                console.log('Aggregate transaction (' + aggBondHash + ') announced');
            }, 
            (error)=>{
                console.log(error);
                if (txConfirmed) {
                    txConfirmed.unsubscribe();
                }
                if (txStatus) {
                    txStatus.unsubscribe();
                }
                reject(error);
            }
        );

      }).catch(reason => {
        console.error('Aggregate bonded transaction (' + aggBondHash + ') listener exception caught: ' + reason);
        listener.terminate();
        reject(reason);
      });
    });
  }

```

&nbsp;

## Cosign Multisig Transaction

With SDK:
```js
// tsjs-xpx-chain-sdk

const sdk = require('tsjs-xpx-chain-sdk');

var API_URL = "https://arcturus.xpxsirius.io";

const transactionHttp = new sdk.TransactionHttp(API_URL);
const accountHttp = new sdk.AccountHttp(API_URL);

// please use the correct network type for different blockchain network
let network = sdk.NetworkType.MAIN_NET;  

const account = sdk.Account.createFromPrivateKey(<privateKey>, network);

const cosignAggregateBondedTransaction = (signedAggregateBoundedTransaction, account) => {
    const cosignatureTransaction = sdk.CosignatureTransaction.create(signedAggregateBoundedTransaction);
    return account.signCosignatureTransaction(cosignatureTransaction);
};

accountHttp.aggregateBondedTransactions(account.publicAccount).subscribe((transactions)=>{

        for(const aggregateTx of transactions){

			// view transaction details...
            let cosigned = aggregateTx.signedByAccount(account.publicAccount)

            if(!cosigned && aggregateTx.transactionInfo.hash === <transactionHash>){
                let cosignatureSignedTransaction = cosignAggregateBondedTransaction(aggregateTx, account);

                transactionHttp.announceAggregateBondedCosignature(cosignatureSignedTransaction);
            }
        }
    });
```

&nbsp;

## Modify Multisig Account

With SDK:
```js
// tsjs-xpx-chain-sdk

// basically the steps is the same as create multisig account, just some little changes

const convertedMultisigAccount = sdk.PublicAccount.createFromPublicKey(<publicKey>, network);

const oneOfCosigner = sdk.Account.createFromPrivateKey(<privateKey>, network);

const modifyMultisigTransaction = sdk.ModifyMultisigAccountTransaction.create(
    sdk.Deadline.create(),
    -1, // relative value of current min approval
	0, // relative value of current min removal
    [
		// add or Remove, leave empty to maintain current cosigner
		/*
		new sdk.MultisigCosignatoryModification(
            sdk.MultisigCosignatoryModificationType.Remove,
            cosigner1
        )
		*/
    ],
    network
);

const aggregateTransaction = sdk.AggregateTransaction.createBonded(
    sdk.Deadline.create(),
    [   
		modifyMultisigTransaction.toAggregate(convertedMultisigAccount)
    ],
    network
);

const signedAggregateBoundedTransaction = oneOfCosigner.sign(aggregateTransaction, generationHash);

...

// whoever signed will be paying the lock fund
const lockFundsTransactionSigned = oneOfCosigner.sign(lockFundsTransaction, generationHash);

...

// remember to update which address to track for the announcing LockFundTransaction and AggregateBondedTransaction

const confirmedTx = await announceLockfundAndWaitForConfirmation(API_URL, oneOfCosigner.address, lockFundsTransactionSigned, lockFundsTransactionSigned.hash);

var aggregateTx = await announceAggregateBonded(API_URL, oneOfCosigner.address, signedAggregateBoundedTransaction, signedAggregateBoundedTransaction.hash )

// refer the code above #Create Multisig Account

```

&nbsp;

## Sending transaction with multisig account

Likewise, it use the lockfundTransaction and aggregateBondedTransaction. All transaction which outgoing from the multisig account will need to be included in aggregateBondedTransaction.

With SDK:
```js
// tsjs-xpx-chain-sdk

...

const aggregateTransaction = sdk.AggregateTransaction.createBonded(
    sdk.Deadline.create(),
    [   
		anyTransactionToSignByMultisigAcc.toAggregate(convertedMultisigAccount)
    ],
    network
);

const signedAggregateBoundedTransaction = oneOfCosigner.sign(aggregateTransaction, generationHash);

...

// whoever signed will be paying the lock fund
const lockFundsTransactionSigned = oneOfCosigner.sign(lockFundsTransaction, generationHash);

...

// announce lockfund and aggregateBond transaction, refer the code above #Create Multisig Account

```

&nbsp;

## View Multisig

With REST API: 

\<REST API URL>/account/{publicKey | address}/multisig

Responses:
- cosignatories - Cosigners of this account
- multisigAccounts - Multisig accounts that current account are part of 
- minApproval - Minimum cosigner to approve outgoing transaction
- minRemoval - Minimum cosigner to approve the removal of cosigner

[View Multisig Reference](https://bcdocs.xpxsirius.io/endpoints/#operation/getAccountMultisig)

&nbsp;

## View Multisig

With REST API: 

\<REST API URL>/account/{publicKey | address}/multisig/graph

[View Multisig Graph Reference](https://bcdocs.xpxsirius.io/endpoints/#operation/getAccountMultisigGraph)


## API Endpoints

Can connect with https or http.

https  - 
example: https://arcturus.xpxsirius.io

http  - 
example: http://arcturus.xpxsirius.io:3000

### List of Mainnet API

- arcturus.xpxsirius.io
- aldebaran.xpxsirius.io
- betelgeuse.xpxsirius.io
- bigcalvin.xpxsirius.io
- delphinus.xpxsirius.io
- lyrasithara.xpxsirius.io

### List of Testnet API

- bctestnet1.brimstone.xpxsirius.io
- bctestnet2.brimstone.xpxsirius.io

## References

[List of Sdks](https://bcdocs.xpxsirius.io/docs/sdks/languages/) 
- Please go to selected SDK github repo wiki to have more understanding of the SDK of specific language
- Please Use IDE along with the SDK to ease the development

