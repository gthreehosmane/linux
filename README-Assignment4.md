Assignment 4: Nested Paging vs. Shadow Paging

Team members - Teja Ganapati Jaddipal(SJSU ID-015957526)


Prerequisite - Working assignment 3

Steps followed - 

1. Ran assignment 3 code and booted inner VM using that code.
2. Once the VM has booted, recorded total exit count information (total count for each type of exit handled by KVM) using a series of queries of CPUID leaf function
0x4FFFFFFD. 
   - cpuid -l 0x4ffffffd -s{exit_type}
3. Tured off inner VM.
4. Remove the ‘kvm-intel’ module from your running kernel using command
   - sudo rmmod kvm-intel
5. Reload the kvm-intel module with the parameter ept=0  and vpid=0 (this will disable nested paging and force KVM to use shadow paging instead). This is done using command 
   - insmod /lib/modules/5.15.0+/kernel/arch/x86/kvm/kvm-intel.ko ept=0 vpid=0
6. Booted the same test VM again, and recorded total exit count information (total count for each type of exit handled by KVM) using a series of queries of CPUID leaf function 0x4FFFFFFD. 
   - cpuid -l 0x4ffffffd -s{exit_type}

Things to note - 
1. As per assignment 3 description and implementation, 0x4ffffffd is used to record total exits for each exit type not 0x4FFFFFFE(in my case).
2. The version of new kernel that is built in assignment 1 is 5.15.0+ so, I replaced XXX with 5.15.0+
3. When I used rmmod and insmod commands without sudo I got permission denied error. So I had to run those commands with sudo.
4. When I reloaded kvm_intel module with ept=0, It was not possible to boot up inner vm, it crashed frequently. So after little bit of research I got to know that adding vpid=0 parameter along with ept=0 could work. Tried that using below command and the inner vm booted successfully. Inner vm was very slow with shadow paging enabled.
   - insmod /lib/modules/5.15.0+/kernel/arch/x86/kvm/kvm-intel.ko ept=0 vpid=0
   
   
Sample from demsg

With ept

Without ept

About exit counts - 

What did you learn from the count of exits? Was the count what you expected? If not, why not?

- I was expecting the that the exit counts will be increased by a few thousands in case of shadow paging since there will be  more page faults, but to my surprise some exit counts were around 3 times more than that of nested paging. 

Exit types that are around 3 times more than that of nested paging.

 - Exit 0 : Exception or NMI
 - Exit 1 : External interrupt
 - Exit 7 : Interrupt window.
 - Exit 12 : HLT
 - Exit 28 : Control-register accesses.
 - Exit 32 : WRMSR
 
 
Also, there were 2 new type of exits that occured in shadow paging.
- Exit 14 : INVLPG 
       - Since exits on TLB flushes is enabled in shadow paging. 
- Exit 33 : VM-entry failure due to invalid guest state. 
       - Many VM-entry failures happened because checks performed on few of the guest state registers failed.
 

What changed between the two runs (ept vs no-ept)?
-  As far as my understanding, shadow paging requires more exits to be enabled compared to that of nested paging. These exits include exits on %cr3 read and write, exits on page faults occuring in shadow page table and guest page table, exits on TLB flushes to remove stale entries when there is a free. This is the reason for increase in the number of exits.
   
   
Assignment 4 is completed using VMware Fusion, Ubuntu 20.04 as outer VM and Fedora 35 as inner VM. You may need to modify the steps above if using different outer VM and inner VM.



 








 





