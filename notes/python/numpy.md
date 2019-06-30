# Numpy Notes

## General
* When using logical indexing, different masks can by combined using bitwise & and | operators:
    ```python
    random = np.random.rand(5,5)
    mask1 = random < 0.2
    mask2 = random > 0.8
    binary = np.zeros((5,5))
    binary[mask1 | mask2] = 1
    ```
`and` and `or` will fail because that attempts to evaluate the entire mask array as a single boolean value.  This is ambiguous since it is unclear whether an array should evaluate to `True` if any or if all elements are nonzero.  Arrays can be evaluated as booleans in this manner using the `any()` or `all()` functions.  [source](https://stackoverflow.com/questions/10062954/valueerror-the-truth-value-of-an-array-with-more-than-one-element-is-ambiguous)

# Select and Choose, Boolean Indexing
