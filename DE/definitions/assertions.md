Assertions are checks you add to your code to verify that something is true and to fail immediately if it isn't. They're commonly used to catch bugs, validate assumptions, and ensure data quality. For example, in a Spark pipeline you might assert that no trips have a negative fare or that the cleaned dataset contains rows before writing it out. If the condition is true, the program continues normally; if it's false, Python raises an `AssertionError` and stops execution, making the problem obvious instead of allowing bad data to flow through the pipeline.

```python
assert df.count() > 0, "Dataset is empty after cleaning"
```
