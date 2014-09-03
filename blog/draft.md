----
title: Main title
author: <a href="//github.com/cheripai">Dat Do</a>
----

It is known that C++ is an ideal language for reaching optimal runtime efficiency, which is why so many processor intensive applications are written in the language. On the other hand, Haskell is known for creating bug-free, compact, and readable programs without sacrificing too much performance. Therefore, it would be interesting to see if Haskell is capable of beating C++ in terms of runtime speed.

To test the runtime performance between the two languages, we will be inserting a list of 713,000 strings from a file into an AVL Tree, which is a $O(n \log n)$ operation. The C++ AVL tree was created in a data structures course that I took recently and the Haskell AVL tree is from the Haskell library Data.Tree.AVL [(View my source code)](https://github.com/CheriPai/AVLComparison). Additionally, the Haskell AVL tree will be using ByteStrings because they are much more efficient than the notoriously slow String. To see how the runtime is affected by files of different sizes, the file will be partitioned into 10 segments. The first segment will have 71,300 words, the second will have 71,300 * 2 words, and so on. Both the C++ and Haskell tests will be performed with the -O2 compile flag for optimization with each test being the average of 3 separate runs. Let’s take a look at the results.

![C++vHaskell](/img/CvsH.png)

So it’s evident that C++ is quite a bit faster than Haskell during this test. This appears to be the case because Haskell operates on immutable data. This means that every time a new element is to be inserted into the Haskell AVL tree, new parent nodes must be created because the old parent nodes cannot be changed. This creates quite a bit of overhead for the Haskell AVL tree. C++ on the other hand, does have mutable data and can simply change the node that a parent node is pointing to, making it faster than Haskell. These results, however, bring up an interesting question: is there an easy way to attain faster than C++ speeds using Haskell?

Conveniently, there exists a Haskell library called monad-par that can be used for parallel computation, which can speed up runtime quite a bit. Yes, one might think that it is unfair to compare multithreaded Haskell against C++ that is not multithreaded. But let’s be honest, working with pthreads can be quite the headache, while parallelism in Haskell requires a few extra lines of code. Here’s a snippet of how parallelized code works running on two threads.

```
runPar $ do
    t1 <- new
    t2 <- new
    t1 <- spawn (return t1 (f x))
    t2 <- spawn (return t2 (g x))
    t1’ <- get t1
    t2’ <- get t2
    return (t1’ + t2’)
```

Great, so now that the Haskell code has been parallelized, we can compile and run the program again to see the difference. To compile for parallelism, we must use some special flags.

```
ghc –O2 filename -rtsopts –threaded
```

And to run the program (N4 means to run the program on 4 cores).

```
./filename +RTS –N4
```

![C++vHaskell4](/img/CvsH4.png)

Impressively, Haskell now gets much better runtimes than C++. Now that we know Haskell is capable of dramatically increasing its speeds through parallelism, it would be interesting to see how the runtime is affected by the degree of parallelism.
According to [Amdahl’s law](http://en.wikipedia.org/wiki/Amdahl%27s_law), a program that is 100% parallelized will see a proportional speed up based on the number of threads of execution. For example, if a program that is 100% parallelized takes 2 seconds to run on 1 thread, then it should take 1 second to run using 2 threads. The code used for our test, however, is not 100% parallelized since there a union operation performed at the end to combine the trees created by the separate threads. The union of the trees is a $O(n)$ operation while the insertion of the strings into the AVL tree is a $O\left(\frac{n \log n }{p}\right)$ operation, where p is the number of threads. Therefore, the runtime for our test should be

$$O\left(\frac{n\log{n}}{p} + n\right)$$

Here is a graph showing the runtime of the operation on the largest set (713,000 strings) across increasing levels of parallelism.

![HaskellParallelization](/img/HParallelism.png)

Taking a look at the results, we can see that the improvement in runtime does not fit the 100% parallelized theoretical model, but does follow it to some extent. Rather than the 2 core runtime being 50% of the 1 core runtime, the 2 core runtime is 55% of the 1 core runtime, with decreasing efficiency as the number of cores increases. Though, it is clear that there are significant improvements in speed through the use of more processor cores and that parallelism is an easy way to get better runtime speeds with little effort. 

During this test, we can see that non-parallelized Haskell is slower than C++. However, through little effort, the runtime of Haskell can be dramatically improved through the use of parallelism. It is clear from Amdahl’s law and through our comparison of runtime speeds on an increasing number of cores that the runtime of Haskell can be improved by using a greater number of threads. Knowing this, perhaps C++ isn’t always the best choice for heavy computation.

