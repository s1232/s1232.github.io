---
layout: post
title:  "Apache DataFusion"
author: Arild Eide
tags: Apache DataFusion
---

# Apache Arrow
Starting out reading [the frontpage of the Apache DataFusion website](https://datafusion.apache.org/) quickly detoured into [Apache Arrow](https://arrow.apache.org).

Apache Arrow is a **columnar memory format specification** allowing zero-copy data transfer through a shared memory represntation and inter-process communication (IPC). It eliminates the need for data copy and (de)serialization across runtime and language barriers. There are native implementations of Arrow across a range of languages and ecosystems: **C++, .Net, Go, Java, Rust and Swift** as well as libraries in other langugaes built on top of the C++ implementation. There's an **Apache IPC file format**, storing to disk in the same format as the memory representation. IPC files different from the Parquet files and generally bigger no column compression is used. Compression is an important element in Parquet, it has support for **brotli, gzip, lz4, zstd and snappy**.

The **Rust implementation** of Arrow and Parquet is in the a shared repo at [https://github.com/apache/arrow-rs](https://github.com/apache/arrow-rs), but published as distinct crates.

## Arrow the format
Prominently, the official documentation presents these key features:

- Data adjacency for sequential access (scans)
- O(1) (constant-time) random access 
- SIMD and vectorization-friendly
- Relocatable without “pointer swizzling”, allowing for true zero-copy access in shared memory

The first two combined is really a key feature as these have traditionally been "either or". The relocatable element is achieved through offset rather than absolute memory addresses and really is the pillar for zero-copy.

Arrow depends on Google's Flatbuffers for metadata serialization, on which the zero-copy capability is built. Flatbuffers is created by Google (2014) and Apache 2.0 licensed. This [Wikipedia article](https://en.wikipedia.org/wiki/Comparison_of_data-serialization_formats) provides a table of recognized serialization formats.

The arrow repo holds **.fbs** files that defines the binary serialization format of metadata. These files are part of the specification itself. Looking at the first few built-in data types:

![Arrow data types](/images/arrow_data_types.png)

There is a definition in [Schema.fbs](https://github.com/apache/arrow/blob/main/format/Schema.fbs) for each type, for instance **Int**:

```
table Int {
  bitWidth: int; // restricted to 8, 16, 32, and 64 in v1
  is_signed: bool;
}
```

## Apache Arrow: Java - Rust interoperability
The zero-copy interop to and from Java is based Java's ability to make use of off-heap memory! This is clearly worth a detour of its own. Efficient use of heap memory and GC is a core part of Java's value proposition.

![Gemini Java off-heap](/images/java_off_heap.png)

Usually, application developers using Java only relates to the stack and the heap, but behind the scenes direct memory access is used everywhere when accessing network and disk. Increasingly, prominent Java-based projects like **Apache Spark** and **Apache Kafka** and machine learning applications have scaled out from the heap and GC controlled memory for performance reasons. These implementations rely on **sun.misc.Unsafe** assuming liabilities and risk in both spatial and temporal dimensions of memory management.

Through **Project Panama** the JDK has started offering the **Foreign Function & Memory API (FFM)** released in JDK 22, to strengthen safety and ergonomics of direct memory access and offer predictable use Vector computation.

I have been hearing about FFM for quite some time without realizing how much of a cornerstone for modern use of Java it really is. Migrating off of the unsafe implementation is a massive undertaking for existing projects and applications. It turns out the JDK ecosystem is currently in at a very critical point in time. High-profile and performance critical projects running atop **sun.misc.Unsafe** which is slated for deprecation and already blasting out console warnings.

Traditionally, the **Java Native Interface (JNI)** offered the ability for calling into C implementations from Java, but it does not offer the performance needed in AI applications. Adoption of FFM is crucial and almost existential for Java and its relevance in the coming decades.

## Next up
The Arrow crate offers the Rust native implementation of Arrow.
![Arrow crate modules](/images/arrow_crate.png)

I want to get hands-on experience using this crate, either directly or through the Parquet crate. 

