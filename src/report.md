## Cryptocurrencies WS2023 - Task 6a - Payment Channels

Author: Rúben Samuel Alves Santos

# Exercise 1

solution: OP_0 sigAlice sigBob

To understand why this solution works we need to understand the locking script of the given input:
    
    OP_2 <Alice's Public Key> <Bob's Public Key> OP_2 OP_CHECKMULTISIG

    or the original script: 
    [   
        OP_2,
        ALICE_PUBKEY (hex),
        BOB_PUBKEY (hex),
        OP_2,
        OP_CHECKMULTISIG
    ]

This locking script requires two signatures from Alice and Bob, this is represented by the first number two and the two public keys of Alice and Bob. The OP_CHECKMULTISIG checks if the signatures are valid.

The solution starts with a OP_0 value that serves as a dummy value for the OP_CHECKMULTISIG bug (which pops one extra element from the stack). We then add the signatures for Alice and Bob.

The transaction had two outputs (1000 satoshis to Alice and 2000 satoshis to Bob) and each output was secured using a Pay-to-PubKey-Hash (P2PKH) script. The remainer 1500 satoshis were given to the miner as a fee.

Bonus Question: As described, this final state differs from a regular state, as it has no revocation mechanism, timelock and is not duplicated. Why do we need these things in a regular state and how does the duplication work?

In regular states of the Lightning Network's payment channels, the incorporation of revocation mechanisms ensures that once a new state is agreed upon, the previous state becomes invalid.
Timelocks provide safety by delaying the finalization of broadcasted states on the blockchain. This delay allows the parties to contest the transaction in case an older, invalidated state is used.
State duplication, where each party holds a version of the channel's state, allows any participant to unilaterally enforce the state on the blockchain if needed.
These mechanisms are not necessary in the final state of a cooperative channel, as there's a mutual agreement on the funds' distribution, eliminating the need for additional security against unilateral actions or disputes.

# Exercise 2

solution: sig_carol, id_carol.pk.to_hex(), PUNISH_SECRET, 'OP_TRUE'

This solution solves the locking script of exercise 2. Here is the locking script:

    [   
    'OP_IF', 
    'OP_SHA256', PUNISH_HASH, 'OP_EQUALVERIFY', 'OP_DUP', 'OP_HASH160', CAROL.get_pk_hash(), 
    'OP_ELSE', 
    PUNISH_TIME, 'OP_CHECKSEQUENCEVERIFY', 'OP_DROP', 'OP_DUP', 'OP_HASH160', MALLORY_PK_HASH,
    'OP_ENDIF',
    'OP_EQUALVERIFY',
    'OP_CHECKSIG'
    ]

The value of OP_TRUE is meant to trigger the first part of the if statement. This first part will calculate the sha256 of the PUNISH_SECRET and then compare it to the PUNISH_HASH with OP_EQUALVERIFY. Since this is true, then the hexadecimal representation of Carol's public key is duplicated with OP_DUP and then hashed with OP_HASH160. The hash of Carol's public key is also in stack and when the if statement exits, both these values are compared with OP_EQUALVERIFY. This is true, so the script continues to the next step. Finally, the signature of Carol is checked with OP_CHECKSIG.

Bonus question: To have multi-hop payments in a Payment Channel Network (PCN), payment channels can be used to route HTLC-based (Hash Time-Lock Contract) payments. These HTLCs are additional outputs in a channel, that need to be punished. Following the example above, assume that Mallory has posted an old state that holds one (or more) HTLCs. Now Carol needs to punish the output holding Mallory’s balance plus each output holding an HTLC. How can she make this punishment more efficient? Is there a way to decrease the amount of things you need to put on-chain?

Carol can make this punishment more efficient by using the HTLCs' preimages to punish the outputs. This way, she only needs to put the preimages on-chain, instead of the HTLCs themselves. This is possible because the HTLCs are secured by the preimages, so if Carol has the preimages, she can claim the HTLCs' outputs.

In summary, for every HTLC, there's a preimage (a secret value) whose hash was used to lock the funds. When the recipient of an HTLC reveals the preimage, they can claim the funds. If Carol has the preimages of these HTLCs (which she should have if the payments were settled correctly), she can use them to claim the HTLCs on-chain efficiently. Instead of creating separate transactions for each HTLC and the main channel balance, Carol includes the preimages in a single transaction. This approach significantly reduces the number of transactions Carol has to make on the blockchain.

Note: In cryptography, a preimage is the original input that was used to generate a specific hash output.

# Exercise 3
TODO: Include a copy of your script here
TODO: Explain why your solution works

