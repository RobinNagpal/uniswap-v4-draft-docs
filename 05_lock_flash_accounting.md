# Intro to Locking
The locking mechanism in v4 ensures that certain operations are executed atomically without interference, ensuring 
consistency and correctness in the PoolManager's state. PoolManager, uses `LockDataLibrary` to manager a queue of 
lockers, allowing nested locks, and ensures that all currency deltas are settled before releasing a lock.

Pool actions can be taken by acquiring a lock on the contract and implementing the `lockAcquired` callback to 
then proceed with any of the following actions on the pools:

- `swap`
- `modifyPosition`
- `donate`
- `take`
- `settle`
- `mint`

# Simple Locking Analogy

Imagine a public library where people can come in and borrow books. Now, let's say a librarian wants to update the
records for a specific book (e.g., mark it as borrowed). To do so without interruptions, the librarian puts up a
sign that says, "Please wait, updating records." This sign prevents other librarians or staff from making changes
to the same record at the same time. Once the updating is done, the sign is removed, and others can now access and
modify the record.

In this analogy:
- The book record is like the state of the PoolManager.
- The sign the librarian puts up is equivalent to acquiring a lock.
- Other staff waiting or not being able to update the record represents the protection given by the lock.


# Main Components
In `PoolManager` "locking" is essentially a way to ensure certain operations are coordinated and don't interfere with each other
Here are the main components of the locking mechanism:

### 1. **Locking and Unlocking:**

- The `lock` function is where the locking mechanism is initiated. It pushes the `msg.sender` (the caller of the 
  function) to the `lockData` which acts as a queue.

    ```solidity
    function lock(bytes calldata data) external override returns (bytes memory result) {
        lockData.push(msg.sender);

        result = ILockCallback(msg.sender).lockAcquired(data);

        if (lockData.length == 1) {
            if (lockData.nonzeroDeltaCount != 0) revert CurrencyNotSettled();
            delete lockData;
        } else {
            lockData.pop();
        }
    }
    ```

- During the lock, a callback function `ILockCallback(msg.sender).lockAcquired(data)` is called, where the locked 
  contract can perform necessary operations.

- After the operations in the callback are completed, it either deletes the `lockData` if it is the only element, 
  signifying the release of the lock, or it pops the last element from `lockData`, signifying that the lock is 
  released by that particular address.

### 2. **Queue Management:**

- The queue management is handled by the `LockDataLibrary`. This library provides functionality to push, pop, and 
  get active locks from the `lockData`.

- The queue is managed in a way that it allows you to keep a track of locker addresses and ensure that only the 
  active locker can perform certain operations.

### 3. **Non-Zero Deltas Tracking:**

- `nonzeroDeltaCount` is a variable in `LockData` structure that keeps track of non-zero deltas across all lockers. A 
   delta here appears to represent a kind of balance or a state that changes during operations.

- In the `_accountDelta` function, if a delta changes from or to zero, the `nonzeroDeltaCount` is updated accordingly. 
  This is crucial for tracking the net changes made by each locker and ensuring that everything nets to zero at the end of operations.

### 4. **Restricted Access:**

- The modifier `onlyByLocker` is used to restrict access to certain functions. It ensures that a function can only be 
  called by the address that currently holds the lock.
    ```solidity
    modifier onlyByLocker() {
        address locker = lockData.getActiveLock();
        if (msg.sender != locker) revert LockedBy(locker);
        _;
    }
    ```
- Several functions in the contract, such as `modifyPosition`, `swap`, `donate`, `take`, `mint`, and `settle`, use 
  the `onlyByLocker` modifier. This ensures that only the entity that has currently locked the contract can modify 
  positions, swap tokens, etc. This helps ensure operations are atomic and avoid potential race conditions.

# Locking Diagram

![Locking Diagram](images/05_locking_mechanism/LockingMechanism_excali.png)

* A locker (caller) initiates a lock function execution in the PoolManager contract.
* The locker gets added to the lock queue by the LockDataLibrary.
* An ILockCallback is used where the locker acquires the lock.
* Balances and currency deltas are managed and updated within the Balance and CurrencyDelta.
* The locker can account for pool balance deltas which handle the currency and its corresponding deltas.

### Working
The locking mechanism in the PoolManager contract works as follows:

1. When a user wants to lock, they call the `lock()` function with the data that they want to be passed to the callback.
2. The `lock()` function pushes the user's address onto the locker queue.
3. The `lock()` function then calls the `ILockCallback(msg.sender).lockAcquired(data)` callback.
4. The callback can do whatever it needs to do, such as updating the user's balances or interacting with other contracts.
5. Once the callback is finished, it returns to the `lock()` function.
6. The `lock()` function checks if there are any other lockers in the queue. If there are, it pops the next locker off the queue and calls the callback for that locker.
7. If there are no more lockers in the queue, the `lock()` function returns.

# Lock Data Structure

- **Struct:**
  The `LockData` struct is used to store information related to locks. It has two fields:
   - `length`: Represents the current number of active locks.
   - `nonzeroDeltaCount`: Represents the total number of non-zero deltas over all active and completed locks.

- **Library (`LockDataLibrary`):**
  The `LockDataLibrary` is a library used for managing the custom storage implementation of the queue that tracks current lockers.

