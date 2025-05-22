# Distributed Systems Implementations
This project contains implementations of several key concepts in distributed systems. It appears to be based on labs from a course like MIT's 6.824.

The major components include:
*   A MapReduce framework for distributed data processing.
*   An implementation of the Raft consensus algorithm.
*   A fault-tolerant key-value store built using Raft.
*   A sharded key-value store for improved scalability.

## Project Structure

Here's a breakdown of the main directories within the project:

*   `mr/`: Contains the implementation of the MapReduce framework, including the master, worker, and RPC logic for their communication.
*   `raft/`: Houses the implementation of the Raft consensus algorithm, a protocol for managing a replicated log.
*   `kvraft/`: Implements a fault-tolerant key-value store. This store utilizes the Raft protocol to ensure consistency across replicas.
*   `shardmaster/`: Contains the code for a service that manages the configuration of a sharded key-value store, deciding which shards are served by which replica groups.
*   `shardkv/`: Implements a sharded key-value store, which distributes data across multiple replica groups (managed by Raft) to achieve scalability.
*   `labrpc/`: A library for remote procedure calls (RPC). It simulates a lossy network environment where messages can be delayed or lost, which is crucial for testing the robustness of distributed algorithms.
*   `labgob/`: A utility for encoding and decoding Go objects, likely used for serializing data for RPC or persistent storage.
*   `main/`: Contains various Go programs that serve as entry points for running different parts of the system (e.g., MapReduce master, workers) and includes test scripts and sample data.
*   `mrapps/`: Holds example MapReduce applications, such as word count (`wc.go`) or an indexer. These are compiled into shared object files (`.so`) to be dynamically loaded by the MapReduce worker.
*   `linearizability/`: Provides tools and tests for checking the linearizability of the distributed data stores, ensuring that they behave like a single, correct copy of the data despite replication.

## Running the MapReduce System

The MapReduce framework can be run using the programs in the `main/` directory and the example applications in `mrapps/`.

### 1. Start the Master

The master process coordinates the MapReduce tasks. To start the master, provide it with a list of input text files. The example text files (e.g., `pg-*.txt`) are located in the `main/` directory.

```bash
go run main/mrmaster.go main/pg*.txt
```

The master will output information about its status and wait for workers to connect. It typically requires a specified number of reduce tasks (often hardcoded or as a parameter in `mr.MakeMaster`).

### 2. Start Workers

Workers execute the map and reduce tasks assigned by the master. You need to provide a MapReduce application to the worker. Example applications (like word count, `wc.so`) are built from the source files in `mrapps/`.

First, ensure the MapReduce application is built (e.g., for `wc.go`):

```bash
go build -buildmode=plugin ../mrapps/wc.go
```

This command should be run from the `main/` directory, or adjust the path to `wc.go` accordingly. It will produce `wc.so`.

Then, start one or more workers:

```bash
go run main/mrworker.go wc.so
```

You can start multiple worker processes, and they will request tasks from the master.

### 3. Testing the MapReduce Implementation

A test script is provided in the `main/` directory to verify the MapReduce implementation:

```bash
cd main
./test-mr.sh
cd ..
```

This script usually runs a sequence of MapReduce jobs (like word count and indexer) with various configurations, including tests for fault tolerance.

## Running Tests for Other Components

The project includes tests for various components like Raft, the key-value store, and the sharded key-value store. These tests are typically located in `_test.go` files within their respective package directories.

To run these tests, navigate to the specific directory and use the `go test` command.

For example, to test the Raft implementation:

```bash
cd raft
go test
cd ..
```

Similarly, for the key-value store (kvraft):

```bash
cd kvraft
go test
cd ..
```

And for the sharded key-value store (shardkv) and shard master:

```bash
cd shardkv
go test
cd ..

cd shardmaster
go test
cd ..
```

The tests often involve creating multiple instances (peers) of the system and simulating network conditions to check for correctness, fault tolerance, and concurrency bugs. The `-race` flag can be added to `go test` (e.g., `go test -race`) to detect race conditions.

## Prerequisites

*   **Go:** The project is written in Go. You will need to have Go installed on your system to build and run the components and tests. You can find installation instructions on the [official Go website](https://golang.org/doc/install).
