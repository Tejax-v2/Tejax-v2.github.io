---
title: "Simulating the Enigma Machine"
date: 2024-09-03
categories: [How things work]
tags: [enigma, hardware, cryptography]
---

Ever wondered how the famous Enigma machine from World War II worked? In this post, we'll break down a C program that simulates the Enigma machine—an encryption device used to encode secret messages. Don't worry, we'll keep it simple and straightforward.

## Introduction

This C program simulates the core components of the Enigma machine: three rotors, a reflector, and a plugboard. You input a plaintext message using uppercase letters, and the program scrambles it into ciphertext that only someone with the correct settings could decode. Let’s break it down step by step.

## The Code

```c
#include <stdio.h>
#include <string.h>

#define ALPHABET_SIZE 26

const char ROTORS[3][ALPHABET_SIZE] = {
    "QWERTYUIOPASDFGHJKLZXCVBNM",
    "MKOPLJINBHUYGVCFTRDXZSEWAQ",
    "CJHEWOIRPQXZTNSYVUBGKFLDMA"
};

const int NOTCHES[3] = { 2, 9, 24 };

const char REFLECTOR[ALPHABET_SIZE] = "YRUHQSLDPXNGOKMIEBFZCWVJAT";

int rotor_positions[3] = {4, 12, 6};

char plugboard[ALPHABET_SIZE];

int forward_rotor(int rotor, int c) {
    int pos = (c + rotor_positions[rotor]) % ALPHABET_SIZE;
    pos = ROTORS[rotor][pos] - 'A';
    return (pos - rotor_positions[rotor] + ALPHABET_SIZE) % ALPHABET_SIZE;
}

int backward_rotor(int rotor, int c) {
    int pos = (c + rotor_positions[rotor]) % ALPHABET_SIZE;
    for (int i = 0; i < ALPHABET_SIZE; i++) {
        if (ROTORS[rotor][i] - 'A' == pos) {
            return (i - rotor_positions[rotor] + ALPHABET_SIZE) % ALPHABET_SIZE;
        }
    }
    return c;
}

int reflect(int c) {
    return REFLECTOR[c] - 'A';
}

void step_rotors() {
    rotor_positions[0]++;
    if (rotor_positions[0] == ALPHABET_SIZE) {
        rotor_positions[0] = 0;
        rotor_positions[1]++;
        if (rotor_positions[1] == ALPHABET_SIZE) {
            rotor_positions[1] = 0;
            rotor_positions[2]++;
            if (rotor_positions[2] == ALPHABET_SIZE) {
                rotor_positions[2] = 0;
            }
        }
    }
    if (rotor_positions[0] == NOTCHES[0]) rotor_positions[1]++;
    if (rotor_positions[1] == NOTCHES[1]) rotor_positions[2]++;
}

void init_plugboard() {
    for (int i = 0; i < ALPHABET_SIZE; i++) {
        plugboard[i] = i;
    }
}

int plugboard_substitution(int c) {
    return plugboard[c];
}

char encrypt_char(char ch) {
    if (ch < 'A' || ch > 'Z') return ch;
    int c = ch - 'A';
    c = plugboard_substitution(c);
    for (int i = 0; i < 3; i++) {
        c = forward_rotor(i, c);
    }
    c = reflect(c);
    for (int i = 2; i >= 0; i--) {
        c = backward_rotor(i, c);
    }
    c = plugboard_substitution(c);
    step_rotors();
    return c + 'A';
}

void encrypt_message(const char *message, char *encrypted) {
    for (int i = 0; i < strlen(message); i++) {
        encrypted[i] = encrypt_char(message[i]);
    }
    encrypted[strlen(message)] = '\0';
}

int main() {
    char message[100];
    char encrypted[100];

    init_plugboard();

    plugboard['H' - 'A'] = 'X' - 'A';
    plugboard['X' - 'A'] = 'H' - 'A';
    plugboard['L' - 'A'] = 'V' - 'A';
    plugboard['V' - 'A'] = 'L' - 'A';

    printf("Enter Plaintext (A-Z only): ");
    scanf("%s", message);

    encrypt_message(message, encrypted);

    printf("Encrypted message: %s\n", encrypted);

    return 0;
}
```

## How Does It Work?

1. **The Plugboard:** 
    - Think of it as a basic letter-swapping system. Before a letter even hits the rotors, it might get swapped with another letter. For example, ‘H’ might become ‘X’ and ‘L’ might turn into ‘V’.
  
2. **The Rotors:**
    - The rotors are like spinning wheels that shift the alphabet around. Each rotor has a different scrambling pattern. As each letter is processed, the rotors move, changing the encryption pattern.

3. **The Reflector:**
    - After passing through the rotors, the letter hits the reflector, which bounces it back through the rotors in reverse order, further scrambling the message.

4. **Stepping the Rotors:**
    - Just like a mechanical watch, the rotors step forward after each letter is processed. If one rotor completes a full rotation, it causes the next rotor to step forward as well.

5. **The Encrypted Message:**
    - Once all letters in your message are processed, you get a scrambled ciphertext that looks like gibberish—unless you have the correct settings to decode it.

## Breaking Down the Code

Here’s how the magic happens under the hood:

### Constants and Variables
- **`ALPHABET_SIZE`**: We’re working with the 26 letters of the English alphabet.
- **`ROTORS`**: This is where we define the scrambling patterns for our three rotors.
- **`NOTCHES`**: Each rotor has a notch, which is like a trigger that makes the next rotor step when it reaches a certain point.
- **`REFLECTOR`**: Another scrambled alphabet array that reflects the signal back through the rotors.
- **`rotor_positions`**: Keeps track of where each rotor is positioned.
- **`plugboard`**: Handles those initial letter swaps before and after the rotors.

### Rotor Mechanism
- **`forward_rotor` function**: This function processes a letter from right to left through the rotor, considering its current position.
- **`backward_rotor` function**: Similar to `forward_rotor`, but it handles the signal in the reverse direction after reflection.

### Reflector Mechanism
- **`reflect` function**: Takes the letter after it’s passed through the last rotor and maps it through the reflector, sending it back through the rotors.

### Rotor Stepping
- **`step_rotors` function**: Steps the rotors forward, simulating that mechanical ticking of the original machine.

### Plugboard
- **`init_plugboard` function**: Initializes the plugboard, so each letter initially maps to itself.
- **`plugboard_substitution` function**: Swaps letters based on the plugboard configuration.

### Character Encryption
- **`encrypt_char` function**: This is where a single character gets encrypted by running it through the plugboard, the rotors (forward and backward), and the reflector. The rotors then step, and the function returns the encrypted character.

### Message Encryption
- **`encrypt_message` function**: Encrypts an entire message by processing each character through the `encrypt_char` function and collecting the results.

### The Main Function
- The `main` function is the starting point. It sets up the plugboard with specific swaps, gets your message, and runs it through the encryption process. The result? A secret message that only someone with the correct settings can read.

## Wrapping Up

And that's a wrap! If you’re into encryption or just love tinkering with code, give this project a try!

Happy coding!
