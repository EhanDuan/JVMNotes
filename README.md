Notes About JVM(Java Virtual Machine)
=====
Introdution to JMM(Java Memory Model)
-----
Typically, start from the Run-time Data Areas
![image](https://github.com/EhanDuan/Java/blob/main/Img/Java%20JVM%20Run-Time%20Data%20Areas.svg)

Dive deeper for Stacks
![image](https://github.com/EhanDuan/Java/blob/main/Img/Java%20JVM%20Stack.png)

Let's go to the Heap
![image](https://github.com/EhanDuan/Java/blob/main/Img/JVM%20Heap%20(View%20of%20Generation).png)

Some confusing part about Method Area and Heap relationships during different editions
![image](https://github.com/EhanDuan/Java/blob/main/Img/JVM%20Heap%20and%20Method%20Area%20Developing.png)



GC(Garbage Collection) and Memory Strategy
-----
# Review
"There is a high 'wall' made of dynamic memory allocation and skills of garbage collection between Java and C++. People inside wanna step outside. People outside wanna jump into it." This is a sentence from the Chinese book of understanding Java Virtual Machine. I like it.

Just let us out of this cyber-punk. When talking about garbage, if you are responsible for your garage in your house, how do you deal with it? Left for your mom? That might be a "good" strategy. But if you are alone, what should you start from? 
(1) Find what stuff belongs to garbage; 
(2) Think When you should collect them;(Think about how your LOL teammate will kill you if you clean your room during the ranking :))
(3) Figure out how to collect your garbage;

Why should we learn the mechanism of the garage collection and memory dynamic allocation even JVM has taken charge of it? 
Very easy to answer. 
(1) GC PART : If you mom cleans your room regularly in her pattern, imaging you play WOW for a very long time in front of your gameing computer. If she keeps going into your room and clean your stuff even cleaning the dust on your keyboard, do you think you can beat the boss successfully? Especially when if you and your brothers play the games in your room, will you enjoy the time when the room is cleaning at the same time? The way to solve it is to talking with your mom to clean after your playing or you stop the battle for a while.
(2) Momery Dynamic Allocation PART: Image your mom is responsible for your stuff storing. Everytime you buy some toys or posters, she needs to arrange them in a good order. One day, you take an elephant back to home, you mom says "oh gosh, we do not have enough room to keep it." You need to find out why and how to solve it by yourself.
[acedemic version]
When there is StackOverFlowError or Momeny Leak Issue, JVM cannot help you. You need to solve the problems.

# GC Algorithms
## Mark-Sweep
It was proposed by John McCarthy in 1960. It has two phrases just like its name: mark and sweep. Mark the objects which need to be recycled and after all marks, clean these objects, which is sweep. Vice Versa(i.e. mark survivor and sweep unmarked ones)

This is the most fundamental algorithm since the new-coming algorithms are developed based on this one. 
![image](https://github.com/EhanDuan/Java/blob/main/Img/JVM%20GC%20Algorithms%20-MS.svg)

## Mark-Copy
In 1969, Semispace Copying was proposed by Fenichel. It seperates the memory into 2 equal-size parts. Each time, only one of these two is used. Only when current one is depleted, the survivor in this memory will be copied to another memory. Then the previous memory will be cleaned at once.
![image](https://github.com/EhanDuan/Java/blob/main/Img/JVM%20Garbage%20Collection%20Algorithms%20MC.svg)

