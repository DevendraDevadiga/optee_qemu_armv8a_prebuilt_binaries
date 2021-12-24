# OP-TEE 3.15 on QEMUv8
Prebuilt OP-TEE 3.15 Binaries for QEMUv8 and Run x-test

# How to run OP-TEE on QEMU and run the x-test.

Download the binaries provided in this repository.

```
$ git clone https://github.com/DevendraDevadiga/optee_qemu_armv8a_prebuilt_binaries.git

$ cd optee_qemu_armv8a_prebuilt_binaries

$ ls

optee_qemu_armv8a  README.md

$ cd cd optee_qemu_armv8a

$ chmod 777 *

$ sudo ./run_qemu.sh
```

Two terminals will be opened ans pop-up. One for Secure world (OP-TEE) and another for Normal World (Linux) as below

![qemu_secure_and_normal_world_terminals](https://user-images.githubusercontent.com/36186082/147329631-b5e6bca0-c334-4418-971b-f210c0a6038e.png)

Type "c" (Continue) on qemu console. OP-TEE prints will display on Secure world console. 

Firmware loading log and Linux logs will display on Normal world console.

![FirmwarePrints](https://user-images.githubusercontent.com/36186082/147330261-8a9b687a-a466-40db-b33d-513d8e580682.png)

OP-TEE will display Memory map details and OP-TEE version details. You can obeserve OP-TEE verison as 3.15.0

_I/TC: OP-TEE version: 3.15.0-dev (gcc version 10.2.1 20201103 (GNU Toolchain for the A-profile Architecture 10.2-2020.11 (arm-10.16))) #8 Thu Dec 23 15:55:20 UTC 2021 aarch64_


![SecureWorldConsole](https://user-images.githubusercontent.com/36186082/147330441-34c8cc27-5b4d-48dd-a1ba-4b7703f22499.png)

Linux logs will display on Normal world console once OP-TEE loadded successfully and switched back to Normal world..

![LinuxConsole](https://user-images.githubusercontent.com/36186082/147330267-4d0c32a6-db87-441f-8e76-26ecde11c877.png)

In Linux you can able to see Linux OP-TEE Driver probe message:

[    2.019187] optee: probing for conduit method.

[    2.021676] optee: revision 3.15 (6be0dbca)

[    2.027383] optee: dynamic shared memory is enabled

[    2.058506] optee: initialized driver

Buildroot is used for rootfs. Login as "root".

![OP-TEE_Driver_Probe](https://user-images.githubusercontent.com/36186082/147330828-96bd324a-1af6-418d-8c3b-57119cac456a.png)

If the OP-TEE Linux driver is loaded properly you can able to see below nodes under "/dev"

#ls -al /dev/tee*

crw-rw----    1 root     teeclnt   247,   0 Dec 24 07:33 /dev/tee0

crw-rw----    1 root     tee       247,  16 Dec 24 07:33 /dev/teepriv0

# 

OP-TEE Client (tee-supllicant) should run as background process. And during the boot itself it is already started and no need to run manually in this case.

#ps -A | grep tee

  145 tee      /usr/sbin/tee-supplicant -d /dev/teepriv0

  146 root     [optee_bus_scan]
  
  180 root     grep tee

# 

![tee_supplicannt](https://user-images.githubusercontent.com/36186082/147331268-857454cd-929a-4b47-8846-17ac80363858.png)






