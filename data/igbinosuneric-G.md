## Description
The batchClaiming function attempts to claim fees for multiple tokens in a batch. Depending on the number of tokens, this function may exceed the gas limit due to the gas cost of iterating through the list.

## Recommendation
Consider optimising the batchClaiming function for scalability, especially when dealing with a large number of tokens. You may want to implement a pagination or batching mechanism to handle large lists.