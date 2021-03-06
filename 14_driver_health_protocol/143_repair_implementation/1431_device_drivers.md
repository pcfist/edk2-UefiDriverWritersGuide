<!--- @file
  14.3.1 Device Drivers

  Copyright (c) 2012-2018, Intel Corporation. All rights reserved.<BR>

  Redistribution and use in source (original document form) and 'compiled'
  forms (converted to PDF, epub, HTML and other formats) with or without
  modification, are permitted provided that the following conditions are met:

  1) Redistributions of source code (original document form) must retain the
     above copyright notice, this list of conditions and the following
     disclaimer as the first lines of this file unmodified.

  2) Redistributions in compiled form (transformed to other DTDs, converted to
     PDF, epub, HTML and other formats) must reproduce the above copyright
     notice, this list of conditions and the following disclaimer in the
     documentation and/or other materials provided with the distribution.

  THIS DOCUMENTATION IS PROVIDED BY TIANOCORE PROJECT "AS IS" AND ANY EXPRESS OR
  IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF
  MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO
  EVENT SHALL TIANOCORE PROJECT  BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
  SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
  PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS;
  OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
  WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR
  OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS DOCUMENTATION, EVEN IF
  ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

-->

### 14.3.1 Device Drivers

Device drivers implementing `Repair()` must verify that _ChildHandle_ is `NULL`
and that _ControllerHandle_ represents a device the device driver is currently
managing. The following example shows the steps required to check these
parameters.

If these checks pass, the health status is returned. In this specific example,
the driver opens the PCI I/O Protocol in its Driver Binding `Start()` function.
This is why `gEfiPciIoProtocolGuid` is used in the call to the EDK II Library
`UefiLib` function `EfiTestManagedDevice()` that checks to see if the UEFI
Drivers providing the `GetHealthStatus()` service is currently managing _ControllerHandle_. If the
private context structure is required, typically, the UEFI Boot Service
`OpenProtocol()`opens one of the UEFI Driver produced protocols on
_ControllerHandle_ and then uses a `CR()` based macro to retrieve a pointer to
the private context structure. This example also calls _ProgressNotification_
from 10% to 100% at 10% increments.

###### Example 155-Repair() Function for a Device Driver

```c
#include <Uefi.h>
#include <Protocol/DriverHealth.h>

EFI_STATUS EFIAPI
AbcRepair (
  IN EFI_DRIVER_HEALTH_PROTOCOL                *This,
  IN EFI_HANDLE                                ControllerHandle,
  IN EFI_HANDLE                                ChildHandle,           OPTIONAL
  IN EFI_DRIVER_HEALTH_REPAIR_PROGRESS_NOTIFY  ProgressNotification   OPTIONAL
  )
{
  EFI_STATUS  Status;
  UINTN       Index;
  
  //
  // ChildHandle must be NULL for a Device Driver
  //
  if (ChildHandle != NULL) {
    return EFI_UNSUPPORTED;
  }
  
  //
  // Make sure this driver is currently managing ControllerHandle
  //
  Status = EfiTestManagedDevice (
             ControllerHandle,
             gAbcDriverBinding.DriverBindingHandle,
             &gEfiPciIoProtocolGuid
             );
  if (EFI_ERROR (Status)) {
    return Status;
  }
  
  //
  // Repair ControllerHandle
  //
  for (Index = 0;
       Index < 10; Index++) {
    //
    // Perform 10% of the work required to repair ControllerHandle
    //
    if (ProgressNotification != NULL) {
      ProgressNotification (Index, 10);
    }
  }
  
  return EFI_SUCCESS;
}
```
