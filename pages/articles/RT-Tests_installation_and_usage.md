# RT-Tests (cyclictest) installation and usage

<https://wiki.linuxfoundation.org/realtime/documentation/howto/tools/rt-tests>

RT-Tests is a test suite made for various real-time Linux features.

First you need to install the prerequisites and clone the project:

```
sudo apt-get install build-essential libnuma-dev

git clone git://git.kernel.org/pub/scm/utils/rt-tests/rt-tests.git
cd rt-tests
git checkout stable/v1.0
```

You can compile all tests or just the one you need, I will compile just `cyclictest`, which measures the latency of an interrupt.

```
make cyclictest
```

If you want a statically linked binary, to use on another device, add `-static` to the `LDFLAGS` in the Makefile. You should also `strip` it so the binary is smaller.

From the README, a basic `cyclictest` run is started with:

```
cyclictest --mlockall --smp --priority=80 --interval=200 --distance=0
```

 * --mlockall : prevents memory from being paged out
 * --smp : Symmetric Multiprocessing CPU
 * --priority=80 : the realtime priority
 * --interval=200 : execution period (distance between) of measuring threads (in us)
 * --distance=0 : difference of interval between threads (0 if all have the same interval)

When running, you can see the thread latencies displayed.


An alternative option is to output a histogram

```
sudo cyclictest -l10000000 -m -S -p90 -i200 -h800 -q > output.txt
```

 * -l10000000 : number of iterations (multiply this by the interval to get ~execution time)
 * -h800 : histogram up to 800us
 * -q : don't print interactive output

You can get the max latency with:

```
grep "Max Latencies" output | tr " " "\n" | sort -n | tail -1 | sed "s/^0*//"
```

And get a .csv output to process later with:

```
grep -v -e "^#" -e '^$' output | tr " " "," | tr "\t" ","
```

### Sources

 * <https://lemariva.com/blog/2018/02/raspberry-pi-rt-preempt-vs-standard-kernel-4-14-y>
 * <https://wiki.linuxfoundation.org/realtime/documentation/howto/tools/cyclictest/start>
 * <https://www.osadl.org/Create-a-latency-plot-from-cyclictest-hi.bash-script-for-latency-plot.0.html>