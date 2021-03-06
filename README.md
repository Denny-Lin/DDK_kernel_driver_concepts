# DDK_kernel_driver_concepts
* It is a repository for practicing Linux kernel driver in kernel space.
* I learned linux driver by myself.
* I will record the learning porcesses of drivers in linux kernel moudule here.
* What are the strategies of memory protection in kernal space?
* What is system call?
* user program -> interface / system calls -> inode -> driver
* ...

## Platform
* Linux
<br><br>


## Let's get started
* Applications on Linux access character and block devices through device nodes, and network devices through network interfaces.
* All the behaviors of read and write should be permitted by kernel which is in the kernel space of ram. <br>

![image](https://user-images.githubusercontent.com/67073582/122648803-5e593380-d15d-11eb-9aaf-fc7de2f3f8cb.png) <br>

* A inode records where the file is.

![image](https://user-images.githubusercontent.com/67073582/124983485-94a61680-e06a-11eb-8a77-4958fc778afe.png) <br>

* User space programs access character and block devices through device nodes also referred to as device special files. 
* When a device node is created, it is associated with a major and minor number.
* By definition, device nodes correspond to resources that have already been allocated by the operating system Kernel. 
* The resources are identified by a major number and a minor number, which are stored as part of the structure of a node.

![image](https://user-images.githubusercontent.com/67073582/122663434-0a863300-d1cd-11eb-8d8d-4a152fe5ecdb.png) <br>

![image](https://user-images.githubusercontent.com/67073582/122663574-13c3cf80-d1ce-11eb-833e-e793b3e56dbd.png) <br>

![image](https://user-images.githubusercontent.com/67073582/122388792-d0394d80-cfa2-11eb-912a-1f32f38a87c4.png) <br>

![image](https://user-images.githubusercontent.com/67073582/122389029-1393bc00-cfa3-11eb-90b1-da17e137e61f.png) <br>

* So what we should do is ...
* To know the interface and what function pointer we can use.
* Yes, we need to know some structures, cuz it is not like the 8051, we just use our private struct or even just array.
* Linux have lots of specific given structs, the data will be sent to other machines following the given structs via wires in phiscial layer. 
* ...
<br><br>

## Use alloc_chrdev_region(), rather than register_chrdev()
* For new drivers, we strongly suggest that you use dynamic allocation to obtain your major device number, rather than choosing a number randomly from the ones that are currently free.

* To have more control over the device numbers and the device creation you could do the following steps <br>
  (instead of register_chrdev()):

 1. Call alloc_chrdev_region() to get a major number and a range of minor numbers to work with.
 2. Create device class for your devices with class_create().
 3. For each device, call cdev_init() and cdev_add() to add the character device to the system.
 4. For each device, call device_create(). <br>
   As a result, among other things, Udev will create device nodes for your devices. <br>
   No need for mknod or the like. 
   device_create() also allows you to control the names of the devices. <br>

## Summary
### Step 1: Write your driver. (Char, Block, ...)
```C
#include <linux/init.h>
#include <linux/module.h>

static struct file_operations simple_driver_fops = 
{
    .owner   = THIS_MODULE,
    .read    = device_file_read,
    ...
    .
};

//dynamic major_num for module_exit
//read, copy_to_user
//write, copy_from_user
//ioctl
//mining bitcoin??
//...

static int my_init(void)
{
    //register for major_num, name, fops
    return  0;
}
    
static void my_exit(void)
{
    //unregiser for major_num
    //
    return;
}
    
module_init(my_init);
module_exit(my_exit); 
```
### Step 2: Compile and mount it.
* write your Makefile.
1. insmod 
2. lsmod 
3. rmmod 

### Step 3: Device file (your program should connect this file to find the driver.)
* We can create it automatically in driver (Hotplug) or by ourself. <br>
* int mknod(const char \*pathname, mode_t mode, dev_t dev); <br>
ex: mknod /dev/ttyUSB32 c 188 32 <br>

### Step 4: Write your program. 
```C
#include <stdio.h>
int main(){
   char buf[512];
   FILE *fp = fopen("/dev/XXXXXX", "w+"); // Call the function in step 1. and open device file you create in step 3.
   if(fp == NULL) {
      printf("can't open device!\n");
      return -1;
   }
   fread(buf, sizeof(buf), 1, fp); // Call the function in step 1.
   fwrite(buf, sizeof(buf), 1, fp); // Call the function in step 1.
   fclose(fp);
   return 0;
}
```

## References
* https://www.youtube.com/watch?v=R5qSTZA0PuY
* https://www.youtube.com/watch?v=oX9ZwMQL2f4
* https://docs.huihoo.com/doxygen/linux/kernel/3.7/structfile__operations.html
* http://0975128810.blogspot.com/2016/01/raspberry-usb-driver.html
* https://www.ibm.com/docs/en/linux-on-systems?topic=hdaa-names-nodes-numbers
* https://www.programmersought.com/article/46983724452/
* https://stackoverflow.com/questions/5970595/how-to-create-a-device-node-from-the-init-module-code-of-a-linux-kernel-module
* https://www.apriorit.com/
<br><br>
