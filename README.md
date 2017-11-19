# 作業系統 HW2

## Part I : Explain Modified Code

### 1. /userprog/addrspace.h
#### (1) 添加 usedPhysPages 紀錄已被使用的 page
```clike = 
// /userprog/addrspace.h

static bool usedPhysPages[NumPhysPages];
```

### 2. /userprog/addrspace.cc
#### (1) 初始化 usedPhysPages 
```clike =
// /userprog/addrspace.cc

bool AddrSpace::usedPhysPages[NumPhysPages] = {0};
```
#### (2) 將設定 pageTable 的實作位置改到 load() 中
```clike = 
// userprog/addrspace.cc

bool
AddrSpace::Load(char *fileName)
{
    // ...
    for(unsigned int i = 0; i<numPages; i++) {
        pageTable[i].virtualPage = i;
        // looking for unused physical page                                  
        int j = 0;
        while(j < NumPhysPages && usedPhysPages[j] == true) j++;
        usedPhysPages[j] = true;
        pageTable[i].physicalPage = j;
        pageTable[i].valid = true;
        pageTable[i].use = false;
        pageTable[i].dirty = false;
        pageTable[i].readOnly = false;
    }
    // ...
}
```
#### (3) 修改 MainMemory 讀取的位置
```clike = 
// userprog/addrspace.cc

bool AddrSpace::Load(char *fileName)
{
    // ...
    int codeAddr = 
        pageTable[noffH.code.virtualAddr/PageSize].physicalPage*PageSize +
            (noffH.code.virtualAddr%PageSize);
    int initDataAddr = 
        pageTable[noffH.initData.virtualAddr/PageSize].physicalPage*PageSize +
            (noffH.initData.virtualAddr%PageSize);
    if (noffH.code.size > 0) {
        DEBUG(dbgAddr, "Initializing code segment.");
	DEBUG(dbgAddr, noffH.code.virtualAddr << ", " << noffH.code.size);
        executable->ReadAt(
		&(kernel->machine->mainMemory[codeAddr]),
			noffH.code.size, noffH.code.inFileAddr);
    }
    if (noffH.initData.size > 0) {
        DEBUG(dbgAddr, "Initializing data segment.");
	DEBUG(dbgAddr, noffH.initData.virtualAddr << ", " << noffH.initData.size);
        executable->ReadAt(
		&(kernel->machine->mainMemory[pageTable[initDataAddr]),
			noffH.initData.size, noffH.initData.inFileAddr);
    }
    // ...
}
```
#### (4) 最後釋放使用的 page
```clike = 
// userprog/addrspace.cc

AddrSpace::~AddrSpace()
{
    for(int i=0; i<numPages; i++){
        usedPhysPages[pageTable[i].physicalPage] = false;
    }
    delete pageTable;
}
```

