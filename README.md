# OP-TEE 3.15 on QEMUv8
Prebuilt OP-TEE 3.15 Binaries for QEMUv8 and Run x-test

# How to run OP-TEE on QEMU and run the x-test.

Download the binaries provided in this repository.

```ruby
$ git clone https://github.com/DevendraDevadiga/optee_qemu_armv8a_prebuilt_binaries.git
$ cd optee_qemu_armv8a_prebuilt_binaries
$ ls
optee_qemu_armv8a  README.md
$ cd cd optee_qemu_armv8a
$ chmod 777 *
```
If want share the folders between your Host PC and QEMU Linux, run below command:
```ruby
$ sudo ./run_qemu_share_files.sh
```
Otherwise run the below command:
```ruby
$ sudo ./run_qemu.sh
```

Two terminals will be opened ans pop-up. One for Secure world (OP-TEE) and another for Normal World (Linux) as below

![qemu_secure_and_normal_world_terminals](https://user-images.githubusercontent.com/36186082/147329631-b5e6bca0-c334-4418-971b-f210c0a6038e.png)

Type "c" (Continue) on qemu console. OP-TEE prints will display on Secure world console. 

Firmware loading log and Linux logs will display on Normal world console.

![FirmwarePrints](https://user-images.githubusercontent.com/36186082/147330261-8a9b687a-a466-40db-b33d-513d8e580682.png)

OP-TEE will display Memory map details and OP-TEE version details. You can obeserve OP-TEE verison as 3.15.0

```ruby
I/TC: OP-TEE version: 3.15.0-dev (gcc version 10.2.1 20201103 (GNU Toolchain for the A-profile Architecture 10.2-2020.11 (arm-10.16))) #8 Thu Dec 23 15:55:20 UTC 2021 aarch64
```

![SecureWorldConsole](https://user-images.githubusercontent.com/36186082/147330441-34c8cc27-5b4d-48dd-a1ba-4b7703f22499.png)

Linux logs will display on Normal world console once OP-TEE loadded successfully and switched back to Normal world..

![LinuxConsole](https://user-images.githubusercontent.com/36186082/147330267-4d0c32a6-db87-441f-8e76-26ecde11c877.png)

In Linux you can able to see Linux OP-TEE Driver probe message:

```ruby
[    2.019187] optee: probing for conduit method.
[    2.021676] optee: revision 3.15 (6be0dbca)
[    2.027383] optee: dynamic shared memory is enabled
[    2.058506] optee: initialized driver
```

Buildroot is used for rootfs. Login as "root".

![OP-TEE_Driver_Probe](https://user-images.githubusercontent.com/36186082/147330828-96bd324a-1af6-418d-8c3b-57119cac456a.png)

If the OP-TEE Linux driver is loaded properly you can able to see below nodes under "/dev"

```ruby
#ls -al /dev/tee*
crw-rw----    1 root     teeclnt   247,   0 Dec 24 07:33 /dev/tee0
crw-rw----    1 root     tee       247,  16 Dec 24 07:33 /dev/teepriv0
# 
```

OP-TEE Client (tee-supllicant) should run as background process. And during the boot itself it is already started and no need to run manually in this case.

```ruby
#ps -A | grep tee
  145 tee      /usr/sbin/tee-supplicant -d /dev/teepriv0
  146 root     [optee_bus_scan]
  180 root     grep tee
# 
```

![tee_supplicannt](https://user-images.githubusercontent.com/36186082/147331268-857454cd-929a-4b47-8846-17ac80363858.png)


In th Secure Console we can't enter any commands. But if type any key its ASCII value will be displayed. Interrupt handler for capturing the UART keys is registered. (console_itr_cb)

https://github.com/OP-TEE/optee_os/blob/master/core/arch/arm/plat-vexpress/main.c#L117
https://github.com/OP-TEE/optee_os/blob/master/core/arch/arm/plat-vexpress/main.c#L109

For example if we type the "ENTER" in keyboard in Secure console ASCII value (in HEX) 0xD will display as below:
```ruby
D/TC:0   console_itr_cb:167 cpu 0: got 0xd
```
For testing I typed capital "A" and small "a" and able to see those typed keys ASCII values in HEX as below:

```ruby
D/TC:0   console_itr_cb:167 cpu 0: got 0x41
D/TC:0   console_itr_cb:167 cpu 0: got 0x61
```
![interrupt_handle](https://user-images.githubusercontent.com/36186082/147333310-83d16d5d-2a22-426a-b103-2f88249a0988.png)

# Share the files/folders between Host PC and QEMU 

This step is usefull when you want to copy the TA.CA to QEMU Linux file system from your host PC.
In this path "optee_qemu_armv8a/shared_folder" already folder is provided and in script "run_qemu_share_files.sh" its hard coded.

Once Linux is booted in QEMU, mount the shared the folder using below command:

```ruby
$ mkdir shared && mount -t 9p -o trans=virtio host <mount point>
```
Example:

```ruby
$ mkdir /mnt/shared && mount -t 9p -o trans=virtio host /mnt/shared
```

Now you can check you are able to access files availables in your host pc or not.
```ruby
$ ls /mnt/shared
sharedfile.sh
$ cat /mnt/shared/sharedfile.sh
This is shared path
```

# Run the Example Trusted applications

For OP-TEE Example application details go through below link:
https://optee.readthedocs.io/en/latest/building/gits/optee_examples/optee_examples.html


The Trusted Applications are generated in the "*.ta" format. In the file system these applications are stored in this path "/lib/optee_armtz/". 

https://github.com/OP-TEE/optee_client/blob/fa1d30c95d6f84cffed59220c0443709c303866c/tee-supplicant/src/tee_supplicant.c#L456

When host application is called TEEC_OpenSession() with UUID information, the tee-supplicant will load the corresponding Trusted application (TA) i.e /lib/optee_armtz/<uuid>.ta
  
Check TA's are available or not. It will display all the TA which is required by "xtest" and optee example applications:

```ruby
# ls /lib/optee_armtz/
12345678-5b69-11e4-9dbb-101f74f00099.ta
25497083-a58a-4fc5-8a72-1ad7b69b8562.ta
2a287631-de1b-4fdd-a55c-b9312e40769a.ta
380231ac-fb99-47ad-a689-9e017eb6e78a.ta
484d4143-2d53-4841-3120-4a6f636b6542.ta
528938ce-fc59-11e8-8eb2-f2801f1b9fd1.ta
5b9e0e40-2636-11e1-ad9e-0002a5d5c51b.ta
5ce0c432-0ab0-40e5-a056-782ca0e6aba2.ta
5dbac793-f574-4871-8ad3-04331ec17f24.ta
60276949-7ff3-4920-9bce-840c9dcf3098.ta
614789f2-39c0-4ebf-b235-92b32ac107ed.ta
731e279e-aafb-4575-a771-38caa6f0cca6.ta
873bcd08-c2c3-11e6-a937-d0bf9c45c61c.ta
8aaaf200-2450-11e4-abe2-0002a5d5c51b.ta
a4c04d50-f180-11e8-8eb2-f2801f1b9fd1.ta
a734eed9-d6a1-4244-aa50-7c99719e7b7b.ta
b3091a65-9751-4784-abf7-0298a7cc35ba.ta
b689f2a7-8adf-477a-9f99-32e90c0ad0a2.ta
b6c53aba-9669-4668-a7f2-205629d00f86.ta
c3f6e2c0-3548-11e1-b86c-0800200c9a66.ta
cb3e5ba0-adf1-11e0-998b-0002a5d5c51b.ta
d17f73a0-36ef-11e1-984a-0002a5d5c51b.ta
e13010e0-2ae1-11e5-896a-0002a5d5c51b.ta
e626662e-c0e2-485c-b8c8-09fbce6edf3d.ta
e6a33ed4-562b-463a-bb7e-ff5e15a493c8.ta
ee90d523-90ad-46a0-859d-8eea0b150086.ta
f157cda0-550c-11e5-a6fa-0002a5d5c51b.ta
f4e750bb-1437-4fbf-8785-8d3580c34994.ta
ffd2bded-ab7d-4988-95ee-e4962fff7154.ta
# 
```

The Host applications are in "/usr/bin" path in the file system.
 
```ruby
# ls /usr/bin/optee*
/usr/bin/optee_example_acipher         /usr/bin/optee_example_plugins
/usr/bin/optee_example_aes             /usr/bin/optee_example_random
/usr/bin/optee_example_hello_world     /usr/bin/optee_example_secure_storage
/usr/bin/optee_example_hotp
# 
# ls /usr/bin/xtest 
/usr/bin/xtest
# 
```
![Host_and_Trusted_applications](https://user-images.githubusercontent.com/36186082/147335792-5b18369e-350f-4cd4-919d-7917b5d47756.png)

Now run the each test application and observe the results.
  
# 1. OP-TEE Hello World example.

When Hello World CA is called with TEEC_OpenSession(), its corresponding TA will be loaded.
  
  https://github.com/linaro-swg/optee_examples/blob/master/hello_world/host/main.c#L44 ==> https://github.com/linaro-swg/optee_examples/blob/master/hello_world/ta/include/hello_world_ta.h#L35
  
  https://github.com/linaro-swg/optee_examples/blob/master/hello_world/host/main.c#L56

Below is the UUID of "optee_example_hello_world":
  
```ruby
  #define TA_HELLO_WORLD_UUID \
	{ 0x8aaaf200, 0x2450, 0x11e4, \
		{ 0xab, 0xe2, 0x00, 0x02, 0xa5, 0xd5, 0xc5, 0x1b} }
```
  
So corresponding TA for this Host application is : 
```ruby
/lib/optee_armtz/8aaaf200-2450-11e4-abe2-0002a5d5c51b.ta
```
Run the CA (Client Application) or Host application:

![run_ca](https://user-images.githubusercontent.com/36186082/147337844-0e1e3148-15ce-4904-ba03-4ba10d8f60ca.png)
  
In Secure console, you will get the messages related to TA as below:
```ruby
D/TC:? 0 tee_ta_init_pseudo_ta_session:299 Lookup pseudo TA 8aaaf200-2450-11e4-abe2-0002a5d5c51b
D/TC:? 0 ldelf_load_ldelf:91 ldelf load address 0x40006000
D/LD:  ldelf:134 Loading TS 8aaaf200-2450-11e4-abe2-0002a5d5c51b
D/TC:? 0 ldelf_syscall_open_bin:142 Lookup user TA ELF 8aaaf200-2450-11e4-abe2-0002a5d5c51b (Secure Storage TA)
D/TC:? 0 ldelf_syscall_open_bin:146 res=0xffff0008
D/TC:? 0 ldelf_syscall_open_bin:142 Lookup user TA ELF 8aaaf200-2450-11e4-abe2-0002a5d5c51b (REE)
D/TC:? 0 ldelf_syscall_open_bin:146 res=0
D/LD:  ldelf:168 ELF (8aaaf200-2450-11e4-abe2-0002a5d5c51b) at 0x40087000
D/TA:  TA_CreateEntryPoint:39 has been called
D/TA:  TA_OpenSessionEntryPoint:68 has been called
I/TA: Hello World!
D/TA:  inc_value:105 has been called
I/TA: Got value: 42 from NW
I/TA: Increase value to: 43
D/TC:? 0 tee_ta_close_session:512 csess 0xbded1c90 id 1
D/TC:? 0 tee_ta_close_session:531 Destroy session
I/TA: Goodbye!
D/TA:  TA_DestroyEntryPoint:50 has been called
D/TC:? 0 destroy_context:308 Destroy TA ctx (0xbded1c30)
```
![SEure_disp_ta_msg](https://user-images.githubusercontent.com/36186082/147338117-7e5013f0-d54e-4ce4-b287-f1188d1a3100.png)

# 2. OP-TEE Test for String/data Encryption by genrating RSA key pair.
  
Normal World console:
  
```ruby
# optee_example_acipher 
optee_example_acipher: Unexpected number of arguments 0 (expected 2)
usage: optee_example_acipher <key_size> <string to encrypt>
# 
# optee_example_acipher 256 "Testing"
Encrypted buffer: 84 68 a9 c5 1a 39 93 25 ab 1a 4a 98 c5 89 4f 7f 59 79 8b 8d e9 9e e1 4d 12 34 89 48 72 00 b4 40 
# 
```
Secure World Console:

```ruby
D/TC:? 0 tee_ta_init_pseudo_ta_session:299 Lookup pseudo TA a734eed9-d6a1-4244-aa50-7c99719e7b7b
D/TC:? 0 ldelf_load_ldelf:91 ldelf load address 0x40006000
D/LD:  ldelf:134 Loading TS a734eed9-d6a1-4244-aa50-7c99719e7b7b
D/TC:? 0 ldelf_syscall_open_bin:142 Lookup user TA ELF a734eed9-d6a1-4244-aa50-7c99719e7b7b (Secure Storage TA)
D/TC:? 0 ldelf_syscall_open_bin:146 res=0xffff0008
D/TC:? 0 ldelf_syscall_open_bin:142 Lookup user TA ELF a734eed9-d6a1-4244-aa50-7c99719e7b7b (REE)
D/TC:? 0 ldelf_syscall_open_bin:146 res=0
D/LD:  ldelf:168 ELF (a734eed9-d6a1-4244-aa50-7c99719e7b7b) at 0x40042000
E/TA:  cmd_enc:100 TEE_AsymmetricEncrypt(7, 0): 0xffff0010
D/TC:? 0 tee_ta_close_session:512 csess 0xbded1c90 id 1
D/TC:? 0 tee_ta_close_session:531 Destroy session
D/TC:? 0 destroy_context:308 Destroy TA ctx (0xbded1c30)
```
# 3. OP-TEE AES encryption Example.
  
Normal World console:
  
```ruby
# optee_example_aes 
Prepare session with the TA
Prepare encode operation
Load key in TA
Reset ciphering operation in TA (provides the initial vector)
Encode buffer from TA
Prepare decode operation
Load key in TA
Reset ciphering operation in TA (provides the initial vector)
Decode buffer from TA
Clear text and decoded text match
# 
# 
# ls -al /data/tee/
total 40
drwxrwx---    2 tee      tee              0 Dec 24 09:14 .
drwxr-xr-x    3 root     root             0 Dec 24 07:33 ..
-rw-------    1 tee      tee           4228 Dec 24 09:14 0
-rw-------    1 tee      tee          16384 Dec 24 09:14 1
-rw-------    1 tee      tee          16384 Dec 24 09:14 dirf.db
# 
```
For information about /data/tee/0, /data/tee/1, /data/tee/dirf.db go through below link:
  
https://optee.readthedocs.io/en/latest/architecture/secure_storage.html#tee-file-structure-in-linux-file-system
  
 
Secure World Console:

```ruby
D/TC:? 0 tee_ta_init_pseudo_ta_session:299 Lookup pseudo TA 5dbac793-f574-4871-8ad3-04331ec17f24
D/TC:? 0 ldelf_load_ldelf:91 ldelf load address 0x40006000
D/LD:  ldelf:134 Loading TS 5dbac793-f574-4871-8ad3-04331ec17f24
D/TC:? 0 ldelf_syscall_open_bin:142 Lookup user TA ELF 5dbac793-f574-4871-8ad3-04331ec17f24 (Secure Storage TA)
D/TC:? 0 ldelf_syscall_open_bin:146 res=0xffff0008
D/TC:? 0 ldelf_syscall_open_bin:142 Lookup user TA ELF 5dbac793-f574-4871-8ad3-04331ec17f24 (REE)
D/TC:? 0 ldelf_syscall_open_bin:146 res=0
D/LD:  ldelf:168 ELF (5dbac793-f574-4871-8ad3-04331ec17f24) at 0x40020000
D/TA:  TA_OpenSessionEntryPoint:394 Session 0x4003d4c0: newly allocated
D/TA:  alloc_resources:124 Session 0x4003d4c0: get ciphering resources
D/TA:  set_aes_key:240 Session 0x4003d4c0: load key material
D/TA:  reset_aes_iv:308 Session 0x4003d4c0: reset initial vector
D/TA:  cipher_buffer:340 Session 0x4003d4c0: cipher buffer
D/TA:  alloc_resources:124 Session 0x4003d4c0: get ciphering resources
D/TA:  set_aes_key:240 Session 0x4003d4c0: load key material
D/TA:  reset_aes_iv:308 Session 0x4003d4c0: reset initial vector
D/TA:  cipher_buffer:340 Session 0x4003d4c0: cipher buffer
D/TC:? 0 tee_ta_close_session:512 csess 0xbded1c90 id 1
D/TC:? 0 tee_ta_close_session:531 Destroy session
D/TA:  TA_CloseSessionEntryPoint:404 Session 0x4003d4c0: release session
D/TC:? 0 destroy_context:308 Destroy TA ctx (0xbded1c30)
```
# 2. Generating HMAC based One Time Passwords securely.
  
Normal World console:
  
```ruby
# optee_example_hotp
Register the shared key: 12345678901234567890��
HOTP: 755224
HOTP: 287082
HOTP: 359152
HOTP: 969429
HOTP: 338314
HOTP: 254676
HOTP: 287922
HOTP: 162583
HOTP: 399871
HOTP: 520489
# 
```
Secure World Console:

```ruby
D/TC:? 0 tee_ta_init_pseudo_ta_session:299 Lookup pseudo TA 484d4143-2d53-4841-3120-4a6f636b6542
D/TC:? 0 ldelf_load_ldelf:91 ldelf load address 0x40006000
D/LD:  ldelf:134 Loading TS 484d4143-2d53-4841-3120-4a6f636b6542
D/TC:? 0 ldelf_syscall_open_bin:142 Lookup user TA ELF 484d4143-2d53-4841-3120-4a6f636b6542 (Secure Storage TA)
D/TC:? 0 ldelf_syscall_open_bin:146 res=0xffff0008
D/TC:? 0 ldelf_syscall_open_bin:142 Lookup user TA ELF 484d4143-2d53-4841-3120-4a6f636b6542 (REE)
D/TC:? 0 ldelf_syscall_open_bin:146 res=0
D/LD:  ldelf:168 ELF (484d4143-2d53-4841-3120-4a6f636b6542) at 0x4002e000
D/TA:  register_shared_key:150 Got shared key 12345678901234567890 (20 bytes).
D/TA:  get_hotp:182 HOTP is: 755224
D/TA:  get_hotp:182 HOTP is: 287082
D/TA:  get_hotp:182 HOTP is: 359152
D/TA:  get_hotp:182 HOTP is: 969429
D/TA:  get_hotp:182 HOTP is: 338314
D/TA:  get_hotp:182 HOTP is: 254676
D/TA:  get_hotp:182 HOTP is: 287922
D/TA:  get_hotp:182 HOTP is: 162583
D/TA:  get_hotp:182 HOTP is: 399871
D/TA:  get_hotp:182 HOTP is: 520489
D/TC:? 0 tee_ta_close_session:512 csess 0xbded1c90 id 1
D/TC:? 0 tee_ta_close_session:531 Destroy session
D/TC:? 0 destroy_context:308 Destroy TA ctx (0xbded1c30)
```
 
