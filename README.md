CRDT-works-on-POC
============

WRITE:

When a client inserts a key-value (save key 1 => a) to the distributed cache, make asynchronous, parallel HTTP PUT requests to all three nodes. The end state will make all nodes received and saved ‘a’ to key ‘1’. If the WRITE request considers success if you have received successful responses (completed() method in unirest Java callback) from at least two servers. Otherwise, it should return as failed to the client and then make necessary HTTP DELETE calls to clean up partial state. To support rollback, you need to add a DELETE operation to the /{key} resource. You may need to update this client code to make HTTP calls asynchronous and to support the DELETE method.

Once you have implemented the DELETE method in both client and server side, you put the following key-value pair into the three cache servers:

http://localhost:300{0|1|2}/cache/ 

{Key => Value}

    1 => a

Assuming all three inserts (PUT calls) went through, all three cache servers should have the key “1” and its value.

READ on Repair: 

When you can reading any key from the cache cluster, you not only read the value, but alos need to repair if you find any inconsistency. Similar to the insert/update part, you need to send asynchronous, parallel HTTP GET read requests to all cache servers. Suppose, if majority of servers (2 out of 3) return the same value, take the majority response as the correct value and then update any incorrect server by making additional HTTP PUT call with the correct value. This PUT call can be done either synchronously or asynchronously as long as you receive a successful response from the update call.

To create inconsistent state, bring down the server A and update the existing key with new value:

http://localhost:300{0|1|2}/cache/ 

{Key => Value}

    1 => b

Finally, make a READ (HTTP GET /1) call to key “1” and see what the value responded. Then, check each server to make sure all have the latest value “b” which means your read-on-repair worked.

Your new Client.java should have these three calls only and the rest of implementation details should hide in a different class (E.g. CRDTClient.java):

    First HTTP PUT call to store “a” to key 1. (Then, sleep for ~30 seconds so that you will have enough time to stop the server A)
    Second HTTP PUT call to update key 1 value to “b”. (Then, sleep again for another ~30 seconds while bringing the server A back)
    Final HTTP GET call to retrieve key “1” value.  

Put System.out.println(“xxxxx step x”) in each step so that you will know when to stop and start the server A.
