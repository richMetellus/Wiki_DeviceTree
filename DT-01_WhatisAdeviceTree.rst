##############################
Device Tree 101
##############################


*********
Scope
*********

In this new document, I will put and organize relevant information I found
concerning device tree.

**The goals:**

* understand the syntax of device tree
* motivation behind device tree
* Understand the specification that governs what goes into a the device tree file so that a device
  tree compiler (dtc) can parse it.
* Device tree binding to a kernel image

    * Use of device tree to initialize peripherals.


**Target Audience**

* Software Developers

**Road map to understand device tree**

* Understand the definition of a device tree

* Understand what goes into a device tree file 

    * basic syntax tree

* understand the standards and common practices that govern what to include
  in the device tree.

    * look at the device tree specification (devicetree.org, ePAPR)
    * Node name
    * Node label
    * best practice for properties naming:

        * standard property (compatible, status, etc...)
        * non-standard properties (<manufacture-ticker-symbol>,<property-name> = <value> )
    
    * Different data type

        * Boolean
        * u32
        * string
        * stream of strings
        * byte
        * byte stream
        * etc ...
        
    * some stuff that goes into the device depends on the driver, device tree bindings.
      so get a good grasp of that.

* Understand how we go from device tree to binary, and how the bootloader and 
  kernel uses it.



****************************
What is a device tree?
****************************

A device tree is 

* a data structure (a tree) used in embedded systems to **describe the 
  hardware components** of a device in a OS-agnostic way.
    
    .. drawio-image:: ../_images/src/DT-1_DeviceTree.drawio

    * written by a developer in a device tree source file. (.dtsi, .dts)
    * process by the device tree compiler (dtc)
    * produce a more efficient representation: device tree blob (.dtb), also
      called a *flattened device tree* **FDT**

* The device tree is a tree structure with nodes that describe the physical 
  devices in the system that cannot be dynamically detected by software [3]_

* Each node in the tree describes the characteristics of the device being 
  represented. [3]_
  
  * The purpose of the device tree is to describe device information in a 
    system that cannot necessarily be dynamically detected or discovered by 
    a client program

* can be used to describe the hardware components of a system in a 
  platform-independent way, allowing the same firmware image to be used 
  across multiple platforms. [1]_

********************************************************
Motivation Behind Device tree : Why was it invented?
********************************************************

Before device tree was used, developing for ARM used to be hard. hardware devices 
are directly hard-coded into the operating system kernel, which makes 
it difficult to support multiple hardware configurations without recompiling 
the kernel for each configuration. This approach is also not very scalable, 
making it challenging to develop and maintain embedded systems. [1]_

Before the advent of the device tree, the kernel contained device specific code. 
A small change, such as the modification of an I2C peripheral's address would 
force a recompilation of the kernel image to be run. [3]_

1. Discoverable vs non-discoverable hardware

    * Some hardware busses provide **discoverability** mechanisms
        * One does not need to know ahead of time what will be connected on 
          these busses

            * e.g: PCI(e), USB
        
        * Devices can be enumerated and identified at runtime 
            
            * Concept of vendor ID, product ID, device class, etc.

**********
History
**********

* Originates from **OpenFirmware** in 1999, defined by Sun. [2]_

    * that's why many Linux/U-Boot functions related to DT have a ``of_`` prefix

* adopted by the Linux kernel community in 2001
* standardized by the IEEE as the IEEE 1275-1994 Open Firmware standard, which 
  specifies the format and content of device trees. [1]_

* Power.org wrote a specification called ePAPR (e par per)

    * it doesn't cover arm and many new things, so that is no longer used.

* devicetree.org seems to be the one leading the governance about the standardization.

********************************************
Basic Device Tree Syntax and Structure
********************************************

.. grid:: 2
   
   .. grid-item-card:: Basic Generic Syntax of Device tree
      
      .. literalinclude:: ../_resources/Devicetree/basicdevicetreesyntax.dts
         :language: c 

Concrete Examples:

* example of device tree source files:

    * .dtsi (device tree source include)
    * .dts (device tree source)

* example of device tre blob (.dtb)


*****************************
Device tree - Cells Concept
*****************************



*******************************************
Where are Device Tree Typical Stored?
*******************************************

**On the host (development machine)**

* On the Host, the device tree sources locations varies. There is no central and
  OS-neutral way to host Device tree sources and share them between projects.

    * often discussed, never done.
    * Duplicated/synced in various projects.

        * U-Boot, Barebox, TF-A

Typical locations and build: The people at bootlin often developed device tree for
ST Microelectronics: so for a dev board like the ``STM32MP157A-DK1``

    * 1st stage: TF-A

        * DT in: ``fdts/<device-name>.dts``

            * example: ``fdts/stm32m157a-dk1.dts``
        
        * Build with ``PLAT=stm32m1 DTB_FILE_NAME=stm32mp157a-dk1.dtb``
        * Bundles the DTB in the resulting ``tf-a-stmp157a-dk1.stm32``

    * 2nd stage: U-boot

        * DT in : from <Uboot-source-tree-DIR> ``arch/arm/dts/<device-name>.dts``

            * example: ``arch/arm/dts/stm32mp157a-dk1.dts`` 
            * Configure with ``stm32mp15_trusted_defconfig``
            * Build with ``DEVICE_TREE=stm32mp157a-dk1`` 
            * Bundles the DTB in the resulting ``u-boot.stm32``

    * Linux Kernel
        
        - DT in: from <Linux-source-tree-DIR>: ``arch<ARCH>/boot/dts``
        - Configure with multi_v7_defconfig 
        - Build 
        - Kernel image: ``arch/arm/boot/zImage``, 
        - DTB: ``arch/arm/boot/dts/stm32mp157a-dk1.dtb``

**On the target embedded device:**

* On target (R. Linux-based) embedded device, the device tree is typically stored in a binary 
  format and is parsed by the bootloader or kernel during the boot process [1]_

    * contains information about the various hardware components of the 
      system, including their addresses, interrupt lines, memory maps, 
      and other characteristics. [1]_

    * Locations: you can stop the booting prompt if the image is configured to
      stop autoboot.

        * DT: 
          
          .. code-block:: console
             :caption: from u-boot shell

             > printenv fdt_addr_r
        

        * Linux Kernel: 
          
          .. code-block:: console
             :caption: from u-boot shell

             > printenv kernel_addr_r
        
    *  You can also explore the device tree on the target if the utility ``dtc``
       is available on the target

       * example:
         
         * In stm32mp157a-dk1 ``/sys/firmware/devicetree/base``, there is a 
           directory/file representation of the Device Tree contents
         
         * If dtc is available on the target, possible to "unpack" the Device Tree using:
           
           ``dtc -I fs /sys/firmware/devicetree/base``




****************
References
****************

.. [1] 
   
   * :ref:`ChatGPT Device Tree Query (modified) <chatGPTDeviceTreeQueries>`
   * :ref:`<chatGPTDevicetreeMotivationQuery>`

.. [2] Bootlin Device tree 101 presentation
.. [3] NXP AN515 Introduction to Device Tree rev 0, 09/2015
.. [4] Youtube video on device tree standardization
