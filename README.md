# Parallel N-Gram Analyze with MPI Library

## 1. Introduction

This is a project implemented as Midterm Project for the Parallel Computing Course. The main idea is under-
standing parallel computing. MPI was used for parallel computing. The goal of the project is writing a parallel program as an MPI application that finds 2-grams (bigram) in the given news dataset and shows frequency analysis in the shortest time.

### 1.1 MPICH

The Message Passing Interface Standard (MPI) is a message passing library standard. The goal of the Message
Passing Interface is to establish a portable, efficient, and flexible standard for message passing that will be widely
used for writing message-passing programs. MPICH is a high performance and widely portable implementation of the Message Passing Interface (MPI) standard. MPICH and its derivatives form the most widely used
implementations of MPI in the world.

### 1.2 N - Gram

N-gram is a method to find the count of the repetitions in a sequence. Using Latin numerical prefixes, an n-gram
of size 1 is referred to as a ”unigram”; size 2 is a ”bigram”; etc. The items will be examined can be phonemes,
syllables, letters, words or base pairs according to the application. The n-grams typically are collected from a
text of speech data.

## 2. Method

I have used the Master-Slave model on my project. The steps of the algorithm given on the following; The role of
The slave is computing frequencies of bigrams of their interval. When a slave completes its job, it sends results
to the master node. The role of Master is combining the results as slaves send. After each node ends its job,
Master print outputs to the console and exports outputs to the file.

## 3. Implementation

### 3.1 Arguments

The program needs two parameters. One for the input file path. It is the path of the file which contains the text that.
The program finds 2-grams (bigram) in the given input file and shows frequency analysis. The other argument is
the output file path. The outputs of the program will be exported to that file. To handle these arguments I have
used getopt. In arg.h header you can find the related method to handle arguments.

### 3.2 Algorithm

After the input and output files defined program load input file content into a global buffer that all nodes can
access it. Master has a 2D array which represents frequencies of each valid characters for my system. And also,
for each worker node, a 2D array declared and send via MPIIrecv method and starts a non-blocking wait with
MPIWait method. Then each worker node starts computing frequencies and stores results on its 2D array.
When a worker node completes its job, sends its 2D array that contains the computed frequencies to the Master
node via MPIIsend method. Then Master node combines the results and exports them to the file given as output
file argument. There are some challenges in the project. Let’s see them on following. Challenges

#### 3.2.1 Parallelization

The first challenge is computing the frequency values in a parallel manner. I decided to divide the data into
intervals. Each worker node is responsible for its interval. They start at its rank - 1 and increment by the number of nodes - 1. Let’s see an example case. Assume I have three nodes (one master - two slaves). The data must be divided for two slave nodes.

```
x: the first character of bigram
y: the second character of bigram
xi: index of character x
yi: index of character y

Node 1
Step 1: (xi, yi) = (0, 1)
Step 2: (xi, yi) = (2, 3)
Step 3: (xi, yi) = (4, 5)

Node 2
Step 1: (xi, yi) = (1, 2)
Step 2: (xi, yi) = (3, 4)
Step 3: (xi, yi) = (5, 6)
```

#### 3.2.2 Storing Results

Another challenge is storing frequency values in a structure. First, I thought about storing them in a map. Later
on, I decided to store it in a 2D array. The row and column index values of the array represent a character.
For the characters in the alphabet, there are 26 character exists. So my array at least has size 26x26. Of course,
if you could some extra characters that number may increase. The next challenge is mapping ASCII characters to
that index values. To do that I first map alphabetic characters into index values. The possible cases are given below;

```
c: Ascii code of the character
p: (1 to EXTRACHARACTERS.length)

Case a)For a-z (97−122) :c− 97
Case b)For A-Z (65−90) :c+ 32− 97
Case c).Extra Characters : (122 +p)− 97
```

#### 3.2.3 Combining Results

The combining part of each result computed by worker nodes is another challenging part of the project. After
each result combined under the master node, the results must be sorted. To do that, I map 2D array into a 1D array and sort it with using the BubbleSort algorithm. The challenge here is mapping 2D array to 1D array. As discussed above, row and column indexes represent characters. If I map the array directly, then I lose which
frequency belongs to which characters. So I defined a struct named bigramt which consists of first and second characters
and the frequency value of these two characters. Then I store the 1D array of bigramt.
The final step of the program is iterating over each bigramt in 1D array and print outputs to the console and
write outputs to the file.


## 4. Summary of Implementation

- In project.c; I implement the master-slave model on nodes. I have used MPI for communication between
 master and worker nodes.
- In arg.h header; I handle the arguments given to program with getopt.
- In readfile.h header; I implemented the file reading operation method.
- In writefile.h header; I implemented file writing operation, sorting and other helper methods. I defined
 a struct to store bigram chars and its frequency value to not lose while sorting.
- In filter.h header; I have used the intervals in ASCII table to store my bigram results as indexes. I map
 ASCII characters to indexes with asciitoindex method, indexes to ASCII characters with indextoascii method. I mapped uppercase characters to lowercase characters.
- In Makefile; I defined clean, build and run operations.
 - Build operation compiles the code with mpicc and creates executable under build directory.
 - Run operation runs the executable with mpirun with default 4 processes. The number of processes can assignable during make command as a parameter. Input and output file names are defined input.txt
 and output.txt as default.
 - Clean operation cleans the build directory.
- I have used built-in MPIWtime method to calculate spent times on each operation. Root node spent
 time, Worker nodes spent times, File read operation spent time and Total spent time calculated.

