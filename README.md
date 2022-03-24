# P7: Huffman Coding/Compression

- [Overview and Background (not required)](#overview-and-background-not-required)
    - [Acknowledgments and History](###acknowledgments-and-history)
- [Overview: What To Do](#overview-what-to-do)
    - [What To Do Details](#what-to-do-details)
- [Git Repositories](#git-repositories)
    - [Pushing to Git](#pushing-to-git)
    - [Git With Partners](#git-with-partners)
- [Before You Start: The BitInputStream and BitOutputStream Classes](#before-you-start-the-bitinputstream-and-bitoutputstream-classes)
- [Implementing `HuffProcessor.decompress`](#implementing-huffprocessordecompress)
- [Implementing `HuffProcessor.compress`](#implementing-huffprocessorcompress)
- [Analysis](#analysis)
- [Submitting](#submitting)
- [Reflect](#reflect)
- [Grading](#grading)
- [Appendix: How the Tree in `decompress` was generated](#appendix-how-the-tree-in-decompress-was-generated)
- [Appendix: Print Debugging Levels](#appendix-print-debugging-levels)

## Overview and Background
<details>
<summary>**Please Read** Background</summary>

There are many techniques used to compress digital data. This assignment covers Huffman Coding.

Huffman coding was invented by David Huffman while he was a graduate student at MIT in 1950 when given the option of a term paper or a final exam. For details, see [this 1991 Scientific American Article][Huffman Article]. In an autobiography Huffman had this to say about the epiphany that led to his invention of the coding method that bears his name:

"*-- but a week before the end of the term I seemed to have nothing to show for weeks of effort. I knew I'd better get busy fast, do the standard final, and forget the research problem. I remember, after breakfast that morning, throwing my research notes in the wastebasket. And at that very moment, I had a sense of sudden release, so that I could see a simple pattern in what I had been doing, that I hadn't been able to see at all until then. The result is the thing for which I'm probably best known: the Huffman Coding Procedure. I've had many breakthroughs since then, but never again all at once, like that. It was very exciting.*"

[The Wikipedia reference][Huffman Wikipedia] is extensive as is [this online material][Duke Huffman ] developed as one of the original [Nifty Assignments][Nifty Assignments]. Both jpeg and mp3 encodings use Huffman Coding as part of their compression algorithms. In this assignment you'll implement a complete program to compress and uncompress data using Huffman coding.
**You should read about the Huffman algorithm, and work to understand it.** You won't be able to complete the project easily without the overview you'll get from this reading: https://www.cs.duke.edu/csed/poop/huff/info/ .

When you've read the description of the algorithm and data structures used you'll be ready to implement both decompression (aka uncompressing) and compression  using Huffman Coding. You'll be using input and output or I/O classes that read and write 1 to many bits at a time, i.e., a single zero or one to several zeros and ones. This will make debugging your program a challenge.

### Acknowledgments and History

We first gave a Huffman coding assignment at Duke in Spring of 1994. Over the years many people have worked on creating the infrastructure for the bit-reading and -writing code (as we changed from C to C++ to Java at Duke) and the GUI that drives the Huffman assignment. It was one of the original so-called "nifty assignments" (see http://nifty.stanford.edu) in 1999. In Fall of 2018 we moved away from the GUI and using a simple main and the command-line. This was done for pragmatic and philosophical reasons.

</details>

## Overview: What To Do
<details>
<summary>High-level overview, using Git, How to Run programs</summary>

You may work with a partner, ***preferably from your discussion section***, but you can work with anyone who is taking 201 this semester. You *must* complete this partner form, even if you don't want a partner: https://do-compsci.com/201fall21-p7-partner

You'll first implement decompression, by writing `HuffProcessor.decompress`. This method is called from `HuffMainDecompress`, the program you'll run. You'll make sure you can decompress **both** text and image files. When you're confident you can decompress both types of files, you'll implement compression by writing `HuffProcessor.compress`. This method is called from `HuffMainCompress`, which is the program you'll run. The class `HuffProcessor` will have helper methods --- some used by both `compress` and `decompress`, so these methods are in the same class to make that sharing easier to accomplish.

Verify compress works (see below)  -- this is feasible because you already have confidence that your decompress/uncompress program works. Test that you can compress (and decompress) both text and image files.

### Self Assessment 

You should read about the algorithm using the references above (https://www.cs.duke.edu/csed/poop/huff/info/) and watch the [class Video W](https://www.youtube.com/watch?v=P_2dA2YfDOo) before you start writing code. You should be able to answer these questions, and you should try to do so, before writing code:

1. Why are two passes over the input file to be compressed required when creating a compressed version of the input file?
2. What aspects of creating the Huffman tree from counts account for that process being a greedy algorithm?
3. At a high-level, how is the tree used to create 8-bit char/chunk encodings?
4. What is written first, after the magic number, in the compressed file?
5. Why are the bits written at the end of the compressed file representing PSEUDO_EOF required?
6. After reading the magic number and tree, how are the bits representing compressed data read when decompressing, e.g., how many bits are read each time the compressed data is accessed?


### What To Do Details

You'll first write a program to decompress an already compressed file. _**When that works**_ you'll write a program to compress any file: text, image, sound, etc.

1.  **Run `HuffMainDecompress`**. This prompts for a file to decompress, then calls `HuffProcessor.decompress`. You're given a stub version of that method; it initially ***simply copies the first file to another file***, it doesn’t actually decompress it. To make sure you know how to use this program, we recommend you run the program as described.

Choose `mystery.tif.hf` from the data folder to decompress (the `.hf` suffix indicates this has been compressed by a working `HuffProcessor.compress`). When prompted with a name for the file to save, use a `UHF prefix`, i.e., save with the name `UFHmystery.tif.uhf (that suffix is the default).  Then run _diff_ on the command line/terminal (or the _Diff.java_ program as described below). Use diff to compare two files: the original, `mystery.tif.hf` and the uncompressed version: `UHFmystery.tif.uhf`. The diff/Diff program should say these files are the same. This is because the code you first get from git simply copies the first file to another file, it doesn't actually decompress it. 

The main takeaways here in running before implementing `HuffProcess.decompress` are to 
- Understand what to run when decompressing.
- Understand how to use _diff_ on the command line or the _Diff.java_ program to compare files. The command line _diff_ is easier to run, see the section below for details.

Implement `HuffProcessor.decompress` as described in the section below. Only when you can decompress mystery/hidden files and verify their contents are you ready to proceed to the `HuffProcessor.compress` method which is described in its own section below.



## Git Repositories

Fork, clone, and import the cloned project from the file system. Use this URL from the course GitLab site: https://coursework.cs.duke.edu/201fall21/P7-Huffman. ***Be sure to fork first*** (see screen shot). Then, clone using the SSH URL after using a terminal window to cd into your IntelliJ workspace. 


<div align="center">
  <img width="421" height="61" src="p7-figures/gitClone.png">
</div>


### Pushing to Git

When you make a series of changes you want to 'save', you'll push those changes to your GitLab repository. You should do this after major changes, certainly every hour or so of coding. You'll need to use the standard Git sequence to commit and push to GitLab:

```bash
git add .
git commit -m 'a short description of your commit here'
git push
```

### Git With Partners

You may work with a partner, ***preferably from your discussion section, but can be anyone on this assignment***. One person should fork-and-clone from the GitLab repo. That person will add the other person/partner as a collaborator on the project. For full information, see the documentation here:  https://docs.gitlab.com/ee/user/project/members/. [^1]
Choose Settings->Members->Invite Members. Then use the autocomplete feature to invite your partner to the project. Both of you can clone and push to this project.

As long as partners are modifying different files, this process works seamlessly. Modifying the same file can lead to issues in resolving conflicts. Git will deal with this with your help, but it's better to take turns in working on the same file, or to work on different files within the project (not very possible in Huffman Coding as done in 201). ***Ideally you'll always be together in-person, via Zoom, or otherwise when working on the project***.
When submitting you'll use the partner/team in Gradescope. Refer to the directions from [P4](https://coursework.cs.duke.edu/201-public-documentation/p4-dna-linkstrand) if you need a refresher on using Git/Gradescope with a partner.

There is only one file you will modify in this project: `HuffProcessor.java`. ***Try to work together, side-by-side or via Zoom, if you have a partner.***

## Before You Start: The `BitInputStream` and `BitOutputStream` Classes

These two classes are provided to help in reading/writing bits in (un)compressed files. They extend the Java [InputStream](https://docs.oracle.com/javase/9/docs/api/java/io/InputStream.html) and [OutputStream](https://docs.oracle.com/javase/9/docs/api/java/io/OutputStream.html) classes, respectively. They function just like Scanner, except instead of reading in / writing out arbitrarily delimited “tokens”, they read/write a specified number of bits. Note that two consecutive calls to the `readBits` method will likely return different results since InputStream classes maintain an internal "cursor" or "pointer" to a spot in the stream from which to read -- and once read the bits won't be read again (unless the stream is reset).

The only methods you will need to interact with are the following:
1. *`int BitInputStream.readBits(int numBits)`*: This method reads from the source the specified number of bits and returns an integer. Since integers are 32-bit in Java, the parameter `numBits` must be between 1 and 32, inclusive. **It will return -1 if there are no more bits to read.**
2. `void BitInputStream.reset()`: This method repositions the “cursor” to the beginning of the input file.
3. `void BitOutputStream.writeBits(int numBits, int value)`: This method writes the least-significant `numBits` bits of the value to the output file.

[^1]: [Here is a video](https://duke.zoom.us/rec/play/7sJ_f-n7_D83G9KUtwSDA6B7W9ToJ66s2yMd_PBfzRzkVHlVYVbwN7IR0JVARrVzkz27qCD6UJSD3w?continueMode=true) from a previous semester showing how partners can get started with Git (cloning, pushing and pulling, resolving merge conflicts).

</details>

## Implementing `HuffProcessor.decompress`
<details>
<summary>Writing Decompression Code First</summary>

You'll remove the code you're given intially in `HuffProcessor.decompress` and implement code to actually compress as described in this section. You **must remember to close the output file** before `decompress` returns. The call `out.close` is in the code you're given, be sure it's in the code you write as well.



There are four conceptual steps in decompressing a file that has been compressed using Huffman coding:
1. Read the 32-bit "magic" number as a check on whether the file is Huffman-coded (see lines 150-153 below)
2. Read the tree used to decompress, this is the same tree that was used to compress, i.e., was written during compression (helper method call on line 154 below).
3. Read the bits from the compressed file and use them to traverse root-to-leaf paths, writing leaf values to the output file. Stop when finding `PSEUDO_EOF` (hidden while loop on lines 156-174 below).
4. Close the output file (line 175 below).

<div align="center">
  <img width="837" height="254" src="p7-figures/decompress.png">
</div>


We recommend using the code above as a base for your decompress method because it illustrates how to use the bit-stream classes and how to throw appropriate exceptions. However, you will not be penalized if you use an alternative method to code it. To understand this in more detail, please read the explanation in https://www2.cs.duke.edu/csed/poop/huff/info/ -- in particular you'll need to know how the tree was written to write code that reads the tree.

As you can see, a `HuffException` is thrown if the file of compressed bits does not start with the 32-bit value `HUFF_TREE`. Your code should also throw a `HuffException` if reading bits ever fails, i.e., the `readBits` method returns -1. That could happen in the helper methods when reading the tree and when reading the compressed bits.


### Reading the Tree (private HuffNode readTree)

<br>

Reading the tree using a helper method is required since reading the tree, stored using a pre-order traversal, is much simpler with recursion. You don't have to use the names or parameters described above, though you can.
In the 201 Huffman tree protocol, interior tree nodes are indicated by the single bit 0 and leaf nodes are indicated by the single bit 1. No values are written for internal nodes and a 9-bit value is written for a leaf node. This leads to the following pseudocode to read the tree.
``` java
private HuffNode readTree(BitInputStream in) {
bit = in.readBits(1);
if (bit == -1) throw exception
if (bit == 0) {
    left = readTree(...)
    right = readTree(...)
    return new HuffNode(0,0,left,right);
}
else {
    value = read BITS_PER_WORD+1 bits from input
    return new HuffNode(value,0,null,null);
    }
}
``` 
For example, the tree below corresponds to the bit sequence `0001X1Y01Z1W01A1B`, with each letter representing a 9-bit sequence to be stored in a leaf as shown in the tree to the right. You'll read these 9-bit chunks with an appropriate call of `readBits`. Rather than use 9, you should use `BITS_PER_WORD + 1`, the +1 is needed since one leaf stores `PSEUDO_EOF` so all leaf nodes store a 9-bit value. See the [appendix](#appendix-how-the-tree-in-decompress-was-generated) for a detailed explanation on how this tree was constructed.

<div align="center">
  <img width="291" height="213" src="p7-figures/huffheadtreeNODES.png">
</div>

### Reading Compressed Bits (while (true))

<br>

Once you've read the bit sequence representing the tree, you'll read the remaining bits from the `BitInputStream` representing the compressed file one bit at a time, traversing the tree from the root and going left or right depending on whether you read a zero or a one. This is represented in the pseudocode for `decompress` by the hidden while loop.

The pseudocode from https://www.cs.duke.edu/csed/poop/huff/info/  is reproduced here, this is the same code shown in that document -- it's not perfect Java, hence pseudocode. (Note: you break when reaching `PSEUDO_EOF`, and then no bits are written to the output file. Otherwise, when writing a value stored in a leaf to the output stream, you write 8, or `BITS_PER_WORD` bits).

``` java
  HuffNode current = root; 
  while (true) {
      int bits = input.readBits(1);
      if (bits == -1) {
          throw new HuffException("bad input, no PSEUDO_EOF");
      }
      else { 
          if (bits == 0) current = current.left;
          else current = current.right;

          if (current is a leaf node) {
              if (current.value == PSEUDO_EOF) 
                  break;   // out of loop
              else {
                  write BITS_PER_WORD bits to output for current.value;
                  current = root; // start back after leaf
              }
          }
      }
  }
  close output file
```


### Command-line diff and Diff.java

There is a mac/unix command `diff` you can run in a terminal/bash shell on Mac/Windows. This command-line `diff`  compares two files and indicates if they are the same bit-for-bit or not. 

You type (terminal/shell):    `diff foo.txt bar.txt`

If the files are the same _nothing is printed_. If the files are different there's an indication of where they are different for text files, 
and just `different` if the files are binary/compressed/image files. For huff, you'll likely use debugging print statements if files aren't the same. 
You can also run Diff.java, provided to you for this assignment. It will ask you to select two files (from the same folder). The program prompts you to select files and displays a message if the files are the same or different.
You use command-click or control-click to choose a second file on Mac/Windows. 


</details>

## Implementing `HuffProcessor.compress`
<details>
<summary>Writing Compress after finishing Decompress</summary>

There are five conceptual steps to compress a file using Huffman coding. You do not need to use helper methods for these steps, but for some steps helper methods are extremely useful and will facilitate debugging.
1. Determine the frequency of every eight-bit character/chunk in the file being compressed (see line 78 below).
2. From the frequencies, create the Huffman trie/tree used to create encodings (see line 79 below).
3. From the trie/tree, create the encodings for each eight-bit character chunk (see lines 83-84 below).
4. Write the magic number and the tree to the beginning/header of the compressed file (see lines 81-82 below).
5. Read the file again and write the encoding for each eight-bit chunk, followed by the encoding for PSEUDO_EOF, then close the file being written (not shown).
You won't need to throw exceptions for the steps outlined. A brief description of each step follows. More details can be found in the explanation of the Huffman algorithm [here][Duke Huffman]

<div align="center">
  <img width="798" height="244" src="p7-figures/newcompress.png">
</div>

### Determining Frequencies (private int[] getCounts)

<br>

Create an integer array that can store 256 values (use `ALPH_SIZE`). You'll read 8-bit characters/chunks, (using `BITS_PER_WORD` rather than 8), and use the read/8-bit value as an index into the array, incrementing the frequency. Conceptually this is a map from 8-bit chunk to frequency for that chunk, it's easy to use an array for this, mapping the index to the number of times the index occurs, e.g., `counts['a']` is the number of times 'a' occurs in the input file being compressed. The code you start with in compress (and decompress) illustrates how to read until the sentinel -1 is read to indicate there are no more bits in the input stream. 


### Making Huffman Trie/Tree (private HuffNode makeTree)

<br>

You'll use a greedy algorithm and a `PriorityQueue` of `HuffNode` objects to create the trie. Since `HuffNode` implements `Comparable` (using weight), the code you write will remove the minimal-weight nodes when `pq.remove()` is called as shown in the pseudocode below.
``` java
PriorityQueue<HuffNode> pq = new PriorityQueue<>();
for(every index such that freq[index] > 0) {
    pq.add(new HuffNode(index,freq[index],null,null));
}
pq.add(new HuffNode(PSEUDO_EOF,1,null,null)); // account for PSEUDO_EOF having a single occurrence

while (pq.size() > 1) {
   HuffNode left = pq.remove();
   HuffNode right = pq.remove();
   // create new HuffNode t with weight from
   // left.weight+right.weight and left, right subtrees
   pq.add(t);
}
HuffNode root = pq.remove();
return root;
```

You'll need to ***be sure that `PSEUDO_EOF` is represented in the tree. *** As shown above, you should only add nodes to the priority queue for indexes/8-bit values that occur, i.e., that have non-zero weights.

### Make Codings from Trie/Tree (private makeEncodings)

<br>

For this, you'll essentially implement a recursive helper method, similar to code you've seen in discussion for the [LeafTrails APT problem](https://www2.cs.duke.edu/csed/newapt/leaftrails.html). As shown in the example of compress above, this method populates an array of Strings such that `encodings[val]` is the encoding of the 8-bit chunk val. See the debugging runs at the end of this write-up for details. As with the LeafTrails APT, the recursive helper method will have the array of encodings as one parameter, a node that's the root of a subtree as another parameter, and a string that's the path to that node as a string of zeros and ones. The first call of the helper method might be as shown, e.g., in the helper method `makeEncodings`.
``` java
    String[] encodings = new String[ALPH_SIZE + 1];
    makeEncodings(root,"",encodings);
```

In this method, if the `HuffNode` parameter is a leaf, an encoding for the value stored in the leaf is added to the array, e.g.,
``` java
   if (root is leaf) {
        encodings[root.value] = path;
        return;
   }
```
If the root is not a leaf, you'll need to make recursive calls adding "0" to the path when making a recursive call on the left subtree and adding "1" to the path when making a recursive call on the right subtree. Every node in a Huffman tree has two children. ***Be sure that you only add a single "0" for left-call and a single "1" for right-call. Each recursive call has a String path that's one more character than the parameter passed, e.g., path + "0" and path + "1".***

### Writing the Tree (private void writeTree)

<br>

Writing the tree is similar to the code you wrote to read the tree when decompressing. If a node is an internal node, i.e., not a leaf, write a single bit of zero. Else, if the node is a leaf, write a single bit of one, followed by _nine bits_ of the value stored in the leaf.  This is a pre-order traversal: write one bit for the node, then make two recursive calls if the node is an internal node. No recursion is used for leaf nodes. You'll need to write 9 bits, or `BITS_PER_WORD + 1`, because there are 257 possible values including `PSEUDO_EOF`.

### Writing Compressed Bits

After writing the tree, you'll need to read the file being compressed one more time. As shown above, the ***`BitInputStream` is reset, then read again to compress it***. The first reading was to determine frequencies of every 8-bit chunk. The encoding for each 8-bit chunk read is stored in the array created when encodings were made from the tree. These encodings are stored as strings of zeros and ones, e..g., "010101110101". To convert such a string to a bit-sequence you can use `Integer.parseInt` specifying a radix, or base of two. For example, to write the encoding for the 8-bit chunk representing 'A', which has an ASCII value of 65, you'd use:
``` java
    String code = encoding['A']
    out.writeBits(code.length(), Integer.parseInt(code,2));
```
You'll use code like this for every 8-bit chunk read from the file being compressed. You must also write the bits that encode `PSEUDO_EOF`, i.e.,
``` java
    String code = encoding[PSEUDO_EOF]
    out.writeBits(code.length(), Integer.parseInt(code,2));
```
You'll write these bits _after_ writing the bits for every 8-bit chunk. The encoding for `PSEUDO_EOF` is used when decompressing, ***so you'll need to write the encoding bits before the output file is closed.***

</details>

## Analysis
<details> 
<summary>Analysis, Submitting, Grading</summary>

No analysis is required. However, you should be able to answer these questions, and they may appear on the final.
1. Why did you implement decompress first?
2. What is the purpose of PSEUDO_EOF?
3. How can a compressed file have more bits than the file being compressed? When does this happen?
4. What compresses more: an image file or a text file, why do you think this happens?

## Submitting

You'll submit the code to Gradescope after pushing your program to GitLab. This is a partner/team project in Gradescope. ***If you're working with a partner only one person will submit and then will be able to add the other as partner.***

Refer to this document for submitting to Gradescope with a partner: https://docs.google.com/document/d/1L_zqiecoK6GXKyaW1cSE-msE1C7QJYhcz7FRKusmjik/edit#heading=h.e0pq4rh5kfd2

## Reflect

You can access the reflect: https://do-compsci.com/201fall21-p7-reflect

## Grading

Points are awarded equally for compression and decompression. You'll get points for decompressing and compressing text and image files. These are 10 points each, for a total of 40 points possible. There is no graded analysis portion for this project. Completing the reflect is two points. Completing the partner form is two points

- Reflect: https://do-compsci.com/201fall21-p7-reflect
- Partner: https://do-compsci.com/201fall21-p7-partner


</details>

## Appendix: How the Tree in `decompress` was generated
<details>
<summary>Development and Debugging</summary>

A 9-bit sequence represents a "character"/chunk stored in each leaf. This character/chunk, actually a value between 0-255, will be written to the output stream when uncompressing. One leaf stores PSEUDO_EOF, this won't be printed during uncompression, but will stop the process of reading bits.

<div align="center">
  <img width="291" height="213" src="p7-figures/huffheadtreeNODES.png">
</div>

When you read the first 0, you know it's an internal node (it's a 0), you'll create an internall `HuffNode` node, and recursively read the left and right subtrees. The left subtree call will read the bits 001X1Y01Z1W as the left subtree of the root and the right subtree recursive call will read  01A1B as the right subtree. Note that in the bit-sequence representing the tree, a single bit of 0 and 1 differentiates INTERNAL nodes from LEAF nodes, not the left/right branch taken in uncompressing -- that comes later. The internal node that's the root of the left subtree of the overall root has its own left subtree of 01X1Y and a right subtree of 01Z1W. When you read the single 1-bit, your code will need to read 9-bits to obtain the value stored in the leaf.

## Appendix: Print Debugging Levels

The class `HuffProcessor` can be constructed debugging as on, or true. The default is that debugging is off or false.
``` java
 HuffProcessor hp  = new HuffProcessor();       // default, debugging off
 HuffProcessor hp = new HuffProcessor(true);    // here set to true
```
You can then write printing/debugging code in your helper methods that you can "turn off" by simply constructing the `HuffProcessor` with on off/false value for debugging. In my code I have statements like the following:
Which prints information about the frequencies of each 8-bit chunk.

<div align="center">
  <img  src="p7-figures/newdebug.png">
</div>

For the file `small.txt` which consists of the single line ***Compsci 201: Duke***, the full debugging output is shown below for my program that compresses the data. You don't need to replicate this, but you may find that having some debugging output is useful. This file is small, so there's not much debugging output. There would be much more for a larger or image file.

```
Huffman Compress Main
10        1
32        2
48        1
49        1
50        1
58        1
67        1
68        1
99        1
101        1
105        1
107        1
109        1
111        1
112        1
115        1
117        1
pq created with 18 nodes
encoding for 10 is 0010
encoding for 32 is 010
encoding for 48 is 11010
encoding for 49 is 0001
encoding for 50 is 1011
encoding for 58 is 11011
encoding for 67 is 1100
encoding for 68 is 11110
encoding for 99 is 11101
encoding for 101 is 0111
encoding for 105 is 1001
encoding for 107 is 1000
encoding for 109 is 0110
encoding for 111 is 1010
encoding for 112 is 11100
encoding for 115 is 11111
encoding for 117 is 0000
encoding for 256 is 0011
compress from small.txt to small.hf
file: 144 bits to 312 bits
read 144 bits, wrote 309 bits
bits saved = -165
```


</details>

[Huffman Article]: https://www.huffmancoding.com/my-uncle/scientific-american
[Huffman Wikipedia]: https://en.wikipedia.org/wiki/Huffman_coding
[Duke Huffman]: https://www2.cs.duke.edu/csed/poop/huff/info/
[Nifty Assignments]: http://nifty.stanford.edu
