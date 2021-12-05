Assignment 1: Discovering vmx features

Team members - Teja Ganapati Jaddipal(SJSU ID-015957526)

Steps Followed to complete  assignment 1.


1. Install vmware fusion on mac
2. Install Ubuntu 20.04.3 in vmware fusion
3. Create Github account on github.com using preferred email id.
4. Goto the master linux github repository - https://github.com/torvalds/linux
5. Fork the master linux github repository using the fork button at top right corner of the page. Once the fork is complete, the repository will be available in your github account.
6. Next step is to clone the repository to your local machine. Cloning can be done in different ways. I am doing it through git, git must be installed first(if not present already) before cloning.
7. Git can be installed using command
   - sudo apt install git
8. You will be asked for password and provide your Ubuntu user account password and press enter to begin installation process.
9. To make sure that the installation is complete, you can type "git --version" command in terminal and check the version of git. If git is installed correctly, the version will be displayed.
10. GitHub uses your commit email address to associate commits with your account on GitHub.com. Setup username and email for your git using the following commands. 
    - to setup git username - $ git config --global user.name "Teja"
    - to setup git email id - $ git config --global user.email "teja.jaddipal@gmail.com"

More detailed informationcan be found in https://docs.github.com/en/get-started/quickstart/set-up-git#next-steps-authenticating-with-github-from-git. To make commits from git the primary email id associated with Github account must be same as the email id that is used with git. If you want to use a different email id with git, add that email id to your github.com account.

11. Below commands install packages that are helpful during kernel compilation process

    - sudo apt install htop (used to monitor system resources)
    - sudo apt install gparted (to resize boot partition if necessary)

During vm setup vm was only allocated 20GB of disk space, 4GB ram and 2 cpu cores. Change this to the values you want. I increased disk space from 20GB to 200GB, allocated 6 cpu cores and 8 GB of ram. You may need to pause /stop the vm to do this. Also enable nested virtualization option for the vm. After this, I used gparted to increase the boot partition size. During previous tries to compile the kernel, disk space ran out and compilation stopped so it is necessary to a have large enough disk space.

12. Clone the repository that you just forked(cloning steps described below) to your local machine (the repository that is in your github account, not from linux master repository).
13. In terminal change the current working directory to the directory where you want to clone the repository using "cd" command
14. In github.com goto your linux repository. Click on the "Code" button(green color button). There will be 3 options to clone the repository, since we are using git to clone the repository over HTTPS,select HTTPS option and copy the repo link(in my case link is- https://github.com/gthreehosmane/linux.git). Make sure that you are cloning the repository from correct place(repository under your account, not the torvalds/linux repository). More detailed information can be found in official guide https://docs.github.com/en/get-started/quickstart/fork-a-repo and https://docs.github.com/en/get-started/getting-started-with-git/about-remote-repositories#cloning-with-https-urls
15. In terminal, once you are in the directory, where you want to clone the repository type "git clone" and after that add a space and paste the link that you copied from previous step and press enter. The cloning should start after this. Cloning may take some time to complete.


16. Installing packages necessary for kernel compilation and installation: During code compilation, I encountered errors for each missing library, so after multiple tries to compile the code, made list of libraries required.

     - sudo apt-get update
     - sudo apt-get upgrade
     - sudo apt-get install build-essential
     - sudo apt-get install libelf-dev
     - sudo apt-get install flex
     - sudo apt-get install bison
     - sudo apt-get install libncurses5-dev gcc make git exuberant-ctags bc libssl-dev
     - sudo apt install dwarves (to resolve CONFIG_DEBUG_INFO_BTF related error)

17. steps to build kernel

     - goto the linux folder(linux folder will be created after cloning is complete)
     - make menuconfig - no changes were made
     - copy the current config file to a new config file in current directory(Config version may vary based on the linux distro you are using). command uname -a can be used to check the current config using command 
     - cp /boot/config-5.11.0-38-generic  .config
     - make oldconfig
     - make prepare
     - created following files manually**
       - x509.genkey file under /linux/certs
       - signing_key.pem file under/linux/certs 
       - canonical-certs.pem file under /linux/debian  
       - canonical-revoked-certs.pem file under /linux/debian
     - make -j 6 modules (replace 6 with the number of vcpus you have allocated to the vm - this is for compiling modules in parallel)
     - make -j 6
     - sudo make modules_install
     - sudo make install
     - sudo reboot
18. Once the reboot is successful, you can check if you are using newly compiled kernel by doing uname -a command in terminal. If the kernel version has changed to the latest one(in my case the version changed from 5.11.0-38-generic to 5.15.0+) And the time stamp should also reflect the date and time on which you have compiled and installed the kernel(updated timestamp- Linux ubuntu 5.15.0+ #2 SMP PREEMPT Sat Nov 6 10:46:00 PDT 2021).

19. Create a folder 283-1 inside linux folder and add cmpe283-1.c and Makefile to this folder. 
20. Goto 283-1 folder and use  make command to generate cmpe283-1.ko file and check if cmpe283-1.ko is generated using command - ls | grep *.ko
     - if you get error regarding missing license add this line at the end of cmpe283-1.c file - MODULE_LICENSE("GPL v2");
21. Use command 
     - sudo insmod cmpe283-1.ko to load module and check if module loaded using command 
     - lsmod | grep cmpe283
22. Use command dmesg to check vmx features
23. Commit cmpe283-1.c and Makefile to your git repository. Make sure you commit to the correct repository. The repository you are commiting can be found out using command
     - git remote -v



** I encountered errors during make modules_install step regarding missing signing key certificates(This may not happen in all linux distros). I created following missing certificates manually.
     
     - Create x509.genkey file under /linux/certs using vi command in terminal
     - Contents of the x509.genkey file
     
	[ req ]
	default_bits = 4096
	distinguished_name = req_distinguished_name
	prompt = no
	string_mask = utf8only
	x509_extensions = myexts

	[ req_distinguished_name ]
	O = teja
	CN = teja signing key
	emailAddress = XXXobscuredXXX

	[ myexts ]
	basicConstraints=critical,CA:FALSE
	keyUsage=digitalSignature
	subjectKeyIdentifier=hash
	authorityKeyIdentifier=keyid
        
        
     -  To create signing_key.pem file under/linux/certs goto the certs folder in linux folder, open a terminal and use the below command
     - openssl req -new -nodes -utf8 -sha512 -days 36500 -batch -x509 -config x509.genkey -outform DER -out signing_key.x509 -keyout signing_key.pem
      
      
        
     -  Create canonical-certs.pem under /linux/debian using vi command in terminal(If debian folder does not exists create on under linux folder)
      Contents of canonical-certs.pem - Please refer this link - https://salsa.debian.org/kernel-team/linux/-/blob/master/debian/certs/debian-uefi-certs.pem
      
      
     -  Create canonical-revoked-certs.pem under /linux/debian using vi command in terminal(If debian folder does not exists create on under linux folder)
      Contents of canonical-revoked-certs.pem - Please refer this link - https://salsa.debian.org/kernel-team/linux/-/blob/master/debian/certs/debian-uefi-certs.pem

Screenshot of the output
<img width="1057" alt="283-1-01" src="https://user-images.githubusercontent.com/13237444/141885271-f638c61a-0bf0-4ceb-91bd-c8c5a92c41d2.png">
<img width="1021" alt="283-1-02" src="https://user-images.githubusercontent.com/13237444/141885330-2a321549-a2fc-4eda-950c-5f43493531ab.png">


This steps are followed for Ubuntu 20.04.3 vm on vmware fusion. You may need to install additional packages for successful kernel compilation depending on the linux distribution you are using and the setup



Assignment 2: Instrumentation via hypercall - Special CPUID leaf nodes - 0x4FFFFFFF and 0x4FFFFFFE

Team members - Teja Ganapati Jaddipal(SJSU ID-015957526)

Steps Followed to complete the assignment 2.

Prerequisite - Working assignment 1

1. Goto path-to-linux-folder-/linux/arch/x86 
2. Modify the vmx.c and cpuid.c by adding required code to handle special CPUID leaf nodes 0x4FFFFFFF and 0x4FFFFFFE. vmx.c will be located in ../x86/kvm/vmx/vmx.c and cpuid.c in ../x86/kvm/cpuid.c

   a. Initially declared total_exits and total_time_in_vmm variables in vmx.c, exported them using EXPORT_SYMBOL macro,used them as extern variables in cpuid.c but       got cyclic dependency error between kvm and kvm_intel modules during make modules_install step. Because of this error global variables were not exported. So 	     as a work around declared global variables in cpuid.c and used them as extern variables in vmx.c. It is also necessary to initialize these variables during         declaration to avoid undefined variable error during compilation. Cyclic dependency error is due to the reason that, cpuid.c is in kvm module and vmx.c is in       kvm_intel module. kvm_intel modules is dependent on kvm module. When we use use a global variable in cpuid.c, which is declared and exported in vmx.c, a           cyclic dependency is created in my case. 
   
   b. Could not use __rdtsc() function from intel intrinsic to read the time as it was causing some errors. So used inline assembly code snippet to read time.
  
	// reference https://www.mcs.anl.gov/~kazutomo/rdtsc.html
	
	
			static __inline__ unsigned long long read_time(void)
			{
			  unsigned hi, lo;
			  __asm__ __volatile__ ("rdtsc" : "=a"(lo), "=d"(hi));
			  return ( (unsigned long long)lo)|( ((unsigned long long)hi)<<32 );
			}
	
    
    c. For printk macro to work we need to add kernel.h header in cpuid.c using #include <linux/kernel.h>
    
3. Follow below steps to compile the modified code, if there are mistakes in the code, you will get errors. Make sure the code compiles without any errors.
     - sudo make -j 6 (Replace 6 with number of vcpus allocated to your vm)
     - sudo make modules_install
     - sudo make install
     - sudo rmmod kvm_intel (Remove existing kvm and kvm_intel modules. kvm module cannot be removed first since it is kvm_intel module is depending on kvm module, so kvm_intel must be removed first.)
     - sudo rmmod kvm
     - sudo modprobe kvm (load the modules with newly added code.)
     - sudo modprobe kvm_intel
     - Check if modules are loaded using command  
        - $lsmod | grep kvm
     - We can also check if global variables are exported correctly using command 
        - $cat /proc/kallsyms | grep 'variable name'

4. To test the code which is added, we need to create a new VM inside our currently running VM(Ubuntu, the outer VM).
5. Inner VM can be created by installing virt-manager and other required libraries on Ubuntu, the outer VM.
6. Follow below steps to install and create a new inner VM inside outer VM. I followed the steps mentioned in https://medium.com/codemonday/setup-virt-manager-qemu-libvert-and-kvm-on-ubuntu-20-04-fa448bdecde3 and  https://linuxize.com/post/how-to-install-kvm-on-ubuntu-20-04/

     - sudo apt update
     - sudo apt install cpu-checker
     - kvm-ok (This command is to check if our outer VM can run a VM inside itself - nested virtualization)
     - sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst virt-manager (To install virt-manager and libraries)
     - sudo systemctl is-active libvirtd (See if libvertd daemon is active)
    Add user group and permissions with 
     - sudo usermod -aG kvm $USER
     - sudo usermod -aG libvirt $USER
     - newgrp libvirt

7. Start virt-manager using command - $virt-manager
8. Follow the onscreen instructions to set up inner VM. I followed steps provided in this link - https://linuxize.com/post/how-to-install-kvm-on-ubuntu-20-04/
9. I changed the cpu configuration to kvm64 from host configuration(By default this will be selected). This has to be done before inner vm installation. I was facing some errors for using host configuration(which is the default option), to resolve error I selected kvm64 cpu configuration.
10. For this assignment, I installed fedora as inner VM
   - Install cpuid package using command - $sudo dnf install cpuid
   - Run the commands (-l option for the cpuid command specifies the leaf node(eax value))
     - $cpuid -l 0x4fffffff 
     - $cpuid -l 0x4ffffffe
   
   

11. Check the message buffer of outer VM using $dmesg command

12. You may want disable autostart default network in kvm else you wont be able to restart inner VM once you shutdown outer VM(I have faced this issue).

Detailed information about kvm installation and setting up a VM on Ubuntu can be found here.

1. https://medium.com/codemonday/setup-virt-manager-qemu-libvert-and-kvm-on-ubuntu-20-04-fa448bdecde3
2. https://linuxize.com/post/how-to-install-kvm-on-ubuntu-20-04/

Output screenshots
<img width="1702" alt="283-2-01" src="https://user-images.githubusercontent.com/13237444/141885425-34a9b21a-fae6-4f95-bd6f-d0901d49b94d.png">
<img width="1705" alt="283-2-02" src="https://user-images.githubusercontent.com/13237444/141885476-8231176c-7815-4bf9-9081-7b3231aa5311.png">
<img width="1708" alt="283-2-03" src="https://user-images.githubusercontent.com/13237444/141885509-90a12a97-171d-4b46-8fdb-b97d39f5f2b8.png">


Assignment 2 is completed using VMware Fusion, Ubuntu 20.04 as outer VM and Fedora 35 as inner VM. You may need to modify the steps above if using different outer VM and inner VM.


Assignment 3: Instrumentation via hypercall - Special CPUID leaf nodes - 0x4FFFFFFD and 0x4FFFFFFC

Team members - Teja Ganapati Jaddipal(SJSU ID-015957526)

Comments on frequency of exits:

1. Number of exits do not increase at a stable rate. Only certain exits are common. More exits will happen in operations like installing new packages which require memory and network access. A full vm boot resulted in 956897 exits in my case.
2. Most frequent exits:

- Exit 32 : WRMSR.
- Exit 48 : EPT violation
- Exit 49 : EPT misconfiguration
- Exit 12 : HLT 
- Exit 1 : External interrupt.

3. Least frequent exits: 

- Exit 29 : MOV DR  
- Exit 54 : WBINVD or WBNOINVD. 
- Exit 31 : RDMSR
- Exit 0 : Exception or non-maskable interrupt (NMI).



Steps Followed to complete the assignment 3.

Prerequisite - Working assignment 1 and 2

1. Goto path-to-linux-folder-/linux/arch/x86 
2. Modify the vmx.c and cpuid.c by adding required code to handle special CPUID leaf nodes 0x4FFFFFFD and 0x4FFFFFFC. vmx.c will be located in ../x86/kvm/vmx/vmx.c and cpuid.c in ../x86/kvm/cpuid.c

   a. Declared and exported the global variables vm_exits_per_reason and time_per_exit_type in cpuid.c and used them in vmx.c using extern keyword to avoid cyclic dependency errors.
   
   b. Reused inline assembly code snippet to read time from assignment 2.
  
	// reference https://www.mcs.anl.gov/~kazutomo/rdtsc.html
	
	
			static __inline__ unsigned long long read_time(void)
			{
			  unsigned hi, lo;
			  __asm__ __volatile__ ("rdtsc" : "=a"(lo), "=d"(hi));
			  return ( (unsigned long long)lo)|( ((unsigned long long)hi)<<32 );
			}
	
    
    
3. Follow below steps to compile the modified code, if there are mistakes in the code, you will get errors. Make sure the code compiles without any errors.
     - sudo make -j 6 (Replace 6 with number of vcpus allocated to your vm)
     - sudo make modules_install
     - sudo make install
     - sudo rmmod kvm_intel (Remove existing kvm and kvm_intel modules. kvm module cannot be removed first since it is kvm_intel module is depending on kvm module, so kvm_intel must be removed first.)
     - sudo rmmod kvm
     - sudo modprobe kvm (load the modules with newly added code.)
     - sudo modprobe kvm_intel
     - Check if modules are loaded using command  
        - $lsmod | grep kvm
     - We can also check if global variables are exported correctly using command 
        - $cat /proc/kallsyms | grep 'variable name'

4. To test the code which is added, we need to create a VM inside our currently running VM(Ubuntu, the outer VM). We have installed kvm and created a inner vm alreday in assignment 2. So, the same inner VM can be reused to test this assignment.
5. Start virt-manager using command - $virt-manager

6. For this assignment, I used fedora as inner VM
   - cpuid package is already installed in inner vm in assignment 2.
   - Run the commands below with different exit types (-l option for the cpuid command specifies the leaf node(eax value) and -s specifies the subleaf node(ecx value)).
   	- $cpuid -l 0x4ffffffd -s{exit_type} 
   	- $cpuid -l 0x4ffffffc -s{exit_type} 
   

7. Check the message buffer of outer VM using $dmesg command



Detailed information about kvm installation and setting up a VM on Ubuntu can be found here.

1. https://medium.com/codemonday/setup-virt-manager-qemu-libvert-and-kvm-on-ubuntu-20-04-fa448bdecde3
2. https://linuxize.com/post/how-to-install-kvm-on-ubuntu-20-04/

Output screenshots

<img width="1699" alt="Screen Shot 2021-11-29 at 12 40 03 PM" src="https://user-images.githubusercontent.com/13237444/143941165-b9d66c79-1b3a-4bed-a6f7-43c1e549088f.png">
<img width="1640" alt="Screen Shot 2021-11-29 at 12 40 35 PM" src="https://user-images.githubusercontent.com/13237444/143941352-d022177f-da49-47e4-ab34-333797b99691.png">
<img width="1503" alt="Screen Shot 2021-11-29 at 12 46 28 PM" src="https://user-images.githubusercontent.com/13237444/143941394-ea326467-1cd3-4937-a6d9-58e3ac66dc23.png">
<img width="1529" alt="Screen Shot 2021-11-29 at 12 47 31 PM" src="https://user-images.githubusercontent.com/13237444/143941451-f6a41f54-167d-4d6e-af7a-299d0f18dac1.png">
Assignment 3 is completed using VMware Fusion, Ubuntu 20.04 as outer VM and Fedora 35 as inner VM. You may need to modify the steps above if using different outer VM and inner VM.





Assignment 4: Nested Paging vs. Shadow Paging

Team members - Teja Ganapati Jaddipal(SJSU ID-015957526)


Prerequisite - Working assignment 3

Steps followed - 

1. Ran assignment 3 code and booted inner VM using that code.
2. Once the VM has booted, recorded total exit count information (total count for each type of exit handled by KVM) using a series of queries of CPUID leaf function
0x4FFFFFFD. 
   - $cpuid -l 0x4ffffffd -s{exit_type}
3. Tured off inner VM.
4. Remove the ‘kvm-intel’ module from your running kernel using command
   - $sudo rmmod kvm-intel
5. Reload the kvm-intel module with the parameter ept=0  and vpid=0 (this will disable nested paging and force KVM to use shadow paging instead). This is done using command 
   - $insmod /lib/modules/5.15.0+/kernel/arch/x86/kvm/kvm-intel.ko ept=0 vpid=0
6. Booted the same test VM again, and recorded total exit count information (total count for each type of exit handled by KVM) using a series of queries of CPUID leaf function 0x4FFFFFFD. 
   - $cpuid -l 0x4ffffffd -s{exit_type}

Things to note - 
1. As per assignment 3 description and implementation, 0x4ffffffd is used to record total exits for each exit type not 0x4FFFFFFE(in my case).
2. The version of new kernel that is built in assignment 1 is 5.15.0+ so, I replaced XXX with 5.15.0+
3. When I used rmmod and insmod commands without sudo I got permission denied error. So I had to run those commands with sudo.
4. When I reloaded kvm_intel module with ept=0, It was not possible to boot up inner vm, it crashed frequently. So after little bit of research I got to know that adding vpid=0 parameter along with ept=0 could work. Tried that using below command and the inner vm booted successfully. Inner vm was very slow with shadow paging enabled.
   - $insmod /lib/modules/5.15.0+/kernel/arch/x86/kvm/kvm-intel.ko ept=0 vpid=0


<img width="1258" alt="283-4-08" src="https://user-images.githubusercontent.com/13237444/144732917-9fdf9db0-671d-43c2-9b2d-02f6edb7f9a1.png">

   
   
Sample from demsg

With ept - 

<img width="1691" alt="283-4-01" src="https://user-images.githubusercontent.com/13237444/144732872-a5716d8b-b671-4ed3-9779-15a50e851e18.png">

<img width="1696" alt="283-4-02" src="https://user-images.githubusercontent.com/13237444/144732878-a8a1e23e-9f10-44b2-82ee-1ca4d3ff84af.png">

<img width="1695" alt="283-4-05" src="https://user-images.githubusercontent.com/13237444/144732906-de9670fb-90b4-461e-9a30-1cb12ab2248f.png">

<img width="1704" alt="283-4-07" src="https://user-images.githubusercontent.com/13237444/144733038-11414f31-1d53-4234-8bcc-25a38b730b7b.png">



Without ept

<img width="1689" alt="283-4-09" src="https://user-images.githubusercontent.com/13237444/144732948-db348e2c-8782-451b-8838-56f88cc5be58.png">

<img width="1667" alt="283-4-11" src="https://user-images.githubusercontent.com/13237444/144732987-ac41b62a-7b90-4c7d-8b18-d5978798d29d.png">

<img width="1674" alt="283-4-14" src="https://user-images.githubusercontent.com/13237444/144733006-1d16bf8b-0002-4d39-b3f7-00dc07af1efd.png">

<img width="1681" alt="283-4-18" src="https://user-images.githubusercontent.com/13237444/144733030-16afaff0-a381-4a38-b207-2de03fca1871.png">



About exit counts - 

What did you learn from the count of exits? Was the count what you expected? If not, why not?

- I was expecting the that the exit counts will be increased by a few thousands in case of shadow paging since there will be  more page faults, but to my surprise some exit counts were around 3 times more than that of nested paging. Also, there were 2 new exits types, this is because, for shadow paging to work correctly more exit types needs to be enabled than nested paging.

Exit types that are around 3 times more than that of nested paging.

 - Exit 0 : Exception or NMI
 - Exit 1 : External interrupt
 - Exit 7 : Interrupt window.
 - Exit 12 : HLT
 - Exit 28 : Control-register accesses.
 - Exit 32 : WRMSR
 
 
Also, there were 2 new type of exits that occured in shadow paging.
- Exit 14 : INVLPG 
     - This is because exits on TLB flushes is enabled in shadow paging. 
- Exit 33 : VM-entry failure due to invalid guest state. 
     - Many VM-entry failures happened because checks performed on few of the guest state registers failed.
 

What changed between the two runs (ept vs no-ept)?
-  As far as my understanding, shadow paging requires more exits to be enabled compared to that of nested paging. These exits include exits on %cr3 read and write, exits on page faults occuring in shadow page table and guest page table, exits on TLB flushes to remove stale entries when there is a free. This is the reason for increase in the number of exits.
   
   
Assignment 4 is completed using VMware Fusion, Ubuntu 20.04 as outer VM and Fedora 35 as inner VM. You may need to modify the steps above if using different outer VM and inner VM.




 










 






