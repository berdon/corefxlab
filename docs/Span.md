#Span\<T\>

## Introduction

Span\<T\> is a new type we are adding to the platform to represent contiguous regions of arbitrary memory, with perfromance characteristics on pair with T[]. Its APIs are similar to the array, but unlike arrays, it can point to either managed or native memory, or to memory allocated on the stack. 

```c#
// managed memory
var arrayMemory = new byte[100];
var arraySpan = new Span<byte>(arrayMemory);

// native memory
var nativeMemory = Marshal.AllocHGlobal(100);
Span<byte> nativeSpan;
unsafe {
    nativeSpan = new Span<byte>(nativeMemory.ToPointer(), 100);
}
SafeSum(nativeSpan);
Marshal.FreeHGlobal(nativeMemory);

// stack memory
Span<byte> stackSpan;
Unsafe {
    byte* stackMemory = stackalloc byte[100];
    stackSpan = new Span<byte>(stackMemory, 100);
}
SafeSum(stackSpan);
```

As such, Span\<T\> is an abstraction over all types of memory available to .NET programs.

```c#
// this method does not care what kind of memory it works on
static ulong SafeSum(Span<byte> bytes) { 
    ulong sum = 0;
    for(int i=0; i < bytes.Length; i++) {
        sum += bytes[i];
    }
    return sum;
}
```
When wrapping an array, Span\<T\> is not limited to pointing to the first element of the array. It can point to any sub-range. In other words, it supports slicing.
```c#
var array = new byte[] { 1, 2, 3 };
var slice = new Span<byte>(array, start:1, length:2); 
Console.WriteLine(slice[0]); // prints 2
```
##API Surface
The full API surface of Span\<T\> is not yet finalized, but the main APIs we will expose are the following:   

```c#
public struct Span<T> {
    public Span(T[] array)
    public Span(T[] array, int index)
    public Span(T[] array, int index, int length)
    public unsafe Span(void* memory, int length)

    public static implicit operator Span<T> (ArraySegment<T> arraySegment);
    public static implicit operator Span<T> (T[] array);

    public int Length { get; }
    public ref T [int index] { get; }

    public Span<T> Slice(int index);
    public Span<T> Slice(int index, int length);
    public bool TryCopyTo(T[] destination);
    public bool TryCopyTo(Span<T> destination);

    public T[] ToArray();
}
```
In addition, we will provide a read-only version of Span\<T\>. The ReadOnlySpan\<T\> is required to represent slices of immutable and read-only structures, e.g. System.String slices. This is discussed below.

## Scenarios

TODO