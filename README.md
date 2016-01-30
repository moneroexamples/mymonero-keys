# Recovery of monero address and private view and spend keys using MyMonero's mnemonic seed

How to recover address, private view and spend keys in C++ based on
13 word mnemonic seed provided by [MyMonero](https://mymonero.com/).

[MyMonero](https://mymonero.com/) uses 13 word mnemonic seed, which
cant be used in the Monero's `simplewallet`. The reason is, that the `simplewallet`
uses 25 word mnemonic seed, and also, generation of private view and spend keys
by the `simplewallet` is different than that used by [MyMonero](https://mymonero.com/).

More information on the differences between the `simplewallet` and mymonero,
can be found:

  - [Cryptnote Tests](http://xmrtests.llcoins.net/addresstests.html)
  - [Why MyMonero key derivation is different than for the simplewallet (Mnemonic)](https://www.reddit.com/r/Monero/comments/3s80l2/why_mymonero_key_derivation_is_different_than_for/)

The example was prepared and tested on Xubuntu 15.10 x64 and Monero 0.9.

Instruction for Monero 0.9 compilation and setup of Monero's header files and libraries are at:
 - [Compile Monero 0.9 on Ubuntu 15.10 and 14.04 x64](https://github.com/moneroexamples/compile-monero-09-on-ubuntu)


## C++: main.cpp

```c++
int main(int ac, const char* av[]) {

    // get command line options
    xmreg::CmdLineOptions opts {ac, av};

    auto help_opt = opts.get_option<bool>("help");

    // if help was chosen, display help text and finish
    if (*help_opt)
    {
        return 0;
    }

    // default language for the mnemonic
    // representation of the private spend key
    string language {"English"};

    // get 13 word mnemonic seed from MyMonero
    auto mnemonic_opt = opts.get_option<string>("mnemonic");

    // get the program command line options, or
    // some default values for quick check
    string mnemonic_str = mnemonic_opt
                          ? *mnemonic_opt
                          : "slid otherwise jeers lurk swung tawny zodiac tusks twang cajun swagger peaches tawny";

    cout << "\n"
         << "Mnemonic seed    : " << mnemonic_str << endl;


    // change the MyMonero 13 word mnemonic seed
    // to its 16 byte hexadecimal version
    xmreg::secret_key16 hexadecimal_seed;

    // use modified words_to_bytes function.
    xmreg::ElectrumWords::words_to_bytes(mnemonic_str, hexadecimal_seed, language);

    cout << "\n"
         << "Hexadecimal seed : " << hexadecimal_seed << endl;


    // take the 16 byte hexadecimal_seed, and
    // and perform Keccak hash on it. It will
    // produce 32 byte hash.
    crypto::hash hash_of_seed;

    cn_fast_hash(hexadecimal_seed.data, sizeof(hexadecimal_seed.data), hash_of_seed);

    cout << "\n"
         << "Hash of seed     : " << hash_of_seed<< endl;


    // having the hashed seed, we can proceed
    // with generation of private and public spend keys.
    // the keccak hash of the seed is used as a seed
    // to generate the spend keys.
    crypto::public_key public_spend_key;
    crypto::secret_key private_spend_key;

    crypto::generate_keys(public_spend_key, private_spend_key,
                          xmreg::get_key_from_hash<crypto::secret_key>(hash_of_seed),
                          true);

    cout << "\n"
         << "Private spend key: " << private_spend_key << "\n"
         << "Public spend key : " << public_spend_key  << endl;

    // now we get private and public view keys.
    // to do this, we keccak hash the hash_of_seed again
    crypto::hash hash_of_hash;
    cn_fast_hash(hash_of_seed.data, sizeof(hash_of_seed.data), hash_of_hash);

    crypto::public_key public_view_key;
    crypto::secret_key private_view_key;


    crypto::generate_keys(public_view_key, private_view_key,
                          xmreg::get_key_from_hash<crypto::secret_key>(hash_of_hash),
                          true);

    cout << "\n"
         << "Private view key : " << private_view_key << "\n"
         << "Public view key  : " << public_view_key  << endl;



    // having all keys, we can get the corresponding monero address
    cryptonote::account_public_address address {public_spend_key, public_view_key};


    cout << "\n"
         << "Monero address   : " << address << endl;


    cout << "\nEnd of program." << endl;

    return 0;
}
```

## Example 1

```bash
./mymonero  -m "slid otherwise jeers lurk swung tawny zodiac tusks twang cajun swagger peaches tawny"
```
Result:

```bash
Mnemonic seed    : slid otherwise jeers lurk swung tawny zodiac tusks twang cajun swagger peaches tawny

Hexadecimal seed : <5878efd0a6d45b0374b49c000da07cd2>

Hash of seed     : <6d23fe14606a0d5fe62d05a78c4b5b1cae2f38f9330e42e86a50286db16ad61e>

Private spend key: <804f08b84507fb0610910d04ae517c07ae2f38f9330e42e86a50286db16ad60e>
Public spend key : <b54264728412c71bb62caca5b3cb57eb96a48a580c97a65257290243e3adf401>

Private view key : <dbb024c981cf7dc797593713f5f426c3b1cad50542c0814d4786e12e768be504>
Public view key  : <75fc0c90732a3db632bc0a169328067fa3f4c52e80ae067aa60bae8c4ccd8711>

Monero address   : <48VWHdLTEpE5dqa5VpAy5xgQWBHZARiGmEmojNLqD4ef1FAzkPxCe9JXUYwCShRR5XNMGnyusrnkmMWr2HMdfDRx2vsrG7c>
```

The private keys agree with those generated by MyMonero.

## Example 2

```bash
./mymonero  -m "nearby hover hiker renting giddy purged chrome paddles point tsunami hoax pledge point"
```
Result:

```bash
Mnemonic seed    : nearby hover hiker renting giddy purged chrome paddles point tsunami hoax pledge point

Hexadecimal seed : <fbdf0efd70ff3c5a321e0f0969041948>

Hash of seed     : <a16321b78892ff31facf9070f33eb445e2510bfd0e45e86891264771de5884f4>

Private spend key: <bef8b944fdc3eb086b9f0ee4e79aa30ce1510bfd0e45e86891264771de588404>
Public spend key : <39812cdb0d7f29bc8c70c319cb381b8f943b9a3fa35d40f59e93b63edd012605>

Private view key : <8952f17e4378c74fcd99ed6c70e799db60a601edf2685407087cc8c06d4db808>
Public view key  : <fb227e0e28539fd6ca13fc7d5b2da19307665ae0d152eb87b1239a00c6c66b07>

Monero address   : <43oVy226ybWYYA93SExWoLR1tznhhaXRqi5pLtYWhcM3212LriHRkLecvj6zoehqkLRbN9P18LDBGPhP9fHHY7Ar1qi9QDM>
```
The private keys agree with those generated by MyMonero.

## Example 3

```bash
./mymonero  -m "suddenly opus governing mystery divers obvious royal icon hefty nautical boxes among boxes"
```
Result:

```bash
Mnemonic seed    : suddenly opus governing mystery divers obvious royal icon hefty nautical boxes among boxes

Hexadecimal seed : <852249bca4446e65501cc7f8338027ec>

Hash of seed     : <30c0dc774aef038b78079f2478cf89d14f2d75c0a02bda4c5dbb14ce2913b97c>

Private spend key: <b5f423ed913983229cbdd9af61fa703f4f2d75c0a02bda4c5dbb14ce2913b90c>
Public spend key : <2c78b799c4d26a2dabb5ceaaff07efff9bd9ffc08cbc9b17634800019f3ca641>

Private view key : <601e6488942149980d9cd50298ef7cdb623cc58e8045f70de994faaea0a97a0e>
Public view key  : <7e9d7fcf3e20a8a3a02f0d73dd676dcc80d0d45c7efdd7d9c783453df150cca3>

Monero address   : <43JrWR7mQYh8e4gLAhVeXpjkj5b4RePpv4utgkFt7bpRBxP1t5J4wQKUNNiapRubXSbCwMLZDJ9CJdRjV1m26Kz3KRB4iFJ>
```

The private keys agree with those generated by MyMonero.

## How can you help?

Constructive criticism, code and website edits are always good. They can be made through github.

Some Monero are also welcome:
```
48daf1rG3hE1Txapcsxh6WXNe9MLNKtu7W7tKTivtSoVLHErYzvdcpea2nSTgGkz66RFP4GKVAsTV14v6G3oddBTHfxP6tU
```    
