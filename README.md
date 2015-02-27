# Cellophane

Tools for manipulating cell expression format (CEF and CEB) files

**Note: Cellophane is not yet ready for use, nor do I know when it will be**

## Overview

Cellophane is a set of tools designed to manipulate large-scale gene expression data. It aims to be for gene 
expression data what [samtools](http://samtools.github.io) is for sequence data. It was designed to simplify the exchange
and manipulation of very large-scale transcriptomics data, particularly from single-cell RNA-seq. However, it can process
any omics dataset that can be represented as an annotated matrix of numbers.

Cellophane is implemented as a command-line utility called `cef`, which operates on an annotated matrix of gene-expression 
values. It supports useful operations such as filtering, sorting, splitting, joining and transposing the input. Multiple 
commands can be chained to perform more complex operations.

Cellophane works with binary files in CEB format ('cell expression binary', `.ceb`), which are compact but not human-readable. CEB files can be
converted to and from the text-based CEF format ('cell expression format', `.cef`) suitable for human consumption. CEF files are tab-delimited 
text files that can be easily parsed or imported into e.g. Excel. 

## Synopsis

```
cef info            - overview of file contents
cef transpose	  	- transpose rows and columns
cef T               - transpose rows and columns (synonym for 'transpose')
cef join		  	- join two datasets by given identifier
cef remove 			- remove attribute
cef annotate        - insert attribute from file
cef filter			- filter rows by given criteria
cef normalize		- normalize rows
cef sort			- sort by attribute or column value, or by correlation
cef aggregate		- calculate aggregate statistics for every row
cef view			- print parts of the matrix
cef import			- import a .cef file and output a .ceb file
cef export			- read a .ceb file and export a .cef
cef validate		- validate a .cef or .ceb file 

cef A : B           - pipe command A into command B (internal pipe)
```


## CEF and CEB file formats

### Detecting the file format

CEF files are tab-delimited text files in UTF-8 encoding. The first four characters are 'CEF\t' (that's a single tab character at then end), equivalent to the hexadecimal 4-byte number 0x43454609. There is no byte order mark (BOM).

CEB files are binary. The first four bytes are UTF-8 encoded 'CEB\t' (that's a single tab character at the end), equivalent to the hexadecimal 4-byte number 0x43454209.

The first four bytes of a file therefore unambiguously indicate if it's a CEF or CEB file (assuming it's known to be one of the two).


### CEF file format

Tab-delimited file, UTF-8 encoding, no BOM

Header line starting with 'CEF' and followed by row attribute count, column attribute count, row count and column count

Example of a file with 4 Row Attributes, 2 Column Attributes, 345 Rows, 123 Columns

|   |   |   |   |    |    |    |
|---|---|---|---|----|----|----|
|CEF| 4 | 2 |345|123 |    |    |
|	|	|	|   |Sex |Male|Female|
|	|	|	|   |Age |P22|P28|
|Gene|Chromosome|Position|Strand|    |    |    |
|Actb|2|12837184|+|    |11 |24 |
|Nkx2-1|17|33432|-|    |0 |41 |
|   |   |   |   |    |    | ...|



### CEB file format

Binary file, little endian

Strings are stored as UTF-8 with a leading int32 length indicator, and no terminator

File structure

	byte[4]	'CEB\t' magic word

	int64	Row count
	int64	Column count

	int32	Column attribute count
	string	Column attribute name #1 
	string	Column attribute name #2 
	...
	string	Column attribute name #n 

	string[] Column attribute #1 values (total equal to column count)
	string[] Column attribute #2 values
	...		
	string[] Column attribute #n values

	int32	Row attribute count
	string	Row attribute name #1
	string	Row attribute name #2
	...
	string	Row attribute name #n

	string[] Row attribute #1 values (total equal to row count)
	string[] Row attribute #2 values
	...		
	string[] Row attribute #n values

	float32[] Values, total of [Row count x Column count] values
