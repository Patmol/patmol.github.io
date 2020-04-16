---
title: "Why you need to learn algorithm"
date: 2020-03-15 14:10:00 +0800
categories: [Computer Science, Algorithm]
tags: [cs, algorithm]
---

If you are a self-taught developer, you may direclty learn how to build something, without studying computer science or at least, how a good or a bad alorithm can change a lot the performance of the application you are working on.

In a lot of language tutorial I've read in the past, there is usually not a single word on the importance on thinking about what you are doing and the importance of a good algorithm. As a toturial reader, we want to directly build something with the new language we are learning, see all the code we had written in action without thinking about the performance of this code and on small applications and small datasets, there is not a big difference between good and bad codes.

But when you will start to work on bigger application with bigger dataset, the difference can be huge!

## Array ordering

Let's take a classical example, the array ordering. Of course, most of the languages are provinding methods to order an array but it's always to understand what's under the hood and it's an easy example to start with.

### The _naïve_ method
An easy way to order an array is to compare an element with the next one and if the first element is bigger than the second one, swap them otherwise, leave them like this. Then, do the compaison between the second and the third element. And once again, if the second is bigger than thrid, swap them.
By performing this action from the first element to the last one, you will put the biggest element at the end of the array.

To order the array, you will need to perform those action again, from the first one to element before the one you set at the end before.

Now we have an idea of what we need to do to order an array, let's write some C++ code to do it!

```cpp
void sortArrayNaive(int array[], int arraySize) {
    int tmp;
    // arraySize - 1 because we don't need to test the last element of the array.
    // We decrease the size of the array to order.
    for (int i(arraySize - 1); i >= 0 ; i--) {
        // We always start from the beginning (we could have also start for the end).
        for (int j(0); j < i; j++) {
            // We check if the current element is bigger than the next one ...
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

Once we have our fonction, we need to test its performance. 
For those tests, I've launch this function with arrays of different size and here are the results.

- Test with 100 elements : 27 microseconds
- Test with 1,000 elements : 2,402 microseconds
- Test with 10,000 elements : 286,790 microseconds
- Test with 100,000 elements : 38,754,410 microseconds (38 seconds)

With those tests, we can see with just 100,000 elements, it take more than 35 seconds to be ordered, which is a lot!

### A better method
Lets see how we can improve those times and for this, we will use the merge sort algorithm.

With the merge sort, we divide our array in two equal part and do this division until we reach our basic case which is have an array of size of 1.
Once the division is perform, we will merge our smaller array into a bigger one and while merging, we will order the values.

You can find more details information on [Wikipedia](https://en.wikipedia.org/wiki/Merge_sort).

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

- Test with 100 elements : 77 microseconds
- Test with 1,000 elements : 447 microseconds
- Test with 10,000 elements : 5,541 microseconds
- Test with 100,000 elements : 49,838 microseconds (0,049 seconde)
- Test with 10,000,000 elements : 4,581,554 microseconds

Except when we have 100 elements or less, the merge sort is faster. And when we have more than 100,000 elements, we start to have a huge difference.
From 38 secondes, we now have 1/20 of a second!

I've tested the merge sort with 10,000,000 elements and it's only took 4.5 seconds! I don't have want to imagine how long it's would have take with the _naive_ approach.

So, even if the code is a little more complicate, the difference between the first implementation and the second one is huge! Using the right algorithm in you application can be a time saver.

## Fibonacci numbers

The Fibonacci numbers is a sequence of number such as each number is the sum of the two previous one, starting to 0 and 1.

We have 
    F_0 = 0
    F_1 = 1
and
    F_n = F_(n - 1) + F_(n - 2)

for n > 1

### The _naïve_ way
Lets try to calculate the Xth element of the Fibonacci sequence. It can be done quite easily with some recurisivity.

```cpp
long long int fib(int n) {
    if (n == 0 || n == 1) {
        return n;
    }

    return fib(n - 1) + fib(n - 2);
}
```

Easy, right ? Now, we will perform some testing on this code.

Test for the 10th element : 0 microseconds
Test for the 30th element : 8,273 microseconds
Test for the 40th element : 512,690 microseconds
Test for the 50th element : 61,766,410 microseconds (60 seconds)

The main problem with this code is that we are performing a lot of re-computation.
To compute for 5, we need to compute for 4 and for 3, to compute for 4, we need to compute for 3 and for 2, so with just this case, we can see we are computing 3, two times. And if you check with bigger number, you will a see all those re-computation.

### The #MethodWay

## The big O notation 

## So, why do I need to learn alogrithm ?
