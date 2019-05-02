---
layout: post
title:  "How to Make a Hexdump in C++"
date:   2019-03-02 18:42:15 -0300
categories: programming
---
A [hexdump][wikipedia-hexdump] is a hexadecimal view of computer data. Today, I'm going to show you
how to make a file hex dumper in C++!

For this tutorial, I'll assume you know how to compile a C++ program, so that I can
jump right to the point.

The final result can be seen in this [github project][git-hexdump]

<h2>So, what exactly do we see in a hexdump?</h2>
In most hexdumpers, we view three kinds of information at once:
1. address/offset (in hex)
2. value of the bytes at that address (in hex)
3. an [ASCII][ascii-table] character for each byte shown

<h3>Example:</h3>
test.txt:
```
This is just a test file...

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla ultrices, leo vitae 
vestibulum rhoncus, velit mi interdum quam, in maximus tortor ligula eu ex. Donec laoreet 
quam a orci laoreet tincidunt.
```

Program execution:
```
$ hexdump test.txt

Offset    00 01 02 03 04 05 06 07  08 09 0a 0b 0c 0d 0e 0f   ASCII
00000000  54 68 69 73 20 69 73 20  6a 75 73 74 20 61 20 74  |This is just a t|
00000010  65 73 74 20 66 69 6c 65  2e 2e 2e 0a 0a 4c 6f 72  |est file.....Lor|
00000020  65 6d 20 69 70 73 75 6d  20 64 6f 6c 6f 72 20 73  |em ipsum dolor s|
00000030  69 74 20 61 6d 65 74 2c  20 63 6f 6e 73 65 63 74  |it amet, consect|
(...)
```

In the example above, the offset 0x00000000 is the beginning of the file.
In the second part, the first 16 bytes are shown (0x00 to 0x0f), then,
in the next line (offset 0x00000010), the next 16 bytes at that offset are
shown (0x10 to 0x1f) and so on...

The third part is, again, just an ASCII character for each byte. As
we can see:
* the byte at [0x14] = 0x66 = 'f'
* the byte at [0x15] = 0x69 = 'i'
* the byte at [0x16] = 0x6c = 'l'
* the byte at [0x17] = 0x65 = 'e'

<h2>But what is <i>hexadecimal</i>?</h2>
[Hexadecimal][wikipedia-hexadecimal] is a numerical base that has 16 possible digits,
while the decimal system, which we commonly use and know, has 10 possible digits.

```
Decimal: 00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 ...
Hexadec: 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f 10 11 12 ...
```

<h2>Setting up headers and typedefs</h2>
```cpp
#include <iostream>	// cin and cout
#include <fstream>	// ifstream
#include <iomanip>	// setfill and setw
#include <string.h>	// strcpy

typedef unsigned long ulong;
typedef unsigned int  uint;

using namespace std;
```

<b>Friendly tip:</b> search about these headers :)

Also, `using namespace std` is a [bad practice][namespace-std], but I'm keeping
it just for the sake of the simplicity.

<h2>Main function</h2>
Let's first declare the variables we'll be using and
set the basic program structure:

```cpp
int main(int argc, char *argv[]) {
	ifstream ifs;		// Input file stream
	ulong offset = 0;	// Address offset
	filebuf* pbuf;		// File stream buffer
	size_t size;		// File size
	char* buffer;		// Buffer for the file data
	char fileName[FileNameBufferSize];

	// Read arguments:
	(...)

	// Read the file:
	(...)

	// Main loop:
	(...)

	// Clear memory:
	ifs.close();
	delete[] buffer;
	
	return 0;
}
```

<h3>Read arguments</h3>

With this code we can pass a file as an argument and dump it or
just execute the program and then indicate a file:
* `$ hexdump <filename>`
* ```
$ hexdump
> File to dump: <filename>
```

```cpp
// Read arguments:
if (argc == 2) {
	strcpy(fileName, argv[1]);
} else if (argc == 1) {
	cout << "File to dump: ";
	cin.get(fileName, FileNameBufferSize);
} else if (argc > 2) {
	cout << "Please, provide only one file.\n";
	return 1;
}
```

<h3>Read the file</h3>

Maybe the most complex part of the program, but we are doing
the following:
1. open the file
2. get the file stream buffer and determine it's size
3. allocate memory based on the size
4. point to the beginning of the file

```cpp
// Read the file:
ifs.open(fileName, ifstream::binary); 		// Open file in binary mode
pbuf = ifs.rdbuf();				// Get pointer to associated buffer object
size = pbuf->pubseekoff(0, ifs.end, ifs.in); 	// Get file size using buffer's members
buffer = new char[size];			// Allocate memory to contain file data

pbuf->pubseekpos(0, ifs.in);			// Set internal position pointer to absolute position
pbuf-> sgetn (buffer, size);			// Get file data

cout << hex << setfill('0');			// Set output to hexadecimal
```

<h3>Main loop</h3>
Here we set the main loop: for each <i>BytesPerLine</i> bytes of the file,
we show it's info.

```cpp
// Main loop:
showHeaders();

// loop for each line
for (ulong bytesRead = 0; bytesRead < size; bytesRead += BytesPerLine) {
	uint bytesLeft = size - bytesRead;
    if ( bytesLeft > BytesPerLine )
		bytesLeft = BytesPerLine;

	showOffset(offset);
	showHexCodes(buffer, offset, bytesLeft);
	showCharacters(buffer, offset);
	
   	offset += BytesPerLine;
}
```
<h2>Methods to show info</h2>

```cpp
void showHeaders() {
	cout << "Offset   ";
	
	for (uint i = 0; i < BytesPerLine; ++i) {
		if(i % 8 == 0) cout << ' ';
		cout << setw(2) << i  << ' ';
	}
	
	cout << "  ASCII\n";
}

void showOffset(ulong offset) {
	cout << setw(8) << offset;
}

void showHexCodes(char* buffer, ulong offset, ulong bytesLeft) {
	for(uint i = 0; i < BytesPerLine; i++) {
		if(i % 8 == 0) cout << ' '; // Little space every 8 bytes
		if(i < bytesLeft)
			cout << ' ' << setw(2) << (unsigned)buffer[i + offset];
		else 
			cout << "   ";
    }
}

void showCharacters(char* buffer, ulong offset) {
	cout << "  |";
	;
	for(uint i = 0; i < BytesPerLine; i++) {
		if( (unsigned)buffer[i + offset] < AsciiThreshold )
			cout << '.';
		else
			cout << buffer[i + offset];
	}
	
	cout << "|\n";
}
```

<h2>Done!</h2>
So that's pretty much it for this tutorial. Go ahead and
try to dump a file. You will note, however, that there is
no control of how much info is displayed per dump.

To do that, we would need a "page" system -- which is not
that hard ;) -- or a more graphical solution, like the
<b>ncurses</b> library.

In the future, I will post another tutorial about how to make
a hex editor, so be sure check here every now and then!

[ascii-table]: http://www.asciitable.com/
[wikipedia-hexdump]: https://en.wikipedia.org/wiki/Hex_dump
[wikipedia-hexadecimal]: https://en.wikipedia.org/wiki/Hexadecimal
[namespace-std]: https://stackoverflow.com/questions/1452721/why-is-using-namespace-std-considered-bad-practice
[git-hexdump]: https://github.com/zetamek/hexdump
