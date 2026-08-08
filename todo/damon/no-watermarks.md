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

Maybe can able users write a new path named `schemes/<N>/trigger`, this is its tree:

```txt
./trigger/
|-- metric 
|-- high, mid, low
```
And here is the options of `metric`:

- `DAMOS_TRIG_WMARKS_AVAILABLE`

- `DAMOS_TRIG_WMARKS_NONE`

- `DAMOS_TRIG_WMAKRS_FREE`

- `DAMOS_TRIG_SOMEPSI_{CPU, MEM, IO}`

- `DAMOS_TRIG_FULLPSI_{CPU, MEM, IO}`

It only support a single metric in a same time, because if want to support multi metric will be VEEERY (WRYYY!!!) COMPLEX!!

Maybe should add a `pause/stop` parameter for each schemes, to let users use their complex scripts to control their schemes. But I don't think it is too necessary, since userspace should not frequently communicate with kernelspace (it will make many overhead!).

### Implementation

I KNOW HOW TO IMPLEMENT IT NOW!! It is very easy to implement.

```C
static unsigned long damos_wmark_wait_us(struct damos *scheme)
{
	unsigned long metric;

- if (damos_get_wmark_metric_value(scheme->wmarks.metric, &metric))
+ if (damos_get_metric_value(scheme->trig.metric, &metric))
		return 0;
```
```C
static int damos_get_metric_value(enum damos_trig_metric metric,
					unsigned long *metric_value)
{
  switch (metric) {
  case DAMOS_TRIG_WMARKS_AVAILABLE:
  case DAMOS_TRIG_WMARKS_FREE:
  case DAMOS_TRIG_WMARKS_NONE:
    return damos_get_wmarks_metric_value(metric, metric_value);
  case DAMOS_TRIG_SOMEPSI_CPU:
  case DAMOS_TRIG_SOMEPSI_MEM:
  case DAMOS_TRIG_SOMEPSI_IO:
  case DAMOS_TRIG_FULLPSI_CPU:
  case DAMOS_TRIG_FULLPSI_MEM:
  case DAMOS_TRIG_FULLPSI_IO:
    return damos_get_psi_metric_value(metric, metric_value);
  default:
    return -EINVAL;
  }
```

And then just change some "watermark(s)" words to "triggers", hehehe (>▽

### OT (Off-Topic)

Seems like this task is very hard, but I will try it first, to make DAMON better, so DAMON can make my laptop better also.

Small TODO: I'm using phone to writing this now, later use laptop to make this more beautiful.