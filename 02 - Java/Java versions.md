### Most important recent releases additions

1. Switch expression
2. `instanceof` pattern matching - auto-casting
3. Text blocks
4. Records
5. NPE stack-trace enhancement
6. Sealed classes and Hidden classes
7. String templates
8. Virtual threads

### Java 8 (March 2014)
1. **Lambda Expressions**: Introduced a concise way to represent anonymous functions.
2. **Stream API**: Added a powerful API for processing sequences of elements.
3. **Optional Class**: Introduced to handle null values more gracefully.
4. **Default Interface Methods**: Allowed interfaces to have methods with implementations.
5. **Date and Time API**: Introduced a new date and time API (`java.time` package) based on Joda-Time.
6. **Concurrent Accumulators**: Enhanced concurrency utilities with classes like `LongAdder` and `DoubleAdder`.
7. **Type Annotations**: Allowed annotations to be used in more places, including type declarations.

### Java 9 (September 2017)
1. **Module System (Project Jigsaw)**: Introduced the Java Platform Module System (JPMS) for modularizing the JDK and applications.
2. **JShell**: Added an interactive REPL (Read-Eval-Print Loop) tool.
3. **Process API Improvements**: Enhanced the API for managing and controlling operating system processes.
4. **Multi-Release JAR Files**: Allowed JAR files to contain version-specific class files.
5. **Stream API Enhancements**: Added new methods like `takeWhile`, `dropWhile`, and `iterate`.

### Java 10 (March 2018)
1. **var keyword**: Introduced the `var` keyword for local variable type inference.
2. **Garbage Collector Interface**: Provided a clean interface for garbage collectors.
3. **Application Class-Data Sharing**: Improved startup and footprint by sharing common class metadata.
4. **Parallel Full GC for G1**: Enhanced the G1 garbage collector to use parallel threads for full GC.

### Java 11 (September 2018)
1. **HTTP Client**: Standardized the HTTP client API introduced in Java 9.
2. **var for Lambda Parameters**: Allowed `var` to be used in lambda expressions.
3. **New String Methods**: Added methods like `isBlank`, `lines`, `strip`, `repeat`.
4. **File Methods**: Added methods like `readString` and `writeString` to `java.nio.file.Files`.
5. **Removed Java EE and CORBA Modules**: Removed modules related to Java EE and CORBA.

### Java 12 (March 2019)
1. **Switch Expressions (Preview)**: Introduced switch expressions to simplify switch statements.
2. **Default CDS Archives**: Enabled Class-Data Sharing (CDS) by default to improve startup time.
3. **G1 Garbage Collector Enhancements**: Improved garbage collection performance.
4. **Microbenchmark Suite**: Added a suite for microbenchmarking.

### Java 13 (September 2019)
1. **Text Blocks (Preview)**: Introduced text blocks for multi-line string literals.
2. **Switch Expressions (Second Preview)**: Continued the preview of switch expressions with refinements.
3. **Reimplementation of the Legacy Socket API**: Improved performance and maintainability of the socket API.

### Java 14 (March 2020)
1. **Pattern Matching for `instanceof` (Preview)**: Simplified type checks and casting.
2. **Records (Preview)**: Introduced records to simplify the creation of data carrier classes.
3. **Text Blocks (Second Preview)**: Continued the preview of text blocks with refinements.
4. **Helpful NullPointerExceptions**: Enhanced NullPointerExceptions with more informative messages.

### Java 15 (September 2020)
1. **Text Blocks**: Standardized text blocks.
2. **Sealed Classes (Preview)**: Introduced sealed classes to restrict which classes can extend or implement them.
3. **Hidden Classes**: Added support for hidden classes, which are intended for use by frameworks.
4. **Pattern Matching for `instanceof` (Second Preview)**: Continued the preview of pattern matching for `instanceof`.

### Java 16 (March 2021)
1. **Records**: Standardized records.
2. **Pattern Matching for `instanceof`**: Standardized pattern matching for `instanceof`.
3. **Sealed Classes (Second Preview)**: Continued the preview of sealed classes.
4. **Vector API (Incubator)**: Introduced an incubating API for vector computations.
5. **Foreign Linker API (Incubator)**: Introduced an incubating API for linking native code.

### Java 17 (September 2021)
1. **Sealed Classes**: Standardized sealed classes.
2. **Pattern Matching for `switch` (Preview)**: Introduced pattern matching for switch statements.
3. **Foreign Function & Memory API (Incubator)**: Introduced an incubating API for working with foreign functions and memory.
4. **Context-Specific Deserialization Filters**: Enhanced security by allowing context-specific deserialization filters.

### Java 18 (March 2022)
1. **Simple Web Server**: Introduced a simple web server for prototyping and testing.
2. **UTF-8 by Default**: Made UTF-8 the default charset for the standard Java APIs.
3. **Code Snippets in Java API Documentation**: Enhanced API documentation with code snippets.
4. **Vector API (Third Incubator)**: Continued the incubating Vector API with refinements.

### Java 19 (September 2022)
1. **Virtual Threads (Preview)**: Introduced virtual threads to simplify concurrency.
2. **Structured Concurrency (Incubator)**: Introduced an incubating API for structured concurrency.
3. **Pattern Matching for `switch` (Third Preview)**: Continued the preview of pattern matching for switch statements.
4. **Foreign Function & Memory API (Second Incubator)**: Continued the incubating API for foreign functions and memory.

### Java 20 (March 2023)
1. **Pattern Matching for `switch` (Fourth Preview)**: Continued the preview of pattern matching for switch statements.
2. **Record Patterns (Preview)**: Introduced record patterns for deconstructing record values.
3. **Foreign Function & Memory API (Third Incubator)**: Continued the incubating API for foreign functions and memory.
4. **Virtual Threads (Second Preview)**: Continued the preview of virtual threads with refinements.

### Java 21 (September 2023)
1. **Pattern Matching for `switch`**: Standardized pattern matching for switch statements.
2. **Record Patterns**: Standardized record patterns.
3. **Virtual Threads**: Standardized virtual threads.
4. **Structured Concurrency**: Standardized structured concurrency.
5. **Foreign Function & Memory API**: Standardized the API for foreign functions and memory.
6. **Sequenced Collections**: Introduced sequenced collections to provide a common interface for collections with a defined encounter order.
7. **String Templates (Preview)**: Introduced string templates for easier and safer string interpolation.