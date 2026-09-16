<div align="center">

# DNA SEARCH

### Disk-Based Genomic Search Engine in C++

[![C++](https://img.shields.io/badge/C%2B%2B-17%2B-00599C?style=for-the-badge\&logo=cplusplus\&logoColor=white)](https://isocpp.org/)
[![CMake](https://img.shields.io/badge/CMake-Build%20System-064F8C?style=for-the-badge\&logo=cmake\&logoColor=white)](https://cmake.org/)
[![Algorithms](https://img.shields.io/badge/Algorithms-Suffix%20Array%20%7C%20Segment%20Tree-orange?style=for-the-badge)](#architecture)
[![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](#license)

**A high-performance DNA sequence search system built from fundamental data structures and systems-level I/O.**

</div>

---

## 🧬 Overview

**DNA Search** is a disk-based genomic sequence search engine written in C++.

The system is designed to search large DNA sequence files efficiently without loading the entire dataset into memory. It combines classical string-search algorithms, range-query data structures, custom hashing, caching, and memory-mapped file access into a single search pipeline.

Instead of treating genomic search as a simple substring problem, the project explores how **data structures and systems programming can work together to build a scalable search engine**.

---

## ⚡ Core Capabilities

* 🔎 **Exact DNA sequence search**
* 🧠 **Suffix Array + Binary Search**
* 🌳 **Segment Tree range filtering**
* ⚡ **Custom Hash Map cache**
* 🔢 **Rabin-Karp rolling hash**
* 💾 **Memory-mapped file access**
* 🧬 **Automatic reverse-complement searching**
* 📍 **Position and frequency reporting**
* 🧪 **Unit + integration testing**
* 🖥️ **Interactive and command-line search modes**

---

# 🏗️ Architecture

```text
                         ┌──────────────────────┐
                         │       User Query     │
                         │  Pattern + X..Y      │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │    Input Parser      │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │ Reverse Complement   │
                         │      Generator       │
                         └──────────┬───────────┘
                                    │
                         ┌──────────▼───────────┐
                         │   Segment Tree       │
                         │ Optional Range Filter│
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │     Hash Cache       │
                         │   Cache Lookup       │
                         └──────────┬───────────┘
                              Hit   │   Miss
                               │    │
                               │    ▼
                               │ ┌────────────────┐
                               │ │  Suffix Array  │
                               │ │ + Binary Search│
                               │ └───────┬────────┘
                               │         │
                               │         ▼
                               │ ┌────────────────┐
                               │ │ Store Results  │
                               │ │   in Cache     │
                               │ └───────┬────────┘
                               │         │
                               └────┬────┘
                                    ▼
                         ┌──────────────────────┐
                         │      Results         │
                         │ Positions / Strand / │
                         │ Frequency            │
                         └──────────────────────┘
```

---

# 🧠 Algorithmic Architecture

The project deliberately combines multiple data structures, each solving a different part of the search problem.

| Component              | Purpose              | Core Idea                                              |
| ---------------------- | -------------------- | ------------------------------------------------------ |
| **Suffix Array**       | Sequence indexing    | Efficient lexicographic organization of suffixes       |
| **Binary Search**      | Pattern lookup       | `O(log n)` search over suffix positions                |
| **Segment Tree**       | Range filtering      | Restricts candidate positions to `[X, Y]`              |
| **Custom Hash Map**    | Query cache          | Constant-time average lookup                           |
| **Rolling Hash**       | Cache key generation | Rabin-Karp style hashing                               |
| **Memory Mapping**     | File access          | Access genome data without loading everything into RAM |
| **Reverse Complement** | Strand coverage      | Searches both DNA orientations                         |

---

# 🔬 Search Pipeline

A query moves through the system as follows:

```text
User Query
    │
    ▼
Parse Pattern + Optional Range
    │
    ▼
Generate Reverse Complement
    │
    ▼
Apply Segment Tree Range Filter
    │
    ▼
Check Custom Hash Cache
    │
    ├──────────── Cache Hit ────────────► Return Result
    │
    ▼
Suffix Array
    │
    ▼
Binary Search
    │
    ▼
Exact Match Positions
    │
    ▼
Store Result in Cache
    │
    ▼
Report Results
```

The output includes:

* Match positions
* Strand information
* Frequency count

---

# 📚 Suffix Array Search

The suffix array acts as the primary index.

Instead of scanning the genome character-by-character for every query, the system organizes suffixes so that matching sequences can be located using binary search.

```text
Genome
  │
  ▼
Suffix Array
  │
  ├── suffix 0
  ├── suffix 1
  ├── suffix 2
  ├── ...
  └── suffix n
        │
        ▼
   Binary Search
        │
        ▼
 Matching Positions
```

This gives the query stage logarithmic search over the suffix index rather than repeatedly performing a full linear scan.

---

# 🌳 Range Queries with Segment Tree

Searches can optionally be restricted to a genomic interval.

Example:

```text
ATGCGT 1000 5000
```

means:

> Search for `ATGCGT` only between positions `1000` and `5000`.

The segment tree provides the range-query structure used to restrict candidate positions before the final search stage.

```text
Genome
───────────────────────────────────────────────
0                1000                5000      N
                 │────────────────────│
                    Search Region
```

This allows the search engine to combine **string indexing with positional constraints**.

---

# ⚡ Custom Hash Cache

Repeated queries can avoid recomputing the same search.

The cache uses:

```text
Query
  │
  ▼
Rolling Hash
  │
  ▼
Custom Hash Map
  │
  ├── Hit  → Return cached result
  │
  └── Miss → Perform search
                    │
                    ▼
               Cache result
```

The hashing mechanism is based on a Rabin-Karp style rolling hash implemented specifically for the project.

---

# 💾 Memory-Mapped I/O

Large genomic files do not necessarily need to be copied entirely into RAM.

The project uses memory-mapped file access to allow the operating system to manage file-backed memory.

```text
DNA File
   │
   ▼
Memory Mapping
   │
   ├── Accessed region
   │       ↓
   │     RAM
   │
   └── Unaccessed region
           ↓
        Remains on disk
```

This makes the system better suited to experimenting with datasets larger than the available physical memory.

---

# 🧬 Reverse Complement Search

DNA is double-stranded, so a pattern can occur on either strand.

For every query, the system automatically computes the reverse complement.

```text
Forward Strand

5' ── A T G C C A ── 3'


Reverse Complement

3' ── T A C G G T ── 5'
```

The search therefore considers both orientations and reports the corresponding strand.

---

# 🖥️ Usage

## Build

### Linux / macOS

```bash
mkdir build
cd build
cmake ..
make
```

### Windows + MinGW

```bash
mkdir build
cd build
cmake .. -G "MinGW Makefiles"
mingw32-make
```

---

## Interactive Mode

```bash
./bin/dna_search ../data/sample.fasta
```

The program then accepts DNA patterns interactively.

---

## Single Query

```bash
./bin/dna_search ../data/sample.fasta ATGCGT
```

---

## Range-Restricted Search

```bash
./bin/dna_search ../data/sample.fasta ATGCGT 1000 5000
```

---

# 🧪 Testing

The project includes tests covering the major algorithmic components.

```bash
cd build
ctest --output-on-failure
```

### Test Coverage

```text
tests/
├── test_suffix_array.cpp
│   └── Suffix array construction + search
│
├── test_segment_tree.cpp
│   └── Range filtering
│
├── test_cache.cpp
│   └── Rolling hash + custom hash map
│
├── test_complement.cpp
│   └── Reverse complement logic
│
└── test_integration.cpp
    └── Full search pipeline
```

---

# 📂 Project Structure

```text
DNAsearch/
│
├── src/
│   ├── main.cpp
│   ├── suffix_array.cpp
│   ├── suffix_array.h
│   ├── segment_tree.cpp
│   ├── segment_tree.h
│   ├── hash_cache.cpp
│   ├── hash_cache.h
│   ├── mmap_reader.cpp
│   ├── mmap_reader.h
│   ├── input.cpp
│   ├── input.h
│   ├── output.cpp
│   └── output.h
│
├── tests/
│   ├── test_suffix_array.cpp
│   ├── test_segment_tree.cpp
│   ├── test_cache.cpp
│   ├── test_complement.cpp
│   └── test_integration.cpp
│
├── data/
│   └── sample.fasta
│
├── build/
│
├── CMakeLists.txt
└── README.md
```

The repository currently follows this source/test/data separation, with CMake as the build system.

---

# 🧪 Example Patterns

| Pattern     | Description                           |
| ----------- | ------------------------------------- |
| `ATG`       | Start codon                           |
| `TAA`       | Stop codon                            |
| `ATGCGT`    | Example gene fragment                 |
| `ACGT`      | Self-complementary sequence           |
| `ATG 0 120` | Search `ATG` within positions `0–120` |

---

# 🛠️ Technology Stack

| Technology             | Usage                  |
| ---------------------- | ---------------------- |
| **C++**                | Core implementation    |
| **CMake**              | Build system           |
| **Suffix Arrays**      | Primary sequence index |
| **Segment Trees**      | Range filtering        |
| **Rabin-Karp Hashing** | Rolling hash           |
| **Custom Hash Map**    | Query cache            |
| **Memory Mapping**     | Large-file access      |
| **GoogleTest / CTest** | Testing infrastructure |

---

# 🎯 Project Goals

DNA Search was built to explore how classical algorithms and low-level systems techniques can be combined into a practical search engine.

The project focuses on:

```text
String Algorithms
       +
Data Structures
       +
Caching
       +
Memory Management
       +
Systems Programming
       +
Genomic Data
```

Rather than using a library that hides the indexing and search process, the core structures are implemented explicitly to expose their behavior and trade-offs.

---

# 🚧 Future Improvements

Potential extensions include:

* Parallel suffix-array construction
* Multi-threaded query execution
* Compressed genomic indexes
* FM-index / Burrows-Wheeler Transform
* Persistent index files
* Larger reference genomes
* More advanced query operators
* Benchmarking against naive substring search
* Query-result visualization
* Expanded biological annotation support

---

# 📄 License

This project is licensed under the **MIT License**.

