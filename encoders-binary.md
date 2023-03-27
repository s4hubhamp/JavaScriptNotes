## Bits and Bytes

https://web.stanford.edu/class/cs101/bits-bytes.html

## Numbers

### Decimal System (Base 10)

The decimal number system is based on the ten digits 0 through 9. Using these digits, we can create any number we want by following a few rules. Firstly, each digit in a number has a value based on its position from right to left, starting from zero. The value of each position is a power of ten. Secondly, we can multiply the digit by its position's power of ten value, and then add up the results to get the final number. For example, the number 10 can be broken down into 1 times 10 to the power of 1, plus 0 times 10 to the power of 0, which equals 10.

`1 * 10^1 + 0 * 10^0 = 10`

| 10^3 (1000) |  10^2 (100) |  10^1 (10)  |  10^0 (1)  | Calculation | Result |
| :---        | :---        | :---        |   :---     | :---        | :---   |
| 0           |      0      |   1         |     0      |  10 * 1 + 1 * 0 | 10 |
| 0           |      0      |   1         |     1      |  10 * 1 + 1 * 1 | 11 |
| 0           |      0      |   3         |     9      |  10 * 3 + 1 * 9 | 39 |
| 0           |      0      |   1         |     0      |  100 * 1 + 10 * 2 + 1 * 3 | 123|
| 9           |      0      |   1         |     9      |  1000 * 9 + 100 * 0 + 10 * 1 + 1 * 9 | 9019|

### Binary System (Base 2)

Available digits are 0 and 1.

| 2^3 (8) |  2^2 (4) |  2^1 (2)  |  2^0 (1)  | Calculation | Result in Binary | Result in Decimal |
| :---        | :---        | :---        |   :---     | :---        | :---   | :---   |
| 0           |      0      |   0         |     0      | 1 * 0 | 0 | 0 |
| 0           |      0      |   0         |     1      | 1 * 1 | 1 | 1 |
| 0           |      0      |   1         |     0      | 2 * 1 + 1 * 0 | 10 | 2 |
| 0           |      0      |   1         |     1      | 2 * 1 + 1 * 1 | 11 | 3 |
| 0           |      1      |   0         |     0      | 4 * 1 + 2 * 0 + 1 * 0 | 100 | 4 |
| 0           |      1      |   0         |     1      | 4 * 1 + 2 * 0 + 1 * 1 | 101 | 5 |

### Hexadecimal System (Base 16)

Available digits are 0-9 and A-F.

| 16^3 (4096) |  16^2 (256) |  16^1 (16)  |  16^0 (1)  | Calculation | Result in Hexadecimal | Result in Decimal |
| :---        | :---        | :---        |   :---     | :---        | :---   | :---   |
| 0           |      0      |   0         |     0      | 1 * 0 | 0 | 0 |
| 0           |      0      |   0         |     1      | 1 * 1 | 1 | 1 |
| 0           |      0      |   0         |     2      | 1 * 2 | 2 | 2 |
| 0           |      0      |   0         |     A (10) | 1 * A => 1 * 10 | A | 10 |
| 0           |      0      |   0         |     F (15) | 1 * F => 1 * 15 | F | 15 |
| 0           |      0      |   1         |     0      | 16 * 1 + 1 * 0 | 16 | 16 |
| 0           |      0      |   F         |     F      | 16 * F + 1 * F => 16 * 15 + 1 * 15 | FF | 255 |


## ASCII

ASCII abbreviated from American Standard Code for Information Interchange, is a character encoding standard for electronic communication. ASCII codes represent text in computers, telecommunications equipment, and other devices. Most modern character-encoding schemes are based on ASCII, although they support many additional characters.

ASCII uses 7-bit code points to represent 128 different characters. These code points are divided into 95 printable characters, which include the 26 letters of the English alphabet (A to Z, both upper- and lower-case), the 10 digits (0 through 9), and a variety of punctuation and other symbols.

There are also 33 non-printable characters, which include control characters like carriage return and line feed, as well as various other characters that are used for things like formatting text.

## Unicode

Unicode, formally The Unicode Standard is an information technology standard for the consistent encoding, representation, and handling of text expressed in most of the world's writing systems. The standard, which is maintained by the Unicode Consortium, defines 144,697 characters covering 159 modern and historic scripts, as well as symbols, emoji, and non-visual control and formatting codes.

Unicode's success at unifying character sets has led to its widespread and predominant use in the internationalization and localization of computer software. The standard has been implemented in many recent technologies, including modern operating systems, XML, and most modern programming languages.

Unicode can be implemented by different character encodings. The Unicode standard defines three Unicode Transformation Formats (UTF), all in practice variable-length encodings, and several other encodings exist. The most common encodings are:

- UTF-8, the dominant encoding on the World Wide Web (used in over 97% of websites as of 2022)[6] and on most Unix-like operating systems, uses one byte[note 3] (8 bits) for the first 128 code points, and up to 4 bytes for other characters.[7] The first 128 Unicode code points represent the ASCII characters, which means that any ASCII text is also a UTF-8 text.
- The obsolete UCS-2 uses two bytes (16 bits) for each character but can only encode the first 65,536 code points. UTF-16 extends UCS-2, by using a 4-byte encoding for the other planes. These are both common on Windows.

Different type of encoders based on Unicode standard(utf8, utf16, utf32) are able to encode all 1,112,064 characters code points in Unicode.

## UTF-8

UTF-8 is a variable-width character encoding used for electronic communication. Defined by the Unicode Standard, the name is derived from Unicode (or Universal Coded Character Set) Transformation Format – 8-bit.
UTF-8 is a variable width that's why it can take 8N bits into account for saving characters.

When representing characters in UTF-8, each code point is represented by a sequence of one or more bytes. The number of bytes used depends on the code point being represented by the character. Here's a breakdown of the usage range:

code points in the ASCII range (0-127) are represented by a single byte
code points in the range (128-2047) are represented by two bytes
code points in the range (2048-65535) are represented by three bytes
and code points in the range (65536-1114111) are represented by four bytes. (This may seem like a lot of possible characters, but keep in mind that in Chinese alone, there are 100,000s of characters.)
The first byte of a UTF-8 sequence is called the "leader byte". The leader byte provides information about how many bytes are in the sequence, and what the code point value of the character is.

The leader byte for a single-byte sequence is always in the range (0-127). The leader byte for a two-byte sequence is in the range (194-223). The leader byte for a three-byte sequence is in the range (224-239). And the leader byte for a four-byte sequence is in the range (240-247).

The remaining bytes in the sequence are called "trailing bytes." The trailing bytes for a two-byte sequence are in the range (128-191). The trailing bytes for a three-byte sequence are in the range (128-191). And the trailing bytes for a four-byte sequence are in the range (128-191).

You can calculate the code point value of a character by looking at the leader byte and the trailing bytes. For a single-byte sequence, the code point value is equal to the value of the leader byte.

For a two-byte sequence, the code point value is equal to ((leader byte - 194) \* 64) + (trailing byte - 128).

For a three-byte sequence, the code point value is equal to ((leader byte - 224) _ 4096) + ((trailing byte1 - 128) _ 64) + (trailing byte2 - 128).

For a four-byte sequence, the code point value is equal to ((leader byte - 240) _ 262144) + ((trailing byte1 - 128) _ 4096) + ((trailing byte2 - 128) \* 64) + (trailing byte3 - 128).

### UTF-8 is a Sound Choice for Encoding

Again, UTF-8 is a super efficient encoding system. It can represent a wide range of characters while still being compatible with ASCII. This makes it a sound choice for use in internationalized software.

## UTF-16

UTF-16 (16-bit Unicode Transformation Format) is a character encoding capable of encoding all 1,112,064 valid character code points of Unicode (in fact this number of code points is dictated by the design of UTF-16). The encoding is variable-length, as code points are encoded with one or two 16-bit code units. UTF-16 arose from an earlier obsolete fixed-width 16-bit encoding, now known as UCS-2 (for 2-byte Universal Character Set), once it became clear that more than 216 (65,536) code points were needed.

## UTF-32

UTF-32 (32-bit Unicode Transformation Format) is a fixed-length encoding used to encode Unicode code points that uses exactly 32 bits (four bytes) per code point (but a number of leading bits must be zero as there are far fewer than 232 Unicode code points, needing actually only 21 bits).[1] UTF-32 is a fixed-length encoding, in contrast to all other Unicode transformation formats, which are variable-length encodings. Each 32-bit value in UTF-32 represents one Unicode code point and is exactly equal to that code point's numerical value.
The main advantage of UTF-32 is that the Unicode code points are directly indexed. Finding the Nth code point in a sequence of code points is a constant-time operation. In contrast, a variable-length code requires linear-time to count N code points from the start of the string. This makes UTF-32 a simple replacement in code that uses integers that are incremented by one to examine each location in a string, as was commonly done for ASCII. Other than converting old code, this constant time access is far less advantageous than novice programmers may assume.
The main disadvantage of UTF-32 is that it is space-inefficient, using four bytes per code point, including 11 bits that are always zero. Characters beyond the BMP are relatively rare in most texts (except for e.g. texts with some popular emojis), and can typically be ignored for sizing estimates. This makes UTF-32 close to twice the size of UTF-16. It can be up to four times the size of UTF-8 depending on how many of the characters are in the ASCII subset.

Unicode defines a single huge character set, assigning one unique integer value to every graphical symbol (that is a major simplification, and isn't actually true, but it's close enough for the purposes of this question). UTF-8/16/32 are simply different ways to encode this.

In brief, UTF-32 uses 32-bit values for each character. That allows them to use a fixed-width code for every character.

UTF-16 uses 16-bit by default, but that only gives you 65k possible characters, which is nowhere near enough for the full Unicode set. So some characters use pairs of 16-bit values.

And UTF-8 uses 8-bit values by default, which means that the 127 first values are fixed-width single-byte characters (the most significant bit is used to signify that this is the start of a multi-byte sequence, leaving 7 bits for the actual character value). All other characters are encoded as sequences of up to 4 bytes (if memory serves).

And that leads us to the advantages. Any ASCII-character is directly compatible with UTF-8, so for upgrading legacy apps, UTF-8 is a common and obvious choice. In almost all cases, it will also use the least memory. On the other hand, you can't make any guarantees about the width of a character. It may be 1, 2, 3 or 4 characters wide, which makes string manipulation difficult.

UTF-32 is opposite, it uses the most memory (each character is a fixed 4 bytes wide), but on the other hand, you know that every character has this precise length, so string manipulation becomes far simpler. You can compute the number of characters in a string simply from the length in bytes of the string. You can't do that with UTF-8.

UTF-16 is a compromise. It lets most characters fit into a fixed-width 16-bit value. So as long as you don't have Chinese symbols, musical notes or some others, you can assume that each character is 16 bits wide. It uses less memory than UTF-32. But it is in some ways "the worst of both worlds". It almost always uses more memory than UTF-8, and it still doesn't avoid the problem that plagues UTF-8 (variable-length characters).

Finally, it's often helpful to just go with what the platform supports. Windows uses UTF-16 internally, so on Windows, that is the obvious choice.

Linux varies a bit, but they generally use UTF-8 for everything that is Unicode-compliant.

So short answer: All three encodings can encode the same character set, but they represent each character as different byte sequences.

**Programs that read text files often use BOM(Byte order mark) to know which encoding is used to encode a peace of text. It also help in determining The byte order, or endianness, of the text stream in the cases of 16-bit and 32-bit encodings**

More at -

https://en.wikipedia.org/wiki/Byte_order_mark

https://mathiasbynens.be/notes/javascript-encoding

https://mothereff.in/utf-8

https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/

## Stanford

## Text files and Binary files

At a generic level of description, there are two kinds of computer files: text files and binary files.[1]

### Text files

A text file (sometimes spelled textfile; an old alternative name is flatfile) is a kind of computer file that is structured as a sequence of lines of electronic text. A text file exists stored as data within a computer file system. In operating systems such as CP/M and MS-DOS, where the operating system does not keep track of the file size in bytes, the end of a text file is denoted by placing one or more special characters, known as an end-of-file marker, as padding after the last line in a text file. On modern operating systems such as Microsoft Windows and Unix-like systems, text files do not contain any special EOF character, because file systems on those operating systems keep track of the file size in bytes. Most text files need to have end-of-line delimiters, which are done in a few different ways depending on operating system. Some operating systems with record-orientated file systems may not use new line delimiters and will primarily store text files with lines separated as fixed or variable length records.
The ASCII character set is the most common compatible subset of character sets for English-language text files, and is generally assumed to be the default file format in many situations.
On Microsoft Windows operating systems, a file is regarded as a text file if the suffix of the name of the file (the "filename extension") is .txt. However, many other suffixes are used for text files with specific purposes. For example, source code for computer programs is usually kept in text files that have file name suffixes indicating the programming language in which the source is written.
Most Microsoft Windows text files use "ANSI", "OEM", "Unicode" or "UTF-8" encoding.

### Binary Files

A binary file is a computer file that is not a text file.[1] The term "binary file" is often used as a term meaning "non-text file".[2] Many binary file formats contain parts that can be interpreted as text; for example, some computer document files containing formatted text, such as older Microsoft Word document files, contain the text of the document but also contain formatting information in binary form.

Binary files are usually thought of as being a sequence of bytes, which means the binary digits (bits) are grouped in eights. Binary files typically contain bytes that are intended to be interpreted as something other than text characters. Compiled computer programs are typical examples; indeed, compiled applications are sometimes referred to, particularly by programmers, as binaries. But binary files can also mean that they contain images, sounds, compressed versions of other files, etc. – in short, any type of file content whatsoever.
Some binary files contain headers, blocks of metadata used by a computer program to interpret the data in the file. The header often contains a signature or magic number which can identify the format. For example, a GIF file can contain multiple images, and headers are used to identify and describe each block of image data. The leading bytes of the header would contain text like GIF87a or GIF89a that can identify the binary as a GIF file. If a binary file does not contain any headers, it may be called a flat binary file.
Some binary files contain headers, blocks of metadata used by a computer program to interpret the data in the file. The header often contains a signature or magic number which can identify the format. For example, a GIF file can contain multiple images, and headers are used to identify and describe each block of image data. The leading bytes of the header would contain text like GIF87a or GIF89a that can identify the binary as a GIF file. If a binary file does not contain any headers, it may be called a flat binary file.