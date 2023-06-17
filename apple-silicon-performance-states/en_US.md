The Apple M1/M2 family of chips have built-in mechanisms
to dynamically scale their battery usage. Doing this allows
Apple Silicon devices to have speed when you want it,
with the ability to *slow things down* to benefit battery usage.

Here are some of the built-in levers you can pull:

## Clock speed controller

The M2 chipset, my current laptop, has two clusters: a cluster of 4 efficiency cores, and
a cluster of 4 performance cores. You can modify the frequency of both of these
core clusters anytime!

This allows the kernel to speed up and slow down the CPUs.
You can change the cpu frequency from userpsace using tools
like cpupower on Linux.

I decided to dive into the code and exlore how it's implemented!
Here's the low-level [code](https://github.com/AsahiLinux/linux/blob/95c5e22cd2e75b624d78bb3504c2e489b0dd86c6/drivers/clk/clk-apple-cluster.c) from the Asahi Linux kernel to that
directly sets the cpu frequency, with annotations. This code is called
by higher level modules in the Linux kernel, like `cpufreq`, which I'll
go over later.

This function sets the clock rate of a cluster of `cpus`, which are
more commonly known as **cores** outside of kernel-land.

```c
static int apple_cluster_clk_set_rate(struct clk_hw *hw, unsigned long rate,
					 unsigned long parent_rate)
{
```

`apple_cluster_clk` is hardware specific, unlike `clk_hw`, a generic
implementation for clock rate logic across all platforms.

```c
	struct apple_cluster_clk *cluster = to_apple_cluster_clk(hw);
```

An `opp` is an "operating performance point", not an "operation".
A list of supported frequencies/voltages is already in the kernel.
Here we fetch the lowest opp <= the user supplied clock rate.

```c
	struct dev_pm_opp *opp;
	opp = dev_pm_opp_find_freq_floor(cluster->dev, &rate);
```

Return early if there's an error.

```c
	if (IS_ERR(opp))
		return PTR_ERR(opp);
````

The hardware doesn't understand linux internal data structures
this translates the opp struct into an index understood by the
hardware, like 0, 1, 2, etc... 

```c
	unsigned int level;
	level = dev_pm_opp_get_level(opp);
```

Log work so far.

```c
	dev_dbg(cluster->dev, "set_rate: %ld -> %d\n", rate, level);
```

`readq_poll_timeout` reads a quad (64 bits) from memory, and keeps trying until timeout.
The table below demonstrates all the arguments of the macro.

| arg | parameter | value |
|-----|-----------|-------|
| 1 | `addr` | `cluster->reg_base + APPLE_CLUSTER_PSTATE` |
| 2 | `val` | `reg` |
| 3 | `cond` | `!(reg & APPLE_CLUSTER_PSTATE_BUSY)` |
| 4 | `delay_us` | 2us |
| 5 | `timeout_us` | `APPLE_CLUSTER_SWITCH_TIMEOUT` = 100us |

Now we read the hardware register associated with this cpu cluster and
keep polling until it's no longer busy. Then, we read the value into `reg`
and if it's still busy after 100us, return with an error.

```c
	u64 reg;
	if (readq_poll_timeout(cluster->reg_base + APPLE_CLUSTER_PSTATE, reg,
				 !(reg & APPLE_CLUSTER_PSTATE_BUSY), 2,
				 APPLE_CLUSTER_SWITCH_TIMEOUT)) {
		dev_err(cluster->dev, "timed out waiting for busy flag\n");
		return -EIO;
	}
```

Do some bitwise logic to set the hardware register to desired level.

```c
	reg &= ~(APPLE_CLUSTER_PSTATE_DESIRED1 | APPLE_CLUSTER_PSTATE_DESIRED2);
	reg |= FIELD_PREP(APPLE_CLUSTER_PSTATE_DESIRED1, level);
	reg |= FIELD_PREP(APPLE_CLUSTER_PSTATE_DESIRED2, level);
	reg |= APPLE_CLUSTER_PSTATE_SET;
```

Write the register using hardware relaxed memory
model, which default on modern devices and arm64 Apple Silicon.

```c
	writeq_relaxed(reg, cluster->reg_base + APPLE_CLUSTER_PSTATE);
```

If the cluster has an associated power domain, modify it
to reflect the new operating performance point.

```c
	if (cluster->has_pd)
		dev_pm_genpd_set_performance_state(cluster->dev,
							 dev_pm_opp_get_required_pstate(opp, 0));

	return 0;
}
```
