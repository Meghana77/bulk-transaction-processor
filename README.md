# Bulk Transaction Ingestion Engine

A high-performance backend microservice designed to ingest, parse, and store **millions of financial records** from massive CSV files into MongoDB.

Built with **Node.js Streams** to ensure constant memory usage (O(1)) regardless of file size, implementing **Backpressure** and **Batch Processing** to handle scale efficiently.

---

## 🚀 The Challenge
In enterprise FinTech environments (like HighRadius or JPMC), systems often need to process "End of Day" reports containing millions of transaction records.
* **Naive Approach:** Reading a 1GB file into memory (`fs.readFile`) causes `heap out of memory` crashes.
* **Standard Approach:** Inserting rows one-by-one into the database kills network latency and database throughput.

## 🛠️ The Solution
This engine solves scalability issues using a **Stream-Based Pipeline**:
1.  **Streaming Read:** Reads the file chunk-by-chunk (Buffers) instead of loading it all.
2.  **Application-Level Backpressure:** Manually pauses the file read stream when the Database write queue is full, preventing memory leaks.
3.  **Database Batching:** Groups records into chunks of 1,000 before sending to MongoDB, optimizing IO operations.

### System Architecture
```mermaid
graph LR
    A[Huge CSV File] -->|Stream Pipe| B(Node.js Stream)
    B -->|Parse Row| C{Batcher Buffer}
    C -- Accumulate 1000 Rows --> D[Batch Full?]
    D -- Yes --> E[Pause Stream]
    E --> F[Bulk Insert to MongoDB]
    F --> G[Resume Stream]
