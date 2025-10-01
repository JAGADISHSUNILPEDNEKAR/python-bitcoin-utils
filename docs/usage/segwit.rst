SegWit Functionality
==================

SegWit (Segregated Witness) is a Bitcoin protocol upgrade that separates witness data (signatures and scripts) from the rest of the transaction, resulting in several benefits such as increased transaction capacity and fixing transaction malleability.

The Python Bitcoin Utils library provides comprehensive support for SegWit, including both version 0 (P2WPKH, P2WSH) and version 1 (Taproot/P2TR).

SegWit Versions
--------------

The library supports different versions of SegWit:

* **SegWit v0**: Original SegWit implementation (P2WPKH and P2WSH)
* **SegWit v1**: Taproot update (P2TR)

Address Types
------------

Native SegWit Addresses
^^^^^^^^^^^^^^^^^^^^^^^

P2WPKH (Pay to Witness Public Key Hash)
""""""""""""""""""""""""""""""""""""""""

.. code-block:: python

    from bitcoinutils.setup import setup
    from bitcoinutils.keys import PrivateKey

    setup('testnet')
    priv = PrivateKey()
    pub = priv.get_public_key()
    segwit_addr = pub.get_segwit_address()
    print(f"P2WPKH address: {segwit_addr.to_string()}")

P2WSH (Pay to Witness Script Hash)
""""""""""""""""""""""""""""""""""

.. code-block:: python

    from bitcoinutils.setup import setup
    from bitcoinutils.keys import PublicKey
    from bitcoinutils.script import Script

    setup('testnet')
    pub1 = PublicKey("03a1af804ac108a8a51782198c2d034b28bf90c8803f5a53f76276fa69a4eae77f")
    pub2 = PublicKey("02530c548d402670b13ad8887ff99c294e67fc18097d236d57880c69261b42def7")

    # Create a 2-of-2 multisig redeem script
    redeem_script = Script(['OP_2', pub1.to_hex(), pub2.to_hex(), 'OP_2', 'OP_CHECKMULTISIG'])
    witness_script_addr = redeem_script.get_segwit_address()
    print(f"P2WSH address: {witness_script_addr.to_string()}")

Nested SegWit Addresses
^^^^^^^^^^^^^^^^^^^^^^^

P2SH-P2WPKH
"""""""""""

.. code-block:: python

    from bitcoinutils.setup import setup
    from bitcoinutils.keys import PrivateKey

    setup('testnet')
    priv = PrivateKey()
    pub = priv.get_public_key()
    p2sh_p2wpkh_addr = pub.get_p2sh_p2wpkh_address()
    print(f"P2SH-P2WPKH address: {p2sh_p2wpkh_addr.to_string()}")

P2SH-P2WSH
""""""""""

.. code-block:: python

    from bitcoinutils.setup import setup
    from bitcoinutils.keys import PublicKey
    from bitcoinutils.script import Script

    setup('testnet')
    pub1 = PublicKey("03a1af804ac108a8a51782198c2d034b28bf90c8803f5a53f76276fa69a4eae77f")
    pub2 = PublicKey("02530c548d402670b13ad8887ff99c294e67fc18097d236d57880c69261b42def7")

    # Create a 2-of-2 multisig redeem script
    redeem_script = Script(['OP_2', pub1.to_hex(), pub2.to_hex(), 'OP_2', 'OP_CHECKMULTISIG'])
    p2sh_p2wsh_addr = redeem_script.get_p2sh_p2wsh_address()
    print(f"P2SH-P2WSH address: {p2sh_p2wsh_addr.to_string()}")

Taproot Addresses (SegWit v1)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    from bitcoinutils.setup import setup
    from bitcoinutils.keys import PrivateKey

    setup('testnet')
    priv = PrivateKey()
    pub = priv.get_public_key()
    taproot_addr = pub.get_taproot_address()
    print(f"P2TR address: {taproot_addr.to_string()}")

Creating SegWit Transactions
---------------------------

Sending to a P2WPKH Address
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    from bitcoinutils.setup import setup
    from bitcoinutils.keys import PrivateKey, P2wpkhAddress, P2pkhAddress
    from bitcoinutils.transactions import Transaction, TxInput, TxOutput
    from bitcoinutils.script import Script
    from bitcoinutils.utils import to_satoshis

    setup('testnet')

    # Create a P2WPKH address to send to
    recipient_addr = P2wpkhAddress('tb1qw508d6qejxtdg4y5r3zarvary0c5xw7kxpjzsx')

    # Create transaction input (from a previous P2PKH transaction)
    txin = TxInput('a16f3ce4dd5deb92d98ef5cf8afeaf0775ebca408f708b2146c4fb42b41e14be', 0)

    # Create transaction output
    txout = TxOutput(to_satoshis(0.001), recipient_addr.to_script_pub_key())

    # Create transaction (not segwit since we're spending from P2PKH)
    tx = Transaction([txin], [txout])

    # Sign the transaction
    priv_key = PrivateKey('cTALNpTpRbbxTCJ2A5Zq6NwAnBSQjguuuhdyzLbWXDuA8ExBq58d')
    from_addr = P2pkhAddress('n4bkvTyU1dVdzsrhWBqBw8fEMbHjJvtmJR')
    
    sig = priv_key.sign_input(
        tx, 0, 
        from_addr.to_script_pub_key()
    )
    
    # Set the scriptSig
    pub_key = priv_key.get_public_key()
    txin.script_sig = Script([sig, pub_key.to_hex()])

    print(f"Signed transaction: {tx.serialize()}")

Spending from a P2WPKH Address
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    from bitcoinutils.setup import setup
    from bitcoinutils.keys import PrivateKey, P2pkhAddress
    from bitcoinutils.transactions import Transaction, TxInput, TxOutput, TxWitnessInput
    from bitcoinutils.script import Script
    from bitcoinutils.utils import to_satoshis

    setup('testnet')

    # Create a transaction input (from a P2WPKH UTXO)
    txin = TxInput('a16f3ce4dd5deb92d98ef5cf8afeaf0775ebca408f708b2146c4fb42b41e14be', 0)

    # Create a P2PKH address to send to
    recipient_addr = P2pkhAddress('n4bkvTyU1dVdzsrhWBqBw8fEMbHjJvtmJR')

    # Create transaction output
    txout = TxOutput(to_satoshis(0.0009), recipient_addr.to_script_pub_key())

    # Create transaction with has_segwit=True
    tx = Transaction([txin], [txout], has_segwit=True)

    # Prepare for signing
    priv_key = PrivateKey('cTALNpTpRbbxTCJ2A5Zq6NwAnBSQjguuuhdyzLbWXDuA8ExBq58d')
    pub_key = priv_key.get_public_key()
    
    # For P2WPKH, the script code is the same as P2PKH scriptPubKey
    script_code = Script([
        'OP_DUP', 'OP_HASH160', 
        pub_key.to_hash160(), 
        'OP_EQUALVERIFY', 'OP_CHECKSIG'
    ])

    # Sign the segwit input
    amount = to_satoshis(0.001)  # Amount being spent from the UTXO
    signature = priv_key.sign_segwit_input(tx, 0, script_code, amount)

    # Set witness data for the input
    tx.witnesses.append(TxWitnessInput([signature, pub_key.to_hex()]))

    print(f"Signed transaction: {tx.serialize()}")

P2WSH Transaction Example
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    from bitcoinutils.setup import setup
    from bitcoinutils.keys import PrivateKey, P2pkhAddress
    from bitcoinutils.transactions import Transaction, TxInput, TxOutput, TxWitnessInput
    from bitcoinutils.script import Script
    from bitcoinutils.utils import to_satoshis

    setup('testnet')

    # Create a 2-of-2 multisig witness script
    priv1 = PrivateKey('cTALNpTpRbbxTCJ2A5Zq6NwAnBSQjguuuhdyzLbWXDuA8ExBq58d')
    priv2 = PrivateKey('cRvyLwCPLU88jsyj94L7iJjQX5C2f8koG4G2gevN4BeSGcEvfKe9')
    pub1 = priv1.get_public_key()
    pub2 = priv2.get_public_key()
    
    witness_script = Script([
        'OP_2', pub1.to_hex(), pub2.to_hex(), 'OP_2', 'OP_CHECKMULTISIG'
    ])

    # Define recipient address
    recipient_addr = P2pkhAddress('n4bkvTyU1dVdzsrhWBqBw8fEMbHjJvtmJR')

    # Spending from P2WSH
    txin = TxInput('a16f3ce4dd5deb92d98ef5cf8afeaf0775ebca408f708b2146c4fb42b41e14be', 0)
    txout = TxOutput(to_satoshis(0.0009), recipient_addr.to_script_pub_key())
    
    tx = Transaction([txin], [txout], has_segwit=True)
    
    # Sign with both keys
    amount = to_satoshis(0.001)
    sig1 = priv1.sign_segwit_input(tx, 0, witness_script, amount)
    sig2 = priv2.sign_segwit_input(tx, 0, witness_script, amount)
    
    # Witness for P2WSH multisig: empty item, sig1, sig2, witness_script
    tx.witnesses.append(TxWitnessInput([
        '',  # Empty item required for CHECKMULTISIG bug
        sig1,
        sig2,
        witness_script.to_hex()
    ]))

    print(f"Signed transaction: {tx.serialize()}")

Taproot Transactions
-------------------

Key Path Spending
^^^^^^^^^^^^^^^^^

.. code-block:: python

    from bitcoinutils.setup import setup
    from bitcoinutils.keys import PrivateKey, P2trAddress
    from bitcoinutils.transactions import Transaction, TxInput, TxOutput, TxWitnessInput
    from bitcoinutils.utils import to_satoshis

    setup('testnet')

    # Create transaction input from a P2TR UTXO
    txin = TxInput('a16f3ce4dd5deb92d98ef5cf8afeaf0775ebca408f708b2146c4fb42b41e14be', 0)

    # Create a transaction output
    recipient_addr = P2trAddress('tb1p5cyxnuxmeuwuvkwfem96lqzszd02n6xdcjrs20cac6yqjjwudpxqkedrcr')
    txout = TxOutput(to_satoshis(0.0009), recipient_addr.to_script_pub_key())

    # Create transaction with has_segwit=True
    tx = Transaction([txin], [txout], has_segwit=True)

    # Sign the taproot input using key path
    priv_key = PrivateKey('cTALNpTpRbbxTCJ2A5Zq6NwAnBSQjguuuhdyzLbWXDuA8ExBq58d')
    
    # Get the P2TR address and its scriptPubKey for this private key
    taproot_addr = priv_key.get_public_key().get_taproot_address()
    prev_script_pubkey = taproot_addr.to_script_pub_key()
    
    signature = priv_key.sign_taproot_input(
        tx, 0, 
        [prev_script_pubkey],  # List of all input script_pubkeys
        [to_satoshis(0.001)]   # List of all input amounts
    )

    # Set witness data for key path spending (only signature)
    tx.witnesses.append(TxWitnessInput([signature]))

    print(f"Signed transaction: {tx.serialize()}")

Script Path Spending
^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    from bitcoinutils.setup import setup
    from bitcoinutils.keys import PrivateKey, PublicKey, P2pkhAddress
    from bitcoinutils.transactions import Transaction, TxInput, TxOutput, TxWitnessInput
    from bitcoinutils.script import Script
    from bitcoinutils.utils import to_satoshis

    setup('testnet')

    # Create transaction input from a P2TR UTXO
    txin = TxInput('a16f3ce4dd5deb92d98ef5cf8afeaf0775ebca408f708b2146c4fb42b41e14be', 0)

    # Create a transaction output
    recipient_addr = P2pkhAddress('n4bkvTyU1dVdzsrhWBqBw8fEMbHjJvtmJR')
    txout = TxOutput(to_satoshis(0.0009), recipient_addr.to_script_pub_key())

    # Create transaction with has_segwit=True
    tx = Transaction([txin], [txout], has_segwit=True)

    # For script path spending, you need the taproot script
    pub_key = PublicKey('03a1af804ac108a8a51782198c2d034b28bf90c8803f5a53f76276fa69a4eae77f')
    tapscript = Script([pub_key.to_hex(), 'OP_CHECKSIG'])
    
    # Sign the taproot input using script path
    priv_key = PrivateKey('cTALNpTpRbbxTCJ2A5Zq6NwAnBSQjguuuhdyzLbWXDuA8ExBq58d')
    
    # WARNING: This is a SIMPLIFIED EXAMPLE for illustration only!
    # In production, the P2TR address must be properly constructed with:
    # - An internal key (tweaked or untweaked)
    # - A Merkle root derived from the script tree containing tapscript
    # The address below is just a placeholder and won't work with the script above.
    # See the library's taproot construction documentation for proper implementation.
    
    # For this example, we assume a P2TR address that was created with this script
    taproot_addr = P2trAddress('tb1p5cyxnuxmeuwuvkwfem96lqzszd02n6xdcjrs20cac6yqjjwudpxqkedrcr')
    prev_script_pubkey = taproot_addr.to_script_pub_key()
    
    signature = priv_key.sign_taproot_input(
        tx, 0, 
        [prev_script_pubkey],
        [to_satoshis(0.001)],
        ext_flag=1,  # Script path spending
        script=tapscript
    )

    # Note: Actual witness data would include the signature, the script, 
    # and the control block. The control block computation would be 
    # handled by other library functions.
    
    print(f"Signed transaction: {tx.serialize()}")

SegWit Transaction Digest
------------------------

The library uses different digest algorithms for signing SegWit transactions:

SegWit v0 Digest Algorithm
^^^^^^^^^^^^^^^^^^^^^^^^^

For SegWit v0, the `get_transaction_segwit_digest` method implements the BIP143 specification.

Taproot (SegWit v1) Digest Algorithm
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For Taproot (SegWit v1), the `get_transaction_taproot_digest` method implements the BIP341 specification.

Witness Structure
---------------

In SegWit transactions, the witness data is stored separately from the transaction inputs:

P2WPKH Witness
^^^^^^^^^^^^^

.. code-block:: python

    [signature, public_key]

P2WSH Witness
^^^^^^^^^^^^

For multisig:

.. code-block:: python

    ['', sig1, sig2, ..., sigN, witness_script]

Note: The empty string is required due to the CHECKMULTISIG off-by-one bug.

P2TR Key Path Witness
^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    [signature]

P2TR Script Path Witness
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    [signature, script, control_block]

Automatic Handling of Witness Data
--------------------------------

The library handles witness format for different input types when using higher-level transaction construction functions:

* For non-witness inputs in SegWit transactions, empty witnesses are added
* For P2WPKH inputs, create a witness with signature and public key
* For P2WSH inputs, create a witness with signatures and the witness script
* For P2TR inputs, create a witness with one signature for key path spending, or signature, script and control block for script path spending

Mixed Input Transactions
----------------------

When creating transactions with both SegWit and non-SegWit inputs:

.. code-block:: python

    from bitcoinutils.setup import setup
    from bitcoinutils.keys import PrivateKey, P2pkhAddress, P2wpkhAddress
    from bitcoinutils.transactions import Transaction, TxInput, TxOutput, TxWitnessInput
    from bitcoinutils.script import Script
    from bitcoinutils.utils import to_satoshis

    setup('testnet')

    # Create transaction inputs
    # Non-SegWit input (P2PKH)
    txin1 = TxInput('a16f3ce4dd5deb92d98ef5cf8afeaf0775ebca408f708b2146c4fb42b41e14be', 0)
    # SegWit v0 input (P2WPKH)
    txin2 = TxInput('75ddabb27b8845f5247975c8a5ba7c6f336c4570708ebe230caf6db5217ae858', 0)
    # Taproot input (P2TR)
    txin3 = TxInput('1dea7cd05979072a3578cab271c02244ea8a090bbb46aa680a65ecd027048d83', 0)

    # Create transaction output
    recipient_addr = P2pkhAddress('n4bkvTyU1dVdzsrhWBqBw8fEMbHjJvtmJR')
    txout = TxOutput(to_satoshis(0.0027), recipient_addr.to_script_pub_key())

    # Create transaction with has_segwit=True (required for any segwit inputs)
    tx = Transaction([txin1, txin2, txin3], [txout], has_segwit=True)

    # Sign each input with the appropriate method
    
    # 1. Legacy P2PKH input
    priv_key1 = PrivateKey('cTALNpTpRbbxTCJ2A5Zq6NwAnBSQjguuuhdyzLbWXDuA8ExBq58d')
    pub_key1 = priv_key1.get_public_key()
    legacy_addr = P2pkhAddress('n4bkvTyU1dVdzsrhWBqBw8fEMbHjJvtmJR')
    
    sig1 = priv_key1.sign_input(tx, 0, legacy_addr.to_script_pub_key())
    txin1.script_sig = Script([sig1, pub_key1.to_hex()])
    # Add empty witness for non-segwit input
    tx.witnesses.append(TxWitnessInput([]))

    # 2. SegWit v0 P2WPKH input
    priv_key2 = PrivateKey('cRvyLwCPLU88jsyj94L7iJjQX5C2f8koG4G2gevN4BeSGcEvfKe9')
    pub_key2 = priv_key2.get_public_key()
    script_code2 = Script([
        'OP_DUP', 'OP_HASH160', 
        pub_key2.to_hash160(), 
        'OP_EQUALVERIFY', 'OP_CHECKSIG'
    ])
    
    sig2 = priv_key2.sign_segwit_input(tx, 1, script_code2, to_satoshis(0.001))
    tx.witnesses.append(TxWitnessInput([sig2, pub_key2.to_hex()]))

    # 3. Taproot P2TR input (key path)
    priv_key3 = PrivateKey('cN9RbPMNcUwBzNNYa7cDJb2wPEKqCpCfe97KoHAWSdCDkqBTZ7tP')
    
    # Get proper scriptPubKeys for all inputs
    segwit_addr = P2wpkhAddress.from_hash(pub_key2.to_hash160())
    taproot_addr = priv_key3.get_public_key().get_taproot_address()
    
    # Collect all script_pubkeys and amounts for taproot signing
    all_script_pubkeys = [
        legacy_addr.to_script_pub_key(),
        segwit_addr.to_script_pub_key(),  
        taproot_addr.to_script_pub_key()
    ]
    all_amounts = [
        to_satoshis(0.001),  # Amount for input 0
        to_satoshis(0.001),  # Amount for input 1
        to_satoshis(0.001)   # Amount for input 2
    ]
    
    sig3 = priv_key3.sign_taproot_input(tx, 2, all_script_pubkeys, all_amounts)
    tx.witnesses.append(TxWitnessInput([sig3]))

    print(f"Signed mixed transaction: {tx.serialize()}")

OP_CHECKSIGADD Support
--------------------

Taproot introduces the new OP_CHECKSIGADD opcode for more efficient threshold multi-signature scripts:

.. code-block:: python

    from bitcoinutils.setup import setup
    from bitcoinutils.keys import PublicKey
    from bitcoinutils.script import Script

    setup('testnet')

    # Define public keys
    pub1 = PublicKey('03a1af804ac108a8a51782198c2d034b28bf90c8803f5a53f76276fa69a4eae77f')
    pub2 = PublicKey('02530c548d402670b13ad8887ff99c294e67fc18097d236d57880c69261b42def7')
    pub3 = PublicKey('03e9f948b1bca68c97fd22cc52b6930ca4ed5b1bbaf14e52e95903726df26b814f')

    # Create a 2-of-3 multi-signature script using OP_CHECKSIGADD
    multi_sig_script = Script([
        pub1.to_hex(), 'OP_CHECKSIG',
        pub2.to_hex(), 'OP_CHECKSIGADD',
        pub3.to_hex(), 'OP_CHECKSIGADD',
        'OP_2', 'OP_EQUAL'
    ])

    # This is more efficient than the traditional way:
    traditional_multisig = Script([
        'OP_2', pub1.to_hex(), pub2.to_hex(), pub3.to_hex(), 'OP_3', 'OP_CHECKMULTISIG'
    ])