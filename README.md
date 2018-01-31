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
	Open Pintos/threads/thread.h with text editor. You need this variable to store the exit code in the thread.
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
5. ##### You can now repeat steps in FileSystem to create a disk and load a new pogram and you will see the process name and its exit code printed when it terminates  :) That's it!

## PART 2: Argument Passing 'with some sys calls'

1. ##### Edit process.c
	Open Pintos/userprog/process.c with text editor.

    * Line 39, before making a new thread, add these lines.
    ```
	char *save_ptr;
	file_name = strtok_r (file_name," ",&save_ptr);
    ```
   * In line 204, change 
    ```
	static bool setup_stack (void **esp);
    ```
  	* to
    ```
	static bool setup_stack (void **esp, char * cmdline);
    ```
   * In line 233, after file = filesys_open (file_name); add these lines
    ```
	char * fn_cp = malloc (strlen(file_name)+1);
	strlcpy(fn_cp, file_name, strlen(file_name)+1);
	char * save_ptr;
	fn_cp = strtok_r(fn_cp," ",&save_ptr);
	file = filesys_open (fn_cp);
    ```
    ##### Implementing the stack
    
   * In line 320, change 
    ```
	if (!setup_stack (esp))
    ```
   * to
    ```
	if (!setup_stack (esp,file_name))
    ```
     ##### replace the method setup_stack with this
	```
    static bool
    setup_stack (void **esp, char * file_name) 
    {
      uint8_t *kpage;
      bool success = false;

      kpage = palloc_get_page (PAL_USER | PAL_ZERO);
      if (kpage != NULL) 
        {
          success = install_page (((uint8_t *) PHYS_BASE) - PGSIZE, kpage, true);
          if (success)
            *esp = PHYS_BASE;
          else
            palloc_free_page (kpage);
        }

      char *token, *save_ptr;
      int argc = 0,i;

      char * copy = malloc(strlen(file_name)+1);
      strlcpy (copy, file_name, strlen(file_name)+1);


      for (token = strtok_r (copy, " ", &save_ptr); token != NULL;
        token = strtok_r (NULL, " ", &save_ptr))
        argc++;


      int *argv = calloc(argc,sizeof(int));

      for (token = strtok_r (file_name, " ", &save_ptr),i=0; token != NULL;
        token = strtok_r (NULL, " ", &save_ptr),i++)
        {
          *esp -= strlen(token) + 1;
          memcpy(*esp,token,strlen(token) + 1);

          argv[i]=*esp;
        }

      while((int)*esp%4!=0)
      {
        *esp-=sizeof(char);
        char x = 0;
        memcpy(*esp,&x,sizeof(char));
      }

      int zero = 0;

      *esp-=sizeof(int);
      memcpy(*esp,&zero,sizeof(int));

      for(i=argc-1;i>=0;i--)
      {
        *esp-=sizeof(int);
        memcpy(*esp,&argv[i],sizeof(int));
      }

      int pt = *esp;
      *esp-=sizeof(int);
      memcpy(*esp,&pt,sizeof(int));

      *esp-=sizeof(int);
      memcpy(*esp,&argc,sizeof(int));

      *esp-=sizeof(int);
      memcpy(*esp,&zero,sizeof(int));

      return success;
    }
    ```
1. ##### Edit syscall.c
	Open Pintos/userprog/syscall.c with text editor.
    * Add include for 'process.h'
    ```
	#include "process.h"
    ```
    * replace the syscall_handler function
    * This implements three system calls 
    * SYS_HALT
    * SYS_EXIT
    * SYS_WRITE
    * SYS_WRITE is needed to make printf work for programs like echo.c
	```
    static void
	syscall_handler (struct intr_frame *f UNUSED) 
	{
 	 int * p = f->esp;


	  int system_call = * p;

		switch (system_call)
		{
			case SYS_HALT:
			shutdown_power_off();
			break;

            case SYS_EXIT:
            thread_current()->parent->ex = true;
            thread_exit();
            break;

            case SYS_WRITE:
            if(*(p+5)==1)
            {
                putbuf(*(p+6),*(p+7));
            }
            break;

            default:
            printf("No match\n");
        }
    }
    ```
1. ##### Edit thread.c    
	Open Pintos/threads/thread.c with text editor.
    * Line 193 add
    ```
    struct child* c = malloc(sizeof(*c));
    c->tid = tid;
    c->exit_code = t->exit_code;
    c->used = false;
    list_push_back(&running_thread()->child_proc, &c->elem);
    ```
     * Line 482 after t->magic = THREAD_MAGIC;
    ```
    list_init(&t->child_proc);	
    t->ex = false;
    t->parent = running_thread();
    list_init(&t->files);
    t->fd_count=2;
    t->exit_code = -100;
    t->waitingon=0;
    
    // comment out interrupt disable and interrupt set level.

    ```   
1. ##### Edit thread.h  
	Open Pintos/threads/thread.h with text editor.
    * in Line 97 add inside struct thread
    ```
    bool ex;
    struct thread* parent;
    struct list files;
    int fd_count;
    struct list child_proc;
    int waitingon;
    ```    
    * create struct child, outside struct thread
    ```
    struct child{
      int tid;
      struct list_elem elem;
      int exit_code;
      bool used;
    };
    ```
