---
layout: post
title:  "How to Make a Brainfuck Interpreter in C++"
date:   2019-02-28 18:42:15 -0300
categories: programming
---
Brainfuck is an esoteric programming language created to... well, you know.
I thought creating an interpreter for it would be cool, so here is a tutorial!

I will explain each step of the thinking process.
The final result can be seen [in this gist][link-gist-bfcompiler].

Also, I'll assume you know how to compile a C++ program so that i can jump right to the point.
I like to use G++ on Linux, as it's simple like that: `g++ -o <output> <source>.cpp`.

You can try Brainfuck online [in this cool page][link-bfvisualizer] by fatiherikli.

<h2>So, how the f* does Brainfuck work?</h2>
Brainfuck has a tape of 30.000 bytes, where the initial value of each byte is zero.
Also, there is a pointer, which begins pointing to the first byte of the tape:
```
[000][000][000][000][000][000][000][000] ...
  ^
```

There are 8 operators:
```
+ -
< >
, .
[ ]
```

`+` and `-` increase or decrease the value of the current byte by one.

`<` and `>` move the pointer left or right.

`,` reads the user input and stores it on the current byte.

`.` prints [ASCII][link-ascii-table] value of the current byte.

`[` and `]` begin and end a loop, respectively.

Any other characters, including spaces, are not considered at all.

<h3>Example #01:</h3>
Input:
```
>>> +++++ +++++ +++++ +++++ +++++ +++++ +++ .
```

Output:
```
!
```

State of the tape at the end:
```
[000][000][000][033][000][000][000][000] ...
                 ^
```

<h3>Example #02:</h3>
Input:
```
+++ [>++<-]
```

State of the tape at the end:
```
[000][006][000][000][000][000][000][000] ...
  ^
```

First, it sets the first byte's value to 3.
Then, it starts a loop: go right, increase 2, go left, decrease 1.

The loop stops when the current byte at `]` is zero.

<h2>1. First things first</h2>
We will need the `iostream` header to get input with `std::cin()`.
The `stdio.h` header is not necessary in some C++ compilers (like g++), though
it will be required as we'll use `putchar()` and `getchar()`.

Now, we need a tape of 30.000 bytes where all of them are initialized to zero. Also,
we need a pointer pointing to the first byte of the tape:

```cpp
#include <stdio.h>
#include <iostream>

unsigned char tape[30000] = {0};
unsigned char* ptr = tape;
```

Both the tape and the pointer are `unsigned` because we don't have negative bytes in the tape.

The size of a `char` in C/C++ is always 1 byte.



<h2>2. The main() function</h2>
Now, we create an input string of 1024 bytes, get the input, `interpret()` the string,
and print a new line. Nothing really fancy.

```c
int main()
{
	char str[1024];
	std::cin.get(str, 1024);
	
	interpret(str);
	putchar('\n');

	return 0;
}
```

Now, we must implement the `interpret()` function...

<h2>3. The interpret() function</h2>
This function loops through each `char` of the input string and interprets it.

The `loop` variable will be used later on to handle `[` and `]`.

```cpp
void interpret(char* input)
{
	char current_char;
	unsigned int i, loop;

	for (i = 0; input[i] != 0; ++i)
	{
		...
	}
}
```

On the `for` loop, we set the current char and check it's value on a `switch`.

```cpp
for (i = 0; input[i] != 0; ++i)
{
	current_char = input[i];

	switch (current_char)
	{
		case '>':
			++ptr;
			break;
		case '<':
			--ptr;
			break;
		case '+':
			++*ptr;
			break;
		case '-':
			--*ptr;
			break;
		case '.':
			putchar(*ptr);
			break;
		case ',':
			*ptr = getchar();
			break;
		case '[':
			...
		case ']':
			...
	}
}
```

As you can see, we can use `++*ptr;` and `--*ptr` the move the pointer 1 byte left or right.
It's an unusual move if you didn't mess with pointers too much before.

<h2>4. The (confusing) loops</h2>
Don't worry if you don't understand it at first, this part is kinda tricky as we are messing with
two different pointers at the same time.
If you think enough about it, you'll get it, though :)

```cpp
case '[':
	if (*ptr == 0)
	{
		loop = 1;
		while (loop > 0)
		{
			current_char = input[++i];
			if (current_char == '[')
				loop++;
			else if (current_char == ']')
				loop--;
		}
	}
	break;
case ']':
	if (*ptr)
	{
		loop = 1;
		while (loop > 0)
		{
			current_char = input[--i];
			if (current_char == '[')
				loop--;
			else if (current_char == ']')
				loop++;
		}
	}
```

What happens is, if we find a `[`, keep looping until we find it's matching `]`. If we happen to find another
`[`, we increase the number of loops found by one and repeat the process.

The same happens for the `]`, but it seeks it's matching `[`.

<h2>And more, much more than this...</h2>
<strike>...I did it my way.</strike>
Now go ahead and make it better. Maybe try to make a loop for the user to always input a new string after the
output? What about making your own interpreter in another language?

[link-gist-bfcompiler]:	https://gist.github.com/zetamek/b4cd86a8e148b1a20af13b6c6029ef66
[link-ascii-table]: http://www.asciitable.com/
[link-bfvisualizer]: https://fatiherikli.github.io/brainfuck-visualizer/