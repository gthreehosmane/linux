Team members - Teja Ganapati Jaddipal(SJSU ID-015957526)

Steps Followed to complete the assignment.


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

During vm setup vm was only allocated 20GB of disk space, 4GB ram and 2 cpu cores. Change this to the values you want. I increased disk space from 20GB to 200GB, allocated 6 cpu cores and 8 GB of ram. You may need to pause /stop the vm to do this. Also enable nested virtualization option for the vm. After this I used gparted to increase the boot partition size. During previous tries to compile the kernel disk space ran out and compilation stopped so it is necessary to a have large enough disk space.

12. Clone the repository that you just forked(cloning steps described below) to your local machine (the repository that is in your github account, not from linux master repository).
13. In terminal change the current working directory to the directory where you want to clone the repository using "cd" command
14. In github.com goto your linux repository. Click on the "Code" button(green color button). There will be 3 options to clone the repository, since we are using git to clone the repository over HTTPS,select HTTPS option and copy the repo link(in my case link is- https://github.com/gthreehosmane/linux.git). Make sure that you are cloning the repository from correct place(repository under your account, not the torvalds/linux repository). More detailed information can be found in official guide https://docs.github.com/en/get-started/quickstart/fork-a-repo and https://docs.github.com/en/get-started/getting-started-with-git/about-remote-repositories#cloning-with-https-urls
15. In terminal, once you are in the directory, where you want to clone the repository type "git clone" and after that add a space and paste the link that you copied from previous step and press enter. The cloning should start after this. Cloning may take some time to complete.


16. Installing packages necessary for kernel compilation and installation

sudo apt-get update
sudo apt-get upgrade
sudo apt-get install build-essential
sudo apt-get install libelf-dev
sudo apt-get install flex
sudo apt-get install bison
sudo apt-get install libncurses5-dev gcc make git exuberant-ctags bc libssl-dev

17. steps to build kernel

    - goto the linux folder(linux folder will be created after cloning is complete)
    - make menuconfig - no changes were made
    - copy the current config file to a new config file in current directory(Config version may vary based on the linux distro you are using). command uname -a can be used to check the current config
         command - cp /boot/config-5.11.0-38-generic  .config
    - make oldconfig
    - make prepare
    - created x509.genkey file under /linux/certs, signing_key.pem file under/linux/certs, canonical-certs.pem file under /linux/debian,  canonical-revoked-certs.pem file /linux/debian**
    - make -j 6 modules (replace 6 with the number of cup cores you have allocated to the vm - this is for compiling modules in parallel)
    - make -j 6
    - sudo make modules_install
    - sudo make install
    - sudo reboot
18. Once the reboot is successful, you can check if you are using newly compiled kernel by doing uname -a command in terminal. If the kernel version has changed to the latest one(in my case the version changed from 5.11.0-38-generic to 5.15.0+) And the time stamp should also reflect the date and time on which you have compiled and installed the kernel(updated timestamp- Linux ubuntu 5.15.0+ #2 SMP PREEMPT Sat Nov 6 10:46:00 PDT 2021).

19. Create a folder 283-1 inside linux folder and add cmpe283-1.c and Makefile to this folder. 
20. Goto this folder and run make and check if cmpe283-1.ko is generated using command - ls *.ko
    - if you get error regarding missing license add this line at the end of cmpe283-1.c file - MODULE_LICENSE("GPL v2");
21. Use command 
    - sudo insmod cmpe283-1.ko to load module and check if module loaded using command 
    - lsmod | grep cmpe283
22. Use command dmesg to check vmx features
23. Commit cmpe283-1.c and Makefile to your git repository. Make sure you commit to the correct repository. The repository you are commiting can be found out using command
   - git remote -v



** I encountered errors during make modules_install step regarding missing signing key certificates(This may not happen in all linux distros). I created following missing certificates manually
   - Create x509.genkey file under /linux/certs using vi command in terminal
     Contents of the x509.genkey file
     
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
   openssl req -new -nodes -utf8 -sha512 -days 36500 -batch -x509 -config x509.genkey -outform DER -out signing_key.x509 -keyout signing_key.pem
      
      
        
   -  Create canonical-certs.pem under /linux/debian using vi command in terminal(If debian folder does not exists create on under linux folder)
      Contents of canonical-certs.pem - Please refer this link - https://salsa.debian.org/kernel-team/linux/-/blob/master/debian/certs/debian-uefi-certs.pem
      
      
   -  Create canonical-revoked-certs.pem under /linux/debian using vi command in terminal(If debian folder does not exists create on under linux folder)
      Contents of canonical-revoked-certs.pem - Please refer this link - https://salsa.debian.org/kernel-team/linux/-/blob/master/debian/certs/debian-uefi-certs.pem


This steps are followed for Ubuntu 20.04.3 vm on vmware fusion. You may need to install additional packages for successful kernel compilation depending on the linux distribution you are using and the setup




 






