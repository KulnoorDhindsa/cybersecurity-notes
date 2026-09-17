# Cryptography 
*Resources: Understanding Cryptography ~ Christof Paar and Jan Pelzl*

---
Cryptography ages back upto 2000 BC with earliest uses recorded in *scytale of Sparta* and the famous *Caesar cipher* of ancient Rome.

The most general term *cryptology* is divided into two branches:
1. **Cryptography**: Science of writing *secret writings* to hide messages.
    1. **Symmetric Algorithms**: Having encryption and decryption methods to convey secrets.
    2. **Asymmetric/Public-Key Algorithms**: Along with a secret key, there exists a public key with the user.
    3. **Cryptography Protocols**: Protocols to deal with applications having cryptographic algorithms.
        - e.g. TLS (Transport Layer Security, within every browser) 
2. **Cryptanalysis**: Science of *breaking* cryptosystems.
## Symmetric Cryptography
Symmetric Cryptography = Symmetric-key = Secret-key = Single-key schemes

*eavesdropping* is unauthorized listening. 

>Strong encryption patterns look gibberish to malicious users eavesdropping onto our conversation !

|Important Variables|Names|
|-------------------|-----|
|`x`|Plaintext / Cleartext|
|`y`|Ciphertext|
|`k`|Key|
|`key space`|Set of all possible keys|

Only secret to be kept in a *sound* cryptosystem is the key. 

>Problem of transmitting the encrypted code secretly, changes to transmitting the **key** secretly.

## Simple Symmetric Encryption / The Substitution Cipher

Substitution means substitution of each letter with another. 

>Substitution cipher is NOT secure at all!! 

Attacks include:
1. **Brute force** attack: Manually, all possible *keys* are tried  
    - Treated cipher as a *black box* 
    - Incorrect keys can give wrong conclusions, which might still make sense
    - Cypher is said to be *computationaly secure* against brute force attacks if testing all keys takes a long time 
    - Calculation of sample-space: if letter 'a' is assigned any random letter out of 26 alphabets (including a itself), there are 2^88w(26 factorial) possibilities!! excluding assigning all caps letters (a is B, instead of b)) 

2. **Letter Frequency Analysis**: Involves analysis of the cypher
    - If letter 'z' appears frequently, then 'z' is assigned to a 'frequently used letter in English' like e (11-13%), t (9%) etc and vica-versa. 
    - 'q' is *mostly* frequented by 'u' 
    - By chance, if word separators like blanks and spaces are found, short words like 'the' and 'and' van be de-crypted. 
## Cryptanalysis
  ```mermaid
graph TD
    A[Cryptanalysis] --> B[Classical Cryptanalysis]
    A --> C[Implementation Attacks]
    A --> D[Social Engineering]
    B --> E[Mathematical Analysis]
    B --> F[Brute-Force Attacks]
```

**Classical Cryptanalysis**: Recovering *plaintext* x or *key* k from *ciphertext* y.

**Social Engineering Attacks**: Bribing, classical manipulation along with tricking to obtain the *key* k.

**Modular Arithmetics**: Performing arithmetic in a finite set of integers.
- **equivalence class**: Groups of different cryptographic inputs/keys that produce identical observable results under an *equivalence relation*.
---

## Ceasar Cipher (Shift Cipher)
*Mechanism*: Shift every *plaintext* (x) letter by a fixed *number* of positions.
>27 shifts is the same as 1 shift !!
```
k = 17 (Ceasar Shift by 17)
ATTACK: 0,19,19,0,2,10
After Shifting by 17 i.e. add 17: 17,10,10,17,19,1
Result: rkkrtb
```
This is least bit secure and can be solved by:
1. **Brute Force**: Since only 26 different keys are possible, brute forcing makes sense
2. **Letter Frequency**: As in substituion.

### Affine Cipher
To complicate and *generalise* the Ceasar Cipher (to make it *more secure*), **a character is multipled and another is added** to plaintext.
```
key: (a,b)=(9,13)
modular inverse = 3 (as 9.3 = 27 which gives `1` when `mod26` and `1` is a.)
```
Result: N C C N F Z
Encrypt: `y=(a.x +b) (mod 26)`
Decrypt: `x = a inverse . (y-b) (mod 26)`