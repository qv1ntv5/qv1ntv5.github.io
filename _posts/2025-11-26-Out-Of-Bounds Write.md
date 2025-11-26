---
layout: post
title: Out-Of-Bounds Write.
subtitle: Notes from Out-Of-Bounds Write course from OST2.
tags: [csoft]
---

### 1. Definition.

In simple terms, Out-Of-Bounds Writes is a memory corruption vulnerability in which user-controlled data is used to calculate a memory location without proper security measures leading to data being write out of the intended destination in memory. In some ocasions user-controlled data can be written on a user-controlled memory location, this is an escalation of a *OOBW* to a *what-where* write vulnerability, in which a potential attacker controls what is being written and where is being written leading to a much more serious scenario.

Is worth to mention that Linear Stack/Heap/Global (when the overflow affects a global variable on DATA/BSS segments) Buffer Overflows are a subset of Out-Of-Bounds Writes, but it deservers an isolated mention since not all BO are in fact OOBW.

<br>

### 2. Common Causes.

There are two sets of causes in the OOBW problems:

1. **Intrinsic vulnerabilities** in which there are problems related to the logic flow of the code:

    Inside of this subset we can aim to poorly handled pointer arithmetic operations with user-controlled data. As a few examples:

    User-controlled data is used to set the index of an array:

    ```c
    array[user_cont_data] = var //OOB Write
    ```

    Pointer setted with user-controlled data and user-controlled data used to fulfill that pointer region:

    ```c
    ptr = base + user_cont_data1
    *ptr = user_cont_data2 //OOB (this time; what-where) Write
    ```

2. **Manufactured vulnerabilities** in which a Linear Buffer Overflow allows the corruption of data later used to calculated a memory address (like for example a pointer) and later overwrite with user-controled data the contents of that pointer. In this kind of vulnerabilities there is not a direct OOBW flaw and it can not come from the code it self but for example a dependency. Is important to note that a manufactured OOBW starts as some other vulnerability class.

<br>

### 3. Trivial Example.

Let's consider the following C code:

```c
// Out-Of-Bounds Write - oobw.c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int main(int argc, char *argv[]){
    unsigned long long buf[8];
    unsigned long long i = strtoull(argv[1], NULL, 16);
    unsigned long long j = strtoull(argv[2], NULL, 16);
    buf[i] = j;
    printf("Wrote 0x%llx to buf[%llu]\n", j, i);
    return 0;
}
```

The attacker controls the first two arguments of the program the index and the content of a buffer. If the first argument is 8 or more, whatever what is the second argument, is gona get written out of the boundaries of the buffer.

<br>

### 4. Exercises.

#### 4.1. CVE-2019-10540.

Qualcomm System on a Chip (SoC) are popular chips used on smartphones devices. Frequently provide functionality as primary Application Processor (AP) but also integrated peripheral processor for wireless capabilities such as cellular, Wifi, and Bluetooth.

The vulnerability occurs when the WLAN NAN function processes NAN availability attributes without properly validating the count value. This lack of input validation allows an attacker to trigger a buffer overflow condition. The vulnerable chunk of code is as follows:

```c
//user-controlled: Length, OTA_DataPtr
char GlobalBuffer[10 * 0xB0 + 6];

unsigned int itemCount = 0;

for (unsigned int i = 0; i < Length; i+= 0x44){
    memcpy(GlobalBuffer + 6 + itemCount * 0XB0, OTA_DataPtr + i, 0x44);
    itemCount++;
}
```

The vulnerability is kind of trivial, we are in a user-controlled for-loop (Length is user-controlled data) in which the source of the data being copied is also user-controlled (OTA_DataPtr).

Let's note (again) why this isn't a simple Linear Stack Buffer Overflow:

- First, *memcpy(dst, src, n)* copies 'n' bytes from 'src' to 'dst' starting at the address pointed by 'dst' pointer.

- If we look carefully, we will note that we are only copying 0x44 (68 bytes) in each iteration of the *for* loop so this would not be such a problem if also *Length* variable is user-controlled and *itemCount* increments once at a time.

- At the end, since we can perform a huge amount of iterations, we can write what-where in memory (partially, due to the increments of the pointer are multiple of 0xBO, but dangerous enough). We can think of this as if we are taking control of an offset used to craft a pointer (the destination data) that is being used to write user-controlled data in memory (OTA_DataPtr + i). 

The key difference from a Linear Stack Buffer Overflow is that we're not overflowing the buffer with a single large write. Instead, the vulnerability stems from an unchecked loop iteration count: each write is small (0x44 bytes), but the destination offset calculation (itemCount * 0xB0) grows unbounded. The loop lacks validation to ensure itemCount stays within the buffer's capacity (10 items), allowing the destination pointer to advance beyond the allocated memory."

<br>

#### 4.2. CVE-2020-0938.

CVE-2020-0938 is a remote code execution vulnerability in the Windows Adobe Type Manager Library (atmfd.dll) that handles Adobe Type 1 PostScript fonts.

Let's consider the following code:

```c
////ACID: num_master
int SetBlendDesignPositions(void *arg) {
  int num_master;
  Fixed16_16 values[16][15];

  for (num_master = 0; ; num_master++) {
    //KC: break if all tokens have been read from input stream
    if (GetToken() != TOKEN_OPEN) {
      break;
    }
    /* KC: writes a SACI number (0-14) of fixed point values from an ACID token in
     * input stream to &values[num_master] */
    int values_read = GetOpenFixedArray(&values[num_master], 15);
    SetNumAxes(values_read);
  }

  SetNumMasters(num_master);

  for (int i = 0; i < num_master; i++) {
    procs->BlendDesignPositions(i, &values[i]);
  }

  return 0;
}
```

In this block of code the user can control the tokens that have to be read, thus we have a for-loop with user-controlled break condition in which some user-controlled data is written to *&values\[num_master\]* following *GetOpenFixedArray()* function. Since we have control both of the token (what is being write) and the location (num_master is the for-loop index iteration and we can make the loop iterate as we want) this is a what-where memory write vulnerability or Out-Of-Bounds Write vulnerability.

<br>

#### 4.3. CVE-2020-1020.

CVE-2020-1020 is a remote code execution vulnerability in the Windows Adobe Type Manager Library that was patched by Microsoft in April 2020. This vulnerability exists in the way Windows Adobe Type Manager Library (atmfd.dll) improperly handles specially crafted multi-master fonts, specifically when parsing Adobe Type 1 PostScript format fonts. The flaw allows for remote code execution when Windows processes malicious Adobe PostScript or OpenType fonts.

Let's check the code:

```c
////ACID: g_font->numMasters
int ParseBlendVToHOrigin(void *arg) {
  Fixed16_16* ptrs[2];
  Fixed16_16 values[2];

  for (int i = 0; i < g_font->numMasters; i++) { //KC: 0 <= g_font->numMasters <= 16
    ptrs[i] = &g_font->SomeArray[arg->SomeField + i];
  }

  for (int i = 0; i < 2; i++) {        //KC: values becomes ACID here
    int values_read = GetOpenFixedArray(values, g_font->numMasters);
    if (values_read != g_font->numMasters) {
      return -8;
    }

    for (int num = 0; num < g_font->numMasters; num++) {
      ptrs[num][i] = values[num];
    }
  }

  return 0;
}
```

We can see at first that an array of two slots to pointers to *Fixed16_16* is declared:

```c
Fixed16_16* ptrs[2];
```

Then, two for-loops take place:

```c
for (int i = 0; i < 2; i++) {
  //...
  for (int num = 0; num < g_font->numMasters; num++) {
    ptrs[num][i] = values[num];
  }
}
```

Then, look that *ptrs\[num\]* denote the item of *ptrs\[\]* at index *num*, *ptrs\[num\]\[i\]*, since *ptrs* elements are pointers, denote a pointer from a a bse plus an offset (of the type *ptrs\[num\] + i*). 

According to the array declaration, *num* possible values must be or 0 or 1, but since *g_font->numMasters* is user-controlled and no validation of this value is performed, an assignation beyond the intended bounds of the buffer can be performed. If *values\[\]* array's items were also user-controlled (since we don't know what *GetOpenFixedArray()* function does) this would be partially a what-where memory write vulnerability.

Observe that, it does not matter that ptrs and values have only two items. In C, the program has an entire linear memory chunk with smalls blocks of memory being used by buffers an other stuff, this means that values\[3\] memory address exists independently that is not defined in the code and since:

```c
buffer[i] = *(buffer + i).
```

Where buffer is by definition a pointer to the first memory address of the array. The compiler just computes the address and dereferences it—there's no bounds checking whatsoever. So values\[3\] becomes *(values + 3 \* sizeof(Fixed16_16)), and the CPU will happily read or write whatever happens to be at that address.


<br>

#### 4.4. CVE-2020-13995.

CVE-2020-13995 is OOB write vulnerability in the *extract75* utility published by the US Air Force Sensor Data Management System (SDMS) River Loop Security. The vulnerability was found in a NITF (National Imagery Transmission Format) parser, specifically the extract75 utility from the Air Force Research Lab's Sensor Data Management Systems River Loop Security. The flaw is a global buffer overflow that leads to a write-what-where condition.

The tool originated from a DARPA program in 1998 and was designed to parse NITF imagery data River Loop Security. NITF is a format used for transmitting and storing imagery intelligence data.

Let's check the code:

```c
//XENO: Globals
char Gstr[255];
char sBuffer[1000];
//...
/* V2_0, V2_1 */
int number_of_DESs;
segment_info_type *DES_info;
//...
long read_verify(int fh, char *destination, long length, char *sErrorMessage) {
    long rc;
    long start;
    long file_len;
    static char sTemp[150];

    rc = read(fh, destination, length);
    if (rc == -1) {
        start = lseek(fh, 0, SEEK_CUR);
        file_len = lseek(fh, 0, SEEK_END);
        sprintf(sTemp, "Error reading, read returned %ld. (start = %ld, \
read length = %ld, file_length = %ld\n%s\n",
                    rc, start, length, file_len, sErrorMessage);
        errmessage(sTemp);
        iQuit(1);
    }
    else if (rc != length) {
        start = lseek(fh, 0, SEEK_CUR) - rc;
        file_len = lseek(fh, 0, SEEK_END);
        sprintf(sTemp, "Error reading, read returned %ld. (start = %ld, \
read length = %ld, file_length = %ld\n%s\n",
                    rc, start, length, file_len, sErrorMessage);
        errmessage(sTemp);
        printf("errno=%d\n", errno);
        iQuit(1);
    }
    return rc;
}

////ACID: hNITF
int main(int argc, char *argv[]){
	//...
    rc = open(sNITFfilename, O_RDONLY| O_BINARY);
	//...
    hNITF = rc;
	//...
	read_verify(hNITF, (char *) sBuffer, 3, "error reading header (# extension segs");
	    sBuffer[3] = '\0';
	    number_of_DESs = atoi(sBuffer);

	    if (number_of_DESs > 0) {
	        /* Allocate Space for extension segs information arrays */
	        DES_info = (segment_info_type *) malloc(sizeof(segment_info_type) * number_of_DESs);
	        if (DES_info == NULL) {
	            errmessage("Error allocating memory for DES_info");
	            iQuit(1);
	        }

	        /* Read Image subheader / data lengths */

	        read_verify(hNITF, sBuffer, 13 * number_of_DESs, "Error reading header / image subheader data lengths");

	        temp = sBuffer;

	        for (x = 0; x < number_of_DESs; x++) {
	            strncpy(Gstr, temp, 4);
	            Gstr[4] = '\0';
	            DES_info[x].length_of_subheader = atol(Gstr);
	            temp += 4;

	            strncpy(Gstr, temp, 9);
	            Gstr[9] = '\0';
	            DES_info[x].length_of_data = atol(Gstr);
	            temp += 9;

	            DES_info[x].pData = NULL;
	            DES_info[x].bFile_written = FALSE;
	        }
	    }
}
```

<br>

Lets see that there are two buffers defined as global data:

```c
char Gstr[255];
char sBuffer[1000];
//...
/* V2_0, V2_1 */
int number_of_DESs;
segment_info_type *DES_info;
```

Is worth to mention that global variables are not stored on the stack neither on the heap, they get allocated in DATA/BSS segments.

First, we start at the *main()* function, the user controlled data gets read from the file and stored on *hNITF* then, the buffer along with the user-controlled data gets introduced in the *read_verify()* function.

```c
int main(int argc, char *argv[]){
//...
  rc = open(sNITFfilename, O_RDONLY| O_BINARY);
//...
  hNITF = rc;
//...
  read_verify(hNITF, (char *) sBuffer, 3, "error reading header (# extension segs");
//...
}
```

Let's check now what *read_verify()* function does. At a first glance, it appears that the function is a wrapper to *read()* that reads 'length' data from 'fh' and store it on 'destination', in this case, the length is only 3 bytes, so only *sBuffer\[0\], sBuffer\[1\], sBuffer\[2\]*.

But, later, something interesting happens, *sBuffer\[3\]* gets filled with a null-terminator byte and then the first string representation of the buffer gets converted to an integer which gets stored in *number_of_DESs* and later a memory chunk for *number_of_DESs* elements of *segment_info_type* data type.

```c
sBuffer[3] = '\0';
number_of_DESs = atoi(sBuffer);

if (number_of_DESs > 0) {
    /* Allocate Space for extension segs information arrays */
    DES_info = (segment_info_type *)
              malloc(sizeof(segment_info_type) * number_of_DESs);
    if (DES_info == NULL) {
        errmessage("Error allocating memory for DES_info");
        iQuit(1);
    }
    //...
}
```

This essentially means that both variables *number_of_DESs* and *DES_info* are used-controlled. Then, *read_verify()* function is called again, but this time with *length* parameter as *13 * number_of_DESs*

```c
read_verify(hNITF, sBuffer, 13 * number_of_DESs,
    "Error reading header / image subheader data lengths");
```

read_verify as a wrapper is reading into a fixed-size wrapper a user-controlled amount of data from user-controlled source, if there is not regulation on the wrapper this could potentially lead to a buffer overflow.

```c
long read_verify(int fh, char *destination, long length, char *sErrorMessage)
{
    long rc;
    long start;
    long file_len;
    static char sTemp[150];

    rc = read(fh, destination, length);
    //...
}
```

However there is something more. If we assume that the second call of *read_verify()* leads to a global buffer overflow, then the global variables gets overwritten, this means that eventually *segment_info_type \*DES_info* is a user-controlled pointer after the overflow.

On the following diagram we can check how the buffer overflow of the *sBuffer* array eventually reachs BSS segment and *DES_info* pointer (remember that, despite memory allocation is towards lower-address, memory write direction always is towards higher address, thus the overwrite would follow:  sBuffer -> number_of_DESs -> DES_info). 

```text
Higher Addresses
┌─────────────────────────────────────────────────────────────┐
│                           STACK                             │
│                      (grows downward)                       │
├─────────────────────────────────────────────────────────────┤
│                        unused memory                        │
├─────────────────────────────────────────────────────────────┤
│                            HEAP                             │
│                       (grows upward)                        │
├─────────────────────────────────────────────────────────────┤
│                             ...                             │
├─────────────────────────────────────────────────────────────┤
│                         BSS Segment                         │
│            (Uninitialized global/static variables)          │
├─────────────────────────────────────────────────────────────┤
│        DES_info              │ 4-8 bytes (pointer)          │
│        (uninitialized ptr)   │ initialized to NULL?         │
├─────────────────────────────────────────────────────────────┤
│        number_of_DESs        │ 4 bytes (int)                │
│        (uninitialized)       │ value = 0?                   │
├─────────────────────────────────────────────────────────────┤
│                       DATA Segment                          │
│           (Initialized global/static variables)             │
├─────────────────────────────────────────────────────────────┤
│        sBuffer[1000]         │ 1000 bytes                   │
│        (char array)          │ all zeros initially?         │
├─────────────────────────────────────────────────────────────┤
│        Gstr[255]             │ 255 bytes                    │
│        (char array)          │ all zeros initially?         │
├─────────────────────────────────────────────────────────────┤
│                       TEXT Segment                          │
│                      (Program code)                         │
└─────────────────────────────────────────────────────────────┘
Lower Addresses
```


Thus, on the following for-loop, where the break-condition is user-controlled, several assignations are being performed to a user-controlled pointer destination (DES_info) and also the first 4 bytes of temp (our controlled-buffer) and the following 9 bytes are being copied to Gstr which later would be the source of the assignation. Thus, this is a what-where write vulnerability which borns from a global buffer-overflow.


```c
temp = sBuffer;

for (x = 0; x < number_of_DESs; x++) {
    strncpy(Gstr, temp, 4);
    Gstr[4] = '\0';
    DES_info[x].length_of_subheader = atol(Gstr);
    temp += 4;

    strncpy(Gstr, temp, 9);
    Gstr[9] = '\0';
    DES_info[x].length_of_data = atol(Gstr);
    temp += 9;

    DES_info[x].pData = NULL;
    DES_info[x].bFile_written = FALSE;
}
```


<br>

#### 4.5. CVE-2021-26675.

CVE-2021-26675 is a stack-based buffer overflow vulnerability in the dnsproxy component of ConnMan (Connection Manager), the vulnerability stems from insufficient length checks before performing memory copy operations (memcpy) in the DNS proxy code. The fix involved adding length checks to prevent buffer overflow.

Let's check the code:

```c
static char *uncompress(int16_t field_count, 	/*KC: ACID from packet header */
                        char *start,            /*KC: Starting header of ACID input packet */
                        char *end,              /*KC: End of ACID input packet */
                        char *ptr,              /*KC: Current offset in ACID input packet */
                        char *uncompressed,     /*KC: Base of [1025] output buffer */
                        int uncomp_len,         /*KC: Hardcoded 1025 */
                        char **uncompressed_ptr)/*KC: Offset to end of uncompressed data */
{
	char *uptr = *uncompressed_ptr; /* position in result buffer */

	debug("count %d ptr %p end %p uptr %p", field_count, ptr, end, uptr);

	while (field_count-- > 0 && ptr < end) {
		int dlen;		/* data field length */
		int ulen;		/* uncompress length */
		int pos;		/* position in compressed string */
		char name[NS_MAXLABEL]; /* tmp label */ /*KC: fixed-size 63 byte buffer*/
		uint16_t dns_type, dns_class;
		int comp_pos;

		if (!convert_label(start, end, ptr, name, NS_MAXLABEL,
					&pos, &comp_pos))
			goto out;

		/*
		 * Copy the uncompressed resource record, type, class and \0 to
		 * tmp buffer.
		 */

		ulen = strlen(name);
		strncpy(uptr, name, uncomp_len - (uptr - uncompressed)); /*KC: 1025 - (current offset-base) */

		debug("pos %d ulen %d left %d name %s", pos, ulen,
			(int)(uncomp_len - (uptr - uncompressed)), uptr);

		uptr += ulen;
		*uptr++ = '\0';

		ptr += pos;

		/*
		 * We copy also the fixed portion of the result (type, class,
		 * ttl, address length and the address)
		 */
		memcpy(uptr, ptr, NS_RRFIXEDSZ); /*KC: NS_RRFIXEDSZ = 10*/

		dns_type = uptr[0] << 8 | uptr[1];
		dns_class = uptr[2] << 8 | uptr[3];

		if (dns_class != ns_c_in)
			goto out;

		ptr += NS_RRFIXEDSZ;
		uptr += NS_RRFIXEDSZ;
    //...
  }
}
```

From comments we can see that the DNS received packet is user-controlled, in the *uncompress()* function, the parameters field_count, start, end and ptr are user-controlled data.

Then, an assignation is performed and the code enters in a while loop with user-controlled break condtion:

```c
	char *uptr = *uncompressed_ptr; /* position in result buffer */

	debug("count %d ptr %p end %p uptr %p", field_count, ptr, end, uptr);

	while (field_count-- > 0 && ptr < end) {
```

Then some declarations are performed, a strncpy() operation aswell (but this is without interest since the parameters involved are not user-controlled) and then we find the following lines:

```c
ptr += pos;

/*
  * We copy also the fixed portion of the result (type, class,
  * ttl, address length and the address)
  */
memcpy(uptr, ptr, NS_RRFIXEDSZ); /*KC: NS_RRFIXEDSZ = 10*/

dns_type = uptr[0] << 8 | uptr[1];
dns_class = uptr[2] << 8 | uptr[3];

if (dns_class != ns_c_in)
  goto out;

ptr += NS_RRFIXEDSZ;
uptr += NS_RRFIXEDSZ;
```

The first thing we must look at is that a memcpy operation is being performed with a user-controlled source to a destination but without control the length so at first this is valid and no overflow can be performed, after the operation, the destination and the source of the copy operation gets incremented.

However, we must not forgot that the while loop has a user-controlled exit-condition, this means that essentially since neither of the two pointers gets restablished at the start of the loop, eventually *uptr* would point out of the bounds of the intedended structure, thus, this is a what-where memory write vulnerability.

<br>

#### 4.6. CVE-2021-28216. 

Consider the following code:

```c
////ACID: NV Var data returned in PerformanceVariable
////NOT ACID: Variables named *Guid
//
// Update S3 boot records into the basic boot performance table.
//
VarSize = sizeof (PerformanceVariable);
Status = VariableServices->GetVariable(VariableServices, EFI_FIRMWARE_PERFORMANCE_VARIABLE_NAME, &gEfiFirmwarePerformanceGuid, NULL, &VarSize, &PerformanceVariable);

if (EFI_ERROR (Status)) {
	return Status;
}
BootPerformanceTable = (UINT8*) (UINTN)PerformanceVariable.BootPerformanceTablePointer;
//
// Dump PEI boot records
//
FirmwarePerformanceTablePtr = (BootPerformanceTable + sizeof(BOOT_PERFORMANCE_TABLE));

GuidHob = GetFirstGuidHob(&gEdkiiFpdtExtendedFirmwarePerformanceGuid);

while (GuidHob != NULL) {
	FirmwarePerformanceData = GET_GUID_HOB_DATA(GuidHob);
	PeiPerformanceLogHeader = (FPDT_PEI_EXT_PERF_HEADER *)FirmwarePerformanceData;
	CopyMem(FirmwarePerformanceTablePtr, FirmwarePerformanceData + sizeof (FPDT_PEI_EXT_PERF_HEADER), (UINTN)(PeiPerformanceLogHeader->SizeOfAllEntries));

	GuidHob = GetNextGuidHob(&gEdkiiFpdtExtendedFirmwarePerformanceGuid, GET_NEXT_HOB(GuidHob));
	FirmwarePerformanceTablePtr += (UINTN)(PeiPerformanceLogHeader->SizeOfAllEntries);
}
//
// Update Table length.
//
((BOOT_PERFORMANCE_TABLE *) BootPerformanceTable)->Header.Length = (UINT32)((UINTN)FirmwarePerformanceTablePtr - (UINTN)BootPerformanceTable);
```

In the code above, *PerformanceVariable* ends up in *BootPerformanceTable* which also ends up in *FirmwarePerformanceTablePtr* which is being used as a destination in *CopyMem* function. so a data-copy operation is being performed with a user-controlled value as the destination, this means that an attacker could make that uncontrolled data gets written any part in memory.

<br>

#### 4.7. CVE-2022-25636.

Netfilter is a lernel module that provides "firewalling, NAT and packet mangling for Linux", it is commonly interacted with via the iptables firewall rule configuration mechanism.

It is possible to specify a set of firewall rules via the "nft" userspace application, which will then be parsed and handled by the kernel module. Let's check the code:

```c
struct flow_action {
	unsigned int               num_entries;
	struct flow_action_entry   entries[];
};

struct flow_rule {
	struct flow_match          match;
	struct flow_action         action;
};

struct nft_flow_rule {
	__be16                     proto;
	struct nft_flow_match      match;
	struct flow_rule           *rule;
};

struct nft_offload_ctx {
	struct {
		enum nft_offload_dep_type   type;
		__be16                      l3num;
		u8                          protonum;
	} dep;
	unsigned int               num_actions;
	struct net                 *net;
	struct nft_offload_reg     regs[NFT_REG32_15 + 1];
};

/**
 * struct_size() - Calculate size of structure with trailing array.
 * @p: Pointer to the structure.
 * @member: Name of the array member.
 * @count: Number of elements in the array.
 *
 * Calculates size of memory needed for structure @p followed by an
 * array of @count number of @member elements.
 *
 * Return: number of bytes needed or SIZE_MAX on overflow.
 */

#define struct_size(p, member, count) __ab_c_size(count, sizeof(*(p)->member) + __must_be_array((p)->member), sizeof(*(p)))

#define NFT_OFFLOAD_F_ACTION (1 << 0)

struct flow_rule *flow_rule_alloc(unsigned int num_actions) {
	struct flow_rule *rule;
	int i;
	// XENO: allocates space for the rule->action.entries[num_actions] array
	rule = kzalloc(struct_size(rule, action.entries, num_actions),
		       GFP_KERNEL);
	if (!rule)
		return NULL;

	rule->action.num_entries = num_actions;
	/* Pre-fill each action hw_stats with DONT_CARE.
	 * Caller can override this if it wants stats for a given action.
	 */
	for (i = 0; i < num_actions; i++)
		rule->action.entries[i].hw_stats = FLOW_ACTION_HW_STATS_DONT_CARE;

	return rule;
}

static struct nft_flow_rule *nft_flow_rule_alloc(int num_actions) {
	struct nft_flow_rule *flow;

	flow = kzalloc(sizeof(struct nft_flow_rule), GFP_KERNEL);
	if (!flow)
		return NULL;

	flow->rule = flow_rule_alloc(num_actions);
	if (!flow->rule) {
		kfree(flow);
		return NULL;
	}

	flow->rule->match.dissector	= &flow->match.dissector;
	flow->rule->match.mask		= &flow->match.mask;
	flow->rule->match.key		= &flow->match.key;

	return flow;
}

static inline struct nft_expr *nft_expr_first(const struct nft_rule *rule)
{
	return (struct nft_expr *)&rule->data[0];
}

static inline struct nft_expr *nft_expr_last(const struct nft_rule *rule)
{
	return (struct nft_expr *)&rule->data[rule->dlen];
}

static inline bool nft_expr_more(const struct nft_rule *rule, const struct nft_expr *expr)
{
	return expr != nft_expr_last(rule) && expr->ops;
}


int nft_fwd_dup_netdev_offload(struct nft_offload_ctx *ctx, struct nft_flow_rule *flow, enum flow_action_id id, int oif)
{
	struct flow_action_entry *entry;
	struct net_device *dev;

	/* nft_flow_rule_destroy() releases the reference on this device. */
	dev = dev_get_by_index(ctx->net, oif);
	if (!dev)
		return -EOPNOTSUPP;

	entry = &flow->rule->action.entries[ctx->num_actions++];
	entry->id = id;
	entry->dev = dev;

	return 0;
}

static inline void *nft_expr_priv(const struct nft_expr *expr)
{
	return (void *)expr->data;
}

static int nft_dup_netdev_offload(struct nft_offload_ctx *ctx, struct nft_flow_rule *flow, const struct nft_expr *expr)
{
	const struct nft_dup_netdev *priv = nft_expr_priv(expr); // XENO: assume priv != ACID
	int oif = ctx->regs[priv->sreg_dev].data.data[0];

	return nft_fwd_dup_netdev_offload(ctx, flow, FLOW_ACTION_MIRRED /*5*/, oif);
}

////ACID: rule
struct nft_flow_rule *nft_flow_rule_create(struct net *net, const struct nft_rule *rule)
{
	struct nft_offload_ctx *ctx;
	struct nft_flow_rule *flow;
	int num_actions = 0, err;
	struct nft_expr *expr;

	expr = nft_expr_first(rule);
	while (nft_expr_more(rule, expr)) {
		if (expr->ops->offload_flags & NFT_OFFLOAD_F_ACTION)
			num_actions++;

		expr = nft_expr_next(expr);
	}

	if (num_actions == 0)
		return ERR_PTR(-EOPNOTSUPP);

	flow = nft_flow_rule_alloc(num_actions);
	if (!flow)
		return ERR_PTR(-ENOMEM);

	expr = nft_expr_first(rule);

	ctx = kzalloc(sizeof(struct nft_offload_ctx), GFP_KERNEL);
	if (!ctx) {
		err = -ENOMEM;
		goto err_out;
	}
	ctx->net = net;
	ctx->dep.type = NFT_OFFLOAD_DEP_UNSPEC;

	while (nft_expr_more(rule, expr)) {
		if (!expr->ops->offload) {
			err = -EOPNOTSUPP;
			goto err_out;
		}
		err = expr->ops->offload(ctx, flow, expr); //XENO: Calls nft_dup_netdev_offload()
		if (err < 0)
			goto err_out;

		expr = nft_expr_next(expr);
	}
	nft_flow_rule_transfer_vlan(ctx, flow);

	flow->proto = ctx->dep.l3num;
	kfree(ctx);

	return flow;
err_out:
	kfree(ctx);
	nft_flow_rule_destroy(flow);

	return ERR_PTR(err);
}
```

There are two importants parts of the code, the first one have to be with the allocation part:

```c
struct nft_flow_rule *nft_flow_rule_create(struct net *net, const struct nft_rule *rule){
  //...
	struct nft_expr *expr;

	expr = nft_expr_first(rule);
	while (nft_expr_more(rule, expr)) {
		if (expr->ops->offload_flags & NFT_OFFLOAD_F_ACTION)
			num_actions++;

		expr = nft_expr_next(expr);
	}
```

Is easy to see that num_actions++ value is user-controlled, this same value will be used alter in the code to allocate space for an array:

```c
struct flow_rule *flow_rule_alloc(unsigned int num_actions) {
	struct flow_rule *rule;
	int i;
	// XENO: allocates space for the rule->action.entries[num_actions] array
	rule = kzalloc(struct_size(rule, action.entries, num_actions),
		       GFP_KERNEL);
	if (!rule)
		return NULL;

	rule->action.num_entries = num_actions;
	/* Pre-fill each action hw_stats with DONT_CARE.
	 * Caller can override this if it wants stats for a given action.
	 */
	for (i = 0; i < num_actions; i++)
		rule->action.entries[i].hw_stats = FLOW_ACTION_HW_STATS_DONT_CARE;

	return rule;
}

static struct nft_flow_rule *nft_flow_rule_alloc(int num_actions) {
	struct nft_flow_rule *flow;

	flow = kzalloc(sizeof(struct nft_flow_rule), GFP_KERNEL);
	if (!flow)
		return NULL;

	flow->rule = flow_rule_alloc(num_actions);
	if (!flow->rule) {
		kfree(flow);
		return NULL;
	}

	flow->rule->match.dissector	= &flow->match.dissector;
	flow->rule->match.mask		= &flow->match.mask;
	flow->rule->match.key		= &flow->match.key;

	return flow;
}

//...
if (num_actions == 0)
  return ERR_PTR(-EOPNOTSUPP);

flow = nft_flow_rule_alloc(num_actions);
//...
```

We can see that *nft_flow_rule_alloc()* is a wrapper for *kzalloc()* which is allocation space on *flow->rule* for the array of 'num_actions' items.

Then, below in the code an assignation is perfomed:

```c

int nft_fwd_dup_netdev_offload(struct nft_offload_ctx *ctx, struct nft_flow_rule *flow, enum flow_action_id id, int oif)
{
	struct flow_action_entry *entry;
	struct net_device *dev;

	/* nft_flow_rule_destroy() releases the reference on this device. */
	dev = dev_get_by_index(ctx->net, oif);
	if (!dev)
		return -EOPNOTSUPP;

	entry = &flow->rule->action.entries[ctx->num_actions++];
	entry->id = id;
	entry->dev = dev;

	return 0;
}

//...

static int nft_dup_netdev_offload(struct nft_offload_ctx *ctx, struct nft_flow_rule *flow, const struct nft_expr *expr)
{
	const struct nft_dup_netdev *priv = nft_expr_priv(expr); // XENO: assume priv != ACID
	int oif = ctx->regs[priv->sreg_dev].data.data[0];

	return nft_fwd_dup_netdev_offload(ctx, flow, FLOW_ACTION_MIRRED /*5*/, oif);
}

//...
err = expr->ops->offload(ctx, flow, expr); //XENO: Calls nft_dup_netdev_offload()
//...
```

Observe that *entry* is a pointer which is being assigned with the memory address that aims to the memory region allocated before with *num_actions*, since this is user-controlled parameter ti can be a near to 0 value as desired, so *entry* can be a memory region with space enough to one assignation and the consequent assignation 

```c
entry->id = id;
entry->dev = dev;
```

Are out-of-bounds.

<br>





