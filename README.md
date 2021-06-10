Notes About JVM(Java Virtual Machine)
=====
Introdution to JMM(Java Memory Model)
-----
# Run-time Data Areas
Typically, start from the Run-time Data Areas
![image](https://github.com/EhanDuan/Java/blob/main/Img/Java%20JVM%20Run-Time%20Data%20Areas.svg)

# Stack
![image](https://github.com/EhanDuan/Java/blob/main/Img/Java%20JVM%20Stack.png)

# Heap
![image](https://github.com/EhanDuan/Java/blob/main/Img/JVM%20Heap%20(View%20of%20Generation).png)

## Some Confusing Parts Explanation
### Method Areas and Heap relationships during different editions
![image](https://github.com/EhanDuan/Java/blob/main/Img/JVM%20Heap%20and%20Method%20Area%20Developing.png)
### Class Constant Pool? String Pool? StringTable? Run-time Constant Pool? 
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
## Enum the GC Root
## Safe Point
## Safe Region

