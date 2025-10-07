---
title: "Revisiting an Old Project (Huffman Coding in C++)"
date: 2025-10-07
---
I want to get my C++ tooling set up in a way that's comfortable for me. That'll mean a build tool, package manager, unit testing, and benchmarking arrangement that isn't too annoying and is repeatable.

I'm picking Huffman Coding and Decoding as that project, because it has modest scope, can be objectively correct/incorrect, is well-studied, is suited to performance optimization, and I've played with it in the past.

### Background

Huffman Coding (https://en.wikipedia.org/wiki/Huffman_coding) is a data encoding used for compression. In a specific definition (replacing each symbol with a code, and each code using an integer number of bits), Huffman coding is optimal. The principle is to count the frequency of each symbol in the source and then to use those frequencies to build a code table for encoding and decoding.

### Canonical Huffman Codes

Canonical Huffman codes are a subset of Huffman Codes that are more restricted than general Huffman Codes. In general codes, if you invert all bits of each code value, you'll have another perfectly adequate code (there are many other transformations, I'd guess that applying an XOR with any bit-sequence to every code would work too). Canonical codes eliminate that ambiguity (and can simplify decoding as well). See some details here: https://en.wikipedia.org/wiki/Canonical_Huffman_code.

The algorithm given on Wikipedia only requires the symbols and the bit-length for each symbol as inputs.

### Part 1: Constructing the Codes

Constructing the codes is fairly straightforward and done in a few steps.

1. Count symbol frequencies (easily done with std::map)
2. Combine frequencies together, bottom-up, to form a tree. The node counts are known and the node-combining algorithm is given by Huffman (repeatedly take the two lowest-frequency nodes, combine them, and add the combined node to the pool).
3. Walk the tree, assign a bit-length to each symbol/leaf node equal to its depth in the tree[^1].
4. Sort the symbol data by bit-length, then in symbol order. Assign codes as described in the Canonical Huffman Code wikipedia article.

The code for this, so far (and with a build pipeline and some minimal unit testing) is at https://github.com/ljmcg/Huffman

[^1]: This wouldn't work for a tree with a single node: e.g the input is all one symbol. A zero length symbol wouldn't be much use! This is a toy/practice project, so I'm disregarding it.

ljmcg