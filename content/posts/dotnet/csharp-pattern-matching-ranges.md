---
title: "C# Pattern Matching Ranges"
date: 2021-10-21T11:27:58-05:00
draft: false
---

# Pattern matching ranges

C# 9.0 introduced [new features in pattern matching](https://github.com/dotnet/csharplang/issues/2850) that allow range expressions (`<`, `>`, `<=`, `>=`) in `switch`es.

I examined the impact of using these new range expressions in an existing project.

## Original Method

[My original method](https://sharplab.io/#gist:f3261fc29db3bff6c2fc4b734373c55a) was implemented with pattern matching but without range expressions:

```csharp
private static (int month, int day) MonthDayFromJulian(int daysSinceJan1)
=> daysSinceJan1 switch
{
    int x when x >= 0 && x <= 30 => (1, daysSinceJan1 + 1),
    int x when x >= 31 && x <= 58 => (2, daysSinceJan1 - 30),
    int x when x >= 59 && x <= 89 => (3, daysSinceJan1 - 58),
    int x when x >= 90 && x <= 119 => (4, daysSinceJan1 - 89),
    int x when x >= 120 && x <= 150 => (5, daysSinceJan1 - 119),
    int x when x >= 151 && x <= 180 => (6, daysSinceJan1 - 150),
    int x when x >= 181 && x <= 211 => (7, daysSinceJan1 - 180),
    int x when x >= 212 && x <= 242 => (8, daysSinceJan1 - 211),
    int x when x >= 243 && x <= 272 => (9, daysSinceJan1 - 242),
    int x when x >= 273 && x <= 303 => (10, daysSinceJan1 - 272),
    int x when x >= 304 && x <= 333 => (11, daysSinceJan1 - 303),
    int x when x >= 334 && x <= 364 => (12, daysSinceJan1 - 333),
    _ => throw new ArgumentOutOfRangeException(nameof(daysSinceJan1))
};
```

It includes some ugly boilerplate to create the temporary `int x` variable for each case, in order to compare the pattern expression. The IL produced by this implementation also sets up a local variable for each `int x` declared:

```cil
.method private hidebysig static 
    valuetype [System.Private.CoreLib]System.ValueTuple`2<int32, int32> MonthDayFromJulian (
        int32 daysSinceJan1
    ) cil managed 
{
    .param [0]
        .custom instance void [System.Private.CoreLib]System.Runtime.CompilerServices.TupleElementNamesAttribute::.ctor(string[]) = (
            01 00 02 00 00 00 05 6d 6f 6e 74 68 03 64 61 79
            00 00
        )
    // Method begins at RVA 0x2080
    // Code size 445 (0x1bd)
    .maxstack 3
    .locals init (
👉        [0] int32 x,
👉        [1] int32 x,
👉        [2] int32 x,
👉        [3] int32 x,
👉        [4] int32 x,
👉        [5] int32 x,
👉        [6] int32 x,
👉        [7] int32 x,
👉        [8] int32 x,
👉        [9] int32 x,
👉        [10] int32 x,
👉        [11] int32 x,
        [12] valuetype [System.Private.CoreLib]System.ValueTuple`2<int32, int32>,
        [13] valuetype [System.Private.CoreLib]System.ValueTuple`2<int32, int32>
    )
```

Fortunately, when this expression is compiled to procedural code, the compiler inlines these temporary variables and generates a gateway-style method with several flat branches:

```csharp
[return: TupleElementNames(new string[] { "month", "day" })]
private static ValueTuple<int, int> MonthDayFromJulian(int daysSinceJan1)
{
    if (daysSinceJan1 >= 0 && daysSinceJan1 <= 30)
    {
        return new ValueTuple<int, int>(1, daysSinceJan1 + 1);
    }
    if (daysSinceJan1 >= 31 && daysSinceJan1 <= 58)
    {
        return new ValueTuple<int, int>(2, daysSinceJan1 - 30);
    }
    if (daysSinceJan1 >= 59 && daysSinceJan1 <= 89)
    {
        return new ValueTuple<int, int>(3, daysSinceJan1 - 58);
    }
    if (daysSinceJan1 >= 90 && daysSinceJan1 <= 119)
    {
        return new ValueTuple<int, int>(4, daysSinceJan1 - 89);
    }
    if (daysSinceJan1 >= 120 && daysSinceJan1 <= 150)
    {
        return new ValueTuple<int, int>(5, daysSinceJan1 - 119);
    }
    if (daysSinceJan1 >= 151 && daysSinceJan1 <= 180)
    {
        return new ValueTuple<int, int>(6, daysSinceJan1 - 150);
    }
    if (daysSinceJan1 >= 181 && daysSinceJan1 <= 211)
    {
        return new ValueTuple<int, int>(7, daysSinceJan1 - 180);
    }
    if (daysSinceJan1 >= 212 && daysSinceJan1 <= 242)
    {
        return new ValueTuple<int, int>(8, daysSinceJan1 - 211);
    }
    if (daysSinceJan1 >= 243 && daysSinceJan1 <= 272)
    {
        return new ValueTuple<int, int>(9, daysSinceJan1 - 242);
    }
    if (daysSinceJan1 >= 273 && daysSinceJan1 <= 303)
    {
        return new ValueTuple<int, int>(10, daysSinceJan1 - 272);
    }
    if (daysSinceJan1 >= 304 && daysSinceJan1 <= 333)
    {
        return new ValueTuple<int, int>(11, daysSinceJan1 - 303);
    }
    if (daysSinceJan1 >= 334 && daysSinceJan1 <= 364)
    {
        return new ValueTuple<int, int>(12, daysSinceJan1 - 333);
    }
    throw new ArgumentOutOfRangeException("daysSinceJan1");
}
```

On x64, this produces jitted output closely matching the procedural output, with several branching comparisons against our `daysSinceJan1` parameter.

## Range-based method

I hoped that [replacing the temporary variable usage with range-based pattern matching](https://sharplab.io/#gist:a6f31c220be9c35c2e8343f6d0b36cc1) would avoid the unnecessary temporary variables and speed up the lookup:

```csharp
private static (int month, int day) MonthDayFromJulian(int daysSinceJan1)
=> daysSinceJan1 switch
{
    (>= 0 and <= 30) => (1, daysSinceJan1 + 1),
    (>= 31 and <= 58) => (2, daysSinceJan1 - 30),
    (>= 59 and <= 89) => (3, daysSinceJan1 - 58),
    (>= 90 and <= 119) => (4, daysSinceJan1 - 89),
    (>= 120 and <= 150) => (5, daysSinceJan1 - 119),
    (>= 151 and <= 180) => (6, daysSinceJan1 - 150),
    (>= 181 and <= 211) => (7, daysSinceJan1 - 180),
    (>= 212 and <= 242) => (8, daysSinceJan1 - 211),
    (>= 243 and <= 272) => (9, daysSinceJan1 - 242),
    (>= 273 and <= 303) => (10, daysSinceJan1 - 272),
    (>= 304 and <= 333) => (11, daysSinceJan1 - 303),
    (>= 334 and <= 364) => (12, daysSinceJan1 - 333),
    _ => throw new ArgumentOutOfRangeException(nameof(daysSinceJan1))
};
```

The generated IL confirms that the additional locals are avoided:

```cil
.method private hidebysig static 
    valuetype [System.Private.CoreLib]System.ValueTuple`2<int32, int32> MonthDayFromJulian (
        int32 daysSinceJan1
    ) cil managed 
{
    .param [0]
        .custom instance void [System.Private.CoreLib]System.Runtime.CompilerServices.TupleElementNamesAttribute::.ctor(string[]) = (
            01 00 02 00 00 00 05 6d 6f 6e 74 68 03 64 61 79
            00 00
        )
    // Method begins at RVA 0x2080
    // Code size 343 (0x157)
    .maxstack 3
👉  .locals init (
        [0] valuetype [System.Private.CoreLib]System.ValueTuple`2<int32, int32>,
        [1] valuetype [System.Private.CoreLib]System.ValueTuple`2<int32, int32>
    )
```

However, the procedural output is now a bubble-style method with nested branches: 

```csharp
[return: TupleElementNames(new string[] { "month", "day" })]
private static ValueTuple<int, int> MonthDayFromJulian(int daysSinceJan1)
{
    if (daysSinceJan1 <= 150)
    {
        if (daysSinceJan1 > 89)
        {
            if (daysSinceJan1 <= 119)
            {
                return new ValueTuple<int, int>(4, daysSinceJan1 - 89);
            }
            return new ValueTuple<int, int>(5, daysSinceJan1 - 119);
        }
        if (daysSinceJan1 > 30)
        {
            if (daysSinceJan1 <= 58)
            {
                return new ValueTuple<int, int>(2, daysSinceJan1 - 30);
            }
            return new ValueTuple<int, int>(3, daysSinceJan1 - 58);
        }
        if (daysSinceJan1 >= 0)
        {
            return new ValueTuple<int, int>(1, daysSinceJan1 + 1);
        }
    }
    else
    {
        if (daysSinceJan1 <= 272)
        {
            if (daysSinceJan1 <= 211)
            {
                if (daysSinceJan1 <= 180)
                {
                    return new ValueTuple<int, int>(6, daysSinceJan1 - 150);
                }
                return new ValueTuple<int, int>(7, daysSinceJan1 - 180);
            }
            if (daysSinceJan1 <= 242)
            {
                return new ValueTuple<int, int>(8, daysSinceJan1 - 211);
            }
            return new ValueTuple<int, int>(9, daysSinceJan1 - 242);
        }
        if (daysSinceJan1 <= 333)
        {
            if (daysSinceJan1 <= 303)
            {
                return new ValueTuple<int, int>(10, daysSinceJan1 - 272);
            }
            return new ValueTuple<int, int>(11, daysSinceJan1 - 303);
        }
        if (daysSinceJan1 <= 364)
        {
            return new ValueTuple<int, int>(12, daysSinceJan1 - 333);
        }
    }
    throw new ArgumentOutOfRangeException("daysSinceJan1");
}
```

On x64, this produces jitted output of a jump table, as if our pattern matching expression was a large `switch` statement with fall-through cases in between the comparison values.

## Performance differences

These differences in code generation do have an effect on real-world performance, primarily through reducing branch mispredictions.

```
BenchmarkDotNet=v0.13.1, OS=Windows 10.0.22000
AMD Ryzen 9 3900X, 1 CPU, 24 logical and 12 physical cores
.NET SDK=6.0.100-rc.2.21505.57
  [Host]     : .NET 6.0.0 (6.0.21.48005), X64 RyuJIT
  Job-PKIECE : .NET 6.0.0 (6.0.21.48005), X64 RyuJIT

RunStrategy=Throughput
```

|         Method |     Mean |     Error |    StdDev | BranchInstructions/Op | BranchMispredictions/Op |
|--------------- |---------:|----------:|----------:|----------------------:|------------------------:|
|    CountMonths | 2.594 us | 0.0160 us | 0.0134 us |                 6,443 |                      22 |
| CountMonthsNew | 2.327 us | 0.0079 us | 0.0070 us |                 3,333 |                      14 |