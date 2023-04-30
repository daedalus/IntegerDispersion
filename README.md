# IntegerDispersion
Bare metal mining for FACT0RN. Meaning, no Docker containers.

## How to use

First, we need to create primorials containing all primes of a fixed bitsize. So all primes of bitsize 10, for example, would be the product of all primes between 512 and 1024....so we would call that the primorial at level 10, denoted P#_10. 

For simplicity, we say P#_1 = 2*3 = 6  is the Level 1 primorial and start from there....so the above example would be primorial level 9 or P#_9. In my testing, the optimal number of levels to have to sieve candidates for mining is somewhere between 23 - 28. After 28 the degradation is significant, ecm sieves faster; depending on many parameters the optimal point changes at lower levels; 23 is the clearly-this-is-better level and 28 is the clearly-this-is-not-optimal.

To get the files containing these levels do the following:


```
cd work/yafu0/yafu/isieve
./sieverb 28
```

The siever can be built from soruce, just run the ``build.sh`` file in the isieve folder. Python is extremely slow generating this leveled primorials so generating them using C++ and loading them into memory is significantly faster.

If you are not running Ubuntu 20.04 on x86 you will need to build yafu, ecm from source and place them in the right directories. I strongly recommend to compile for your architecture with ``-march=native`` or the equivalent for clang when you build; I also strongly recommend to build GMP from source and optimize it, this process may take half an hour, and requires that you build GMP, run a benchmark that runs long, replace a header file, and rebuild again, but it is worth it.

To run the miner, do:

```
cd work/yafu0/yafu
python FACTOR.py <number of cores to use> <core offset> <scriptPubKey>
```

It is best to turn off Hyper-Threadind(Intel)/SMT(AMD) and dedicate a full physical core per thread. The terminology above assumes one thread per core, that is, HT/SMT is turned off. Because FACTOR.py uses taskset to pin threads to cores, the \<core offset\> parameters helps you assign the next miner to  different cores than the previous one. Note that to turn off HT/SMT it is not necessary to go to the BIOS, you can do it from the terminal...though your computer might freeze or slow down for several seconds while it does this. Use ``echo off | sudo tee /sys/devices/system/cpu/smt/control`` or the equivalent if you running on Intel CPUs.



To further increase performance, you may want to run the ``renice_mining.py`` python script as this will further reduce core migrations, context switches and give you even better time slices on the linux scheduler. The way to run it is to execute every second or two with the watch command:

```
watch -n 1 python renice_mining.py
```

You only need to have one instance of ``renice_mining.py`` regardless of how many miners you have running. It will do the right thing.

I would recommend at this point in time to use about 4-8 physical cores per miner. The way to do that is to create a copy of ``yafu0`` into ``yafu1``, ``yafu2``, etc for as many miners as you are going to have running. You will need one separate copy for each. Note, the log files generated by mining get into the GB range over time...so you need to check them and delete them every so often.

So, if you are going to have 5 miners with 4 physical cores each, after turning HT/SMT off, then you would have something that looks like this:

```
cp -R work/yafu0 work/yafu1
cp -R work/yafu0 work/yafu2
cp -R work/yafu0 work/yafu3
```

and then on 4 terminals you would have running one instance from each folder in each one. So,

```
------------------------
Terminal 0
------------------------
cd /word/yafu0/yafu
python 4 0 <scriptPubKey>
------------------------

------------------------
Terminal 1
------------------------
cd /word/yafu1/yafu
python 4 4 <scriptPubKey>
------------------------

------------------------
Terminal 2
------------------------
cd /word/yafu2/yafu
python 4 8 <scriptPubKey>
------------------------

------------------------
Terminal 3
------------------------
cd /word/yafu3/yafu
python 4 12 <scriptPubKey>
------------------------

------------------------
Terminal 4
------------------------
watch -n 1 renice_python.py
------------------------

```

Remember to update lines 73 and 74 in ``FACTOR.py`` with the rpc credentials to your FACT0RN daemon.

There probably is a much better way of doing this and much better way to organize this entire process....feel free to contribute to make this better. I will merge improvements as they show up on the PR list.




