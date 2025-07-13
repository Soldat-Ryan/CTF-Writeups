# Rule Breaker 1

## Description

I hashed 3 more of my most secret passwords with SHA256. To make the passwords even more unbreakable, I mutated each one according to the following rules:

    Password 1: Append 3 characters at the end, in the following order: a special character, a number, and an uppercase letter
    Password 2: A typo was made when typing the password. Consider a typo to mean a single-character deletion from the password
    Password 3: Make the password leet (and since I'm nice, I'll tell you a hint: only vowels are leetified!)

I'm confident you'll never be able to crack my hashes now! Do your worst!

>5e09f66ae5c6b2f4038eba26dc8e22d8aeb54f624d1d3ed96551e900dac7cf0d fb58c041b0059e8424ff1f8d2771fca9ab0f5dcdd10c48e7a67a9467aa8ebfa8 4ac53d04443e6786752ac78e2dc86f60a629e4639edacc6a5937146f3eacc30f

Use the rockyou.txt wordlist.

Flag format: L3AK{pass1_pass2_pass3}

## First password

Use the hashcat hybrid mode with -a 6 (wordlist + mask)
 
```
?d -> any digit
?s -> any special char
?u -> any uppercase
```

```sh
hashcat -m 1400 -a 6 "5e09f66ae5c6b2f4038eba26dc8e22d8aeb54f624d1d3ed96551e900dac7cf0d" rockyou.txt "?s?d?u"

5e09f66ae5c6b2f4038eba26dc8e22d8aeb54f624d1d3ed96551e900dac7cf0d:hyepsi^4B
```


## Second password

A jumbo rule was enough :   

```sh
john /tmp/a --rules=jumbo -w=rockyou.txt --format=Raw-SHA256

thecowsaysmo     (?)
```

## Third password

Found a leetspeak rule in https://github.com/hashcat/hashcat/tree/master/rules

```sh
hashcat -m 1400 -a 1 "4ac53d04443e6786752ac78e2dc86f60a629e4639edacc6a5937146f3eacc30f" rockyou.txt -r leetspeak.rule

4ac53d04443e6786752ac78e2dc86f60a629e4639edacc6a5937146f3eacc30f:unf0rg1v@bl3
```

FLAG : L3AK{hyepsi^4B_thecowsaysmo_unf0rg1v@bl3}

