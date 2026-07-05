---
layout: post
title:  "Apache DataFusion read up summary"
author: Arild Eide
tags: Apache DataFusion
---

# Apache Arrow - general outline
Starting out reading [the frontpage of the Apache DataFusion website](https://datafusion.apache.org/) quickly detoured into [Apache Arrow](https://arrow.apache.org).

Apache Arrow is a **columnar memory format specification** allowing zero-copy data transfer through a shared memory represntation and inter-process communication (IPC). It eliminates the need for data copy and (de)serialization across runtime and language barriers. There are native implementations of Arrow across a range of languages and ecosystems: **C++, .Net, Go, Java, Rust and Swift** as well as libraries in other langugaes built on top of the C++ implementation. There's an **Apache IPC file format**, storing to disk in the same format as the memory representation. IPC files different from the Parquet files and generally bigger no column compression is used. Compression is an important element in Parquet, it has support for **brotli, gzip, lz4, zstd and snappy**.

The **Rust implementation** of Arrow and Parquet is in the a shared repo at [https://github.com/apache/arrow-rs](https://github.com/apache/arrow-rs), but published as distinct crates.


## Apache Arrow: Java - Rust interoperability
The zero-copy interop to and from Java is based Java's ability to make use of off-heap memory! This is clearly worth a detour of its own. Efficient use of heap memory and GC is a core part of Java's value proposition.

![Gemini Java off-heap](/images/java_off_heap.png)

Usually, application developers using Java only relates to the stack and the heap, but if it weren't for direct memory access Java would be dead in the water by now. Both **Apache Spark** and **Apache Kafka** have scaled out from the heap and GC controlled memory and more recently machine learning has forced off heap for performance reasons. Under **Project Panama** the JDK has evolved the **Foreign Function & Memory API (FFM)** released in JDK 22 to strengthen safety and ergonomics of direct memory access.

I have been hearing about FFM without realizing how much of a cornerstone for modern use of Java it really is. It would be interesting to know to what extent large projects that predates the FFM offering, like Apache Spark, has adopted the FFM API.

It turns out the JDK ecosystem is currently in at a very critical point in time. High-profile and performance critical projects like Spark and Kafka are running atop **sun.misc.Unsafe** which is slated for deprecation and already blasting out console warnings. Traditionally, the **Java Native Interface (JNI)** offered the ability for calling into C implementations from Java, but it does not offer the performance needed in AI applications. Adoption of FFM is crucial and almost existential for Java and its relevance in the coming decades.

# Questions and to-dos
Apache arrow is said to support both **random access and scan-based workloads**. Why is that the case and how can it be demonstrated?

Explore the practical relation between Parquet and Arrow. Would reading a parquet file from disk using the [Parquet crate](https://crates.io/crates/parquet) create an Arrow representation in memory?


