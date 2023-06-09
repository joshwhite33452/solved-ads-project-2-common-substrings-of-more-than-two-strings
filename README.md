Download Link: https://assignmentchef.com/product/solved-ads-project-2-common-substrings-of-more-than-two-strings
<br>
<span class="kksr-muted">Rate this product</span>

Project 2

Common Substrings of More Than Two Strings

This project considers the problem of determining the longest common sub-strings of at least d distinct strings. The delivery deadline for the project code is June 2nd, at 17:00.

Students should not use existing code for the algorithms described in the project, either from software libraries or other electronic sources.

It is important to read the full description of the project before starting to design and implement solutions.

Students should deliver a working implementation to the mooshak system.

1 Longest substrings

The challenge is to implement the suffix tree algorithm to compute the size of the longest substrings that are common to at least d distinct strings. An algorithm for this problem is explained in section 7.6 of Gusfield’s book [1].

The algorithm runs in two phases. First it builds a generalized suffix tree, for the set of strings. This is achieved in linear time using Ukkonen’s algorithm. In the second step the tree is traversed, possibly with a DFS, and information about the set of distinct strings that contain a certain substring is computed, by performing unions of sets of string identifiers whenever such a set contains at least d distinct strings and its string-depth is compared to the longest known value. In fact a slight improvement can be done in this step, because we are interested in the complete table of strings sizes for all the possible values of d.

1

The input will consist of a set of DNA sequences, hence the underlying alphabet will be A,C,T,G.

In this project we do not require a naive implementation, but students are advised to implement one for debugging purposes.

1.1 Description

The input will consist of a set S = S1,S2,…,Sk, with k strings. The total size of the strings is m. The most naive solution for this problem compares every substring of Sj with every suffix of Si until there is a mismatch, or one of the strings ends. This algorithm would require O(m4) time, although this is a rough upper bound. Still it is a good algorithm for debugging.

A more efficient version could use the Boyer-Moore pre-processing, or adapt the Knuth-Morris-Pratt algorithm and obtain O(m2) time. Using suf- fix trees we can obtain the better bound of O(km).

Hence the first step must be constructing the generalized suffix tree. To obtain the previous bound this must be done efficiently, in linear time, by using Ukkonen’s algorithm. This algorithm is fairly challenging to imple- ment and constitutes a major part of this assignment. Particularly because generalized suffix trees require some extra issues. In implementing this data structure students are advised to use the “sentinel” programming technique. This technique consists in augmenting the data structures to avoid writing more code. The idea is that to avoid producing code that contains many conditional selections, if statements mainly, we add extra content in the data structure. An important example for suffix trees is the suffix link of the root node. The general definition does not apply to this case, therefore it is formally undefined. For programming purposes it is best to add a sentinel, that is the suffix link of the root. It should also be possible to Descend from this node to the root with any letter.

A fair amount of information must be stored at each node. A recom- mended structure, in C, is the following:

typedef struct node* node;

struct node {

<pre>  int Ti;  int head;</pre>

<pre>  int sdep;  node child;</pre>

/**&lt; The value of i in Ti */ /**&lt; The path-label start

at &amp;(Ti[head]) */ /**&lt; String-Depth */ /**&lt; Child */

2

};

<pre>node brother;node slink;node* hook;</pre>

/**&lt; brother *//**&lt; Suffix link *//**&lt; What keeps this linked? */

Most fields are explained by the comments. This structure can be used both to represent internal nodes and to represent leaves. For leaves a smaller structure could be used, thus saving overall space, but such optimization is not critical and therefore not recommended. The trickiest field in the previous structure is the hook, which is a pointer to a pointer to a struct. If you find this confusing you can instead use a doubly linked list for the brother, i.e., store two pointers instead of one, and a father pointer that points to the node that is the father of this one. This makes inserting nodes into the tree simpler to implement.

The hook version requires less code, and space. A node struct is pointed to by only one child or brother pointer, the idea of the hook is to keep a pointer to that pointer. With this information it is also simple to update the structure for removals, and moreover the code does not depend on whether the reference was from a child pointer or a brother pointer. Still the hook information must be properly updated, for example if p-&gt;b becomes the child of node x we would use the following code :

<pre>      x-&gt;child = p-&gt;b;      p-&gt;b-&gt;hook = &amp;(x-&gt;child);</pre>

A simple outline of Ukkonen’s algorithm is the following, slightly different from section 6.1:

<pre>j = 0;while(j &lt;= ni[i])</pre>

{while(!DescendQ(p, Ti[i][j]))

{AddLeaf(p, i, j); SuffixLink(p);

<pre>      }    Descend(p, Ti[i][j]);    j++;</pre>

}

This code is used to insert text Ti[i] into the generalized suffix tree, there- fore Ti[i][j] represents the character at position j, the size of this text is stored in ni[i]. For every character Ti[i][j] that need to be inserted we

3

create nodes and follow suffix links until we find a position where it is possi- ble to descend by the letter Ti[i][j]. The variable p represents a point, i.e., a position between two letters of a given branch. The following structure is a possible implementation of this concept:

typedef struct point* point;

<pre>struct point{</pre>

<pre>  node a;  node b;  int s;</pre>

/**&lt; node above */ /**&lt; node bellow */ /**&lt; String-Depth */

};

The DescendQ function returns true when it is possible to descend from p with Ti[i][j]. Then DescendQ function actually descends with the given letter. The AddLeaf function creates a new leaf and inserts it into the tree, in this process it might also be necessary to create a new internal node. The SuffixLink alters the point p, so that it points to the suffix link of the current point. This function must implement the skip/count trick of section 6.1.3. An important detail to be mindful about is that the AddLeaf function should not alter p, the reason for this is that the SuffixLink uses the value in the slink field, and for recently created nodes this value is not yet available. In fact assigning this value, for a new node, is another crucial detail, it should be set by SuffixLink if p is an internal node, otherwise it should be set to the next new internal node, created on the next call of AddLeaf, this issue is referred in Lemma 6.1.1.

There is also an important trick to implement generalized suffix trees, again focusing on reducing the amount of conditional code. In a generalized suffix tree all terminators should be different symbols. If we where to use real characters this would limit k to 255 − 4, which is too small, particularly for the larger tests. Moreover handling which characters are terminators, avoiding the DNA letters could be messy. Alternatively we could use ‘ ’ as a terminator and add code so that the comparison of two terminators depended on Ti field of the node struct. There is a simpler and cleaner approach, we can use a ‘ ’ for all terminators as before, except for the Ti that is currently being inserted into the tree, for that string we use ‘1’. Once the string gets inserted into the tree we switch the terminator to ‘ ’ and proceed with the next text. In essence the previous code we showed can be inserted the middle of the following code:

i = 0;

4

while(i &lt; k) {

Ti[i][ni[i]] = ‘1’; …

Ti[i][ni[i]] = ‘ ’;

i++; }

/**&lt; Force diferent terminators*/

After building the generalized suffix tree the second step is easier. At each node we store a list representing the set of values of i for which some suffix T[i] corresponds to a descendent of the node. These lists can be computed in the finishing time of a DFS visit to the node, by merging the lists of the children of a node. To merge these lists you can use an array, of size k, that indicates for index i if a suffix of T[i] is already know to be in the currently merged list. This allows us to avoid duplicating these indexes, but reseting the modified entries back to false must be done carefully after each use.

Once we know the size of each list it is possible to search for the node with the largest sdep value for which the corresponding list contains at least d elements. Note that we are interested in all the k possible values of d. By using an array, of size k, it is possible to traverse the suffix tree only once, instead of k times. Thus reducing the time, of this particular operation, from O(kn) to O(k + n).

Finally considering memory allocation it is best to make few calls to malloc and allocate big chunks of memory. In a naive implementations these functions can easily occupy 25% of the overall time. All the nodes of the suffix tree can be allocated, after reading the texts, i.e., knowing m, since there can be no more than 2m + 1 nodes. This chunk can be reduced after finishing the tree, but it is not necessary for the memory limits in the mooshak system. The elements of the stacks for the DFS can be implemented with linked lists, likewise it is possible to allocate all in one step. Keeping track of which elements are in use and which are not is straight forward.

1.2 Specification

To automatically validate the index we use the following conventions. The binary is executed with the following command:

<pre>   ./project &lt; in &gt; out</pre>

The file in contains the input commands that we will describe next. The output is stored in a file named out. The input and output must respect

5

the specification below precisely. The output file will be validated against an expected result, stored in a file named check, with the following command:

<pre>   diff out check</pre>

This command should produce no output, thus indicating that both files are identical.

The format of the input file is the following. The first line contains a single integer, k, i.e., the value indicates the number of strings that follow.

Each of the following k lines starts with a number, mi. This number indicates the size of the string that follows. Then there is a white space ‘ ’ and finally a string with mi characters from the ‘A’, ‘C’, ‘T’, ‘G’ alphabet.

The input contains no more data.

The output should consist of a single line containing the sizes of the longest substring that exists in at least d different strings. For all the possible values of d ranging from 2 to k. Every pair of values is separated by a single space ‘ ’. There should also be a space before the newline character.

1.3 Sample Behaviour

The following examples show the expected output for the given input. These files are available on the course web page.

input 1

556 TTACCGCCGCATCGGGCTGAGGGGCTATGGGCACGAGACCGCTTTCAATGCATCTT13 ATCTTTACTTTCA87 ATCCCTAGTCACGCCGTCATTCGTTATAACCTATATAGGACTGGTGGGGGTATTGTGCGGACCGCTTTCAATGCATCT 70 TTTCATATCCCTAGTCACGCCGTCTTTACTTTCATATCCCTAGTCACGCCGTGAGGACGTGCGGACCGCT87 TCAATGCATCTTGATCTTTACTTTCATATCCCTAGTCACGCCGTCGTGCGCTTCTGTAGAACAGAAATATTGGTCTCT

output 1

30 18 6 6

input 2

66 GGCCGC57 GTAGGGCCTTCAGTGACTATAGGATGGGGGAATATTCGGCTTCGAAGTTATGAACTA172 GGCATGGTGACAGTCCGCCGATATTCGCGATTCCAGAGCTATTTTGGACGTCGTTCCGGAGATCCCCTTGGCGGGTC 197 TGCCTACGCTGACTGTTCAAGCGGGATCAAGCCCGTTTAACACTATTCCGCACTATATCGTTCTTCTTCAGTGGAGA

6

<pre>11 GGCAGTTCGGC15 GACCGTTATCCAAGT</pre>

output 2

55 9 4 3 2

input 3

769 GATGCCAGTGACCAGAATACATCATTGTAGAGGCGCGAGATACAGCAGCCGAGGAATGCAAGACCTTTT35 GCAGAAAGACCGGGCCTATCTTCCGTATTGCGGCA182 ATGACTCAAAGACAACTTCCACAGGATGATAAAAGCCAAAGCCTCCCTGCAAGCTGGACGGTTCCTGGCTAGGACGC 278 TATCTTCCGTATTGCGGCAAATGACTCAAAGACAACTTCCACAGGATGATAAAAGCCAAAGGTTTGGTTACACGAAG 49 CGGTGCCGAGAGTTCCTATCTTCCTATCTTCCGTATTGCGGCAAATGAC55 CAAAGACAACTTCCACAGGATGATAAAAGCCAAAGACACACAGGTACAAGGTACC2 AT

output 3

41 35 6 5 3 2

input 4

9295 AGTAGTCTACCGTAGGCGTCCGACCCGCAATTATCGGACTGTGATGAGACTGTGGTAATTCACAGATCCGTTGAAGT 76 TGGTTCGGAGCCGTACAAAATATTTAAAGTAAACTGCACTATACTACACATGCCCTCTGGTTCCATCTAGCTAAAC 116 ATCGATTAGGCGACACGGTCTCCGCTTCGGCCCGAATGTGTAGGCCGTCAAGGTAAGTGCCCAACCTTACAGTTGGG 15 TGTGATGAGACTGTG61 TAATTCACAGATCCGTTGAAGTGACGTGGGGGTGTCAACTTGGTAACATCGTCGTAAGAAA187 AAAACACGGCGTTGTACAGGACGTGGGAAGGGGGGTCAGATGATTTATATGCCATTTACAGAAATTTACGAACTCAT 195 GACTGTGGTAATTCACAGATCCGTTGAAGTGACGTGGGGGTGTCAACTTGGTGATGTGCCCAAACTACTCCTTTCGG 9 ACAGAGTCT26 CCGTAAATATCGTTATCATCCACAAG

output 4

52 44 8 5 4 3 3 2

input 5

11305 ATATATAGGGTTGTAAGATGCCCCTACATACCCCTCCGAAATCGTACTCGGAAAATGCTCGGCGGTCCGCTCGATAT

7

31 TAACGATGCCTGTGCTAAAGACCCATGAGCT1C149 GGGACCTTTAGCTCGGCGCTCAGTCTGGCTGGCCCGGTGATTCGGATTCCGGTAGCCCGCATCTTGGTGCGGTCACG 176 CCTTGCATCCTGATCCATTAATCAGACAGTCGCCCCGGATAGCGCTGGTGGGGCGGAGTAAACACTCCTAACCGATA 67 TCGCGTACGGCTAAATGGTGTTCCTTACATTACCACGCAAGCCCCCGAAATCGTACTCGGAACGAAA253 CGTACTCGGAAAATGCTCGGCGGTCCGCTCGATATCTATGATCGGCTGGGGGTTGGTCTCGGCATTTTACATGATAC 125 AGCCTCAACAAATCACCCTCCCAACGAAATCGTACTCGGAAAATGCTCGGCGGTCCGCTCGATATCTATGATCGGCT 88 TCTGCGCGACCATCCGAAATCGTACTCGGAAAATGCTCGGCGGTCCGCTCGATATCTATGATCGGCTGGGGGTTGGTC 146 GTCTACATGGCCTGAACGCTGAGGTCCTAAGCCACCCCACATATCATGACGGCCTGAATGTGAGTGCAATCCAGATA 93 CACTTAAACGCCGAACAGCCGCACTGTCCTGGTACTTATGACCTAAATGGCTGATATATCATCGCCGGACCCCCCCCC

output 5

<pre>77 75 74 17 7 5 4 3 3 1</pre>

input 6

1376 CATTTCGTGTCTGGCCCTCCACCGATTGTAAGTAGGTACAGCCCCGAACGGCATCAACGTTGGTTTTTCGGCACCC 4 AAAT471 AACGCTGGGGCCAACTTGGCGTGCACCGATTCTGGTGACCTCGCCCGAAATTCTGCCTATATCAAGACCATCGCTAC 207 GGGTAAATGAGCCCGCGGTATGACTAGTAAGATGTCCGGTTCCATATTAGAACGTAATTGTATATTTGATATTATAG 69 GCTATTCACTCGCAACTTGTCGGACATTAGCGGAGTAGTCCCGGCTCAAGTCCTCGAGACAAAAGAGGC365 TCGATCTCTGTGGTTTTGTGAGCACGTTAATTGTCAGGGTGGCCAGGTTGTATCTGGCGTCAATTTAAGACTGCCCC 224 ACAAAGGTGGCGACTGCGGTCTCAGGTGATCAACATGACGTGCAAGGTCACAGCTGGATATAGCCACGCCTCGCCCC 30 TAACTAACATGGTCTTTTGGATCTACACCA25 ATGACATTAACAGGAGTTATCCGTG6 ACCTAG223 GTTATAGACAACCTAAGTGTATCTCCCCACTTATACTGAATCCCGCAATCACAACAAGTTCATTAAAGGTGGCGACT 111 CTCGCGCGTGTGGAATTTTGAGGACCAGTAAAAGGTGGCGACTGCGGTCTCAGGTGATCAACATGACGTGCAAGGTC 287 AATTTCGGTAGGCAGATTCTTTACACCTTATTCCTGTGGGTTACCAGAATCATTATGAGATCGTTTGGCCTACATCG

output 6