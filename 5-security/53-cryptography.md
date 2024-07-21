# Cryptography

- Cryptography achieves such protection by converting the data’s original bit pattern into a different bit pattern, using an algorithm called a cipher.
  - A cipher (or cypher) is an algorithm used for performing encryption or decryption—a series of well-defined steps that can transform readable information (plaintext) into unreadable form (ciphertext) and vice versa. 
- Basic idea
    - Take a piece of data
    - Use an algorithm (i.e. **cipher**), usually augmented with a second piece of information (i.e. **key**)
    - To covert the data into different form
$$
C=E(P, K)
$$
- Data: $P$, or plaintext
- Key: $K$
- Encryption algorithm: $E$
- Symmetric cryptography: use a **single secret key shared** by all parties with rights to access the data

## Public Key Cryptography

- Asymmetric ciphers: use one key to encrypt the data and a second key to decrypt the data, with one of the keys kept secret and the other commonly made public
- Use two different keys for cryptography, one to encrypt and one to decrypt

$$
C=E(P,K_{encrypt})
$$

$$
P=D(C, K_{decrypt})
$$

- Now they can tell everyone their decryption key, but keep the encryption key secret
- This form is called public key crypto
    - One of the two keys can be widely known to the entire public
    - While still achieving desirable results
- Public key: the key everyone knows
- Private key: the key that only the owner knows
- Only holders of the proper key can obtain the cipher’s benefit


## Cryptographic Hash

- Cryptographic hashes: do not allow reversal of the cryptography and do not require the use of keys
- No one should be able to obtain the **original bit pattern from the hash**