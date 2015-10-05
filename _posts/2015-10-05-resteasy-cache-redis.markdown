---
layout: post
title:  "Request caching with Resteasy and Redis"
categories: software
---

I just finished writing [a plugin][resteasy-cache-redis] to use [Redis][redis]
as a backing cache for [Resteasy served requests][restesy-cache-core].

___What's that mean?___

With this plugin, adding `@Cache` annotations to Resteasy resource classes or
methods is all that's needed to serve requests from a Redis cache instead of
performing some expensive operation on a per-request basis.

Responses are cached by URI and content type. Wildcard content types are
supported.

___That's all?___

Not really. Caveats:

- `@Cache` annotations must define a positive `maxAge` value. This value is used
as the time-to-live for each cache entry.
- The plugin wraps the [Jedis][jedis] client for Redis. You are responsible for
providing your own instance of a [JedisPool][jedis-pool]. _Protip_: Start with a
[JedisPoolConfig][jedis-pool-config] instead of GenericObjectPoolConfig.

___Why? Resteasy already uses Infinispan.___

I'm not terribly fond of Infinispan. It's hard to configure. The documentation
is poor. In large distributed environments I've had JGroups fall apart and
leave the cache in a state that required rebooting the cluster.

On the other hand, I love Redis. It's easy to install, its documentation is
great, and it comes with sensible defaults. Plus, its API and data structures
are brilliant.

___Details?___

See the project on [Github][resteasy-cache-redis] for instructions, details,
and to file any issues.

[resteasy-cache-redis]: https://github.com/samolsen/resteasy-cache-redis
[redis]: http://redis.io
[restesy-cache-core]: https://docs.jboss.org/resteasy/docs/3.0.10.Final/userguide/html/Cache_NoCache_CacheControl.html
[jedis]: https://github.com/xetorthio/jedis
[jedis-pool]: https://github.com/xetorthio/jedis/blob/master/src/main/java/redis/clients/jedis/JedisPool.java
[jedis-pool-config]: https://github.com/xetorthio/jedis/blob/master/src/main/java/redis/clients/jedis/JedisPoolConfig.java
