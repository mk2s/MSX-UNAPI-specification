# The FujiNet Firmware UNAPI specification

This document describes an UNAPI compliant API specification for FujiNet firmware for MSX computers.

## Index

[1. Introduction](#1-introduction)

[2. API identifier and version](#2-api-identifier-and-version)

[3. API routines](#3-api-routines)

[3.1. FN_GETINFO: Obtain the implementation name and version](#31-fn_getinfo-obtain-the-implementation-name-and-version)

[3.2. FN_CALL_NOBUFF: Call a FujiNet function with no buffer](#32-fn_call_noparam-call-a-fujinet-function-with-no-buffer)

[3.3. FN_CALL_WRITE: Call a FujiNet function sending data to the device](#33-fn_call_write-call-a-fujinet-function-sending-data-to-the-device)

[3.4. FN_CALL_READ: Call a FujiNet function reading data from the device](#34-fn_call_read-call-a-fujinet-function-reading-data-from-the-device)

[Appendix A. Acknowledgements](#appendix-a-acknowledgements)

[Appendix B. Version history](#appendix-b-version-history)

## 1. Introduction

MSX-UNAPI is a standard procedure for defining, discovering and using new APIs (Application Program Interfaces) for MSX computers. The MSX-UNAPI specification is described [in a separate document](MSX%20UNAPI%20specification%201.1.md).

This document describes an UNAPI compliant API intended for FujiNet firmware on MSX computers. FujiNet is a multi-function network device that provides WiFi connectivity, network protocol handling, and device emulation capabilities. Rather than exposing each FujiNet feature as a separate routine, this API provides a generic interface for calling FujiNet firmware functions using three calling patterns: command with no buffer, sending data to the device, and reading data from the device.

The API abstracts FujiNet firmware function calls behind a small set of routines, so that client software can interact with the device without depending on the details of a particular FujiNet hardware revision or communication mechanism. As long as the routine signatures and behaviors described in this document are preserved, the implementation will be valid regardless of the underlying hardware interface.  The actual FujiNet command payload is defined in a separate FujiNet document.

## 2. API identifier and version

The API identifier for the specification described in this document is: "FujiNet" (without the quotes). Remember that per the UNAPI specification, API identifiers are case-insensitive.

The FujiNet firmware API version described in this document is 1.0. This is the API specification version that the mandatory implementation information routine must return in DE (see [Section 3.1](#31-fn_getinfo-obtain-the-implementation-name-and-version)).

## 3. API routines

This version of the FujiNet firmware API consists of 4 routines: 1 mandatory routine and 3 specification routines, which are described below. API implementations may define their own additional implementation-specific routines (numbered 128 through 254), as described in the [MSX-UNAPI specification](MSX%20UNAPI%20specification%201.1.md). Routine number 255 is reserved.

Register A is used at input to select the routine number. Registers F, BC, DE and HL may be used as input and output parameters, and their values are corrupted after routine execution unless stated otherwise. Registers IX and IY are never used as input parameters, to allow inter-slot and inter-segment calls. Alternate registers (AF', BC', DE', HL') are always corrupted after routine execution.

If client software invokes a non-existing routine number (that is, passes in A a number not assigned to any routine), nothing must happen and the code must return with AF, BC, DE and HL unmodified.

### 3.1. FN_GETINFO: Obtain the implementation name and version

* Input:
  * A = 0

* Output:
  * HL = Address of the implementation name string
  * DE = API specification version supported. D=primary, E=secondary.
  * BC = API implementation version. B=primary, C=secondary.

This routine is mandatory for all implementations of all UNAPI compliant APIs. It returns basic information about the implementation itself: the implementation version, the supported API version, and a pointer to the implementation description string.

The implementation name string must be placed in the same slot or segment of the implementation code (or in page 3), must be zero terminated, must consist of printable characters, and must be at most 63 characters long (not including the terminating zero). Refer to the [MSX-UNAPI specification](MSX%20UNAPI%20specification%201.1.md) for more details.

### 3.2. FN_CALL_NOBUFF: Call a FujiNet firmware function with no buffer

* Input:
  * A = 1
  * HL = Pointer to FujiNetParams defined below.

* Output:
  * L = result code:
    * 1: Success
    * 0: Failure

All FujiNet firmware calls take a pointer to FujiNetParams defined as:
```
typedef struct {
  uint8_t device;
  uint8_t command;
  uint8_t aux_descr;
  uint8_t aux1, aux2, aux3, aux4;
  void *buffer;
  uint16_t length;
} FujiNetParams;
```

This routine calls a FujiNet function that requires no buffer. It is intended for simple commands such as status queries, or toggle operations, where the params alone is sufficient to identify the action to perform.  Consult FujiNet documentation for device, command and other params.  When calling FN_CALL_NOBUFF, length will be ignored and buffer is not used.  

On success (L = 1), On failure (L = 0).

### 3.3. FN_CALL_WRITE: Call a FujiNet function sending data to the device

* Input:
  * A = 2
  * HL = Pointer to FujiNetParams

* Output:
  * L = result code:
    * 1: Success
    * 0: Failure

This routine calls a FujiNet firmware function that sends data from the MSX to the FujiNet device. The data to send must be placed in memory starting at the address specified in buffer, and length must contain the length of the data in bytes.  buffer and FujiNetParams struct must not be in Page1 and Page2.

Consult FujiNet documentation for device, command and other params.  

On success (L = 1), On failure (L = 0).

### 3.4. FN_CALL_READ: Call a FujiNet firmware function reading data from the device

* Input:
  * A = 3
  * HL = Pointer to FujiNetParams


* Output:
  * L = result code:
    * 1: Success
    * 0: Failure

This routine calls a FujiNet firmware function that reads data from the FujiNet device into MSX memory. The received data will be written to memory starting at the address specified in buffer, and length specifies the maximum number of bytes to receive.

Consult FujiNet documentation for device, command and other params.  buffer and FujiNetParams struct must not be in Page1 and Page2.

On success (L = 1), On failure (L = 0).

## Appendix A. Acknowledgements

Thanks to the FujiNet project team and the MSX community.

## Appendix B. Version history

* Version 0.0:

  * Apr-2026 Initial draft of the FujiNet firmware UNAPI specification
