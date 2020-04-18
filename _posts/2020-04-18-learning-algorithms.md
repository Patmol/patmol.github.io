---
title: "Why you need to learn algorithms"
date: 2020-03-15 14:10:00 +0800
categories: [Computer Science, Algorithm]
tags: [cs, algorithm]
---

If you are a self-taught developer, you may directly learn how to build something without studying computer science or, at least, how a good or a lousy algorithm can change a lot the performance of the application you are developing.

In many language tutorials I've read, there is usually not a single word on the importance of thinking about what you are doing and the importance of a first-rate algorithm. As a reader, we want to directly build something with the language we are learning, see all the code we had written in action without thinking about the performance of this code. On small applications and small datasets, there is not a big difference between good and imperfect codes.

But when you start to work on a more significant application with a large dataset, the difference can be huge!

## Array ordering

Let's take a typical example, the array order. Of course, most of the languages are providing methods to order an array, but it's always to understand what's under the hood, and it's an easy example to begin.

### The _naïve_ method
An easy way to order an array is to compare an element with the next one, and if the first element is greater than the second one, swap them otherwise, leave them like this. Then, make a comparison between the second and the third element. And once again, if the second is greater than thrid, swap them.
By performing this action from the first element to the last one, the greater element is put at the end of the array.

To order the array, this action needs to be performed those actions again, from the first to the element before the one you set at the end before.

Now we have an idea of what we need to do to order an array, let's write some C++ code to do it!

```cpp
void sortArrayNaive(int array[], int arraySize) {
    int tmp;
    // arraySize - 1 because we don't need to test the last element of the array.
    // We decrease the size of the array to order.
    for (int i(arraySize - 1); i >= 0 ; i--) {
        // We always start from the beginning (we could have also start for the end).
        for (int j(0); j < i; j++) {
            // We check if the current element is greater than the next one ...
            if (array[j] > array[j + 1]) {
                // ... and we swap them if it's the case.
                tmp = array[j];
                array[j] = array[j + 1];
                array[j + 1] = tmp;
            }
        } 
    }
}
```

Once we have our function, we need to test its performance. 
For those tests, I've launched this function with arrays of different sizes, and here are the results.

- Test with 100 elements: 27 microseconds
- Test with 1,000 elements: 2,402 microseconds
- Test with 10,000 elements: 286,790 microseconds
- Test with 100,000 elements: 38,754,410 microseconds (38 seconds)

With those tests, we can see with just 100,000 elements; it takes more than 35 seconds to be ordered, which is a lot!

### A better method
Let's see how we can improve those times, and for this, we can take the merge sort algorithm.

With the merge sort, we divide our array into two equal parts and do this division until we reach our primary case, which has an array of size of 1.
Once the division is performed, we merge our smaller array into a bigger one, and while merging, we order the values.

You can find more detailed information on [Wikipedia](https://en.wikipedia.org/wiki/Merge_sort).

The code is a little more complicate but it worth it, you will see!
```cpp
void sortArray(int array[], int arraySize) {
    sortArrayDivide(array, 0, arraySize - 1);
}

void sortArrayDivide(int array[], int start, int end) {
    if (start >= end) {
        return;
    }

    int mid = start + (end - start) / 2;

    sortArrayDivide(array, start, mid);
    sortArrayDivide(array, mid + 1, end);

    sortArrayMerge(array, start, mid, end);
}

void sortArrayMerge(int array[], int start, int mid, int end) {
    int arrayLeftSize = mid - start + 1; 
    int arrayRightSize = end - mid;

    // We create two temporaries arrays
    int *arrayLeft = new int[arrayLeftSize];
    int *arrayRight = new int[arrayRightSize];

    // We copy the values of our arrays in the tempoparies ones
    for (int i = 0; i < arrayLeftSize; i++) {
        arrayLeft[i] = array[start + i];
    }

    for (int j = 0; j < arrayRightSize; j++) {
        arrayRight[j] = array[mid + 1 + j];
    }

    int i = 0, j = 0, arrayMergeIndex = start;

    // We merge our tempoparies arrays into our main one
    while (i < arrayLeftSize && j < arrayRightSize) {
        if (arrayLeft[i] <= arrayRight[j]) {
            array[arrayMergeIndex] = arrayLeft[i];
            i++;
        } else {
            array[arrayMergeIndex] = arrayRight[j];
            j++;
        }
        arrayMergeIndex++;
    }

    while (i < arrayLeftSize) {
        array[arrayMergeIndex] = arrayLeft[i];
        i++; arrayMergeIndex++;
    }

    while (j < arrayRightSize) {
        array[arrayMergeIndex] = arrayRight[j];
        j++; arrayMergeIndex++;
    }

    delete arrayLeft;
    delete arrayRight;
}
```

And the results with this algorithm are:

- Test with 100 elements: 77 microseconds
- Test with 1,000 elements: 447 microseconds
- Test with 10,000 elements: 5,541 microseconds
- Test with 100,000 elements: 49,838 microseconds (0,049 seconde)
- Test with 10,000,000 elements: 4,581,554 microseconds

Except when we have 100 elements or less, the merge sort is faster. And when we have more than 100,000 elements, we start to have a huge difference.
From 38 secondes, we now have 1/20 of a second!

I've tested the merge sort with 10,000,000 elements, and it only takes 4.5 seconds! I don't want to imagine how long it would have taken with the _naive_ approach.

So, even if the code is a little more complicated, the difference between the first implementation and the second one is huge! Using the right algorithm in your application can be a time saver.

## Fibonacci numbers

The Fibonacci numbers are a sequence of numbers, such as each number is the sum of the two previous ones, starting to 0 and 1.

We have 
    F_0 = 0
    F_1 = 1
and
    F_n = F_(n - 1) + F_(n - 2)

for n > 1

### The _naïve_ way
Let's try to calculate the Xth element of the Fibonacci sequence. It can be done quite easily with some recursivity.

```cpp
long long int fib(int n) {
    if (n == 0 || n == 1) {
        return n;
    }

    return fib(n - 1) + fib(n - 2);
}
```

Easy, right? Now, we perform some tests on this code.

Test for the 10th element: 0 microseconds
Test for the 30th element: 8,273 microseconds
Test for the 40th element: 512,690 microseconds
Test for the 50th element: 61,766,410 microseconds (60 seconds)

The main problem with this code is that we are performing much re-computation.
To compute 5, we need to compute 4 and 3, to compute 4, we need to compute 3 and 2, so with just this case, we can see we are computing 3, two times. And if you check with a greater number, you see a lot of that re-computation.

### The #MethodWay

An easy way to improve the performance to calculate a Fibonacci sequence is to save the values of already calculated numbers that we reuse.

Here a code which uses this technique.

```cpp
long long int fibMethodWay(int n) {
    long long int fibN1 = 1;
    long long int fibN2 = 0;
    long long int fibN = -1;
    int i = 2;

    if (n == 1) return fibN2;

    if (n == 2) return fibN1;

    while (i < n) {
        fibN = fibN1 + fibN2;
        fibN2 = fibN1;
        fibN1 = fibN;
        i++;
    }

    return fibN;
} 
```

The code is quite easy to understand, and the performances are outstanding!

Test for the 10th element: 0 microseconds
Test for the 30th element: 0 microseconds
Test for the 40th element: 0 microseconds
Test for the 50th element: 0 microseconds

In this case, we need to think about our problem in a different way to improve how we can resolve it.

## So, why do I need to learn algorithms?

Depending on the field in which you are working in computer science, you may think that learn algorithm is not useful for you, but in this case, I think the opposite. Even if you will probably not need to calculate a Fibonacci sequence or order arrays as functions to perform this task are already existing in most languages.

But learning algorithms can give you a better way to think and a new way to tackle problems. Those new skills can help you in your daily tasks.

If you want to start to learn it, here some starting point :
* [Introduction to Algorithms, Third Edition](https://mitpress.mit.edu/books/introduction-algorithms-third-edition)
* [Algorithms, 4th Edition](https://algs4.cs.princeton.edu/home/)
* [MIT 6.006 Introduction to Algorithms, Fall 2011](https://www.youtube.com/playlist?list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb)

Those three sources are excellent. If you prefer a book, the first one is, in my opinion, the best of both. 
If you prefer videos, the ones from MIT are outstanding.