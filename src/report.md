## Cryptocurrencies WS2023 - Task 6a - Payment Channels

Author: Rúben Samuel Alves Santos

# Exercise 1

solution: OP_0 sigAlice sigBob

or with the real values

OP_0 304402203471a575b33f09eceda586c854a07c3ad55289fbfdaf9d01d973bd9f99687d8502205c7b77477a91b0d3f641cfd242ea9e40717f5093b00b68147fffcdfc8790eb8701 3044022013c4d2dc67fc1ef724b9720b54c7ba14b932a737407709c5a0a260f98e41d123022015093e4b8080f65aa837487138685a69b3b30c7907ab27c65d9a54c3bcfcc52a01

To understand why this solution works we need to understand the locking script of the given input:
    
    OP_2 <Alice's Public Key> <Bob's Public Key> OP_2 OP_CHECKMULTISIG

This locking script requires two signatures from Alice and Bob, this is represented by the first number two and the two public keys of Alice and Bob. The OP_CHECKMULTISIG checks if the signatures are valid.

The solution starts with a OP_0 value that serves as a dummy value for the OP_CHECKMULTISIG bug (which pops one extra element from the stack). We then add the signatures for Alice and Bob.

The transaction had two outputs (1000 satoshis to Alice and 2000 satoshis to Bob) and each output was secured using a Pay-to-PubKey-Hash (P2PKH) script. The remainer 1500 satoshis were given to the miner as a fee.

# Exercise 2

solution: sig_carol, id_carol.pk.to_hex(), PUNISH_SECRET, 'OP_TRUE'

or with the real values:

30440220446cd2be21d6312affd51a00f660045fdd05a4e44ddb961bb42a2c9a14a8b9260220153180e388c9b670f4eb61911c92dd3139f95b46e0826d372c38e85c913bb0fc01 0234814cda6823160af45d502e0428e9b5207d306a8bedb7ccaccc729d63e24ee4 b90ec7c29234f4a65f321232f4de7096a5e9551830c5f6b96a89bdf382131117 OP_TRUE

The value of OP_TRUE is meant to trigger the first part of the if statement. This first part will calculate the sha256 of the PUNISH_SECRET and then compare it to the PUNISH_HASH with OP_EQUALVERIFY. Since this is true, then the hexadecimal representation of Carol's public key is duplicated with OP_DUP and then hashed with OP_HASH160. The hash of Carol's public key is also in stack and when the if statement exits, both these values are compared with OP_EQUALVERIFY. This is true, so the script continues to the next step. Finally, the signature of Carol is checked with OP_CHECKSIG.

Bonus question: To have multi-hop payments in a Payment Channel Network (PCN), payment channels can be used to route HTLC-based (Hash Time-Lock Contract) payments. These HTLCs are additional outputs in a channel, that need to be punished. Following the example above, assume that Mallory has posted an old state that holds one (or more) HTLCs. Now Carol needs to punish the output holding Mallory’s balance plus each output holding an HTLC. How can she make this punishment more efficient? Is there a way to decrease the amount of things you need to put on-chain?

Carol can make this punishment more efficient by using the HTLCs' preimages to punish the outputs. This way, she only needs to put the preimages on-chain, instead of the HTLCs themselves. This is possible because the HTLCs are secured by the preimages, so if Carol has the preimages, she can claim the HTLCs' outputs.

In summary, for every HTLC, there's a preimage (a secret value) whose hash was used to lock the funds. When the recipient of an HTLC reveals the preimage, they can claim the funds. If Carol has the preimages of these HTLCs (which she should have if the payments were settled correctly), she can use them to claim the HTLCs on-chain efficiently. Instead of creating separate transactions for each HTLC and the main channel balance, Carol includes the preimages in a single transaction. This approach significantly reduces the number of transactions Carol has to make on the blockchain.

# Exercise 3

This a the solution for exercise 3: sig_dave, id_dave.pk.to_hex(), UNKNOWN_HASH, 'OP_FALSE', sig_dave, id_dave.pk.to_hex()

or with the real values:

304402200a016c311d31336adffe6a5fdbf3c9efa63fac13b42215a2e12c8d4e8cf90d7d02203fc2fd51484e3fd7789151d3b5df16afe571f66ce0d25aedcbe5073b29bc2db201 02c6094e44a3c81d38fbcea6f3753625b2237d150732c0d2f19da6b507c93219ca 147d87d9ef309f0b49846573321199362a8283b1119e68189167fcbe3fe5f8c6 OP_FALSE 304402200a016c311d31336adffe6a5fdbf3c9efa63fac13b42215a2e12c8d4e8cf90d7d02203fc2fd51484e3fd7789151d3b5df16afe571f66ce0d25aedcbe5073b29bc2db201 02c6094e44a3c81d38fbcea6f3753625b2237d150732c0d2f19da6b507c93219ca

Before entering the if statement from the locking script we need to provide the public key of Dave and his signature, similarly to what we have done previously with Pay-to-PubKey-Hash lock scripts.

So after using this two values we have in the stack: sig_dave, id_dave.pk.to_hex(), UNKNOWN_HASH, 'OP_FALSE'

Since we are not yet pass the 'FUTURE_TIME' we need to exploit the else statement, therefore using OP_FALSE.

In the else statement we start by calculating the sha256 of our topmost element: UNKNOWN_HASH. Then UNKNOWN_HASH is added to the stack and we calculate OP_EQUAL. This results in being added the value 0, since the hash of a value is different from itself. The stack is then: sig_dave, id_dave.pk.to_hex(), 0

The two topmost values are duplicated with OP_2DUP and the stack becomes: sig_dave, id_dave.pk.to_hex(), 0, id_dave.pk.to_hex(), 0

The OP_HASH160 is applied to 0 and MALLORY_PK_HASH is pushed into the stack, resulting in: sig_dave, id_dave.pk.to_hex(), 0, id_dave.pk.to_hex(), OP_HASH160(0), MALLORY_PK_HASH

The operation OP_2ROT makes the fifth and sixth items go to the top of the stack resulting in: 0, id_dave.pk.to_hex(), OP_HASH160(0), MALLORY_PK_HASH, sig_dave, id_dave.pk.to_hex()

With two OP_DUP operations the topmost value (id_dave.pk.to_hex()) is duplicated twice. After this operation we exit the if statement and apply the OP_EQUALVERIFY operation which is trivially true, removing both the previously added values.

The operation OP_CHECKSIGVERIFY will use both the sig_dave and his public key and the resulting stack will be: 0, id_dave.pk.to_hex(), OP_HASH160(0), MALLORY_PK_HASH

With the operations OP_2DROP and OP_DROP we drop the three topmost values in the stack and the result is: 0

The final operation OP_NOT transforms this 0 into 1 which equates to true, meaning that our transaction is valid.
