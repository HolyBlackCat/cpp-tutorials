# Passing and returning arrays

Arrays are a bit of a special case, in how they can be passed and returned from functions. (That is the plain arrays, as opposed to `std::vector` and other alternatives.)

## Returning arrays from functions

They can't be returned from functions (at least not directly), attempting that is a compilation error.

Your options are:

* Returning `std::vector`. (Which is what you'll be doing most of the time, since you in general will be using `std::vector` more often than plain arrays, due to it not having a fixed size).

* Making the array a member of a `struct` and returning the entire struct. (Only makes sense if you want a struct in the first place. Creating one just to hold the one array is silly.)


TODO std::array



But putting one in a struct allows you to return the whole struct.

They can serve as parameters, but when used as a parameter, the array size is ignored. E.g. you can do `void foo(int array[10])`, and then pass `int arr[5]` to it, and the array size mismatch isn't an error.

In fact, you can omit the array size in the parameter entirely: `void foo(int array[])`. This is equivalent to the above.

In all those cases, if you want to accept arrays of different sizes, you should manually pass the array size in a separate parameter. That is assuming you need to know the size; e.g. a null-terminated `char` array avoids that.

You may remember `std::size` from previous chapters, which can normally be used to determine the size of an array, but no, it doesn't work on array parameters.

This will be explained in more details later, including the exact nature of those fake array parameters.

**All of this only applies to plain arrays. `std::vector` doesn't have those limitations.**
