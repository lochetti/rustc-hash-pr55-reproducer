Project to reproduce the problem with mass colision of hashs that is being discussed at https://github.com/rust-lang/rustc-hash/pull/55

The project is basically a "macro loop" calling a diesel's macro that creates a database table representation. We noticed that calling diesel's macro 512 times is enough to start to see the degradation in compile times because it generates more than 2^20 elements (`GenericArgs interner`) at the hash table.
The current state of the code is calling 2048 times the macro, but as one could check at the code, it is not hard to change the code to call less times (and see the compile time behaviour and the `GenericArgs interner` count for differente scenarios).

I have tried the reproducer code with 128, 256, 512, 1024 and 2048 table creations and got the following results:

| # of tables created | # of GenericArgs interner - Rust 1.83 | Compile Time (seconds) - Rust 1.83 | # of GenericArgs interner - Rust 1.84.1 | Compile Time (seconds) - Rust 1.84.1 |
| ------ | ---------- | ----- | ---------- | ------- |
| 128    | 470544     | 35    | 471096     | 37      |
| 256    | 935184     | 60    | 936248     | 61      |
| 512    | 1864464    | 112   | 1866552    | 134     |
| 1024   | 3723024    | 223   | 3727160    | 415     |
| 2048   | 7440144    | 452   | 7448376    | 1282    |

You can notice that the # of GenericArgs interner is pretty similar between 1.83 and 1.84, but the compile time starts to degenerate quickly after 1024 tables being created (because the number of elements at the hash table greatly exceeds 2^20)
