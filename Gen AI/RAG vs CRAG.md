| Feature                  | Retrieval-Augmented Generation (RAG)              | Corrective RAG (CRAG)                                              |
| ------------------------ | ------------------------------------------------- | ------------------------------------------------------------------ |
| Main idea                | Retrieve documents and generate an answer         | Retrieve documents, evaluate them, and correct retrieval if needed |
| Retrieval quality check  | Usually none                                      | Includes a verification/correction step                            |
| Handling bad documents   | Model may still use irrelevant context            | System detects weak retrieval and retries or supplements           |
| Accuracy                 | Good, but depends heavily on retrieval quality    | Generally higher because of corrective filtering                   |
| Hallucination reduction  | Partial                                           | Better reduction of hallucinations                                 |
| Complexity               | Simpler architecture                              | More complex pipeline                                              |
| Speed                    | Faster                                            | Slightly slower due to evaluation steps                            |
| Cost                     | Lower compute cost                                | Higher compute cost                                                |
| Reliability              | Can fail if retrieval is poor                     | More robust against noisy/incomplete data                          |
| Typical workflow         | Retrieve → Generate                               | Retrieve → Evaluate → Correct → Generate                           |
| Best use cases           | Simple knowledge assistants, internal docs search | Enterprise QA, research systems, high-accuracy AI assistants       |
| Dependency on retriever  | Very high                                         | Reduced because correction can recover from bad retrieval          |
| External search fallback | Rare                                              | Commonly included                                                  |
| Example problem          | Wrong document leads to wrong answer              | System notices low-quality docs and retries                        |
