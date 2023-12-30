## Cryptocurrencies WS2023 - Task 6a - Payment Channels

Author: Rúben Samuel Alves Santos

# Exercise 1

solution: OP_0 sigAlice sigBob

To understand why this solution works we need to understand the looking script of the given input:
    
    OP_2 <Alice's Public Key> <Bob's Public Key> OP_2 OP_CHECKMULTISIG

    or the original script: OP_PUSHNUM_2 OP_PUSHBYTES_33 03f1416f7a83b2577e684654af5281566e5c70f9538d71b5a4a8ad39711f364e17 OP_PUSHBYTES_33 032c4681910b1ffbfbd1c524ae3cd1e974008415609958018e7ba1f319d657ed35 OP_PUSHNUM_2 OP_CHECKMULTISIG

This looking script requires two signatures from Alice and Bob. At first we add number 2 to the stack, that represents the number of signatures required. Then we add Alice's public key and Bob's public key to the stack. Finally we add number 2 to the stack, that represents the number of public keys required. Then we use the OP_CHECKMULTISIG to check if the signatures are valid.

The solution starts with a OP_0 value that serves as a dummy value for the OP_CHECKMULTISIG bug (which pops one extra element from the stack). We then add the signatures for Alice and Bob.

The transaction had two outputs (1000 satoshis to Alice and 2000 satoshis to Bob) and each output was secured using a Pay-to-PubKey-Hash (P2PKH) script. The remainer 1500 satoshis were given to the miner as a fee.

Bonus Question: As described, this final state differs from a regular state, as it has no revocation mechanism, timelock and is not duplicated. Why do we need these things in a regular state and how does the duplication work?

In regular states of the Lightning Network's payment channels, the incorporation of revocation mechanisms ensures that once a new state is agreed upon, the previous state becomes invalid.
Timelocks provide safety by delaying the finalization of broadcasted states on the blockchain. This delay allows the parties to contest the transaction in case an older, invalidated state is used.
State duplication, where each party holds a version of the channel's state, allows any participant to unilaterally enforce the state on the blockchain if needed.
These mechanisms are not necessary in the final state of a cooperative channel, as there's a mutual agreement on the funds' distribution, eliminating the need for additional security against unilateral actions or disputes.

# Exercise 2
TODO: Include a copy of your script here
TODO: Explain why your solution works

# Exercise 3
TODO: Include a copy of your script here
TODO: Explain why your solution works

