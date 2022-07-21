## Mistakes

1. Do not describe anything which you dont know 
2. Dynamo limit is around 400 KB, good for small files
3. Be realistic about latency requirements, trying to send 500 kb files across n/w cant be done in 100-150 ms. It will take more than that.
4. UUIDs are supposed to be unique across multiple systems.
5. Try to go with Base62 encoding, its safer to use. Base64 adds + and / which might not always work?


### Requirement Collection
1. Always ask about number of users/products ?
2. Ask about size of objects 
3. And latency requirements
