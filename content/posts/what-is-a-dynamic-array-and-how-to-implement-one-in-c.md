---
title: "What Is a Dynamic Array and How to Implement One in C"
date: 2017-08-22
tags: ["c", "arrays", "data structures"]
categories: ["tutorials"]
---

This guide has been written as a complete introduction to dynamic arrays, with comparisons against similar array types like dynamically allocated and fixed-size arrays, and also a tutorial on how to implement one in C.
> # [Link to the repository of code.](https://github.com/shnupta/practice-c/tree/master/darray)

**Prerequisites**: you should have a basic understanding of pointers and some knowledge of programming in C.

Let’s get to it!

## First let’s define an array.

An array is a contiguous area of memory, consisting of equally sized elements, indexed by contiguous integers.

Typically, an array has a fixed size when declared in most languages.

```c
int my_array[10]; // declares an array of integers with capacity 10
```

A fixed-size array means that once the array has been declared with its size, it cannot then be changed. The standard array in C is a fixed size array. However, a fixed size array (static) can present some problems, we must provide the size of the array at compile time. There is a way round this by using dynamically-allocated arrays. Dynamically-allocated arrays can specify the size of the array in run time. Something like this:

```c
int *my_array = new int[size];
```

But the problem that still resides is: what if we don’t know the maximum size of the array when we allocate it?

This is where dynamic arrays come in.

## Dynamic Arrays

The idea of a dynamic array is to have a pointer to a dynamically-allocated array (see above), and then when the size of the array becomes bigger than what was allocated, we create a new dynamically-allocated array, with a larger size, and then reset our pointer to that. We can also call a dynamic array an automatically resizing array.

### What should it be able to do?

Typically a dynamic array should be able to:

* Get( **i** ): return the element at position **i** — O(1)

* Set( **i**, **val** ): set the element at position **i** to the value **val**

* Push( **val** ): add **val** to the end of the array — typically O(1) but can differ (see later)

* Remove( **i** ): remove the element at position **i** — O(n)

* Size(  ): return the number of elements in the array

## The Implementation

The files we will have for the implementation of the dynamic array are:

* darray.h — header file containing the struct definition and function declarations

* darray.c — the source code for the functions

Our *darray.h* file will look like [this](https://github.com/shnupta/practice-c/blob/master/darray/darray.h).

First of all we need to declare our array structure:
```c
typedef struct darray {
    void **items;
    int size;
    int capacity;
} darray;
```

items is our ‘dynamically-allocated’ array. size is pretty self explanatory and capacity is the maximum elements that items can hold at the current time.

We are going to have 15 functions for our *darray*.

```c
void darray_init(darray *);
void darray_init_with_cap(darray *, int);
void darray_destroy(darray *);
int darray_size(darray *);
int darray_capacity(darray *);
int darray_is_empty(darray *);
void* darray_at(darray *, int);
void darray_push(darray *, void *);
void darray_insert(darray *, int, void *);
void darray_prepend(darray *, void *);
void *darray_pop(darray*);
void darray_delete(darray*, int);
void darray_remove(darray*, void *);
int darray_find(darray*, void *);
void darray_resize(darray*, int);
```

*darray_init* will be our allocation function. Here the array will be created and the memory for it will be allocated.

```c
void darray_init(darray *vec) {
    vec->capacity = DARRAY_INIT_CAPACITY;
    vec->size = 0;
    vec->items = malloc(sizeof(void *) * vec->capacity);
}
```

First of all, we set the capacity of the vector to our initialisation capacity (which in this case is 16). Then we set the size to zero as our array contains no elements and finally we use **malloc()** to allocate some memory for our items array. The **malloc()** function allocates the provided number of bytes of memory and returns a pointer to the allocated memory. Now we have some memory to play around with and use to store our data.

The *darray_init_with_cap* is an optional function that I wrote to allow for a bit more freedom. You just pass in the capacity size rather than using the standard macro.

Before we continue, we also need a way to clear up our *darray* when we are finished with it — we don’t want any memory leaks! Our *darray_destroy* function simply uses **free()** to deallocate the memory we allocated for *items* and sets the *size* and *capacity* to NULL;

The next few functions in the header file are pretty self explanatory. Return the size, capacity or 1 if the size is zero. You get the idea.

Now for some more interesting functions…

Obviously we need to access the elements inside our array, but we must do this at O(1) runtime. This is what makes an array special.

Remember our array is indexed by a range of contiguous integers so we can use this to access the element. Accessing an element is fairly straight forward. We need the memory address of where our elements start, the size of each element (in this case it’s the size of a void pointer) and the index that we want to access.

```c
void* darray_at(darray *vec, int index) {
    assert(index < vec->size && index >= 0);
    return *(vec->items + sizeof(void *) * index);
}
```

The assert statement I’m using is just checking that the index we are trying to access actually exists in our array, if it doesn’t we don’t want to let the program continue as it could end up badly. To access the element we take the address of our array vec->items and then add to that the number of bytes until the element that we want to access. We do this by using the **sizeof()** function which returns the size in bytes of the argument we pass in, and we then times this by our index as that is how many bytes until our element begins. We then dereference this memory address which returns our void * element inside our array!

What about adding elements? One of the simplest ways to add an element to a dynamic array is to push to the end of the array.

```c
void darray_push(darray *vec, void *item) {
    if (vec->size == vec->capacity) {
        darray_resize(vec, vec->capacity * 2);
    }
    *(vec->items + sizeof(void *) * vec->size) = item;
    vec->size++;
}
```

We will get to the resizing of our array at the end of the tutorial. Here we are setting the item at the pointer location that is the index of *size* equal to the value of our pointer. Because we are using zero-based indexing, the element at position *size* is the next element at the end of our array. Finally we increment the size of the array.

It’s all well and good slapping a value on the end, but sometimes we want to slot an item into the middle of the array and keep all of our other elements. Here is where inserting comes in.

```c
void darray_insert(darray *vec, int index, void *item) {
    assert(index > 0 && index < vec->size);
    if(vec->size == vec->capacity) {
        darray_resize(vec, vec->capacity * 2);
    }
    for (int i = vec->size - 1; i >= index; i--) {
        *(vec->items + sizeof(void *) * (i + 1)) = *(vec->items + sizeof(void *) * i);
    }
    *(vec->items + sizeof(void *) * index) = item;
    vec->size++;
}
```

Once again we use the **assert()** statement to check the passed in *index* is valid and also check if the array needs to be resized. Before we can actually insert our value into the *index*, we need to make space for it. We cannot simply change the value at the *index* as we’d be overwriting the existing value. So what we do is loop from the last value of our array until we reach the *index* we want to insert at, and copy that value to the index that is one greater than it. We cannot loop from the *index* towards the end of the array as we would just be copying the same value across each time. Once we’ve made the space we set the value at the memory address of our *index*, like before. And again we increment the size of the array.

The darray_prepend() function simply uses the darray_insert() function and passes zero as the index argument.

```c
void* darray_pop(darray *vec) {
    assert(vec->size > 0);
    if(vec->size <= vec->capacity / 4) {
        darray_resize(vec, vec->capacity / 2);
    }
    void *ret_val = *(vec->items + sizeof(void *) * (vec->size-1));
 
    *(vec->items + sizeof(void *) * (vec->size-1)) = NULL;
    vec->size--;

    return ret_val;
}
```

Popping a value is deleting the last value from our array, and returning it. At the beginning we do the normal validation check and resize check. Then we create the ret_val variable. This is a copy of the last element of our array (we need this to be able to return the element after we delete it). Then we simply set the value to be NULL at the address of the last element (which is size — 1 due to zero-based indexing). Don’t forget to decrement the *size* of our array as we’ve deleted an element.

```c
void darray_delete(darray *vec, int index) {
    assert(index > 0 && index < vec->size);
    if(vec->size <= vec->capacity / 4) {
        // half the capacity in resize
        darray_resize(vec, vec->capacity / 2);
    }
    *(vec->items + sizeof(void *) * index) = NULL;
    for (int i = index; i < vec->size; i++) {
        *(vec->items + sizeof(void *) * i) = *(vec->items + sizeof(void *) * (i+1));
    }
    vec->size--;
}
```

Deleting an element in our array is similar to inserting. Only rather then shifting elements to the right before we insert, we shift elements left after deleting the item.

```c
void darray_remove(darray *vec, void *item) {
    for(int i = 0; i < vec->size; i++) {
        if (*(vec->items + sizeof(void *) * i) == item) {
            darray_delete(vec, i);
        }
    }
}
```

Our remove function looks for any elements that have the value of *item*and removes them from our array using the previous function. N.B. this will delete multiple occurrences and not just the first match.

```c
int darray_find(darray *vec, void *item) {
    for(int i = 0; i < vec->size; i++) {
        if (*(vec->items + sizeof(void *) * i) == item) {
            return i;
         }
    }
    return -1;
}
```

Our find function is very similar to our remove function. However on the first occurrence of *item* it will return the index that it is at. If a match is not found, -1 is returned.

Now onto the most important function of them all!

```c
void darray_resize(darray *vec, int new_size) {
    assert(new_size > 0);
    vec->capacity = new_size;
    void **new_items = malloc(sizeof(void *) * vec->capacity);
    new_items = vec->items;
    free(vec->items);
    vec->items = new_items;
}
```

We have called this function multiple times (when deleting, inserting). First of all we check the new size is valid. Then we change our array’s capacity property to be the new size. After we allocate memory in a similar way to how we initialised the array, this time with the new capacity. We then copy our existing elements to our newly allocated memory, afterwards freeing the previous array memory. Finally we set our array’s items collection to be the newly allocated and assigned memory.

Well that’s it! I hope you found this a bit of a no-nonsense, straight to the point way of learning and got something from it. If you have any questions or need any help, feel free to drop me an email.
