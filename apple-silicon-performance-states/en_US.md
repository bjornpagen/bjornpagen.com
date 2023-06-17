The Apple m1/m2 family of chips have built-in mechanisms
to dynamically scale their battery usage. Doing this allows
Apple Silicon devices to have speed when you want it,
with the ability to *slow things down* to benefit battery usage.

Here are some of the built-in levers you can pull:

## Clock speed controller

This allows the kernel to speed up and slow down the CPUs.
Here's the important code from the Asahi Linux kernel to
set the cpu frequency, with annotations:

```c
static int apple_cluster_clk_set_rate(struct clk_hw *hw, unsigned long rate,
					 unsigned long parent_rate)
{
  // apple_cluster_clk is hardware specific, unlike clk_hw, a generic
  // implementation for clock rate logic across all platforms
	struct apple_cluster_clk *cluster = to_apple_cluster_clk(hw);

  // an opp is an "operating performance point", not an "operation"
  // a list of supported frequencies/voltages is already in the kernel
  // here we fetch the lowest opp <= the user supplied clock rate
	struct dev_pm_opp *opp;
	opp = dev_pm_opp_find_freq_floor(cluster->dev, &rate);

  // return early if there's an error
	if (IS_ERR(opp))
		return PTR_ERR(opp);

  // the hardware doesn't understand linux internal data structures
  // this translates the opp struct into an index understood by the
  // hardware, like 0, 1, 2, etc... 
	unsigned int level;
	level = dev_pm_opp_get_level(opp);

  // log work so far
	dev_dbg(cluster->dev, "set_rate: %ld -> %d\n", rate, level);

  // readq_poll_timeout reads a quad (64 bits) from memory, and keeps trying until timeout
  // func sig: readq_poll_timeout(addr, val, cond, delay_us, timeout_us)
  // addr = cluster->reg_base + APPLE_CLUSTER_PSTATE
  // val = reg
  // cond = !(reg & APPLE_CLUSTER_PSTATE_BUSY), ie cluster must not be busy, keep polling until it isn't
  // delay_us = 2 microseconds
  // timeout_us = APPLE_CLUSTER_SWITCH_TIMEOUT = 100 microseconds
  //
  // now we'll read the hardware register associated with this cpu cluster
  // keep polling until it's no longer busy, and read the value into reg
  // if it's still busy after 100us, return with an error
	u64 reg;
	if (readq_poll_timeout(cluster->reg_base + APPLE_CLUSTER_PSTATE, reg,
			       !(reg & APPLE_CLUSTER_PSTATE_BUSY), 2,
			       APPLE_CLUSTER_SWITCH_TIMEOUT)) {
		dev_err(cluster->dev, "timed out waiting for busy flag\n");
		return -EIO;
	}

  // do some bitwise logic to set the hardware register to desired level
	reg &= ~(APPLE_CLUSTER_PSTATE_DESIRED1 | APPLE_CLUSTER_PSTATE_DESIRED2);
	reg |= FIELD_PREP(APPLE_CLUSTER_PSTATE_DESIRED1, level);
	reg |= FIELD_PREP(APPLE_CLUSTER_PSTATE_DESIRED2, level);
	reg |= APPLE_CLUSTER_PSTATE_SET;

  // write the register using hardware relaxed memory model
	writeq_relaxed(reg, cluster->reg_base + APPLE_CLUSTER_PSTATE);

  // if the cluster has an associated power domain, modify it
  // to reflect the new operating performance point
	if (cluster->has_pd)
		dev_pm_genpd_set_performance_state(cluster->dev,
						   dev_pm_opp_get_required_pstate(opp, 0));

	return 0;
}
```
