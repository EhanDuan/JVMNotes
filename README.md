Notes About JVM(Java Virtual Machine)
=====
Under the Hood JMM(Java Memory Model)
-----
* [Run-time Data Areas](#run-time-data-areas)
* [Stack](#stack)
* [Heap](#heap)
* [Method Areas and Heap relationships during different editions](#method-areas-and-heap-relationships-during-different-editions)
* [Class Constant Pool? String Pool? StringTable? Run-time Constant Pool?](#constants)
# Run-time Data Areas
Typically, start from the Run-time Data Areas
![image](https://github.com/EhanDuan/Java/blob/main/Img/Java%20JVM%20Run-Time%20Data%20Areas.svg)

# Stack
![image](https://github.com/EhanDuan/Java/blob/main/Img/Java%20JVM%20Stack.png)

# Heap
![image](https://github.com/EhanDuan/Java/blob/main/Img/JVM%20Heap%20(View%20of%20Generation).png)

## Some Confusing Parts Explanation
### Method Areas and Heap relationships during different editions
![image](https://github.com/EhanDuan/Java/blob/main/Img/JVM%20Heap%20and%20Method%20Area%20Developing.svg)
### Constants 
#### String Pool (String literal Pool)
This is after `class loading` and `verification` and `preparation`, the String instance objects are created in the heap. The references of these string instance objects are stored in string pool. (In Hotspot, the function of string pool is realized by String Table, which is a hash table, which stores the string interning). 

This is shared by all the class.

`Location: Heap`

#### Class Constant Pool
This one is in the `class file`. After abc.java is complied into abc.class, the class constant pool is created in the abc.class file. Each class has one.

It stores `literals` and `symbolic references` during the compilation. In the class file, there isn't any information about how actually the methods and fields deploy in the memory. Only when JVM is running, these are converted to real memory address. Concretely, during the class loading, it will get the references in the constant pool in the class file. And when the class is created or in execution, these contents are in resolution and translated into specific address.

`Location: Class file`

#### Run-time Constant Pool

After the class is loaded in the JVM memory, the Run-time Constant Pool is created for each class file. The constant pool in the class file will be put in the run-time constant pool. After resolution, the symbolic references are replaced with direct references by using checking `StringTable`. 

`Location: Method Area(realized by MetaSpace)` 

Constants Reference: https://blog.csdn.net/qq_26222859/article/details/73135660

GC(Garbage Collection) and Memory Strategy
-----
# Before It
Before everything, the notes in this section is the learning notes by reading the book "Deeply UnderStanding JVM"(author Zhiming Zhou). I collected the important parts here and made some figures to better understand the key points of the book.
# Review
Just let us out of this cyber-punk. When talking about garbage, if you are responsible for your garage in your house, how do you deal with it? Left for your mom? That might be a "good" strategy. But if you are alone, what should you start from? 
(1) Find what stuff belongs to garbage; 
(2) Think When you should collect them;(Think about how your LOL teammate will kill you if you clean your room during the ranking :))
(3) Figure out how to collect your garbage;

Why should we learn the mechanism of the garage collection and memory dynamic allocation even JVM has taken charge of it? 
Very easy to answer. 
* GC PART : If you mom cleans your room regularly in her pattern, imaging you play WOW for a very long time in front of your gameing computer. If she keeps going into your room and clean your stuff even cleaning the dust on your keyboard, do you think you can beat the boss successfully? Especially when if you and your brothers play the games in your room, will you enjoy the time when the room is cleaning at the same time? The way to solve it is to talking with your mom to clean after your playing or you stop the battle for a while.

* Momery Dynamic Allocation PART: Image your mom is responsible for your stuff storing. Everytime you buy some toys or posters, she needs to arrange them in a good order. One day, you take an elephant back to home, you mom says "oh gosh, we do not have enough room to keep it." You need to find out why and how to solve it by yourself.
[acedemic version]
When there is StackOverFlowError or Momeny Leak Issue, JVM cannot help you. You need to solve the problems.

# GC Algorithms
## Mark-Sweep
It was proposed by John McCarthy in 1960. It has two phrases just like its name: `mark` and `sweep`. Mark the objects which need to be recycled and after all marks, clean these objects, which is sweep. Vice Versa(i.e. mark survivor and sweep unmarked ones)

This is the most fundamental algorithm since the new-coming algorithms are developed based on this one. 
![image](https://github.com/EhanDuan/Java/blob/main/Img/JVM%20GC%20Algorithms%20-MS.svg)

## Mark-Copy
In 1969, Semispace Copying was proposed by Fenichel. It seperates the memory into 2 equal-size parts. Each time, only one of these two is used. Only when current one is depleted, the survivor in this memory will be copied to another memory. Then the previous memory will be cleaned at once.

![image](https://github.com/EhanDuan/Java/blob/main/Img/JVM%20Garbage%20Collection%20Algorithms%20MC.svg)

In 1989, Andrew Appel optimized the strategy, which is called "Appel-style garbage collection"(used by HotSpot Serial, ParNew). It seperates the YoungGen into Eden(bigger one) and two small Survivor spaces. Each time, Eden and one of survivor space are used. When gc is triggered, the objects who surivives in this round of gc on Eden and one survivor space will be copied to another survivor at one. Then, the Eden and previous survivor space is cleared at once. Default ratio of Eden to Survivor is 8:1. So compared with the "semispace copying", this strategy will only "waste" 10%.
![image](https://github.com/EhanDuan/Java/blob/main/Img/JVM%20Appel-style%20GC.png)
In addition, if the survivor space is not enought to hold the survivors at a certain Minor GC, other regions are needed to stored, which is the Handle Promotion.

## Mark-Compact
In 1974, Edward Lueders proposed "Mark-Compact" algorithm. Basically, it is the same as the "Mark-Sweep" in the marking. But after than, instead of directly cleaning the space, it compacts the survivors to one side of the memory. Finally, the space out of the boundary will be cleaned at once.
 
![image](https://github.com/EhanDuan/Java/blob/main/Img/JVM%20GC%20Mark-Compact.svg)

# GC Realizations
Talk about HopSpot.
## Enum the GC Root
There are two ways to determine objects to "die" or not : `Reference Counting GC` and `Tracing GC`. Most of the JVMs use the second one. The idea is to 
(1) Firstly, find the gc root;
(2) Trace the objects based on the root; (Reachability)
(3) The objects which could be traced by the roots can survive. Others memory space will be recyled.

![image](https://github.com/EhanDuan/Java/blob/main/Img/JVM%20GC%20Roots.svg)
So it is crucial to enum the gc root correctly.

The objects which are usually gc roots:
(1) Constant (eg. StringTable)
(2) Class static field and variables
(3) Execution context (eg. local variables in the stack frame)
(4) Sychronized objects
(5) JNI handles

Enum gc roots needs `Stop the World`. If the threads continue running, the link relationship bewteen gc roots and other objects could change.

The data structure `OopMap` is adpoted to solved the heavy work to find the gc roots in the HotSpot. Once the `class loading` is done, the information about which type data is stored in which offset in the objects. In the JNI, the location of references in the stack and PC register will be stored. In this case, the collector will know these information during scanning instead of finding one by one from gc roots.

Reference:https://xie.infoq.cn/article/ba113294bb20a41614502a063

## Safe Point
If each `OopMap` instance is created for each command, large extra store space is needed. This price is expensive.

In this case, not every command will have an OopMap. Only reaching `Safepoint`, the information is stored. It means the program needs to run to these locations and then stop and gc. 

Safe points usually appear in long-time command, for example, method call, loop jumping(`break`, `continue`, `return`), excepting jumping and other kinds of instructions reuse.

Two ways for letting threads to stop at certain regions:
* Preemptive Suspension
* Voluntary Suspension

For Preemptive Suspension: When gc happens, system will stop all the user threads. Check if some threads are at safe point or not. If not, let the corresponding threads run a while and then stop them to check iteratively. Almost no JVMs uses this.

For voluntary Suspension: When gc happens, system will not directly act on the threads. Instead, threads always `poll` "safe point flag" during the execution at reasonable intervals. When finding the "stop signs" are true, threads will stop at the neareat safe points.


## Safe Region
`Safe region` is used to deal with the situations when some threads are not in "processing", like `Sleep` state or `Blocked` state. In this case, these threads cannot response to the stop request from the JVM. JVM cannot also wait these threads to be activated.

Safe region is the expanding safe point. In the safe region, the reference links are not going to change. So in this case, any place here is safe to start gc.

When the threads reach the safe regions, they will mark themselves to show they are here. 
(1)When in this time, JVM wants to start gc, JVM will not need to consider these marked threads.
(2)When these threads are going to leave the region, check if gc roots enum has finished or not. If so, threads keep running. If not, threads wait until gc roots enum finish.


## Remembered Set
Remembered Set is abstract data structure realized by `CardTable` implementation. It is used to deal with inter-gen reference situation which means some objects in `Gen A` links to objects on `Gen B` or other gens. It can avoid to enum gc roots in all Tenured(Old) Gen which is INDEED heavy work.

There are three accuracy:
* Word Accuracy: Every record is accurate to each word(i.e. address bits of the processor, 32 or 64.), which includes inter-gen pointers;
* Object Accuracy: Every record is accurate to each object, which has fields which owns inter-gen pointers;
* Card Accuracy: Every record is accuate to a certain memory region, where there are objects have inter-gen pointers;

For `CardTable`, it uses the card accuracy. The most simple form of `CardTable` is a byte array, which is adopted by HotSpot. Each element in the array corresponds to a specific memory block in its marked-memory-region. The memory block is called `Card Page`.

There is not only one object in the memory of a card page. If there is one or more objects which has the inter-gen pointers, these elements will mark as "1", which could call these elements "dirty". No inter-gen pointers will mark "0". When gc starts, it only needs to select the "dirty" elements. So it knows which card pages have inter-gen pointers and add them into gc roots to scan.

## Write Barrier
Thoertically, when the objects in a card page needs to be assigned, it should be mark "1". In HotSpot, `Write Barrier` is used to maintain the state of the `CardTable`. Write barrier can be viewed as the AOP`aspect` of this action when JVM faces this assignment action.  When objects are being assigned, a around notice is generated which allows extra actions. In this case, there are two barries : `Pre-Write Barrier`,`Post-write Barrier`. After applying this, JVM will generate instructions for all assignment commands. 

 Another Problem: False Sharing. 
The cpu cache system uses `Cache Line` as store unit. When more than one threads update their mutually stand-alone varibles, which are in the same cache line, the efficiency will decrease.

The solution is not to use unconditional write barrier. Check the card table marks firstly. Only clean element can be marked "dirty".

## Concurrent Reachability Analysis

Tracing objects from GC roots requires more stop duration when Java Heap memory increases. Bigger heap is, more objects are, more complicated the graph is. `Mark` is the common characteristic for all tracing gc algorithms. So if `Mark` is optimized, the benefit is huge. Before that, let's introduce a new concept "Tri-color Marking".

Tri-color Marking is a tool to better understand the objects which differ in "gc access" state.

* White: objects which have not been accesssed by the garbage collector. Obviously, at starting point, all objects are white. At the end, the white objects mean they are not reachable.
* Black: object which has been accessed by the garbage collector and all the references of the object have been scanned. Black objects mean they are safe. If some object refers to black objects, no need to re-scan.(Reference has direction) Black objects cannot directly link to some white object without passing grey objects.
* Grey: object which has been accessed by teh garbage collector but there is at lease one reference of this object has not been scanned.

Imagine that, the process of the gc root tracing is like the wave in the sea. The wave crest is grey, white is in the front of grey, black is behind.

Let's back to the problem. If the collector and user threads work together, there would be two results:  
(1) "safe" -> "Die"  
(2) "Die" -> "Safe"   
The second one is not serious, which stays more garbage in the memory. But the first one could destroy useful links in the user threads.

1. Starting Point
2a. Scan
3a. Scan Finishing

But the following could happen concurrently
2b. Scan (Reference Change)
2C. Scan (Reference Change)

The key point of the problem is object disappearance(i.e. "black" -> "white"). It happens if and only if the 2 following situations are met at the same time:  
(1) More new references from black to white are created;  
(2) For a certain white object, all direct or indirect references from grey to that white are cut.

The solution aims the aforementioned two conditions:  
(1) Incremental Update;  
(2) Snapshot At The Beginning, SATB;  

Reference: https://en.wikipedia.org/wiki/Tracing_garbage_collection#Tri-color_marking
