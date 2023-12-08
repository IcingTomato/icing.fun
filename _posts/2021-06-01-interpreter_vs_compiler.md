---
layout: post
title:  "Interpreter Vs Compiler : Differences Between Interpreter and Compiler"
tags: programme c-language en-us
---

In this article, you will learn the differences between interpreters and compilers.

We generally write a computer program using a high-level language. A high-level language is one that is understandable by us, humans. This is called **source code**.

However, a computer does not understand high-level language. It only understands the program written in **0**'s and **1**'s in binary, called the **machine code**.

To convert source code into machine code, we use either a **compiler** or an **interpreter**.

Both compilers and interpreters are used to convert a program written in a high-level language into machine code understood by computers. However, there are differences between how an interpreter and a compiler works.

## Interpreter Vs Compiler

|  Interpreter   | Compiler  |
|  ----  | ----  |
| Translates program one statement at a time.  | Scans the entire program and translates it as a whole into machine code. |
| Interpreters usually take less amount of time to analyze the source code. However, the overall execution time is comparatively slower than compilers.  | Compilers usually take a large amount of time to analyze the source code. However, the overall execution time is comparatively faster than interpreters. |
| No Object Code is generated, hence are memory efficient.  | Generates Object Code which further requires linking, hence requires more memory. |
| Programming languages like JavaScript, Python, Ruby use interpreters.  | Programming languages like C, C++, Java use compilers. |

## Working of Compiler and Interpreter

![](http://icing.fun/img/post/2021/06/01/compiler-interpreter-working.webp)

