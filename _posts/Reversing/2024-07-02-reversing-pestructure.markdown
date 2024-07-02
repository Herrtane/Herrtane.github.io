---
layout: post
title: <Reversing> 2. PE 파일 구조
date: 2024-07-02 12:30:23 +0900
category: Reversing
comments: true
---

## PE(Portable Executable) 파일 구조

연구실 내에서 신입생 세미나 리버싱 문제 출제를 준비하다보니, 오랜만에 리버싱 핵심 원리를 다시 펴게 되었다. 매번 책을 펴서 참고하기는 번거로우므로, 이번 기회에 아예 ELF 구조처럼 PE 구조도 따로 포스팅해서 정리해놓으려고 한다. 나중에 악성코드나 랜섬웨어 리버싱에서도 분명 유용히 사용할 수 있을 것이다.

<br/>

우선, PE 파일의 전체 구조도는 다음과 같다.

```
┌─────────────────────┐                                                                  
│ DOS header(0x40)    ├─────► IMAGE_DOS_HEADER                                           
├─────────────────────┤                                                                  
│ DOS stub            │                                                                  
├─────────────────────┤                                                                  
│ NT header(0xF8)     ├─────► IMAGE_NT_HEADERS ─────┬───────► IMAGE_FILE_HEADER          
├─────────────────────┤                             │                                    
│ Section header      ├─────► IMAGE_SECTION_HEADER  └───────► IMAGE_OPTIONAL_HEADER32(64)
├─────────────────────┤                                                                  
│ NULL                │                                                                  
├─────────────────────┤                                                                  
│                     │                                                                  
│                     │                                                                  
│                     │                                                                  
│                     │                                                                  
│                     │                                                                  
│                     │                                                                  
│                     │                                                                  
│                     │                                                                  
│ Section             │                                                                  
│                     │                                                                  
│                     │                                                                  
│                     │                                                                  
│                     │                                                                  
│                     │                                                                  
│                     │                                                                  
│                     │                                                                  
│                     │                                                                  
└─────────────────────┘                                                                  
```

### DOS Header

DOS header의 구조는 다음과 같다. 중요한 부분만 구조체로 정리했다.

```c

typedef struct _IMAGE_DOS_HEADER {
    WORD e_magic;                           // DOS signature : 0x4D5A ("MZ")
    ...
    WORD e_lfanew;                          // offset to NT header
} IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;

```

### DOS stub

DOS stub의 크기는 가변적이며, 정해진 양식이 없으므로 생략한다.

### NT header

NT header의 구조는 다음과 같다.

```c

typedef struct _IMAGE_NT_HEADERS {
    DWORD Signature;                        // PE Signature : 0x50450000 ("PE"00)
    IMAGE_FILE_HEADER FileHeader;
    IMAGE_OPTIONAL_HEADER32 OptionalHeader;
} IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS32;

typedef struct _IMAGE_FILE_HEADER {
    WORD Machine;                           // x86 : 0x014C, x64 : 0x8664, IA-64 : 0x0200 
    WORD NumberOfSections;
    DWORD TimeDateStamp;                    // The build time of this file
    DWORD PointerToSymbolTable;
    DWORD NumberOfSymbols;
    WORD SizeOfOptionalHeader;
    Word Characteristics;                   // It is combined in a bitwise OR format
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;

typedef struct _IMAGE_DATA_DIRECTORY {
    DWORD VirtualAddress;
    DWORD Size;
} IMAGE_DATA_DIRECTORY, *PIMAGE_DATA_DIRECTORY;

#define IMAGE_NUMBEROF_DIRECTORY_ENTRIES 16

typedef struct _IMAGE_OPTIONAL_HEADER {
    WORD Magic;                             // IMAGE_OPTIONAL_HEADER32 : 0X010B, IMAGE_OPTIONAL_HEADER64 : 0X020B
    BYTE MajorLinkerVersion;
    BYTE MinorLinkerVersion;
    DWORD SizeOfCode;
    ...
    DWORD AddressOfEntryPoint;              // RVA(Relative Virtual Address) of EP
    DWORD BaseOfCode;                       // RVA(Relative Virtual Address) of Code Section
    DWORD BaseOfData;                       // RVA(Relative Virtual Address) of Data Section
    DWORD ImageBase;                        // The PE loader creates a process to execute a PE file, 
                                            // loads the file into memory, 
                                            // and then sets the EIP register value to ImageBase + AddressOfEntryPoint.
    DWORD SectionAlignment;                 // The minimum unit of a section in memory
    DWORD FileAlignment;                    // The minimum unit of a section in File
    ...
    DWORD SizeOfImage;                      // The size of PE Image in memory
    DWORD SizeOfHeaders;
    ...
    WORD Subsystem;                         // 1 : System Driver File, 2 : GUI File, 3 : CLI File
    ...
    DWORD NumberOfRvaAndSizes;              // The number of the DataDirectory arrays
    IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];
                                            // DataDirectory[0] = EXPORT Directory
                                            // DataDirectory[1] = IMPORT Directory
                                            // DataDirectory[2] = RESOURCE Directory
                                            // DataDirectory[9] = TLS Directory
} IMAGE_OPTIONAL_HEADER32, PIMAGE_OPTIONAL_HEADER32;

```

### Section Header

각 섹션의 정보를 나타내는 Section Header의 구조는 아래와 같다.

```c

#define IMAGE_SIZEOF_SHORT_NAME 8

typedef struct _IMAGE_SECTION_HEADER {
    BYTE Name[IMAGE_SIZEOF_SHORT_NAME];
    union {
        DWORD PhysicalAddress;
        DWORD VirtualSize;                  // The size of the section in memory
    } Misc;
    DWORD VirtualAddress;                   // The start address of the section in memory
    DWORD SizeOfRawData;                    // The size of the section in file
    DWORD PointerToRawData;                 // The start offset of the section in file
    ...
    DWORD Characteristics;                  // It is combined in a bitwise OR format
} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;

```

## 참고

포스팅을 할 때 위의 ASCII 그림을 사용해서 그림을 그리니까 상당히 간편하고 유용해서, 해당 그림 도구를 제공하는 사이트 링크를 걸어놓겠다.

<br/>

[ASCII Flow 그림 도구 사이트](https://asciiflow.com/#/)