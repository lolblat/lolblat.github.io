# Unpacking BattleEye - Article 1

## Introduction

Hello everyone, this is the first article in a series of articles about unpacking and reversing BattleEye.
The first few article will be about unpacking BattleEye application. These articles are works-in-progress.

Our target, the BattleEye application, is packed with VMProtect. We know that from past researches of BattleEye unpacking.

## Unpacking the protection
### So what is VMProtect?
VMProtect is a commercial software that offers packing and protection for an application. VMProtect is a virtual machine packer.
Regular packers compress the data of the application then decompress it on run time using the stub. The virtual machine packing, of the other hand, creates a virtual CPU with custom opcodes and convert your application's assembly to run on that special virtual CPU.
Because the VM based protection changes the opcodes, it makes reverse engineering extremely complicated as you need to reverse engineer each custom opcode.

On top of changing all the opcodes, the virtual CPU has fake opcodes, jumps and calls (instructions that do nothing). Before getting deep into reversing we need to filter out all these.

After filtering out the useless instructions, we can create a quick summery of the VM itself. The VM creates its own registers and its own stack. The stack is stored inside the EBP register and the registers are stored inside the EDI register. The registers' stracture is essentially an array on the original program's stack (not on the VM's stack). The EDI register is the address of the array.

We have another 2 more important registers; we have the ESI register, and the EBX register. The ESI holds the VM instruction pointer, and the EBX register holds the encryption key (we will take a look at the encryption mechanism and how it affects the program later).

We can split the VM operation to 3 steps: 
1. Initialization of the VM
2. Getting and dispatching the command
3. Running the command

### Step 1 - initialization of the VM
In this step the program creates the stack of the VM and the registers array, stores the VM instruction pointer inside the ESI register and finally creates the encryption key.

### Step 2 - where we are getting and dispatching the command
We are getting the opcode from the VM Instruction pointer, the opcode is the size of 1 byte, We are incrementing the VM Instruction pointer, after We have the opcode, We start to decode the opcode, We are XORing the opcode with the encryption key, and do some more mathematical operation on the opcode, then We are XORing the encryption key with the decoded opcode, We are getting the handler of the opcode from hander's table, pushing the handler into the stack, and dispatching the opcode by returning into the handler.

Step 3: just running the command, and loop into the second step.

We are getting the command from the VM instruction pointer

![first-image](..\images\1569911824139.png)

We are getting the handler, and dispatching it.

![second-image](..\images\1569911974632.png)

## Reverse the unpacking

Now that we know how the VM is working, we can start to reverse engineering the opcodes and the application itself, I created an emulator to emulate the application, I passed over each opcode and reverse it.

The application loading `kernel32.dll`, and get the `VirtualProtect` function, afterwards changes the protection of the application segments, to be able to write to them, afterwards the application loading function, encrypt their loading address, and write it into the application. After all the loading, the application call `VirtualProtect` again on the application segments, and return their protection to their old ones.

The next step is that we return into unpacked code, the unpacked code is initializing global variables, the initializing code is using the loaded functions.

Most of the work in this part, was to reverse each opcode, and create the emulator.

## What's next and used tools

In light of this article representing the start of a series, you can expect more reversing and fun to come. I've came to the decision to create a debugger plugin for `x64dbg` to be able to debug the VM more easily, this plugin will be presented in the next article.

I will upload all my tools into my [github](https://github.com/lolblat/Tools/tree/master/BattleEye)

Thank you for reading :).



