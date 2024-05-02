# 16-Bit processor with unified cache memory

This is a college work developed together with *[Luigi Eliabe](https://github.com/Luigi-Eliabe)*, and other classmates, that aimed to develop a 16-bit data processor with a unified cache memory.

## Cache archtecture

The archtecure designed for the cache is a two-way associative set structure, that means this has 4 sets with 2 line in each set. The cache line has 4 16-bit words, Resulting in a cache size of 64 Bytes.

Futhermore, the writing policy developed was the Write-Back that consists to update the main memory's data only when the cache line be overrided. And, as replacement algorithm, It was developed the Least Recently Used, with a modification:

- The line to be override will be always the older data holder line, independently it was last used or not.
