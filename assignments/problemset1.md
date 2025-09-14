# Problem Set 1
## Exercise 1

**1. Invariants.** What are two invariants of the state? (Hint: one is about aggregation/counts of items, and one relates requests and purchases). Say which one is more important and why; identify the action whose design is most affected by it, and say how it preserves it.

**Answer:** The first invariant is that for every item request, the sum of the count of the purchases linked to this item and the current request count must equal the initial total item request count. The second invariant is that every purchase must be linked to an item request in the same registry.
**The most important invariant is the second one because we must make sure that there are no untracked purchases, so every purchase must be linked to a request. We do not want any purchases outside of the item requests because then the gifts are unwanted.**

**2. Fixing an action.** Can you identify an action that potentially breaks this important invariant, and say how this might happen? How might this problem be fixed?

Answer: Removing an item after a purchase of that item will break this invariant because now there is a purchased item that is now untracked. We can fix this by only allowing items with no purchases to be removed.

**3. Inferring behavior.** The operational principle describes the typical scenario in which the registry is opened and eventually closed. But a concept specification often allows other scenarios. By reading the specs of the concept actions, say whether a registry can be opened and closed repeatedly. What is a reason to allow this?

Answer: Yes a registry can be opened and closed repeatedly as the requirements for open is that the registry exists and is not active and the requirements for close is that the registry exists and is active. This is so that the registry can be paused and reopened later as many times as needed.

**4. Registry deletion.** There is no action to delete a registry. Would this matter in practice?

Answer: This would not matter in practice because the host can just close the registry and then nobody will be able to purchase items.

**5. Queries.** What are two common queries likely to be executed against the concept state? (Hint: one is executed by a registry owner, and one by a giver of a gift.)

Answer: One common query likely to be executed by a registry owner is to show all of the current purchases along with the giver of the gifts. Another common query by the giver of the gift is to see all of the gifts that have not been purchased yet.

**6. Hiding purchases.** A common feature of gift registries is to allow the recipient to choose not to see purchases so that an element of surprise is retained. How would you augment the concept specification to support this?

Answer: I would add a hide_purchases flag to the registries set to false and add an action called toggle_purchase_hiding that requires an active registry and it flips the boolean of the hide_purchases flag.

**7. Generic types.** The User and Item types are specified as generic parameters. The Item type might be populated by SKU codes, for example. Explain why this is preferable to representing items with their names, descriptions, prices, etc.

Answer: Using SKU codes for the item type is more favorable so that the gift giver can find the exact item that needs to be purchased as opposed to guessing which item is the correct one.

## Exercise 2

**1.** Complete the definition of the concept state.

Answer: The concept state is a set of Users with a username String and a password String.

**2.** Write a requires/effects specification for each of the two actions. (Hints: The register action creates and returns a new user. The authenticate action is primarily a guard, and doesn’t mutate the state.)

Answer: The register action requires no User with the same username and the effect is creating a new User with the username of username and a password of password. This action returns this new user. Authenticate requires the username to exist in Users and the password associated to that user to be the same as the password entered. The effect is nothing.

**3.** What essential invariant must hold on the state? How is it preserved?

Answer: The essential invariant that must hold is that there cannot be two Users with the same username, this is preserved in the register action where it checks that the username attempting to be registered is unique.

**4.** One widely used extension of this concept requires that registration be confirmed by email. Extend the concept to include this functionality. (Hints: you should add (1) an extra result variable to the register action that returns a secret token that (via a sync) will be emailed to the user; (2) a new confirm action that takes a username and a secret token and completes the registration; (3) whatever additional state is needed to support this behavior.)

Answer: Here is the final concept:


**concept** PasswordAuthentication

**purpose** Limit access to known users

**principle** After a user registers with a username and a password, they can authenticate with that same username and password and be treated each time as the same user after email confirmation.

**state**
- A set of Users with:
    - username: String
    - password: String
- A set of PendingConfirmations with:
    - username: String
    - token: Token

**actions**
- `register(username: String, password: String): (user: User, token: Token)`
    - **requires:** No User with this username exists
    - **effects:**
        - Creates new User with username of username and password of password
        - Creates a new PendingConfirmation with username of username and a new token (token is unique in PendingConfirmations)
    - **returns:** The new User and token

- `confirm(username: String, token: Token)`
    - **requires:** User exists with this username AND PendingConfirmation exists with this username and token
    - **effects:** Deletes PendingConfirmation for this username and token

- `authenticate(username: String, password: String): (user: User)`
    - **requires:** User exists with this username AND password matches AND username is not in PendingConfirmation
    - **effects:** None (primarily a guard)

## Exercise 3

**PersonalAccessToken Concept**

**concept** PersonalAccessToken

**purpose** Grant users scopable and revokable access to programmatic resources that can expire.

**principle** A signed-in user can create multiple long random tokens. Each active token authenticates as that user for the API. Each token has a set of scopes that limit operations. A token may have an expiration time and can be revoked at any time. For organizations that use SAML single sign-on, the token must be explicitly authorized for that organization to reach its resources.

**state**
- A set of Tokens with:
    - user: User
    - note: String
    - token: String
    - scopes: Set(Scope)
    - active: Flag
    - expiration: Time
    - organizationsAccess: Set(Organization)
    - verifier: Verifier

**actions**
- `createToken(owner: User, scopes: Set(Scope), expiration: Time?): (token: Token, secret: String)`
    - **requires:** Owner exists AND requested scopes exist
    - **effects:**
        - Creates a token for the owner with a new secret
        - Stores only a verifier of the secret
        - Adds the scopes to the token
        - Sets the token to active
        - Optionally sets an expiration time if configured
        - Allows the user to copy the secret once only
    - **returns** token, secret

- `authorizeTokenForSSO(token: Token, organization: Organization)`
    - **requires:** Token is active AND owner is a member of the organization AND organization uses single sign-on
    - **effects:** Adds the organization to the set of organizations that the token is authorized to reach

- `revokeOrDeleteToken(token: Token)`
    - **requires:** Token exists AND token is active AND token is not expired
    - **effects:** Makes the token inactive so it can no longer be used

- `authenticateWithToken(secret: String): (user: User)`
    - **requires:** There exists a token whose stored verifier matches the presented secret AND token is active AND token has not expired
    - **effects:** None
    - **result:** The owner of the token is considered the authenticated user for CLI, API requests, and git over HTTP

**How PersonalAccessToken Differs from PasswordAuthentication**

Password authentication relies on a single secret for the entire account and grants the full set of user privileges after a successful login. Personal access tokens allow a user to create multiple secrets, each with its own expiration so a token can be created, rotated, and revoked independently of other tokens or the account password. Each token carries scopes that restrict what the holder can do. For organizations using single sign-on, a personal access token must be explicitly authorized to access protected resources while password authentication just needs an account. Classic personal access tokens may have no expiration by default but are automatically removed if unused for a year, which is different from most password policies which stay forever.

**How GitHub Documentation Could Be Improved**

The documentation should begin with a clear sentence defining a classic personal access token as a per-user, scoped, and revocable secret for command line and API access and not for web login. The fact that the first couple lines states that PATs are similar to passwords creates confusinos. It should state clearly that a classic token can reach every repository the user can access within its scopes, and that organizations using single sign-on require explicit token authorization. A short comparison table should be included to show when to use passwords for web sessions, personal access tokens for command line and API, SSH keys for git over SSH, and applications for long-lived automation.

## Exercise 4: Defining Familiar Concepts

---

### Concept 1: URL Shortener

**concept** URLShortener

**purpose** Provide short, easy-to-share links that redirect to longer URLs, with either user-defined or autogenerated suffixes.

**principle** A user submits a long URL and optionally a custom suffix. The service generates a short URL (with either the custom or an autogenerated suffix) and stores the mapping. When a short URL is accessed, the service redirects to the original long URL.

**state**
- A set of URLMappings with:
    - shortURL: String
    - shortSuffix: String
    - longURL: String
    - user: User

**actions**
- `createShortURLCustomSuffix(longURL: String, customSuffix: String, user: User): (shortURL: String)`
    - **requires:** customSuffix must not already be in URLMappings and longURL is a valid url
    - **effects:** Stores a new URLMapping with the longURL of longURL and shoftSuffix of customSuffix and user of user
    - **returns:** The shortURL

- `createShortURLNoSuffix(longURL: String, user: User): (shortURL: String)`
    - **requires:** longURL is a valid url
    - **effects:** Stores a new URLMapping with the longURL of longURL and shoftSuffix of a random suffix string that is not in URLMapping and user of user
    - **returns:** The shortURL

- `deleteShortURL(shortURL: String, user: User)`
    - **requires:** shortURL exists and user matches user
    - **effects:** Removes the URLMapping

**notes**
- Custom suffixes must be unique. Autogenerated suffixes are random strings. Deletion is restricted to the creator for security.

---

### Concept 2: Billable Hours Tracking

**concept** BillableHoursTracker

**purpose** Track and record work sessions for billing clients by the hour, ensuring accurate and auditable records.

**principle** An employee starts a session by selecting a project and describing the work. The session is ended with another action. If a session is left open, the system can auto-end it after the work day ends.

**state**
- A set of WorkSessions with:
    - employee: User
    - project: Project
    - description: String
    - startTime: Time
    - endTime: Time?
    - status: {active, ended}

**actions**
- `startSession(employee: User, project: Project, description: String, startTime: Time): (session: WorkSession)`
    - **requires:** No active session for employee
    - **effects:** Creates a new active WorkSession with startTime of startTime and description and project. Initializes end time as 5 pm. Sets status to active.

- `endSession(employee: User, endTime): (session: WorkSession)`
    - **requires:** Active session exists for employee
    - **effects:** Sets endTime to endTime and status to ended

- `autoEndSession(employee: User)`
    - **requires:** current time is later than 5 pm
    - **effects:** Sets endTime to 5 pm and status to ended

**notes**
- I set the end time to 5 pm because we are assuming the workday ends at 5 pm so if an employee forgets to end a session, it auto-ends at 5 pm.

---

### Concept 3: Conference Room Booking

### Concept 3: Conference Room Booking

**concept** ConferenceRoomBooking

**purpose**
Allow users to reserve a specific room for a specific time interval, preventing conflicts and respecting room capacity.

**principle**
A room is defined by its location. Users (owners) create reservations for a room by specifying a start time, end time, reservation name, and setup notes. Each reservation is linked to a specific room and time interval, and only one active reservation is allowed per slot. Reservations must not overlap in time for the same room. Reservation details and history are retained for transparency and review.

**state**
- A set of Rooms with:
    - location: String
- A set of Reservations with:
    - owners: Set(User)
    - slot: Slot
    - setupNotes: String
    - reservationName: String
    - room: Room
    - startTime: Time
    - endTIme: Time

**actions**

- `createRoom(location: String): Room`
    - **requires:** Location is not already used for another room
    - **effects:** Creates a new Room with the given location

- `reserveRoom(owners: Set(User), room: Room, startTime: Time, endTime: Time, reservationName: String, setupNotes: String): Reservation`
    - **requires:** Room exists; startTime < endTime; no overlapping reservation for this room and time; owners is non-empty
    - **effects:** Creates a Reservation with the given owners, room, startTime, endTime, reservationName, and setupNotes

- `cancelReservation(reservation: Reservation)`
    - **requires:** Reservation exists
    - **effects:** Deletes the reservation, freeing the slot

- `updateSetupNotes(reservation: Reservation, setupNotes: String)`
    - **requires:** Reservation exists
    - **effects:** Updates the setupNotes for the reservation

- `changeReservationTime(reservation: Reservation, newStartTime: Time, newEndTime: Time)`
    - **requires:** Reservation exists; newStartTime < newEndTime; no overlap with other reservations for the room
    - **effects:** Updates the startTime and endTime for the reservation

**notes**
- Limiting each slot to one active reservation means that for any given time interval in a room, only one group of owners can use the space.
