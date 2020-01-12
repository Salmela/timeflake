# timeflake
Timeflakes are 64-bit (unsigned), roughly-ordered, URL-safe UUIDs. Inspired by Twitter's Snowflake and Instagram's UUID.

It supports both incremental counters per shard ID, and cryptographically strong pseudo-random numbers for the UUID generation.

The IDs store the following in binary form (in this order):
- Time since custom epoch in seconds (32 bits).
- Logical shard ID (10 bits).
- Counter (22 bits).

```python
timeflake = Timeflake()
timeflake.random()
>>> 'eihdZ7Hqa'
# decoded=4091004504637904 timestamp=1578785711 shard_id=217 counter=393680
```

The resulting string `efqCcXufN` contains the following information:
```
decoded = 4085399177663909
timestamp = 1578784406
shard_id = 123
counter = 5541
```

Some nice properties of having an auto-incrementing sequence as the most significant part of the resulting ID are:
- Reduced performance impact when using clustered indices on relational databases (vs random UUIDs).
- The IDs are (roughly) sortable, so you can tell if one ID was created a few seconds before or after another.

The `.random()` method returns a new UUID using cryptographically strong pseudo-random numbers for the counter segment.

When using the random counter, the probability of a collision per logical shard per second is 2^22 (about 1 in 4 million). If you do not specify the `shard_id`, a random one will be selected for you.

You can also use the `.next()` method to use an auto-incrementing sequence for the counter segment (don't forget to set a shard_id per thread/process/instance):

```python
timeflake = Timeflake(shard_id=7)
timeflake.next()
>>> 'eicbZeGxe'
# decoded=4090831824755682 timestamp=1578785671 shard_id=7 counter=7138
```


By default they are encoded in base57 for (nice) URL-safe representation. This means they are concise (max length of 11 characters).

If you prefer to work with the unsigned 64-bit integer form, simply pass `encoded=False` to the instance:

```python
timeflake = Timeflake(encoded=False)
timeflake.random()
>>> 4085399177663909
```

The default alphabet for base57 encoding is: `23456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz`. It intentionally removes visually similar characters (like 1 and l) while keeping the resulting ID short with a max of 11 characters.

When using the default epoch (2020-01-01), the IDs will run out at around 2088-01-19.

# Supported versions
I'll be adding tests for various python versions. But I only intend to support Python 3.7+ at this moment.

# Dependencies
No dependencies other than the standard library.