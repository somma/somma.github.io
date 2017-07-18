# What does IRP_PAGING_IO means?

[원문](https://www.osronline.com/showthread.cfm?link=256100)

There are a couple of ways to approach this question.  Let's first approach it operationally:

+ This bit indicates that one of the Paging I/O functions in the I/O Manager have been invoked (by MM) to fill in or write out physical pages that contain data that is stored by the file system driver.  This includes IoPageRead, IoSynchronousPageWrite and IoAsynchronousPageWrite

+ This bit indicates (to a filter or a file system) that there is an MDL describing the physical pages.  This doesn't necessarily mean that there is a mapping (to a virtual address) for those pages (MmGetSystemAddressForMdlSafe can be used to create one if necessary, though it's best to be avoided 
unless necessary).

+ This bit indicates (again to a filter or a file system) that the UserBuffer address *may not be a valid address*.  It turns out that it needs to match what's in the MDL but there's no guarantee that it's actually mapped at that address.  I mention this in my file systems class because it's a subtle bug I've seen multiple times over the years.

+ This bit indicates that a filter wishing to use that buffer MUST use the MDL (if you don't the OS will give you the frowny face of death - FFOD - in Windows 8/8.1).  This is because building a new MDL for the same physical pages risks marking the pages as dirty and that can cause them to be WRITTEN, which is definitely not what you want.

+ An IRP_MJ_READ or IRP_MJ_WRITE where this bit is set will not use an APC to perform completion of the original I/O - the I/O Manager will directly set the event object in the IRP to signal back to the original thread that the I/O is completed.

+ When tearing down an IRP, the I/O Manager *will not free* the Irp->MdlAddress when this bit is set. That's because the I/O Manager views the MDL as belonging to the Memory Manager.

Now let's approach it conceptually:

For read, this indicates that the page is being read via the demand page fault mechanism, and rather than the virtual address of a buffer we are given an MDL that describes the newly allocated physical pages that will eventually be used to resolve the virtual address at which the fault occurred.

For write, this indicates that something within the virtual memory system (Mm or Cc) is requesting that the data within the given physical pages be written back to storage by the file system driver. Again, there is not necessarily any valid virtual address mapping.  Writing dirty pages back to their storage location is an essential part of how the Memory Manager reclaims pages.  For Cc, the write-back is part of write-behind cache logic.

Mm doesn't care if dirty pages EVER get written back per-se.  It cares that there aren't "too many" dirty pages.  But it's content to have a handful of old dirty pages sitting around in memory.

Cc doesn't care about page allocation, usage, working sets, etc.  It cares that data in the file system write-behind cache doesn't stick around "too long".

Tony
OSR