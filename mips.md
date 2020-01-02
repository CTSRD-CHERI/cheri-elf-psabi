# CHERI-MIPS ELF psABI extensions

# Capability Registers Conventions

## Hybrid ABI

 Name      | ABI Name | Saver  | Meaning
-----------|----------|--------|-----------------------
 $c0       | $cnull   | N/A    | Null capability
 $c1-$c2   |          | Caller | Code and data arguments for cross-domain calls
 $c3       |          | Caller | Capability return value
 $c3-$c10  |          | Caller | Capability arguments
 $c11-$c15 |          | Caller | Temporary registers
 $c16-$c23 |          | Callee | Saved registers
 $c24-$c31 |          | Caller | Temporary registers

## Pure Capability ABI

 Name      | ABI Name | Saver  | Meaning
-----------|----------|--------|----------------------
 $c0       | $cnull   | N/A    | Null capability
 $c1-$c2   |          | Caller | Code and data arguments for cross-domain calls
 $c3       |          | Caller | Capability return value
 $c3-$c10  |          | Caller | Capability arguments
 $c11      | $csp     | Callee | Stack pointer
 $c12      |          | Caller | `cjalr` destination register 
 $c13      |          | Caller | Pointer to on-stack arguments
 $c14-$c15 |          | Caller | Temporary registers
 $c16      |          | Caller | Exception pointer register
 $c17      | $cra     | Caller | Return address
 $c18-$c23 |          | Callee | Saved registers
 $c24      | $cfp     | Callee | Frame pointer
 $c25      | $cbp     | Callee | Stack base pointer?
 $c26      | $cgp     | Callee | Global pointer
 $c26      | $idc     | N/A    | Initial data capability for `CCall`
 $c27-$c31 |          | Caller | Temporary registers

# Procedure Calling Convention

The stack pointer for both ABIs must be aligned to the size of a capability.

# C type details

Both ABIs add a new base C type to hold capabilities:

 Type      | Size (Bytes) | Alignment (Bytes)
-----------|--------------|-------------------
 uintcap_t | 16 or 32     | 16 or 32

The pure capability ABI changes the following C types:


 Type      | Size (Bytes) | Alignment (Bytes)
-----------|--------------|-------------------
 uintptr_t | 16 or 32     | 16 or 32
 void *    | 16 or 32     | 16 or 32


# ELF Object Files

## File Header

* e_flags: The following new flags are defined.

 Name                    | Value      | Meaning
-------------------------|------------|---------
 `EF_MIPS_ABI_CHERIABI`  | 0x0000c000 | Binary uses pure capability ABI
 `EF_MIPS_MACH_CHERI128` | 0x00c10000 | Binary uses 128-bit capabilities
 `EF_MIPS_MACH_CHERI256` | 0x00c20000 | Binary uses 256-bit capabilities


## Dynamic Tags

Note that dynamic section values remain virtual addresses and not
capabilities.

The following new values for `d_tag` are defined:


 Name                          | Value      | Meaning
-------------------------------|------------|---------
 `DT_MIPS_CHERI___CAPRELOCS`   | 0x7000c000 | Start of `__cap_relocs` section
 `DT_MIPS_CHERI___CAPRELOCSSZ` | 0x7000c001 | Length of `__cap_relocs` section
 `DT_MIPS_CHERI_FLAGS`         | 0x7000c002 | Various CHERI flags
 `DT_MIPS_CHERI_CAPTABLE`      | 0x7000c003 | Start of `.captable` section
 `DT_MIPS_CHERI_CAPTABLESZ`    | 0x7000c004 | Length of `.captable` section
 `DT_MIPS_CHERI_CAPTABLE_MAPPING`| 0x7000c005 | Start of `.captable_mapping` section
 `DT_MIPS_CHERI_CAPTABLE_MAPPINGSZ`| 0x7000c006 | Length of `.captable_mapping` section

The following fields are defined for the value of `DT_MIPS_CHERI_FLAGS`:

   Bits 0 - 2 | Bit 3  | Bit 4  | Bit 5 | Bits 6 - 63
  ------------|--------|--------|-------|-------------
   ABI        | CTFILE | CTFUNC | RELCR | *Reserved*

  * `DF_MIPS_CHERI_ABI_LEGACY` (0): Uses unbounded $pcc and $t9 for ABI calls
  * `DF_MIPS_CHERI_ABI_PCREL` (1): 
  * `DF_MIPS_CHERI_ABI_PLT` (2): 
  * `DF_MIPS_CHERI_ABI_FNDESC` (3): 
  * `DF_MIPS_CHERI_CAPTABLE_PER_FILE` (0x8): 
  * `DF_MIPS_CHERI_CAPTABLE_PER_FUNC` (0x10): 
  * `DF_MIPS_CHERI_RELATIVE_CAPRELOCS` (0x20): 

## Relocations

## Section Alignment

Loadable segments in executables and DSOs should be aligned and padded
to ensure that systems software can generate capabilities for each
segment that do not overlap with other loadable segments.
