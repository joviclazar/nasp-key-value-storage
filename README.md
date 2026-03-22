# Key Value Storage

This project is a implementation of key-value storage in Go. It provides a simple CLI for storing, retrieving and deleting key-value pairs.

## Features

- Memtable - In-memory data structure, has implementations of SkipList, HashMap and B-Tree.
- SSTable - Immutable on-disk data structure with compression and indexing.
- Write-Ahead Log (WAL) - A log file that records all changes to the data, ensuring durability and crash recovery.
- LRU Cache - An in-memory cache that stores recently accessed key-value pairs for faster retrieval.
- Block Manager - Manages the allocation and deallocation of disk blocks for storing data.
- Bloom Filter, Count-Min Sketch, HyperLogLog, SimHash - Probabilistic data structures that can be stored as regular key-value pairs in the storage system.
- CLI - A command-line interface for interacting with the key-value storage system.
- Configuration - A configuration file for setting up the storage system.

## Personal Contribution

- Was the first one to actually run the project as a whole and did most of the work of integrating the different components together
- Complete WAL
- Complete LRU cache
- Implemented storing probabilistic data structures as regular key-value pairs in the storage system
- Complete configuration management

## Usage Paths

### Write Path

1. When a user issues a PUT or DELETE request, the operation is first durably recorded in the Write-Ahead Log (WAL), also known as the Commit Log.
2. After the WAL acknowledges the write, the data is inserted into the Memtable, an in-memory data structure optimized for fast writes.
3. Once the Memtable reaches its configured size threshold, its contents are sorted by key and flushed to disk as a new SSTable.
4. Following the flush, the system evaluates whether compaction criteria are met and triggers compaction if necessary. It is important to note that compactions at one level may cascade and initiate compactions at subsequent levels.

### Read Path

1. When a user issues a GET request, the system first checks whether the record exists in the Memtable. If found, the value is immediately returned.
2. If the record is not present in the Memtable, the system then checks the Cache layer. If the data is found there, it is returned without further lookup.
3. If the record is not found in memory, the system proceeds to search through SSTables. For each SSTable, the Bloom Filter is consulted to determine whether the key might be present. If the Bloom Filter indicates absence, the SSTable is skipped; otherwise, the system continues searching within that table.
Note: The order in which SSTables are searched depends on the chosen compaction strategy. If all candidate SSTables at a given LSM-tree level are exhausted without a match, the search continues at the next level. This process repeats until the key is found or the final level is reached.
4. Within a candidate SSTable, the system first checks whether the key falls within the Summary range. If it does, the corresponding position in the Index structure is identified.
5. The Index is then used to locate the exact offset in the Data file, from which the record is read and returned to the user.

## Usage

1. Build the project using:
   ```
   go build main.go
   ```
2. Ensure that the configuration file `config.yaml` is properly set up according to your environment and requirements. If invalid values are provided, the system will default to predefined settings.
3. Run the CLI:
   ```
    ./main
    ```
4. When in CLI, you can enter '4' to see the help menu with available commands.

## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.