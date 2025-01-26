# Pageserver

Responsibility
- Respond to GetPage@LSN requests from the Compute Nodes
- Receive WAL from WAL safekeeper, and store it
  - reprocess it into a format allowing quick access to any page version
- Upload data to S3 to make it durable, download files from S3 as needed

Non-functional
- High availability?

Concepts
1. incoming WAL
2. layer files
3. page version GetPage@LSN
4. garbage collection

Illustration
- WAL received as stream
- stream captured by page server and stored in memory (reorder buffer)
- pager server flushes mem to disk when enough WAL is accumulated into a new L0 layer file, mem is released
- L1: if enough L0 -> merge them, sliced per key-space producing L1 (each file contains more narrow key range but larger LSN range).
- layers further copied to `cloud storage` for long-term archival
  - a layer can be removed from local disk or kept for fast access.
  - a layer is fetched from cloud storage to local disk if not found.

```
Cloud Storage                   Page Server                           Safekeeper
                        L1               L0             Memory            WAL

+----+               +----+----+
|AAAA|               |AAAA|AAAA|      +---+-----+         |
+----+               +----+----+      |   |     |         |AA
|BBBB|               |BBBB|BBBB|      |BB | AA  |         |BB
+----+----+          +----+----+      |C  | BB  |         |CC
|CCCC|CCCC|  <----   |CCCC|CCCC| <--- |D  | CC  |  <---   |DDD     <----   ADEBAABED
+----+----+          +----+----+      |   | DDD |         |E
|DDDD|DDDD|          |DDDD|DDDD|      |E  |     |         |
+----+----+          +----+----+      |   |     |
|EEEE|               |EEEE|EEEE|      +---+-----+
+----+               +----+----+
```

Refs
- [neon/docs/pageserver-storage](https://github.com/neondatabase/neon/blob/main/docs/pageserver-storage.md)