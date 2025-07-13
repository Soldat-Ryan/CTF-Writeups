# Rule Breaker 2

## Description
If you thought rules were easy after the last challenge, think again! I've concocted more devious password mangling rules to push the limits of your cracking knowledge (and possibly your CPU...):

    Password 1: Prepend 1 uppercase letter, Swap the first 2 characters, Rotate it to the right 3 times, Append a 4-digit year since 1900.
    Password 2: Lowercase the entire password. Apply a random caesar cipher shift to all the letters in the password. Then, replace each alphanumeric character with its right neighbor on the QWERTY keyboard. Finally, reverse it.
    Password 3: Split the password in half, toggle the case of every consonant in the first half, randomly toggle the case of all vowels in the second half, then interleave the halves together. Assume password has an even length and is no more than 14 characters.

> 2a07038481b64a934495e5a91d011ecbf278aba8c5263841e1d13f73975d5397 cd6e58d947e2f7ace23cb6d602daa1ae46934c3c1f4800bfd25e6af2b555f6f5 84b9e0298b1beb5236b7fcd2dd67e67abf62d16fe6d591024178790238cb4453

Use the rockyou.txt wordlist.

Flag format: L3AK{pass1_pass2_pass3}


## First password :
```
# From https://www.openwall.com/john/doc/RULES.shtml
^X	    prefix the word with character X
}	    rotate the word right: "smithj" -> "jsmith"
DN	    delete the character in position N
XNMI	extract substring NM from memory and insert into current word at I
```

Create a custom rule using john :
```
[List.Rules:custom_challenge1]
^[A-Z]X010D2}}}
```

Explanations below
```
^[A-Z]  -> "Prepend 1 uppercase letter"
X010D2  -> "Swap the first 2 characters`
}}}     -> "Rotate it to the right 3 times"
```

Write the new wordlist with --stdout
```sh
john -w=rockyou.txt --rules=custom_challenge1 --stdout > rockyou2.txt
```

Then use hybrid mode with hashcat to add the mask (-a 6) ("Append a 4-digit year since 1900.")
```sh
hashcat -m 1400 -a 6 "2a07038481b64a934495e5a91d011ecbf278aba8c5263841e1d13f73975d5397" rockyou2.txt "19?d?d"

2a07038481b64a934495e5a91d011ecbf278aba8c5263841e1d13f73975d5397:er!bLigbroth1984
```

## Second password : 

```
# From https://www.openwall.com/john/doc/RULES.shtml
l	    convert to lowercase
r	    reverse: "Fred" -> "derF"
```

Create a custom rule to convert the wordlist to lowercase ("Lowercase the entire password.")

```
[List.Rules:custom_challenge2]
l
```

Write the new wordlist with --stdout

```sh
john -w=rockyou.txt --rules=custom_challenge2 --stdout > rockyou_min.txt
```

Performing the ROT1 to ROT25 transformations to the wordlist with a C script
- "Apply a random caesar cipher shift to all the letters in the password"
- "Then, replace each alphanumeric character with its right neighbor on the QWERTY keyboard."

```c
❯ cat qwerty_map.h
#ifndef QWERTY_MAP_H
#define QWERTY_MAP_H

// QWERTY right-shift table for lowercase letters, digits, and symbols
char qwerty_right(char c) {
    switch (c) {
        // Numbers
        case '1': return '2';
        case '2': return '3';
        case '3': return '4';
        case '4': return '5';
        case '5': return '6';
        case '6': return '7';
        case '7': return '8';
        case '8': return '9';
        case '9': return '0';
        case '0': return '-';

        // Top row (QWERTY)
        case 'q': return 'w';
        case 'w': return 'e';
        case 'e': return 'r';
        case 'r': return 't';
        case 't': return 'y';
        case 'y': return 'u';
        case 'u': return 'i';
        case 'i': return 'o';
        case 'o': return 'p';
        case 'p': return '[';

        // Home row (ASDF)
        case 'a': return 's';
        case 's': return 'd';
        case 'd': return 'f';
        case 'f': return 'g';
        case 'g': return 'h';
        case 'h': return 'j';
        case 'j': return 'k';
        case 'k': return 'l';
        case 'l': return ';';

        // Bottom row (ZXCV)
        case 'z': return 'x';
        case 'x': return 'c';
        case 'c': return 'v';
        case 'v': return 'b';
        case 'b': return 'n';
        case 'n': return 'm';
        case 'm': return ',';

        default: return c; // no mapping: keep original
    }
}

#endif // QWERTY_MAP_H
```

```C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_LINE 256

// ROT1 to ROT25
void caesar_shift_lower(const char *input, char *output, int shift) {
    int i = 0;
    while (input[i] != '\0' && i < MAX_LINE - 1) {
        char c = input[i];
        if (c >= 'a' && c <= 'z') {
            output[i] = 'a' + ((c - 'a' + shift) % 26);
        } else {
            output[i] = c;
        }
        i++;
    }
    output[i] = '\0';
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s wordlist.txt\n", argv[0]);
        return 1;
    }

    FILE *infile = fopen(argv[1], "r");
    if (!infile) {
        perror("Error opening input file");
        return 1;
    }

    FILE *outfile = fopen("rot_output.txt", "w");
    if (!outfile) {
        perror("Error opening output file");
        fclose(infile);
        return 1;
    }

    char line[MAX_LINE];
    char shifted[MAX_LINE];
    while (fgets(line, sizeof(line), infile)) {
        // Supprimer \n ou \r
        line[strcspn(line, "\r\n")] = '\0';

        for (int rot = 1; rot <= 26; rot++) {
            caesar_shift_lower(line, shifted, rot);
            fprintf(outfile, "%s\n", shifted);
        }
    }

    fclose(infile);
    fclose(outfile);
    printf("ROT1 to ROT25 variants written to rot_output.txt\n");
    return 0;
}
```

Compilation & execution
```sh
gcc -O2 -o part2 part2.c

./part2 rockyou_min.txt
ROT1 to ROT25 variants written to rot_output.txt
```

Then use the reverse custom rule ("Finally, reverse it.")
```
[List.Rules:custom_challenge2]
r
```

```sh
john -w=rot_output.txt --rules=custom_challenge2 <(echo -n "cd6e58d947e2f7ace23cb6d602daa1ae46934c3c1f4800bfd25e6af2b555f6f5") --format=raw-sha256

o4d@lkny@d       (?)
```

## Third password
New c script to perform all these operations :
- "Split the password in half"
- "toggle the case of every consonant in the first half"
- "randomly toggle the case of all vowels in the second half" 
- "then interleave the halves together" 
- "Assume password has an even length and is no more than 14 characters."

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

#define MAX_LEN 16  // 14 chars max + newline + null
#define MAX_VOWELS 16

const char *VOWELS = "aeiouyAEIOUY";

int is_vowel(char c) {
    return strchr(VOWELS, c) != NULL;
}

char toggle_consonant(char c) {
    if (!isalpha(c) || is_vowel(c)) return c;
    return isupper(c) ? tolower(c) : toupper(c);
}

// Génère toutes les combinaisons de casses pour les voyelles dans la 2e moitié
void generate_variants(const char *first, const char *second, FILE *out) {
    int len = strlen(first);
    int vowel_indices[MAX_VOWELS];
    int vowel_count = 0;

    char base[8];
    strcpy(base, second);

    // Identifier les indices de voyelles dans second
    for (int i = 0; i < len; i++) {
        if (is_vowel(base[i])) {
            vowel_indices[vowel_count++] = i;
        }
    }

    int total = 1 << vowel_count; // 2^n combinaisons

    for (int mask = 0; mask < total; mask++) {
        char variant[8];
        strcpy(variant, base);

        for (int i = 0; i < vowel_count; i++) {
            int index = vowel_indices[i];
            if ((mask >> i) & 1)
                variant[index] = toupper(base[index]);
            else
                variant[index] = tolower(base[index]);
        }

        // Interleave
        char result[15];
        for (int i = 0, j = 0; i < len; i++) {
            result[j++] = first[i];
            result[j++] = variant[i];
        }
        result[2 * len] = '\0';
        fprintf(out, "%s\n", result);
    }
}

void process_word(const char *word, FILE *out) {
    int len = strlen(word);
    if (len % 2 != 0 || len > 14) return;

    int half = len / 2;
    char first[8], second[8];

    strncpy(first, word, half);
    strncpy(second, word + half, half);
    first[half] = second[half] = '\0';

    for (int i = 0; i < half; i++) {
        first[i] = toggle_consonant(first[i]);
    }

    generate_variants(first, second, out);
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s wordlist.txt\n", argv[0]);
        return 1;
    }

    FILE *in = fopen(argv[1], "r");
    if (!in) {
        perror("Cannot open input");
        return 1;
    }

    FILE *out = fopen("transformed_variants.txt", "w");
    if (!out) {
        perror("Cannot open output");
        fclose(in);
        return 1;
    }

    char line[MAX_LEN];
    while (fgets(line, sizeof(line), in)) {
        line[strcspn(line, "\r\n")] = '\0';
        process_word(line, out);
    }

    fclose(in);
    fclose(out);
    printf("Output written to transformed_variants.txt\n");
    return 0;
}
```

Compilation & execution
```sh
gcc -O2 -o part3 part3.c

./part3 rockyou.txt
Output written to transformed_variants.txt

john -w=transformed_variants.txt <(echo -n "84b9e0298b1beb5236b7fcd2dd67e67abf62d16fe6d591024178790238cb4453") --format=raw-sha256

Using default input encoding: UTF-8
Loaded 1 password hash (Raw-SHA256 [SHA256 128/128 ASIMD 4x])
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
CcoATnTdoyNY     (?)
1g 0:00:00:00 DONE (2025-07-12 20:20) 5.000g/s 14652Kp/s 14652Kc/s 14652KC/s CnoETlTloE..CUorSaTs
Use the "--show --format=Raw-SHA256" options to display all of the cracked passwords reliably
Session completed.
```

FLAG : `L3AK{er!bLigbroth1984_o4d@lkny@d_CcoATnTdoyNY}`

