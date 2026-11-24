# StronglyTypedIds

This repo shows how we can use various coding constructs to implement Strongly Typed IDs and also to compare how each technique performs.

Here is a sample run from BenchmarkDotNet on .NET 11:

```text
// * Summary *

BenchmarkDotNet v0.16.0-preview.2, macOS 27.2.1 (26B131) [Darwin 27.2.0]
Apple M4 Pro, 1 CPU, 14 logical and 14 physical cores
.NET SDK 11.0.100
  [Host]     : .NET 11.0.0 (11.0.0, 11.0.26.53018), Arm64 RyuJIT armv8.0-a
  DefaultJob : .NET 11.0.0 (11.0.0, 11.0.26.53018), Arm64 RyuJIT armv8.0-a


| Method                 | Mean      | Error     | StdDev    | Gen0    | Allocated |
|----------------------- |----------:|----------:|----------:|--------:|----------:|
| ReadonlyRecordStructId |  4.909 us | 0.0291 us | 0.0272 us |       - |         - |
| StructId               |  4.934 us | 0.0322 us | 0.0302 us |       - |         - |
| VogenId                |  4.936 us | 0.0193 us | 0.0181 us |       - |         - |
| RecordId               | 27.847 us | 0.0469 us | 0.0366 us | 38.2385 |  320000 B |
| SealedRecordId         | 28.737 us | 0.3960 us | 0.3510 us | 38.2385 |  320000 B |
| ClassId                | 29.495 us | 0.4366 us | 0.3870 us | 38.2385 |  320000 B |
```

Compared with the earlier .NET 9 run on the same machine (Apple M4 Pro):

| Method                 | .NET 9 Mean | .NET 11 Mean | Change | .NET 9 Allocated | .NET 11 Allocated |
|----------------------- |------------:|-------------:|-------:|-----------------:|------------------:|
| ReadonlyRecordStructId |    6.347 us |     4.909 us |   -23% |             64 B |               0 B |
| StructId               |    6.378 us |     4.934 us |   -23% |             64 B |               0 B |
| VogenId                |    6.452 us |     4.936 us |   -24% |             64 B |               0 B |
| RecordId               |   28.931 us |    27.847 us |    -4% |         320064 B |          320000 B |
| SealedRecordId         |   29.340 us |    28.737 us |    -2% |         320064 B |          320000 B |
| ClassId                |   29.952 us |    29.495 us |    -2% |         320064 B |          320000 B |

The struct-based IDs are now allocation free: the 64 B per call on .NET 9 was the closure delegate passed to `List<T>.ForEach`, which the newer JIT's escape analysis can now keep off the heap. The class and record IDs still allocate one 32 B object per ID (10,000 per run), so they remain roughly 6x slower.

> BenchmarkDotNet 0.15.x does not recognise the .NET 11 runtime, so the project uses `0.16.0-preview.2` until a stable 0.16 ships.
