# In a non-ARC environment this Rules to remember

* You have ownership of any objects you create.
* You can take ownership of an object using retain.
* When no longer needed, you must relinquish ownership of an object you own.
* You must not relinquish ownership of an object you don’t own.

# Take ownership of a object in this methods which name likes

* alloc
* new
* copy
* mutableCopy
* and “retain” also take the ownship

# In ARC , follow below rules

* Forget about using retain, release, retainCount, and autorelease.
* Forget about using NSAllocateObject and NSDeallocateObject.
* Follow the naming rule for methods related to object creation.
* Forget about calling dealloc explicitly.
* Use @autoreleasepool instead of NSAutoreleasePool.
* Forget about using Zone (NSZone).
* Object type variables can’t be members of struct or union in C language.(__bridge __bridge_retain __bridge_transfer)
* ‘id’ and ‘void*’ have to be cast explicitly.

# the property qualifier means

<table>
    <tr>
        <td>Property modifier</td>
        <td>Ownership qualifier</td>
    </tr>
    <tr>
    	<td>assign</td>
    	<td>__unsafe_unretained</td>
    </tr>
    <tr>
    	<td>copy</td>
    	<td>__strong(note:new copied object is assigned.)__strong</td>
    </tr>
    <tr>
    	<td>retain</td>
    	<td>__strong</td>
    </tr>
    <tr>
    	<td>strong</td>
    	<td>__strong</td>
    </tr>
    <tr>
    	<td>weak</td>
    	<td>__weak</td>
    </tr>
    <tr>
    	<td>unsafe_unretained</td>
    	<td>__unsafe_unretained</td>
    </tr>
</table>