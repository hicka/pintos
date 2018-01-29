# PintOS

This is a simple guide to get started with PintOS , by Hicker.


## Follow these Steps:
1. ##### Install QEMU Simulator if you havent already.
      ```
      sudo apt-get install qemu
      ```

2. ##### GDBMACROS
	Open Pintos/utils/pintos-gdb with your editor and set it to the absoulte path of your gdb-macros.
    ```
    GDBMACROS=/home/hicker/Desktop/Pintos/misc/gdb-macros
    ```
    
 3. ##### Compile Utils Directory (Pintos/utils)
	
    ```
    cd utils
    make
    ```
 
 4. ##### Compile Threads Directory (Pintos/threads)
	
    ```
    cd threads
    make
    ```
    
 5. ##### Edit pintos file (Pintos/utils)
	Open Pintos/utils/pintos with text editor
    ```
    ln 259 
    	replace 'kernel.bin' with
        	'/home/hicker/Desktop/Pintos/threads/build/kernel.bin'
    
    
    ln 623
    	replace 'qemu-system-i386' with
        	'qemu-system-x86_64'
    ```
    
        
 6. #### Edit pintos.pm file (Pintos/utils)
	Open Pintos/utils/pintos.pm with text editor
    ```
    ln 362 
    	replace 'loader.bin' with
        	'/home/hicker/Desktop/Pintos/threads/build/loader.bin'
    
	```
        
    
 11. ### Export utils directory path to PATH variable
      Open ~/.bashrc
      ```
      sudo nano ~/.bashrc
      ```
      Add this line to the end of the file and save it
      ```
      export PATH=/home/hicker/Desktop/Pintos/utils:$PATH
      ```
12. ### Reload terminal with the new environment variables
      ```
      source ~/.bashrc
      ```
13. ### Run pintos
      ```
      pintos run alarm-multiple
      ```
     
    You should be able to run this command and pintos should run.  
     
     
## User Programs 

1. ##### Compile userprog Directory (Pintos/userprog)
	
    ```
    cd userprog
    make
    ```
2. ##### Compile examples Directory (Pintos/examples)
	
    ```
    cd examples
    make
    ```



3. ##### Edit pintos file (Pintos/utils)
	Open Pintos/utils/pintos with text editor
    
     ```
        ln 259 (replace 'threads' with 'userprog')
                '/home/hicker/Desktop/Pintos/userprog/build/kernel.bin'
     ```

4. ##### Edit pintos.pm file (Pintos/utils)
	Open Pintos/utils/pintos.pm with text editor
    
     ```
        ln 362 (replace 'threads' with 'userprog')
                '/home/hicker/Desktop/Pintos/userprog/build/loader.bin'
     ```
## File Systems
1. ##### Creating a simulated disk
	You need to be in Pintos/userprog/build directory.
	1. run this command to create a disk file with 2mb. Run
	
    ```
    pintos-mkdisk filesys.dsk --filesys-size=2
    ```
    
	2. Format the created disk filesystem , Run
	
    ```
    pintos -f -q
    ```
    
 2. ##### Copy an example program to the filesystem
	While still in Pintos/userprog/build directory.
	
    ```
    pintos -p ../../examples/echo -a echo -- -q
    ```

 3. ##### Running the program
	
    ```
    pintos -q run 'echo'
    ```
 4. ##### Listing programs in file system
	
    ```
    pintos ls
    ```
 5. ##### Removing a program from the filesystem
	
    ```
    pintos -q rm filename
    ```




   
   

## PART 1: Implementing Process termination message:

1. ##### Edit thread.h
	Open Pintos/threads/thread.h with text editor
    ```
    ln 86 
		int exit_code;
	```
    After saving the file run in Pintos/threads direcotory,
    ```
    make clean
    make
    ```
 2. ##### Edit process.c
	Open Pintos/userprog/process.c with text editor. type thse code.
    
    ```
    ln 69 after if(!success){
    
		thread_current()->exit_code = -1;
	```
    
    ```
    ln 109  after uint32_t *pd;
    
		printf("%s: exit(%d)\n",cur->name,cur->exit_code);
	```          
 3. ##### Re-Compile userprog Directory (Pintos/userprog)
	 After saving the file run in Pintos/userprog direcotory,
    ```
	make clean
    make
    ```
 4. ##### Go to build folder
	```
    cd build
    ```
5. ##### You can now repeat steps in FileSystem to load a new pogram and you will see the process name and its exit code printed :) That's it!


