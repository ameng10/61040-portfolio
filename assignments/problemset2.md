# Problem Set 2
## Concept Questions

**1. Contexts.** The NonceGeneration concept ensures that the short strings it generates will be unique and not result in conflicts. What are the contexts for, and what will a context end up being in the URL shortening app?

Answer: The context are for partitioning uniqueness so that a nonce is unique within a context but among multiple contexts, the nonce can be reused. In a URL shortening app, a context will be the url base such as .com or .tinyurl so that the url shorterner can reuse a nonce but not map the url to an existing url.

**2. Storing used strings.** Why must the NonceGeneration store sets of used strings? One simple way to implement the NonceGeneration is to maintain a counter for each context and increment it every time the generate action is called. In this case, how is the set of used strings in the specification related to the counter in the implementation? (In abstract data type lingo, this is asking you to describe an abstraction function.)

Answer: NonceGeneration must store sets of used strings so that when we call 'generate', we know which strings are already used so that we do not reuse a nonce within the same context. We need to be explicitly sure that we are not reusing a nonce so this is the only way to be 100% sure. The set of used strings is related to the counter as the counter can map its integer value to a string using a function f. So every used string in the set can be mapped to an integer and the current integer value of the counter will get incremented so that the next string will have a different letter. The integer would then be in base 26 for example so that if the counter was 1, then the used string would be 'a'.

**3. Words as nonces.** One option for nonce generation is to use common dictionary words (in the style of yellkey.com, for example) resulting in more easily remembered shortenings. What is one advantage and one disadvantage of this scheme, both from the perspective of the user? How would you modify the NonceGeneration concept to realize this idea?

Answer: One advantage for users is that it is easy to remember the url when needing to use it. Random strings are hard to remember so having real words would make the link easily sharable and accessible. One disadvantage is that it makes the url less private as it is easy to guess the url. I would modify NonceGeneration by adding a set of allowed dictionary words in each context state and in 'generate' action, it would generate a random word in the dictionary set and check if it is already in used strings.

## Synchronization Questions

**1. Partial matching.** In the first sync (called generate), the Request.shortenUrl action in the when clause includes the shortUrlBase argument but not the targetUrl argument. In the second sync (called register) both appear. Why is this?

Answer: In the first sync, only the short url base is included because nonce generation needs only the context in which uniqueness must hold. The target url is irrelevant to generating a unique suffix, so it is omitted. In the second sync, both the target url and the base are needed to invoke registration, so both are bound in the when clause.

**2. Omitting names.** The convention that allows names to be omitted when argument or result names are the same as their variable names is convenient and allows for a more succinct specification. Why isn’t this convention used in every case?

Answer: This isn't used in every case when you want to rename the local variable into something that isn't the argument variable, or if you wanted to pass a constant for example, or if two actions result in the same variable name so you need to differentiate the arguments.

**3. Inclusion of request.** Why is the request action included in the first two syncs but not the third one?

Answer: The first two syncs are driven by a user’s shorten url request, so they include the request completion to bind what they need and to order the steps. The third sync reacts to any successful registration, regardless of who requested it, so it triggers on the registration completion itself and does not need the original request action.

**4. Fixed domain.** Suppose the application did not support alternative domain names, and always used a fixed one such as “bit.ly.” How would you change the synchronizations to implement this?

Answer: I would remove the shortUrlBase argument from the actions so that the syncs would look like below:
```markdown
sync generate
when Request.shortenUrl()
then NonceGeneration.generate(context: "bit.ly")

sync register
when
  Request.shortenUrl(targetUrl)
  NonceGeneration.generate(): (nonce)
then UrlShortening.register(shortUrlSuffix: nonce, shortUrlBase: "bit.ly", targetUrl)
```
- The `setExpiry` sync remains unchanged.


**5. Adding a sync.** These synchronizations are not complete; in particular, they don’t do anything when a resource expires. Write a sync for this case, using appropriate actions from the ExpiringResource and URLShortening concepts.

**Answer:**

```markdown
sync expireAndDelete
when ExpiringResource.expireResource(): (resource)
then UrlShortening.delete(shortUrl: resource)
```

---

## Extending the Design


**1. Design additional concepts for analytics (private, per-user counts):**

```markdown
concept: OwnershipAndView
purpose: enforce private analytics by recording the owner of each short URL and authorizing only the owner to view analytics
principle: whenever a short URL is created, its owner is recorded; only the owner can view analytics for that URL
state
  a set of Ownerships with:
    shortUrl: String
    owner: User
actions
  recordOwnership(shortUrl: String, owner: User)
    requires: no ownership exists for this shortUrl
    effects: creates an ownership mapping from shortUrl to owner
  authorizeView(shortUrl: String, requester: User)
    requires: ownership exists for shortUrl AND requester == owner
    effects: none
  forgetOwnership(shortUrl: String)
    requires: ownership exists for shortUrl
    effects: deletes the ownership

```
```markdown

concept: AccessCounts
purpose: count how many times each short URL is used
principle: new short URLs start at zero; each successful lookup increments the count; count can be read when viewing is authorized
state
  a set of Counters with:
    shortUrl: String
    count: Number
actions
  initCounter(shortUrl: String)
    requires: no counter exists for shortUrl
    effects: creates a counter with count zero
  incrementCounter(shortUrl: String)
    requires: counter exists for shortUrl
    effects: increases count by one
  getCount(shortUrl: String)
    requires: counter exists for shortUrl
    effects: none
    returns: the current count
```

---


**2.** Specify three essential synchronizations with your new concepts: one that happens when shortenings are created; one when shortenings are translated to targets; and one when a user examines analytics.

Answer:

```markdown
sync recordOwnership
when Request.shortenUrl(shortUrlBase)
     UrlShortening.register()
then OwnershipAndView.recordOwnership(shortUrl, owner: User)
     AccessCounts.initCounter(shortUrl)

sync incrementCounter
when UrlShortening.lookup(shortUrl)
then AccessCounts.incrementCounter(shortUrl)

sync getCount
when Request.viewAnalytics(shortUrl, requester)
     OwnershipAndView.authorizeView(shortUrl, requester: User)
then AccessCounts.getCount(shortUrl)
```
**3.** As a way to assess the modularity of your solution, consider each of the following feature requests, to be included along with analytics. For each one, outline how it might be realized (eg, by changing or adding a concept or a sync), or argue that the feature would be undesirable and should not be included:
- Allowing users to choose their own short URLs;
- Using the “word as nonce” strategy to generate more memorable short URLs;
- Including the target URL in analytics, so that lookups of different short URLs can be grouped together when they refer to the same target URL;
- Generate short URLs that are not easily guessed;
- Supporting reporting of analytics to creators of short URLs who have not registered as user.

Answer:
- Allowing users to choose their own short URLs;
    - Add a new request path for the desired suffix. For example: `Request.shortenUrlWithSuffix(owner: User, shortUrlBase: String, desiredSuffix: String, targetUrl: String)`

    Then sync like the following:
    ```markdown
    sync registerCustom
    when Request.shortenUrlWithSuffix(owner, shortUrlBase, desiredSuffix, targetUrl)
    then UrlShortening.register(shortUrlSuffix: desiredSuffix, shortUrlBase, targetUrl)
    ```
- Using the “word as nonce” strategy to generate more memorable short URLs;
    - Adjust the NonceGeneration concept like previously by adding a set of allowed dictionary words in each context state and in 'generate' action, it would generate a random word in the dictionary set and check if it is already in used strings. Nothing else would change.

- Including the target URL in analytics, so that lookups of different short URLs can be grouped together when they refer to the same target URL;
    - We can adjust the AccessCounts to include a set of Counters with longUrls instead of shortUrls and then the initCounter, incremenetCounter, and getCount would apply to the longUrls as well as the shortUrls.
- Generate short URLs that are not easily guessed;
    - To generate short URLs that are not easily guessed, we can change generate action in NonceGeneration to choose a random 10 character nonce out of the alphabet and check that this nonce is not in the used set.
- Supporting reporting of analytics to creators of short URLs who have not registered as user.
    - This feature is undesirable because we want the analytics to be secure and we want every owner that creates a short url to be a registered user.
