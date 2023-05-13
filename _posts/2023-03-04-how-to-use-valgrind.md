---
title: How to Use Valgrind
date: 2023-03-04 21:00:00 -0800
categories: [software, tutorial]
tags: [valgrind, cpp]    # tag names should always be lowercase
---

# How to Use Valgrind

In this tutorial, we are going to learn when to and how to use Valgrind

## Motivation

There might be time when we need to allocate memory when we code in C++. It is imperative that we free those memory blocks otherwise we will have memory leaks. [Valgrind](https://valgrind.org/) is a tool that can detect memory leaks, deallocation errors, etc. In this tutorial, we will learn how to use Valgrind to detect memory issues.

## Installation

Valgrind can be installed using the terminal:

```shell
sudo apt-get install valgrind
```

## Usage

Firstly, we have to compile our source code to binary code:

```shell
g++ <filename>.cpp -o <output_binary> -g
```

Do not forget to use the *-g* flag.

Secondly, we run Valgrind and the binary code:

```shell
valgrind ./<output_binary>
```

Finally, we will see some diagnosis from Valgrind on our terminal.

## Error #1: Off by One

In this example, the array is of size 5 but we try to access *arr[5]*.
The following is an example:

```cpp
#include <iostream>
#include <array>
int main()
{
    std::array<int, 5> arr({0, 1, 2, 3, 4});

    // a for loop that is off by one
    for (auto i = 0; i <= 5; ++i)
    {
        std::cout << arr[i] << "\t";
    }
    std::cout << "\n";

    return 0;
}
```

When we run it, Valgrind detects that we try to access an uninitialized value and gives us a warning:

*Conditional jump or move depends on uninitialised value(s)*, and
*Use of uninitialised value of size 8*.

Hence, we know that we should go back and fix our code such that we only access elements from *arr[0]* to *arr[4]*.

## Error #2: Not Delete

In this example, we create two arrays on heap memory but we do not free them using the *delete* keyword.

```cpp
#include <random>   // random_device mt19937 uniform_int_distribution
#include <algorithm>    // generate

int main()
{
    auto N = 1 << 4;
    int* a = new int[N];
    int* b = new int[N];

    // create a seed for the random number engine
    std::random_device rd;
    // create a standard mersenne_twister_engine seeded with rd()
    std::mt19937 gen(rd());
    std::uniform_int_distribution<> distrib(1, 6);

    // the lambda function:
    // use distrib to transform the random unsigned int
    // to an int in [1, 6]

    // initialize with values from the lambda function
    std::generate(a, a+N, [&](){ return distrib(gen); });
    std::generate(b, b+N, [&](){ return distrib(gen); });

    return 0;
}
```

When we run it with Valgrind, it detects that we try to access an uninitialized value and gives us a warning:

```bash
>HEAP SUMMARY:
>in use at exit: 128 bytes in 2 blocks
>total heap usage: 4 allocs, 2 frees, 73,856 bytes allocated
>LEAK SUMMARY:
>definitely lost: 128 bytes in 2 blocks
```

Furthermore, we can add an additional flag *--leak-check=full* to see a more detail diagnosis.

It gives us where we allocate heap memory blocks and the culprit:

```bash
>64 bytes in 1 blocks are definitely lost in loss record 1 of 2
>
>at 0x484A2F3: operator new[](unsigned long)
>
>by 0x10A559: main (error_not_delete.cpp:8)
```

Hence, we know that we should free *a* and *b* at the end of our code.

```cpp
delete a;
delete b;
```

## Error #3: Double Free

In this example, we *free* the variable *a* twice. This will result *core dump error*.

```cpp
#include <random>   // random_device mt19937 uniform_int_distribution
#include <algorithm>    // generate

int main()
{
    auto N = 1 << 4;
    int* a = new int[N];
    int* b = new int[N];

    // create a seed for the random number engine
    std::random_device rd;
    // create a standard mersenne_twister_engine seeded with rd()
    std::mt19937 gen(rd());
    std::uniform_int_distribution<> distrib(1, 6);

    // the lambda function:
    // use distrib to transform the random unsigned int
    // to an int in [1, 6]

    // initialize with values from the lambda function
    std::generate(a, a+N, [&](){ return distrib(gen); });
    std::generate(b, b+N, [&](){ return distrib(gen); });

    delete[] a;
    delete[] a;
    delete[] b;

    return 0;
}
```

Valgrind is also capable of detecting this issue.
It will give us:

```bash
>Invalid free() / delete / delete[] / realloc()
>at 0x484CA8F: operator delete[](void*)
```

Hence, we should only free the variable *a* once.

## Summary

So far, we learn what Valgrind can help us and we should keep in mind that we do not make those mistakes when we code in the future.

Happy coding 😀

## Acknowledgement

Those are the sources where I learn:
1. Valgrind: https://en.wikipedia.org/wiki/Valgrind
1. CoffeeBeforeArch: https://www.youtube.com/watch?v=ykXZdYs9cOw&ab_channel=CoffeeBeforeArch
