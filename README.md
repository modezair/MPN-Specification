# MPN-Specification
Specification for the Mophun executable format (*.mpn), found in numerous old (~2004 AD) mobile games.
```
MOPHUN EXECUTABLE (*.mpn)
READ IN LITTLE ENDIAN
WRITTEN BY MDZAIR

--INFO--
Executable format that runs games for the once-popular Mophun gaming platform.
Compression is done in MoPack - Mophun's proprietary compression algorithm.
Can also be encrypted, which is what most Mophun-based games do.

--FORMAT--

//	ALL numerals are in little endian.

4	Signature
	Must equal 0x50474D56 (endian reversed = "VMGP").
	
4	HeapSize (divided by 4)
2	StackSize (divided by 4)
2	Flags
	Bits: CXXXXXXX XXXXXXXX
	X	= Unknown
	C	= Compressed
	
4	CodeSize
4	InitializedDataSize		(Embedded game data)
4	UninitializedDataSize	(No idea)
4	MetadataSize
4	???
	Usually non-zero if file is compressed...
	...at least from my POV.
	
4	AddressCount
4	FunctionTableSize
	This value, on parsing, must be rounded up such that it becomes
	divisible by 4. At least, that's what the official Mophun
	emulator does.


--CODE-- (size: CodeSize)
if Compressed: {
	4	CompressedSize
}
//	If Compressed is true, data from here until the next section is
//	compressed using MoPack, or by another name, Mophun LZ.

-----------------------------------

Mophun's assembly is documented in the official SDK.
Check "doc/sdk/MophunAsmRef.pdf".

What is not mentioned, however, is the bytecode for each instruction.

Bytecodes are written using 4 or 8 bytes, using the following format:
AA BB CC DD [EEEEEEEE], where:
	AA = Opcode (operation code)
	BB = Argument 1
	CC = Argument 2
	DD = Argument 3
	EEEEEEEE = Argument 4 (for 8-byte operations)
	
One may use pip-objdump.exe (from the official Mophun SDK) with
"--disassemble" on any MPN file to get its assembly & opcodes.
Use the assembly reference (doc/sdk/MophunAsmRef.pdf) to see each
opcode's purpose.
...or just reference any open-source Mophun emulator's source code.

Here's what I'll tell you, though:

	Notice the EEEEEEEE argument.
	If the bitwise operation (EEEEEEEE & 0x80000000) yields 0x80000000,
  	it is an integer.
	Otherwise, it is an address ID. Check --ADDRESS TABLE--.
	
	Examples:
		0x80000232 = Number 562 in hex.
		0xFFFFFFA9 = Number -87 in hex.
		0x00000025 = Address #37.

--RESOURCES-- (size: InitializedDataSize)
//	The game data.
//	This section stores textures, audio and other data sequentially.
//	However, the code needs to be analyzed to get them, because the MPN does
//	not store the file names or sizes - instead it just stores addresses and
//	raw data. The developer must specify the file IDs and sizes
//	themselves. The game code must be analyzed to find each sprite & audio.
//	Or, in the case of most games, you can just extract them by checking
//	each data address & find where it's referenced to get its size.
//	Or, in the case of some games, there's one singular embedded container
//	file storing all the game data, an example being Joe's Treasure Quest
//  3D. Check --ADDRESS TABLE-- for more.

--METADATA-- (size: MetadataSize)
////////////////////////
//	WORK IN PROGRESS  //
////////////////////////
// Stores the app's metadata.
// (for now, just skip `MetadataSize` bytes)

--ADDRESS TABLE--

// NOTE: Mophun counts address IDs starting from 1.

repeat AddressCount: {
	1	AddressType
	2	Argument1
		Used differently depending on AddressType's value:
			0x02 ->	FunctionNameOffset
					Starting from the function table offset.
	1	Argument2
		I haven't seen this one get used.
	4	Argument3
		Used differently depending on AddressType's value:
			0x02 -> Unused
			0x11 -> Code Address
			0x18 -> Heap Address
			0x21 -> Data Address
}

//	From here until the end of the file are all the null-terminated
//	built-in function names.
//	Some executables use custom functions that aren't from Mophun.
//	This results in emulators that don't have the required patches throwing
//	an error, complaining of "unresolved symbol access" or similar.
//	Examples include:
//		Worms: World Party -> {
//			wormsSetFocusFunc
//			wormsGfxCopy
//			...and some arena functions, and more...
//		}
//		Marble Cannon -> {
//			veAddResourceFile
//			veDeleteResourceFile
//			veGetTextDirection
//			veLoadString
//			veQueryText
//			(Context: Marble Cannon's MPN uses files from the
//			game's resource/apps/ folder for in-game strings
//			of different languages)
//		}
```
