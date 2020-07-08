# CHERI-RISC-V ELF psABI Extensions

This document is based on the [RISC-V ELF psABI specification] and references
many aspects of it. Any behaviour not specified by this extension should be
assumed to be as specified by that base document.

## Table of Contents
1. [Capability Register Convention](#capability-register-convention)
2. [Procedure Calling Convention](#procedure-calling-convention)
	* [Integer and Capability Calling Convention](#integer-capability-calling-convention)
	* [IL32PC64E Calling Convention](#il32pc64e-calling-convention)
	* [Named ABIs](#named-abis)
	* [Default ABIs](#default-abis)
3. [C Type Details](#c-types)
	* [C Type Sizes and Alignments](#c-type-sizes)
4. [ELF Object Files](#elf-object-files)
	* [File Header](#file-header)
	* [Relocations](#relocations)
	* [Thread Local Storage](#thread-local-storage)
	* [Dynamic Table](#dynamic-table)
	* [Capability Relocations Section](#capability-relocations-section)
	* [Section Alignment](#section-alignment)

# <a name=capability-register-convention></a> Capability Register Convention

Name    | ABI Mnemonic | Meaning                | Preserved across calls?
--------|--------------|------------------------|------------------------
c0      | cnull        | Null capability        | -- (Immutable)
c1      | cra          | Return capability      | No
c2      | csp          | Stack capability       | Yes
c3      | cgp          | Global capability      | -- (Unallocatable)*
c4      | ctp          | Thread capability      | -- (Unallocatable)
c5-c7   | ct0-ct2      | Temporary registers    | No
c8-c9   | cs0-cs1      | Callee-saved registers | Yes
c10-c17 | ca0-ca7      | Argument registers     | No
c18-c27 | cs2-cs11     | Callee-saved registers | Yes
c28-c31 | ct3-ct6      | Temporary registers    | No

\*: CGP is currently unused in the PCC-relative ABI. Future ABI variants will
make use of it. We leave it unallocatable in the PCC-relative ABI in order to
be comparable with the base RISC-V ABIs without linker relaxation support, as
well as for simplicity.

# <a name=procedure-calling-convention></a> Procedure Calling Convention

## <a name=integer-capability-calling-convention></a> Integer and Capability Calling Convention

TODO

## <a name=il32pc64e-calling-convention></a> IL32PC64E Calling Convention

The IL32PC64E calling convention is based on the ILP32E base RISC-V ABI, and
designed to be usable with the RV32EXcheri ISA. This calling convention is the
same as the integer and capability calling convention, except for the following
differences.

The stack pointer need only be aligned to a 64-bit boundary. Note that this is
stricter than the ILP32E ABI due to the alignment requirements of capabilities.

Registers c16-c31 do not participate in the calling convention, so
there are only six argument registers, ca0-ca5, only two callee-saved
registers, cs0-cs1, and only three temporaries, ct0-ct2.

When used with an ISA with a split register file, the non-aliasing registers
x16-x31 do not participate in the calling convention, so there are only six
integer argument registers, a0-a5, only two callee-saved registers, s0-s1, and
only three temporaries, t0-t2.

If used with an ISA that has any of the registers c16-c31, f0-f31 and, in the
case of a split register file, x16-x31, then these registers are considered
temporaries.

The IL32PC64E calling convention is not compatible with ISAs that have registers
that require load and store alignments of more than 64 bits. In particular,
this calling convention must not be used with the Q ISA extension.

## <a name=named-abis></a> Named ABIs

This specification defines the following named ABIs:

* <abi name=abi-il32pc64></a> **IL32PC64**: Integer and capability
  calling convention only, hardware floating-point calling convention is not
  used (i.e. ELFCLASS32, EF_RISCV_CHERIABI and EF_RISCV_FLOAT_ABI_SOFT).

* <abi name=abi-il32pc64f></a> **IL32PC64F**: IL32PC64 with hardware
  floating-point calling convention for FLEN=32 (i.e. ELFCLASS32,
  EF_RISCV_CHERIABI and EF_RISCV_FLOAT_ABI_SINGLE).

* <abi name=abi-il32pc64d></a> **IL32PC64D**: IL32PC64 with hardware
  floating-point calling convention for FLEN=64 (i.e. ELFCLASS32,
  EF_RISCV_CHERIABI and EF_RISCV_FLOAT_ABI_DOUBLE).

* <abi name=abi-il32pc64e></a> **IL32PC64E**: [IL32PC64E
  calling-convention](#il32pc64e-calling-convention) only, hardware
  floating-point calling convention is not used (i.e. ELFCLASS32,
  EF_RISCV_CHERIABI, EF_RISCV_FLOAT_ABI_SOFT, and EF_RISCV_RVE).

* <abi name=abi-l64pc128></a> **L64PC128**: Integer and capability
  calling convention only, hardware floating-point calling convention is not
  used (i.e. ELFCLASS64, EF_RISCV_CHERIABI and EF_RISCV_FLOAT_ABI_SOFT).

* <abi name=abi-l64pc128f></a> **L64PC128F**: L64PC128 with hardware
  floating-point calling convention for FLEN=32 (i.e. ELFCLASS64,
  EF_RISCV_CHERIABI and EF_RISCV_FLOAT_ABI_SINGLE).

* <abi name=abi-l64pc128d></a> **L64PC128D**: L64PC128 with hardware
  floating-point calling convention for FLEN=64 (i.e. ELFCLASS64,
  EF_RISCV_CHERIABI and EF_RISCV_FLOAT_ABI_DOUBLE).

* <abi name=abi-l64pc128q></a> **L64PC128Q**: L64PC128 with hardware
  floating-point calling convention for FLEN=128 (i.e. ELFCLASS64,
  EF_RISCV_CHERIABI and EF_RISCV_FLOAT_ABI_QUAD).

As with the base RISC-V ABIs, the IL32PC64\* ABIs are only compatible with
RV32\* ISAs, and the L64PC128\* ABIs are only compatible with the RV64\* ISAs.

The \*F ABIs require the \*F ISA extension, the \*D ABIs require the \*D ISA
extension, and the LP64Q ABI requires the Q ISA extension.

All ABIs require the Xcheri ISA extension.

## <a name=default-abis></a> Default ABIs

For the base RISC-V ABIs, it is recommended that the default ABIs be LP64D and
ILP32D. For the same reasons, but to make full use of the additional
protections made available by CHERI, we recommend that the default ABIs on
CHERI-RISC-V be [L64PC128D](#abi-l64pc128d) and [IL32PC64D](#abi-il32pc64d).

# <a name=c-types></a> C Type Details

## <a name=c-type-sizes></a> C Type Sizes and Alignments

CHERI-RISC-V introduces a new base C type to hold capabilities, for which there
are two conventions.

  * **LP64, LP64F, LP64D, LP64Q, L64PC128, L64PC128F, L64PC128D and
    L64PC128Q**: use the following type size and alignment (based on the
    LP64/L64PC128 convention):

    Type                | Size (Bytes) | Alignment (Bytes)
    --------------------|--------------|------------------
    void * __capability | 16           | 16

  * **ILP32, ILP32F, ILP32D, ILP32E, IL32PC64, IL32PC64F, IL32PC64D,
    IL32PC64E**: use the following type size and alignment (based on the
    ILP32/IL32PC64 convention):

    Type                | Size (Bytes) | Alignment (Bytes)
    --------------------|--------------|------------------
    void * __capability | 8            | 8

Just as it defines intptr_t/uintptr_t types that can hold pointers and integers
alike, the C standard library on CHERI-RISC-V also defines intcap_t/uintcap_t
types that can hold capabilities and integers alike. The maximum value that
such a type can hold is limited to the maximum _address_ a capability can take
(where an address is either 32-bit or 64-bit), however, and not the size of the
entire capability (which is either 64-bit or 128-bit).

Additionally, the new pure capability ABIs change the pointer type (and
intptr_t/uintptr_t accordingly) with two conventions.

  * **L64PC128, L64PC128F, L64PC128D, and L64PC128Q**: use the following type size and
    alignment (based on the L64PC128 convention):

    Type   | Size (Bytes) | Alignment (Bytes)
    -------|--------------|------------------
    void * | 16           | 16

  * **IL32PC64, IL32PC64F, IL32PC64D, and IL32PC64E**: use the following type size and
    alignment (based on the IL32PC64 convention):

    Type   | Size (Bytes) | Alignment (Bytes)
    -------|--------------|------------------
    void * | 8            | 8

They are otherwise identical to their LP64\*/ILP32\* counterparts.

For large objects, additional padding and alignment requirements may be
introduced to ensure that capabilities referring to the object can be
represented precisely, as capability compression means the available precision
for bounds decreases as the object size increases above a threshold. See [CHERI
ISAv7] for details of the restrictions.

# <a name=elf-object-files></a> ELF Object Files

## <a name=file-header></a> File Header

* e_flags: The following new flags are defined.

  * EF_RISCV_CHERIABI (0x00010000): This bit is set when the binary targets the
    pure capability ABIs defined by this specification.
  * EF_RISCV_CAP_MODE (0x00020000): This bit is set when the binary requires
    its instructions be decoded in _capability mode_ (see [CHERI ISAv7]).
    Currently this bit will be set if and only if a pure capability ABI is in
    use due to ISA and code generation restrictions, but future ISA revisions
    may provide enough support to allow the two to be independent.

## <a name=relocations></a> Relocations

The following table provides details of the additional CHERI-RISC-V ELF
relocations (instruction specific relocations show the instruction type in the
Details column):

Enum | ELF Reloc Type                         | Description                                    | Field        | Calculation | Details
-----|----------------------------------------|------------------------------------------------|--------------|-------------|----------------------------------
192  | R_RISCV_CHERI_CAPTAB_PCREL_HI20        | PC-relative capability table reference         | _U-type_     | T + A - P   | `%captab_pcrel_hi(symbol)`
193  | R_RISCV_CHERI_CAPABILITY               | Derive a capability                            | _capability_ | C + A       |
194  | R_RISCV_CHERI_CAPABILITY_CALL          | Derive a capability only used for direct calls | _capability_ | C + A       | Reserved; currently unimplemented
195  | R_RISCV_CHERI_SIZE                     | Size of the symbol                             | _wordclass_  | Z           |
196  | R_RISCV_CHERI_TPREL_CINCOFFSET         | TLS LE thread usage                            |              |             | `%tprel_cincoffset(symbol)`
197  | R_RISCV_CHERI_TLS_IE_CAPTAB_PCREL_HI20 | PC-relative TLS IE capability table offset     | _U-type_     |             | Macro `cla.tls.ie`
198  | R_RISCV_CHERI_TLS_GD_CAPTAB_PCREL_HI20 | PC-relative TLS GD capability table offset     | _U-type_     |             | Macro `clc.tls.gd`

### Calculation Symbols

The following table provides details on the additional variables used in
relocation calculation:

Variable | Description
---------|-----------------------------------------------
C        | Capability for the symbol in the symbol table
T        | Offset of the symbol into the capability table
Z        | Size of the symbol in the symbol table

### Field Symbols

The following table provides details on the additional variables used in
relocation fields:

Variable     | Description
-------------|-----------------------------
_capability_ | Specifies a capability field

### Capability Table

Each object (shared library or executable) contains a capability table (the
.captable section) which contains capabilities of global symbols (objects and
functions) referred to by the object. This completely replaces a traditional
GOT. The capability table in each dynamically linked object is filled in by the
dynamic linker during loading, and by the C startup code in statically linked
executables.

Unlike a GOT, the capability table is always required, since capabilities
cannot be materialised and must instead be derived from a sufficiently
authorising capability, provided either to the dynamic linker or to the C
startup code depending on whether the object is statically linked.

Like a GOT, the capability table also includes integer indices and offsets used
for thread local storage, described later in this document.

For loading a global capability from the capability table, the assembly
pseudoinstruction:

```
    clgc        ca0, symbol
```

expands to the following assembly instructions and relocations:

```
label:
    auipcc      ca0, %captab_pcrel_hi(symbol) # R_RISCV_CHERI_CAPTAB_PCREL_HI20 (symbol)
    clc         ca0, %pcrel_lo(label)(ca0)    # R_RISCV_PCREL_LO12_I (label)
```

This is equivalent to the `la` pseudoinstruction loading from the GOT when
assembling position-independent code for the base RISC-V ABIs, but _always_
incurs indirection.

### PC-Relative Addressing

There is also `cllc`, the equivalent of the `lla` pseudoinstruction in the base
RISC-V ABIs. Its use, however, is heavily discouraged, since the resulting
capability's bounds and permissions are derived from PCC and cannot be
guaranteed safe, thus it should only be used when necessary (such as in
low-level startup code that has not yet initialised the capability table) and
when the code in question can be proved safe. In such cases, the assembly
pseudoinstruction:

```
    cllc        ca0, symbol
```

expands to the following assembly instructions and relocations:

```
label:
    auipcc      ca0, %pcrel_hi(symbol)     # R_RISCV_PCREL_HI20 (symbol)
    cincoffset  ca0, ca0, %pcrel_lo(label) # R_RISCV_PCREL_LO12_I (label)
```

### Procedure Calls

Currently, all procedure calls are indirected through the capability table.
Although not currently exploited due to the reliance on PC-relative loads from
the capability table necessitating bounds covering the whole object, this
permits fine-grained bounds on function capabilities. However, it is desirable
to have a code model close to the base RISC-V ABIs, and so a future version of
the ISA will define a full I-type CJALR instruction that will then be used here
to provide a two-instruction PCC-relative call equivalent to the `call`
pseudoinstruction in the base ABIs.

Thus, all procedure calls currently amount to a load from the capability table
as defined above followed by an indirect capability jump and link:

```
    clgc        ct0, symbol
    cjalr       cra, ct0
```

Due to the use of indirect jumps, no PLT stubs are currently used, but that
will change in a future version of this specification when using the new I-type
CJALR described above.

## <a name=thread-local-storage></a> Thread Local Storage

Descriptions of the different models, the compiler flags, variable attributes
and ELF flags are all as specified in the base RISC-V ELF psABI specification.
CHERI-RISC-V only changes the instruction sequences and relocations, which are
described in this section.

These models closely mirror the base specification, and notably have bounds and
permissions derived from either `ctp` in the static models or from the entry in
the DTV for the global dynamic model. Future revisions of this specification
may define new models to impose finer grained protection on thread local
variables, as has been done for global variables with the capability table.

### Local Exec

Example assembly for the load, increment and store of a thread local variable
`symbol` using the `%tprel_hi`, `%tprel_cincoffset` and `%tprel_lo` assembly
modifiers, with the emitted relocations in comments:

```
    lui         a5, %tprel_hi(symbol)                   # R_RISCV_TPREL_HI20 (symbol)
    cincoffset  ca5, ctp, a5, %tprel_cincoffset(symbol) # R_RISCV_TPREL_CINCOFFSET (symbol)
    clw         t0, %tprel_lo(symbol)(ca5)              # R_RISCV_TPREL_LO12_I (symbol)
    addi        t0, t0, 1
    csw         t0, %tprel_lo(symbol)(ca5)              # R_RISCV_TPREL_LO12_S (symbol)
```

The `%tprel_cincoffset` assembly modifier does not return a value and is used
purely to associate the `R_RISCV_TPREL_CINCOFFSET` relocation with the
`cincoffset` instruction.

### Initial Exec

Example assembly load, increment and store of a thread local variable `symbol`
using the `cla.tls.ie` pseudoinstruction:

```
    cla.tls.ie  a0, symbol, ca0
    cincoffset  ca0, ctp, a0
    clw         t0, 0(ca0)
    addi        t0, t0, 1
    csw         t0, 0(ca0)
```

Note that, unlike RISC-V's `la.tls.ie`, CHERI-RISC-V's `cla.tls.ie` takes a
temporary capability register as an additional argument. The assembly
pseudoinstruction:

```
    cla.tls.ie  a0, symbol, ca0
```

expands to the following assembly instructions and relocations:

```
label:
    auipcc      ca0, %tls_ie_captab_pcrel_hi(symbol) # R_RISCV_CHERI_TLS_IE_CAPTAB_PCREL_HI20 (symbol)
    cl[w|d]     a0, %pcrel_lo(label)(ca0)            # R_RISCV_PCREL_LO12_I (label)
```

### Global Dynamic

Example assembly load, increment and store of a thread local variable `symbol`
using the `clc.tls.gd` pseudoinstruction:

```
    clc.tls.gd  ca0, symbol
    clgc        ca1, __tls_get_addr
    cjalr       cra, ca1
    clw         t0, 0(ca0)
    addi        t0, t0, 1
    csw         t0, 0(ca0)
```

The assembly pseudoinstruction:

```
    clc.tls.gd  ca0, symbol
```

expands to the following assembly instructions and relocations:

```
label:
    auipcc      ca0, %tls_gd_captab_pcrel_hi(symbol) # R_RISCV_CHERI_TLS_GD_CAPTAB_PCREL_HI20 (symbol)
    cincoffset  ca0, ca0, %pcrel_lo(label)           # R_RISCV_PCREL_LO12_I (label)
```

The `__tls_get_addr` function has the same type as in the base RISC-V ELF psABI
specification. That is:

```
extern void *__tls_get_addr (tls_index *ti);
```

where the type `tls_index` is defined as:

```
typedef struct {
    unsigned long ti_module;
    unsigned long ti_offset;
} tls_index;
```

## <a name=dynamic-table></a> Dynamic Table

The following table provides details of the additional CHERI-RISC-V dynamic table entries:

Enum       | ELF Dynamic Tab              | Description
-----------|------------------------------|-------------------------------------
0x7000c000 | DT_RISCV_CHERI___CAPRELOCS   | Start of the __cap_relocs section
0x7000c001 | DT_RISCV_CHERI___CAPRELOCSSZ | Length of the __cap_relocs section

## <a name=capability-relocations-section></a> Capability Relocations Section

For capability relocations against preemptible or external symbols,
CHERI-RISC-V uses normal ELF relocations with a type of
R_RISCV_CHERI_CAPABILITY as defined in [Relocations](#relocations). However,
for capability relocations against non-preemptible symbols defined within the
same object, CHERI-RISC-V uses a separate ad-hoc format inherited from
CHERI-MIPS, which itself inherits it from its own legacy pure capability ABI
that lacked support for dynamic linking. Whilst this is a non-standard format
that is intended to be phased out, its specialist nature makes it easier to
parse.

These relocations are defined as an array in a new `__cap_relocs` section, with
corresponding `__start___cap_relocs` and `__stop___cap_relocs` symbols. Each
entry is of the following type:

```
typedef struct {
    unsigned long cr_location;
    unsigned long cr_base;
    unsigned long cr_offset;
    unsigned long cr_length;
    unsigned long cr_flags;
} cap_reloc;
```

* `cr_location`: The ELF virtual address where the capability is to be stored
  within this object.

* `cr_base`: The ELF virtual address of the symbol being pointed to, which will
  also be within this object. This will be the base of the derived capability.

* `cr_offset`: The offset to add to the derived capability (equivalent to an
  ELF addend).

* `cr_length`: The length used for the bounds of the derived capability. This
  will be computed at link time from the size in the ELF symbol table of the
  symbol being pointed.

* `cr_flags`: If the most significant bit of this field is set, the capability
  will be a function (executable) capability, otherwise the capability will be
  a data (non-executable) capability. In the case of data capabilities, if the
  second most significant bit is set, the capability is for read only data and
  will only permit loads, otherwise the capability will permit both loads and
  stores. All other bits are reserved.

Note that, being ELF virtual addresses, the base load address of the object
must be added to `cr_location` and `cr_base` in order to calculate the absolute
address at run time.

## <a name=section-alignment></a> Section Alignment

Loadable segments in executables and shared objects should be aligned and
padded to ensure that systems software can generate capabilities for each
segment that do not overlap with other loadable segments. See [CHERI ISAv7] for
the specification of the capability compression in use and the alignment
requirements the format imposes to ensure representable non-overlapping bounds.

[CHERI ISAv7]: https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-927.pdf
[RISC-V ELF psABI specification]: https://github.com/riscv/riscv-elf-psabi-doc
