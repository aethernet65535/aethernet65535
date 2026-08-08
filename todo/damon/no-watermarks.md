### Why Watermarks Not Good Enough?

This is what watermarks able to do:
- Get FREE watermark metric, like kswapd.

And it may able to get AVAILABLE, CACHED, USED also.

But, what if the running's condition of scheme is complex? Like this:
1. AVAILABLE <= 20%
2. SOME MEM PSI >= 0.10

This is what DAMON Watermarks cannot do now.

### So, What To Do?

This is my experimental idea, 65535% not good on the real production.

```C
/* static int kdamond_wait_activation(struct damon_ctx *ctx) */
wait_time = damos_wmark_wait_us(s);
```

Just change this to a new function named `damos_may_activated()`, it will just return if this scheme should activated, and since always monitoring do not consume too much system resources, DAMON will no need to sleep in this mechanism.

### What `damos_may_activated()` Do?

Actually, I don't know (。﹏。*)

Maybe can able users write a new path named `schemes/<N>/conditions`, this is its tree:

```txt
./conditions/
|-- watermarks/
    |-- metric, high, mid, low
|-- psi/
    |-- some/, full/
        |-- *.mem, *.cpu, *.io
|-- meminfo/
    |-- inactive
    |-- active
```

To simplify it, scheme will only activated when ALL condition is true.

And I think should add a `pause` for each scheme also, to able users to use their own complex scripts in userspace.

Not sure, this task is too complex, but it should make DAMON better, so I will try first (๑•̀ㅂ•́)و✧