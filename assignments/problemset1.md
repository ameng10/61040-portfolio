# Problem Set 1
## Exercise 1

**1. Invariants.** What are two invariants of the state? (Hint: one is about aggregation/counts of items, and one relates requests and purchases). Say which one is more important and why; identify the action whose design is most affected by it, and say how it preserves it.

**Answer:** The first invariant is that for every item request, the sum of the count of the purchases linked to this item and the current request count must equal the initial total item request count. The second invariant is that every purchase must be linked to an item request in the same registry. The most important invariant is the first one because we must make sure that the state is always keeping track of the correct count and makes sure nobody buys too many of an item. The purchase action is most affected by it and it preserves it by decreasing the requests count by the correct amount by increasing the purchase count to balance.

**2. Fixing an action.** Can you identify an action that potentially breaks this important invariant, and say how this might happen? How might this problem be fixed?

Answer: Adding an existing item after a purchase of that item may not update the count correctly, as the count may not update. To account for this, we need to make addItem add the count to the request object directly so that the item request is always updated.

**3. Inferring behavior.** The operational principle describes the typical scenario in which the registry is opened and eventually closed. But a concept specification often allows other scenarios. By reading the specs of the concept actions, say whether a registry can be opened and closed repeatedly. What is a reason to allow this?

Answer: Yes a registry can be opened and closed repeatedly as the requirements for open is that the registry exists and is not active and the requirements for close is that the registry exists and is active. This is so that the registry can be paused and reopened later as many times as needed.

**4. Registry deletion.** There is no action to delete a registry. Would this matter in practice?

Answer:

**5. Queries.** What are two common queries likely to be executed against the concept state? (Hint: one is executed by a registry owner, and one by a giver of a gift.)

Answer:

**6. Hiding purchases.** A common feature of gift registries is to allow the recipient to choose not to see purchases so that an element of surprise is retained. How would you augment the concept specification to support this?

Answer:

**7. Generic types.** The User and Item types are specified as generic parameters. The Item type might be populated by SKU codes, for example. Explain why this is preferable to representing items with their names, descriptions, prices, etc.

Answer:
