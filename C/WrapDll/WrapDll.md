# Wrap Sensor DLL Driver for LabVIEW Call

**Abstract:**

This article describes how to wrap the sensor DLL driver for easily call from LabVIEW. Especially, the DLL has complex data interface: **structure pointer including another structure pointer**. 

Causes some sensor, instruments and customized equipments only have DLL driver, that is  written by C or C++. If the DLL data interface is complex or C++ class, User must wrap the DLL making another has simple interface data type DLL file, provide to LabVIEW to access and control the device. The LabVIEW can directly call the interface is structure pointer of DLL, corresponding to the LabVIEW cluster data type. But if the structure pointer object including another structure pointer, using cluster including cluster is not work. So this article will use Panasonic HL-C2 sensor Driver DLL as example to wrap to a simple data type interface, then using LabVIEW access the sensor data.



## DLL Prototype Function

It is from the sensor programing manual.

![1599228222625](D:\MyFile\mywrite\MarkdownImageLib\1599228222625.png)

Function Return:

```c
// HLC2_STATUS, function return type
// It is a DWORD type, unsigned long
typedef DWORD HLC2_STATUS;
```

Parameters Define:

```C
// hlc2Handle, it is a void pointer as device handle, define as below
typedef void* HLC2_HANDLE;

// dwOUt, 32bit unsigned data
// be used specifies output(outpu1 or 2)
typedef unsigned long DWORD

// pBufferNormal, it is a structure pointer
typedef struct {
	// DWORD,32bit unsigned data
	// typedef unsigned long DWORD;
	DWORD TopPoint;				// such as 1
	DWORD EndPoint;				// such as 400
	
	// WORD, 16bit unsigned data
	// typedef unsigned short WORD;
	WORD dwCount;				// such as 500
	
	// HLC2_NUMERIC11, numerical value form -999.999999 to +999.999999
	// using stucture represent
	// such as -123.123456
	/*
	typedef struct {
		BYTE Sign; 				// Sign +/-
		BYTE Integer[3]; 		// 3-digit integer(no zero suppression)
		BYTE Period;			// Decimal point(".")
		BYTE Decimal[6];		// 6-digit decimal number
	} HLC2_NUMERIC11;
	*/
	HLC2_NUMERIC11 *pGetData;
} HLC2_BUFFERNORMAL;

// bccFlg, 8bit unsigned data
typedef unsigned char BYTE;

```



## Simulation A DLL

Using this function simulates a sensor DLL file, it has the **GetBufferDataNormal** function.

Header file: myDll.h

```c
// setting struct byte align method
#pragma	pack(push, 1)

// defined DSA
typedef struct {
    unsigned char Sign; 			// Sign +/-
    unsigned char Integer[3]; 		// 3-digit integer(no zero suppression)
    unsigned char Period;			// Decimal point(".")
    unsigned char Decimal[6];		// 6-digit decimal number
} Number;

typedef struct {
	unsigned long TopPoint;				// such as 1
	unsigned long EndPoint;				// such as 400
	unsigned short dwCount;				// such as 500
	
	// such as -123.123456
	Number* pValue;
} NormalBuffer;

// export a dll
_declspec(dllexport) void GetBufferDataNormal(NormalBuffer* pNormalBuffer);


#pragma	pack(pop)
```

Source File: myDll.c

```C
#include "mydll.h"

// function defined
// Cause we are only care about the NormalBuffer ptr, so we just write a function
// it make *pBuffer every value increase 1;
void
GetBufferDataNormal(NormalBuffer* pBuffer){

// +1
// update the struct member value
pBuffer->TopPoint = pBuffer->TopPoint + 1;
pBuffer->EndPoint = pBuffer->EndPoint + 1;
pBuffer->dwCount = pBuffer->dwCount + 1;

// keep constant neg char
pBuffer->pValue->Sign = '-';


// although the char shoud is 0~9 refer to number, but we only add 1
// indicate the input parameter successfully pass the fucntion
pBuffer->pValue->Integer[0] = pBuffer->pValue->Integer[0] + 1;
pBuffer->pValue->Integer[1] = pBuffer->pValue->Integer[1] + 1;
pBuffer->pValue->Integer[2] = pBuffer->pValue->Integer[2] + 1;

// keep the point char
pBuffer->pValue->Period = '.';

// add + 1
pBuffer->pValue->Decimal[0] = pBuffer->pValue->Decimal[0] + 1;
pBuffer->pValue->Decimal[1] = pBuffer->pValue->Decimal[1] + 1;
pBuffer->pValue->Decimal[2] = pBuffer->pValue->Decimal[2] + 1;
pBuffer->pValue->Decimal[3] = pBuffer->pValue->Decimal[3] + 1;
pBuffer->pValue->Decimal[4] = pBuffer->pValue->Decimal[4] + 1;
pBuffer->pValue->Decimal[5] = pBuffer->pValue->Decimal[5] + 1;

// no return
}
```

## Compiler Driver Dll

- OS： Win7 32bit, VS2010Express
- Open VS > File > New > Project > Visual C++ > Win32 Project > Nexe > Application type: DLL > Additional  options: Empty project, as below

![1599835038397](D:\MyFile\mywrite\MarkdownImageLib\1599835038397.png)

- Click finish
- Add the *.h and *.c file to the project
- At the head tool bar setting : Release Win32 (Note: the another wrapp DLL project also need same)
- right select build solution
- Get a DLL, it is named your project name, my is **myDll.dll** at release folder



## Wrap myDll.dll

This function will wrap the previous myDLL.dll, get another named “wrapper” DLL to LabVIEW call. Note: this header file decode the complex interface to sample interface.

**wrapperdll.h**

```C
#include <Windows.h>

#pragma	pack(push, 1)

// defined DSA
typedef struct {
    unsigned char Sign; 			// Sign +/-
    unsigned char Integer[3]; 		// 3-digit integer(no zero suppression)
    unsigned char Period;			// Decimal point(".")
    unsigned char Decimal[6];		// 6-digit decimal number
} Number;

typedef struct {
	unsigned long TopPoint;				// such as 1
	unsigned long EndPoint;				// such as 400
	unsigned short dwCount;				// such as 500
	
	// such as -123.123456
	Number* pValue;
} NormalBuffer;


// new a sample struct to change the dll complex struct to a sampe struct
// make LV conveniently call
// the new sample stuct will as function input
typedef struct {
	unsigned long TopPoint;
	unsigned long EndPoint;
	unsigned short dwCount;

	// Sign +/-
	unsigned char Sign;

	// 3-digit integer(no zero suppression)
    unsigned char Integer0;
	unsigned char Integer1;
	unsigned char Integer2;

	// Decimal point(".")
    unsigned char Period;

	// 6-digit decimal number
    unsigned char Decimal0;
	unsigned char Decimal1;
	unsigned char Decimal2;
	unsigned char Decimal3;
	unsigned char Decimal4;
	unsigned char Decimal5;

} Data;

#pragma	pack(pop)


// define function pointer
// the interface is be call function parameter
typedef void (*DLLFUNC)(NormalBuffer* pNormalBuffer);


// export dll
_declspec(dllexport) void GetBufferDataNormalWrapperDll(Data* pData);

_declspec(dllexport) unsigned long GetDllVersion(void);
```

wrapperdll.c

```c
#include "wrapperdll.h"

// define wrapper function
void GetBufferDataNormalWrapperDll(Data* pData)
{
	// new strct vars from deriver
	Number Number;
	NormalBuffer NormalBufferV;
	
	// new function ptr
	DLLFUNC pGetBufferDataNormal;


	// load dll, get dll file handle
	// using relative path, just place myDll.dll file same path
	// with this wrapperdll.dll
	HINSTANCE DllHandle = LoadLibrary(L".\\myDll.dll");
	
	// only for debug
	// if get any error, using GetLastError() get the error code
	/*
	unsigned long error_id;
	error_id = GetLastError();
	pData->TopPoint = error_id;
	*/

	// init the pointer element with input parameter
	// init NumberV struct
	Number.Sign = pData->Sign;
	Number.Integer[0] = pData->Integer0;
	Number.Integer[1] = pData->Integer1;
	Number.Integer[2] = pData->Integer2;
	Number.Period = pData->Period;
	Number.Decimal[0] = pData->Decimal0;
	Number.Decimal[1] = pData->Decimal1;
	Number.Decimal[2] = pData->Decimal2;
	Number.Decimal[3] = pData->Decimal3;
	Number.Decimal[4] = pData->Decimal4;
	Number.Decimal[5] = pData->Decimal5;

	// init the NormalBufferV struct
	NormalBufferV.TopPoint = pData->TopPoint;
	NormalBufferV.EndPoint = pData->EndPoint;
	NormalBufferV.dwCount = pData->dwCount;
	NormalBufferV.pValue = &Number;
	
	if (DllHandle) {
		// pointer is valid
		// init get the function ptr
		pGetBufferDataNormal = (DLLFUNC)GetProcAddress(DllHandle, "GetBufferDataNormal");
		if (pGetBufferDataNormal)
		{
			// pointer valid
			// call the function to change the NormalBufferV struc
			// this function will make normalbufferV head 3 element +1
			// others element get -123.123456
			// pass the parameter to driver dll and call the driver function
            // GetBufferDataNormal
			pGetBufferDataNormal(&NormalBufferV);

			// after call, build return value to output parameter
			pData->TopPoint = NormalBufferV.TopPoint;
			pData->EndPoint = NormalBufferV.EndPoint;
			pData->dwCount = NormalBufferV.dwCount;

			pData->Sign = (NormalBufferV.pValue)->Sign;
			pData->Integer0 = (NormalBufferV.pValue)->Integer[0];
			pData->Integer1 = (NormalBufferV.pValue)->Integer[1];
			pData->Integer2 = (NormalBufferV.pValue)->Integer[2];
			pData->Period = (NormalBufferV.pValue)->Period;
			pData->Decimal0 = (NormalBufferV.pValue)->Decimal[0];
			pData->Decimal1 = (NormalBufferV.pValue)->Decimal[1];
			pData->Decimal2 = (NormalBufferV.pValue)->Decimal[2];
			pData->Decimal3 = (NormalBufferV.pValue)->Decimal[3];
			pData->Decimal4 = (NormalBufferV.pValue)->Decimal[4];
			pData->Decimal5 = (NormalBufferV.pValue)->Decimal[5];

		}
		// free the ptr
		FreeLibrary(DllHandle);
	}
	// no return
}

unsigned long GetDllVersion(void){
	// return a version number
	return 100;
}
```



## LV Call the Wrapped Dll

Test Environment: LabVIEW 2013 32bit. 

Note: you DLL must same with you driver DLL file bit, 32bit or 64bit

Below is the LabVIEW call results.

![RUN](https://github.com/Frank-Instruments/Frank-Instruments.github.io/blob/main/C/WrapDll/LVCode.jpg)



**Done!**
