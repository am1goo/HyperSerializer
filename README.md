# HyperSerializer
![]()
[![NuGet version (HyperSerializer)](https://img.shields.io/badge/nuget-v1.0.6-blue?style=flat-square)](https://www.nuget.org/packages/HyperSerializer/)

Blazing fast binary serialization up to 18 times faster than MessagePack and Protobuf with roughly equivelant memory allocation (see Benchmarks below).  HyperSerializer uses the Span<T> and Memory<T> managed memory structs to acheive high speed and low memory allocation without unsafe code.  HyperSerializer is 100% thread-safe and comes with both sync and async serialization and deserialization methods.  Out of the box support for .NETCoreApp 3.1, net5.0, net6.0.  See "About HyperSerializer" for more detail.
    
HyperSerializer is intended for use cases such as caching, and interservice communication behind firewalls or between known parites.  It is implemented using a customer binary format (aka wire format) and uses bounding techniques to protect against buffer overflows.  As a result, attempting to deserialize a message that exceeds the size of an expected data type will result in an exception in most cases.  For example, the following code which can be found in SerializerTests.cs in the test project attempts to deserialize an 8 BYTE buffer as a 4 BYTE int, which results in an ArgumentOutOfRangeException:

```csharp
long i = 1L << 32;
Span<byte> buffer = default;
MemoryMarshal.Write(buffer, ref i);
var deserialize = HyperSerializer<int>.Deserialize(buffer);
```
In the event the destiation data type was (1) an 8 BYTES in length or (2) an object containing properties with an aggregate size exceeding 8 BYTES, one of the following would occur: (1) a data type specific execption, in most cases - ArguementOutOfRangeException, OR (2) no exception at all if the bytes happen to represent valid values for the destination type(s).

## Usage
HyperSerializer is a contract-less serializalizer that supports serializing primatives, structs and classes.  As a result, changing the order or removing properties from a class that existed at the time of serialization will break deserialization.  If you add proproprites without changing the names, types and order of properties that existed at the time of serialization, it should not break deserialization, but should be tested throughly.  With respect to classes, only properties with public getters and setters will be serialied (fields and properties not matching the aforementioned crieria will be ignored).

```csharp
//Sync Example
Test obj = new();
Span<byte> bytes = HyperSerializer<Test>.Serialize(obj);
Test objDeserialized = HyperSerializer<Test>.Deserialize(bytes);
    
//Async Example
Test obj = new();
Memory<byte> bytes = HyperSerializer<Test>.SerializeAsync(obj);
Test objDeserialized = HyperSerializer<Test>.DeserializeAsync(bytes);

//Deserialize byte array...
byte[] bytes; // example - bytes you received over the wire, from cache etc...
Test objDeserialized = HyperSerializer<Test>.Deserialize(bytes);
Test objDeserialized = HyperSerializer<Test>.DeserializeAsync(bytes);
```
## Benchmarks
Benchmarks performed using BenchmarkDotNet follow the intended usage pattern of serializing and deserializing a single instance of an object at a time (as opposed to batch collection serialization used in the benchmarks published by other libraries such as Apex).  The benchmarks charts displayed below represent 1 million syncronous serialization and deserialization operations of the following object:

```csharp
public class Test {
    public Guid? Gn { get; set; }
    public int A { get; set; }
    public long B { get; set; }
    public DateTime C { get; set; }
    public uint D { get; set; }
    public decimal E { get; set; }
    public TimeSpan F { get; set; }
    public Guid G { get; set; }
    public TestEnum H { get; set; }
}
```
### Benchmark Results
-***Speed - Serializing and Deserializing 1M "Test" Objects***
_(Times in Milliseconds - Lower is Better)_ - HyperSerializer is roughly 3x faster than ApexSerializer and 18x faster than MessagePack and Protobuf...
![Execution Duration](https://github.com/Hyperlnq/HyperSerializer/blob/main/BenchmarkAssets/Time.png)

-***Memory - Serializing and Deserializing 1M "Test" Objects***
_(Memory in Megabytes - Lower is Better)_ - HyperSerializer requres roughly the same memory allocation as MessagePack and half that of ApexSerializer and Protobuf...
    
![Execution Duration](https://github.com/Hyperlnq/HyperSerializer/blob/main/BenchmarkAssets/Space.png)
      
## About HyperSerializer
HyperSerializer is intended for use cases such as caching, and interservice communication behind firewalls or between known parites.  It is implemented using a customer binary format (aka wire format) and uses "bounded" serialization to protect against buffer overflows.  As a result, attempting to deserialize binary that exceeds the size of an expected data type.  For example, the following code which can be found in the test project attempts to deserialize an 8 BYTE long message as a 4 BYTE int resulting in an ArgumentOutOfRangeException:

```csharp
long i = 1L << 32;
Span<byte> buffer = default;
MemoryMarshal.Write(buffer, ref i);
var deserialize = HyperSerializer<int>.Deserialize(buffer);
```
In the event the destiation data type were an 8 BYTES in length or an object containing properties that exceeded 8 BYTES one of the following would occur: (1) no exception if the bytes popluated are within the bounds the deserialized type(s), or (2) a data type specific excption, in most cases - ArguementOutOfRangeException.
    
## Limitations 
### Unsupported types
Serialization of the following types and nested types is planned but not supported at this time (if you would like to contribute, fork away or reach out to collaborate)...

*Classses with omplex type properties (i.e. a class with a property of type ANY class).  If a class contains a nested complex type property, it will still be serialized but the property will be ignored.
*Collections (e.g. List, Dictionary, etc.). If a class contains a property of type collection, it will still be serialized but the property will be ignored.

### Property Exclusion
If you need to exclude a property from being serialized for reasons other then performance (unless nanoseconds actually matter to you), presently your only option is a DTO.  If you would like this feature added feel free to contribute or log an issue.

## NB
Note, the HyperSerializer project contains an unsafe implementation of the serializer.  It is intended for benchmarking purposes only and  In most scenarios the "safe" default implemenation outperforms the the unsafe implementation.  To be clear: DO USE HyperSerializer<T>; DO NOT USE HyperSerializerUnsafe<T>.
    
## Feedback, Suggestions and Contributions
Are all welcome!
