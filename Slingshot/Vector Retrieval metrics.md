During Ingestion:

| Mode                                                   | Fields indexed                  | Index + metric                                            |
| ------------------------------------------------------ | ------------------------------- | --------------------------------------------------------- |
| Dense only (default: `enable_bm25_ingestion=false`)    | `vector` only                   | `HNSW` + `L2`                                             |
| Hybrid (`enable_bm25_ingestion=true`)                  | `vector` + `sparse`             | `HNSW`+`L2` and `SPARSE_INVERTED_INDEX`+`BM25`            |
| Metadata sparse (`enable_metadata_sparse_vector=true`) | `vector` + `sparse` + `sparse2` | Same as above, plus BM25 on `metadata_subset` → `sparse2` |
|                                                        |                                 |                                                           |
Durin Retrieval:

| Retrieval mode                                            | What runs                                  | Metric                                                                              |
| --------------------------------------------------------- | ------------------------------------------ | ----------------------------------------------------------------------------------- |
| Semantic only (`enable_bm25=false` on source)             | ANN on `vector`                            | `L2` via HNSW (`search_type=similarity`)                                            |
| Hybrid (`enable_bm25=true`)                               | Dense ANN on `vector` + sparse on `sparse` | `L2` + `BM25`, fused with `WeightedRanker` (weights `0.3` semantic / `0.7` keyword) |
| Metadata BM25 only (`only_bm25=true`, metadata filtering) | Sparse search on `sparse2`                 | `BM25` only — no embedding query                                                    |


Context-processing collections

| Field                         | Index                             | Metric | When                                       |
| ----------------------------- | --------------------------------- | ------ | ------------------------------------------ |
| `vector`, `vector_ans`        | `HNSW` (M=16, efConstruction=300) | `L2`   | Always                                     |
| `sparse`                      | `SPARSE_INVERTED_INDEX`           | `BM25` | When `enable_bm25_context_processing=true` |
| `session_id`, `user_id`, etc. | `TRIE`                            | `NONE` | Scalar filter speed                        |
| `is_deleted`                  | `BITMAP`                          | `NONE` | Boolean filter                             |

![[Pasted image 20260708004536.png]]
