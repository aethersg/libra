#Testing out facebook's Libra

#### Cloning the git repository
```
git clone https://github.com/libra/libra.git
```

This should create a folder called libra. 

# Setting up Libra core
```
 ./libra/scripts/dev_setup.sh
 
 Welcome to Libra!
 
 This script will download and install the necessary dependencies needed to
 build Libra Core. This includes:
         * Rust (and the necessary components, e.g. rust-fmt, clippy)
         * CMake, protobuf, go (for building protobuf)
 
 If you'd prefer to install these dependencies yourself, please exit this script
 now with Ctrl-C.
 
 Proceed with installing necessary dependencies? (y/N)
 
 it will update the neccessary information of which you will see this when it is finished
 -----------------------------------------------------------------------------------------
 
 Finished installing all dependencies.
 
 You should now be able to build the project by running:
         source /Users/tanj1/.cargo/env
         cargo build

```
# Building the code and connecting to the test-net.

```
It is neccessary to go into the folder to run the script as it will look for Cargo.toml file.

cd libra
./scripts/cli/start_cli_testnet.sh

```
Please note you will need port 8000 to be open for this to work. 
you should see the following when it is successful.
```
Building and running client in debug mode.
    Finished dev [unoptimized + debuginfo] target(s) in 20.44s
     Running `target/debug/client --host ac.testnet.libra.org --port 8000 -s ./scripts/cli/trusted_peers.config.toml`
Connected to validator at: ac.testnet.libra.org:8000
usage: <command> <args>

Use the following commands:

account | a 
        Account operations
query | q 
        Query operations
transfer | transferb | t | tb 
        <sender_account_address>|<sender_account_ref_id> <receiver_account_address>|<receiver_account_ref_id> <number_of_coins> [gas_unit_price_in_micro_libras (default=0)] [max_gas_amount_in_micro_libras (default 10000)] Suffix 'b' is for blocking. 
        Transfer coins (in libra) from account to another.
help | h 
        Prints this help
quit | q! 
        Exit this client


Please, input commands: 

libra% 

```

#Functionality of what libra can do.
1. Account 
2. query
3. Transfer

### Account operations
```
create | c 
        Create an account. Returns reference ID to use in other operations
list | la 
        Print all accounts that were created or loaded
recover | r <file_path>
        Recover Libra wallet from the file path
write | w <file_path>
        Save Libra wallet mnemonic recovery seed to disk
mint | mintb | m | mb <receiver_account_ref_id>|<receiver_account_address> <number_of_coins>
        Mint coins to the account. Suffix 'b' is for blocking

```

### Query operations

```

balance | b <account_ref_id>|<account_address>
        Get the current balance of an account
sequence | s <account_ref_id>|<account_address> [reset_sequence_number=true|false]
        Get the current sequence number for an account, and reset current sequence number in CLI (optional, default is false)
account_state | as <account_ref_id>|<account_address>
        Get the latest state for an account
txn_acc_seq | ts <account_ref_id>|<account_address> <sequence_number> <fetch_events=true|false>
        Get the committed transaction by account and sequence number.  Optionally also fetch events emitted by this transaction.
txn_range | tr <start_version> <limit> <fetch_events=true|false>
        Get the committed transactions by version range. Optionally also fetch events emitted by these transactions.
event | ev <account_ref_id>|<account_address> <sent|received> <start_sequence_number> <ascending=true|false> <limit>
        Get events by account and event type (sent|received).

```

###Transfer operations

```
transfer | transferb | t | tb 
        <sender_account_address>|<sender_account_ref_id> <receiver_account_address>|<receiver_account_ref_id> <number_of_coins> [gas_unit_price_in_micro_libras (default=0)] [max_gas_amount_in_micro_libras (default 10000)] Suffix 'b' is for blocking. 
        Transfer coins (in libra) from account to another.
```

#Examples 
###Creating wallets 
Lets create 2 wallets
using the following command
<br>create | c 
<br>Create an account. Returns reference ID to use in other operations
```
libra% a create
>> Creating/retrieving next account from wallet
Created/retrieved account #0 address c22c69c7bc922a2c406d3669aa9cbb841f0125a739f3f83dd2db83f3a9563ead
libra% a create
>> Creating/retrieving next account from wallet
Created/retrieved account #1 address e96787f2bb9572a34fd351b34a1bdf03c89a8fe9ab31173ebaa2df40cb2eb0bd
```

Now we have :  
<p> account 0 
<br> address : c22c69c7bc922a2c406d3669aa9cbb841f0125a739f3f83dd2db83f3a9563ead

<p> account 1 
<br> address : e96787f2bb9572a34fd351b34a1bdf03c89a8fe9ab31173ebaa2df40cb2eb0bd


Please note: these two accounts are not in the blockchain yet as there is no money ( value ) in them
This process can be done via mining or transferring money into them. 

###Mining some coins
Now lets mint some coins for account 0. 
<br> Using the following command.
<br> mint | mintb | m | mb <receiver_account_ref_id>|<receiver_account_address> <number_of_coins>
<br> Mint coins to the account. Suffix 'b' is for blocking
```
libra% a m 0 100
>> Minting coins
Mint request submitted

```
What this command does is 
<br> a == account
<br> m == mint
<br> 0 == the receiver_account_ref_id of the first account which is '0'
<br> 100 ==  the number of coins we will wish to mint

Note: There is throttling in the test-net do wait a while about a minute or so and you can re-mint.
```
libra% a m 1 50
>> Minting coins
[ERROR] Error minting coins: Failed to query remote faucet server[status=429 Too Many Requests]: "<html><body><h1>429 Too Many Requests</h1>\nYou have sent too many requests in a given amount of time.\n</body></html>\n"

```

###Querying the wallets:
```
libra% q b 0
Balance is: 100

libra% q b 1
Balance is: 50
```

###Transferring money:
For this lets transfer 50 coins from account 0 to account 1 so the values will be reversed.
```
libra% t 0 1 50
>> Transferring
Transaction submitted to validator
To query for transaction status, run: query txn_acc_seq 0 0 <fetch_events=true|false>
``` 
After which lets check the transaction status.
```
libra% q txn_acc_seq 0 0 true
>> Getting committed transaction by account and sequence number
Committed transaction: SignedTransaction { 
 raw_txn: RawTransaction { 
        sender: c22c69c7bc922a2c406d3669aa9cbb841f0125a739f3f83dd2db83f3a9563ead, 
        sequence_number: 0, 
        payload: {, 
                transaction: peer_to_peer_transaction, 
                args: [ 
                        {ADDRESS: e96787f2bb9572a34fd351b34a1bdf03c89a8fe9ab31173ebaa2df40cb2eb0bd},
                        {U64: 50000000}, 
                ]
        }, 
        max_gas_amount: 10000, 
        gas_unit_price: 0, 
        expiration_time: 1561101267s, 
}, 
 public_key: 370e58e749ce8e98cbb4c08678d19ae02ea66e612ae0faa3c656b40f05a6316d, 
 signature: Signature( R: CompressedEdwardsY: [97, 56, 102, 165, 34, 250, 23, 97, 168, 208, 73, 27, 107, 170, 218, 88, 99, 23, 14, 162, 150, 198, 151, 20, 20, 29, 137, 226, 67, 162, 18, 204], s: Scalar{
        bytes: [207, 202, 184, 175, 151, 193, 113, 160, 215, 135, 176, 26, 150, 219, 179, 186, 127, 152, 207, 161, 162, 41, 229, 179, 196, 22, 139, 103, 89, 233, 64, 1],
} ), 
 }
Events: 
ContractEvent { access_path: AccessPath { address: c22c69c7bc922a2c406d3669aa9cbb841f0125a739f3f83dd2db83f3a9563ead, type: Resource, hash: "217da6c6b3e19f1825cfb2676daecce3bf3de03cf26647c78df00b371b25cc97", suffix: "/sent_events_count/" } , index: 0, event_data: AccountEvent { account: e96787f2bb9572a34fd351b34a1bdf03c89a8fe9ab31173ebaa2df40cb2eb0bd, amount: 50000000 } }
ContractEvent { access_path: AccessPath { address: e96787f2bb9572a34fd351b34a1bdf03c89a8fe9ab31173ebaa2df40cb2eb0bd, type: Resource, hash: "217da6c6b3e19f1825cfb2676daecce3bf3de03cf26647c78df00b371b25cc97", suffix: "/received_events_count/" } , index: 0, event_data: AccountEvent { account: c22c69c7bc922a2c406d3669aa9cbb841f0125a739f3f83dd2db83f3a9563ead, amount: 50000000 } }

``` 

As the transaction is successful again we will check the query:
```
libra% q b 0
Balance is: 50

libra% q b 1
Balance is: 100
```
From here we can see the transaction has gone through.

Now lets try sending more transactions and see what happens.
```
libra% t 0 1 10
>> Transferring
Transaction submitted to validator
To query for transaction status, run: query txn_acc_seq 0 1 <fetch_events=true|false>

libra% t 0 1 10
>> Transferring
Transaction submitted to validator
To query for transaction status, run: query txn_acc_seq 0 2 <fetch_events=true|false>

libra% t 0 1 10
>> Transferring
Transaction submitted to validator
To query for transaction status, run: query txn_acc_seq 0 3 <fetch_events=true|false>

```
and again checking the balance
```
libra% q b 0 
Balance is: 20

libra% q b 1
Balance is: 130
```

now lets check the sequence of both accounts since we have sent from only account 0 
<br> and for account 0 we have done no transactions.
```
libra% q s 0
>> Getting current sequence number
Sequence number is: 4

libra% q s 1
>> Getting current sequence number
Sequence number is: 0
```
We can see the current sequence is 4 for account 0 and for account 1 is 0. 
<br>this is due to the fact that account 0 has been the only one sending transactions.
 
 ### Testing edge cases. 
 Now what will happen if we send more then what the wallet has ? 
 <br>First check the balances.
 ```
libra% q b 0
Balance is: 20

libra% q b 1
Balance is: 130
```
Test case 1: transfer 50 dollars from account 0 to account 1 :
```
libra% t 0 1 50
>> Transferring
Transaction submitted to validator
To query for transaction status, run: query txn_acc_seq 0 4 <fetch_events=true|false>
```
Went to the transaction.

```
libra% query txn_acc_seq 0 4 true
>> Getting committed transaction by account and sequence number
Committed transaction: SignedTransaction { 
 raw_txn: RawTransaction { 
        sender: c22c69c7bc922a2c406d3669aa9cbb841f0125a739f3f83dd2db83f3a9563ead, 
        sequence_number: 4, 
        payload: {, 
                transaction: peer_to_peer_transaction, 
                args: [ 
                        {ADDRESS: e96787f2bb9572a34fd351b34a1bdf03c89a8fe9ab31173ebaa2df40cb2eb0bd},
                        {U64: 50000000}, 
                ]
        }, 
        max_gas_amount: 10000, 
        gas_unit_price: 0, 
        expiration_time: 1561101913s, 
}, 
 public_key: 370e58e749ce8e98cbb4c08678d19ae02ea66e612ae0faa3c656b40f05a6316d, 
 signature: Signature( R: CompressedEdwardsY: [119, 107, 197, 72, 50, 108, 112, 231, 22, 124, 33, 205, 13, 12, 219, 84, 41, 199, 99, 212, 222, 20, 157, 128, 181, 185, 81, 26, 85, 168, 195, 187], s: Scalar{
        bytes: [130, 178, 98, 152, 169, 38, 144, 148, 118, 32, 26, 59, 95, 211, 220, 122, 241, 36, 80, 113, 148, 23, 10, 59, 114, 142, 111, 100, 201, 247, 111, 5],
} ), 
 }
Events: 

```
Do note there is no events. thought the status is for the query is true.
it can assumed that the transaction did not go through.
now lets try to transfer 1.50 to account 1 to see how robust the values it can take.
```
libra% t 0 1 1.5
>> Transferring
Transaction submitted to validator
To query for transaction status, run: query txn_acc_seq 0 5 <fetch_events=true|false>

```
it was submitted now lets check the txn.
```
libra% query txn_acc_seq 0 5 true
>> Getting committed transaction by account and sequence number
Committed transaction: SignedTransaction { 
 raw_txn: RawTransaction { 
        sender: c22c69c7bc922a2c406d3669aa9cbb841f0125a739f3f83dd2db83f3a9563ead, 
        sequence_number: 5, 
        payload: {, 
                transaction: peer_to_peer_transaction, 
                args: [ 
                        {ADDRESS: e96787f2bb9572a34fd351b34a1bdf03c89a8fe9ab31173ebaa2df40cb2eb0bd},
                        {U64: 1500000}, 
                ]
        }, 
        max_gas_amount: 10000, 
        gas_unit_price: 0, 
        expiration_time: 1561102521s, 
}, 
 public_key: 370e58e749ce8e98cbb4c08678d19ae02ea66e612ae0faa3c656b40f05a6316d, 
 signature: Signature( R: CompressedEdwardsY: [110, 0, 140, 130, 112, 140, 216, 163, 8, 231, 214, 17, 92, 233, 233, 145, 235, 22, 138, 54, 90, 78, 223, 255, 156, 178, 136, 223, 217, 18, 214, 96], s: Scalar{
        bytes: [29, 217, 112, 225, 31, 85, 206, 93, 62, 64, 94, 80, 72, 10, 167, 24, 1, 30, 214, 81, 160, 10, 216, 58, 171, 107, 237, 51, 81, 163, 70, 8],
} ), 
 }
Events: 
ContractEvent { access_path: AccessPath { address: c22c69c7bc922a2c406d3669aa9cbb841f0125a739f3f83dd2db83f3a9563ead, type: Resource, hash: "217da6c6b3e19f1825cfb2676daecce3bf3de03cf26647c78df00b371b25cc97", suffix: "/sent_events_count/" } , index: 4, event_data: AccountEvent { account: e96787f2bb9572a34fd351b34a1bdf03c89a8fe9ab31173ebaa2df40cb2eb0bd, amount: 1500000 } }
ContractEvent { access_path: AccessPath { address: e96787f2bb9572a34fd351b34a1bdf03c89a8fe9ab31173ebaa2df40cb2eb0bd, type: Resource, hash: "217da6c6b3e19f1825cfb2676daecce3bf3de03cf26647c78df00b371b25cc97", suffix: "/received_events_count/" } , index: 4, event_data: AccountEvent { account: c22c69c7bc922a2c406d3669aa9cbb841f0125a739f3f83dd2db83f3a9563ead, amount: 1500000 } }

```
From the response we can see that it went through lets check the balances again.
```
libra% q b 0
Balance is: 18.5

libra% q b 1
Balance is: 131.5
```
It works :D

now lets try a smaller number to be transferred. lets do 0.0001 thou i would reckon it will not be an issue
```
libra% t 0 1 0.0001
>> Transferring
Transaction submitted to validator
To query for transaction status, run: query txn_acc_seq 0 6 <fetch_events=true|false>

libra% query txn_acc_seq 0 6 true
>> Getting committed transaction by account and sequence number
Committed transaction: SignedTransaction { 
 raw_txn: RawTransaction { 
        sender: c22c69c7bc922a2c406d3669aa9cbb841f0125a739f3f83dd2db83f3a9563ead, 
        sequence_number: 6, 
        payload: {, 
                transaction: peer_to_peer_transaction, 
                args: [ 
                        {ADDRESS: e96787f2bb9572a34fd351b34a1bdf03c89a8fe9ab31173ebaa2df40cb2eb0bd},
                        {U64: 100}, 
                ]
        }, 
        max_gas_amount: 10000, 
        gas_unit_price: 0, 
        expiration_time: 1561102733s, 
}, 
 public_key: 370e58e749ce8e98cbb4c08678d19ae02ea66e612ae0faa3c656b40f05a6316d, 
 signature: Signature( R: CompressedEdwardsY: [131, 187, 194, 29, 59, 75, 169, 219, 17, 139, 243, 72, 136, 185, 84, 92, 4, 18, 222, 55, 183, 40, 191, 129, 145, 71, 154, 155, 23, 26, 96, 91], s: Scalar{
        bytes: [38, 71, 89, 71, 248, 69, 155, 10, 139, 90, 81, 74, 223, 47, 184, 127, 213, 130, 93, 117, 201, 217, 21, 128, 105, 59, 11, 199, 60, 131, 169, 6],
} ), 
 }
Events: 
ContractEvent { access_path: AccessPath { address: c22c69c7bc922a2c406d3669aa9cbb841f0125a739f3f83dd2db83f3a9563ead, type: Resource, hash: "217da6c6b3e19f1825cfb2676daecce3bf3de03cf26647c78df00b371b25cc97", suffix: "/sent_events_count/" } , index: 5, event_data: AccountEvent { account: e96787f2bb9572a34fd351b34a1bdf03c89a8fe9ab31173ebaa2df40cb2eb0bd, amount: 100 } }
ContractEvent { access_path: AccessPath { address: e96787f2bb9572a34fd351b34a1bdf03c89a8fe9ab31173ebaa2df40cb2eb0bd, type: Resource, hash: "217da6c6b3e19f1825cfb2676daecce3bf3de03cf26647c78df00b371b25cc97", suffix: "/received_events_count/" } , index: 5, event_data: AccountEvent { account: c22c69c7bc922a2c406d3669aa9cbb841f0125a739f3f83dd2db83f3a9563ead, amount: 100 } }

libra% q b 0
Balance is: 18.4999
libra% q b 1
Balance is: 131.5001

```

Now lets try again to send 18.5 when we have 18.4999 to account 1. 
```
libra% t 0 1 18.5
>> Transferring
Transaction submitted to validator
To query for transaction status, run: query txn_acc_seq 0 7 <fetch_events=true|false>
libra% query txn_acc_seq 0 7 true
>> Getting committed transaction by account and sequence number
Committed transaction: SignedTransaction { 
 raw_txn: RawTransaction { 
        sender: c22c69c7bc922a2c406d3669aa9cbb841f0125a739f3f83dd2db83f3a9563ead, 
        sequence_number: 7, 
        payload: {, 
                transaction: peer_to_peer_transaction, 
                args: [ 
                        {ADDRESS: e96787f2bb9572a34fd351b34a1bdf03c89a8fe9ab31173ebaa2df40cb2eb0bd},
                        {U64: 18500000}, 
                ]
        }, 
        max_gas_amount: 10000, 
        gas_unit_price: 0, 
        expiration_time: 1561102814s, 
}, 
 public_key: 370e58e749ce8e98cbb4c08678d19ae02ea66e612ae0faa3c656b40f05a6316d, 
 signature: Signature( R: CompressedEdwardsY: [236, 46, 252, 216, 229, 235, 81, 50, 164, 151, 18, 46, 253, 104, 49, 222, 140, 126, 148, 58, 209, 116, 247, 194, 209, 194, 243, 57, 125, 42, 159, 181], s: Scalar{
        bytes: [108, 24, 14, 193, 184, 201, 62, 86, 61, 183, 185, 87, 76, 193, 70, 168, 214, 146, 237, 218, 236, 53, 101, 136, 247, 242, 159, 176, 213, 84, 1, 7],
} ), 
 }
Events: 

```
