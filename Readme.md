
# ![DMUCS](images/dmucs-logo-small.gif) DMUCS – a Distributed Multi-User Compilation System

(for use with [distcc](https://distcc.github.io/) and and the parallel 
version of [luaMakeSystem](https://github.com/stephengaito/luaMakeSystem))

DMUCS is a system that allows a group of users to share a compilation farm. Each
compilation request from each user will be sent to the fastest 
available machine, every time.  The system has these fine qualities:

* Supports multiple users compiling simultaneously, and scales well to 
  handle the new loads.
* Supports multiple operating systems in the compilation farm.
* Uses all processors of a multi-processor compilation host.
* Makes best use of compilation hosts with widely differing CPU speeds.
* Guarantees that a compilation host will not be overloaded by compilations.
* Takes into account the load on a host caused by non-compilation tasks.
* Supports the dynamic addition and removal of hosts to the compilation farm.
* Works with distcc, which need not be altered in any way.

## ![DMUCS](images/dmucs-logo-small.gif) DMUCS consists of these (main) programs:

*   **dmucs**: the “host-server”.  This application reads a configuration 
    file indicating the number of CPUs and the “power” of each potential 
    host in the compilation farm.

    It then receives over the network:
      * load average information from each compilation host.
      * host requests from compile tasks that need remote hosts on which to run.
      * information requests from monitoring applications.
      * status requests from an administrator.

    dmucs maintains the database of hosts in the compilation farm, and 
    assigns hosts to compilation tasks, giving out the best host/cpu 
    available when the compilation task asks.

*   **gethost**: a compilation task uses gethost get a host/cpu from the 
    dmucs server.  In general, a makefile will perform a compilation this 
    way:

        gethost distcc gcc ...

    gethost contacts the server to get a host, which it puts into the 
    environment variable DISTCC_HOSTS.  gethost then calls the program 
    given to it.  After that program ends, gethost releases the assigned 
    host back to the dmucs server.

*   **loadavg**: the administrator of the compilation farm must start this 
    application on each compilation host.  `loadavg` sends the load average of 
    the compilation host to the dmucs server periodically.  The dmucs server 
    will “downgrade” a compilation host if the host’s load averages goes too 
    high.

*   **monitor**: the administrator (or anyone) may use this program to 
    monitor the busy-ness of the compilation farm.  It displays which 
    hosts/cpus are available in the compilation farm, which hosts/cpus have 
    compilation tasks assigned to them, which hosts have been made 
    administratively unavailable, and which hosts are “silent” – i.e., the 
    dmucs server has not received a load average message from the compilation 
    host for a while.

*   **addhost**/**remhost**: tells the dmucs server to add or remove a given 
    host from the dmucs' server's list of "available" hosts.

## ![DMUCS](images/dmucs-logo-small.gif) Installing DMUCS

1. Get the latest version of DMUCS (0.6.1) from 
   [here](https://github.com/stephengaito/dmucs).

2. dmucs installs using the standard `configure`, `make`, and `make install` 
   steps.

3. Run `configure`. You may wish to use the `-–prefix` argument to `configure`
   if you don’t want the executables, documentation, and configuration files
   to be installed under `/usr/local`.  Specifically, the `-–prefix`
   argument will tell dmucs where to go to look for the hosts-info file
   (`/usr/local/share/dmucs/hosts-info`, by default).

4. Run `make CPPFLAGS=-DSERVER_MACH_NAME=”<machine-name>”`, where 
   `<machine-name>` is where you have chosen to run the dmucs server.  NOTE: 
   this machine does not have to be a powerful machine – I’ve used a very 
   wimpy Sun Ultra 5 very successfully.

5. Run `make install`.

6. Make sure you have your compilers in place on each host.  They must all 
   be found in the same directory on every compilation host.  Also, make 
   sure you have in place the distcc executables (distcc and distccd, at 
   least).

7. Make sure the loadavg executable can be accessed and executed on each 
   host.

8. Make sure the dmucs executable can be accessed and executed on the 
   machine you have chosen as your server machine.

9.  Create a hosts-info file in the location
    `/usr/local/share/dmucs/hosts-info` (or, if you specified `–-prefix` when 
    `configure`ing, in `<prefix>/share/dmucs/hosts-info`).  Here is a sample 
    hosts-info file:

    ```
        # Format: machine number-of-cpus power-index
        linux-comp-1       4         10
        solaris-comp-1     2         5
        solaris-comp-2     2         5
        old-linux-comp-1   1         4
        old-solaris-comp-3 1         2
        169.144.80.25      1         2
    ```

    As you can see, the format is simple: each line is a host's name, then 
    the number of cpus on that machine, and then the "power index" of that 
    machine.  If you know that some machines are much more powerful than 
    others, then give them higher power indices.  The value must be >= 1. 
    If you don't want to bother with a hosts-info file, the system will 
    still work, but all machines will be treated equally, and the code will 
    assume each machine has only 1 cpu.  (More information on how to choose 
    power indices is given below.)

10. Go to your host server machine and install and start up the dmucs 
    executable.
 
11. Go to each compilation host machine and run `loadavg` and start up the 
   `distccd` daemons (if you are using `distccd`).  The output from the 
   dmucs executable should show each host being registered.  (Alternatively, 
   you can use the shell script enable-host found in the dmucs/scripts 
   directory.  This will start up the `loadavg` and `distccd` programs on 
   the host machine.)

12. In your Makefiles (or Construct or SConstruct files), make each 
    compilation line look like this:

        gethost distcc gcc ...

13. Run your build script and see the compilations being farmed out to the 
    fastest available compilation machines in your network.

## ![DMUCS](images/dmucs-logo-small.gif) Tips and Tricks

* If you are having problems getting DMUCS to work, first make sure that 
  you can compile using `distcc`, but not dmucs.  If this works, then you 
  are most of the way there.  If not, then you should go to the `distcc` 
  web page for lots of hints on getting `distcc` to work.

* If you don’t know what values to give for the “power index” values in the 
  hosts-info file, you can do this:

    1. Set up `distcc` to run a single compilation on a single compilation
       host.
    2. Repeat this for the same file on each host, measuring the time it 
       takes to compile each time.
    3. Give your slowest machine the power index 1.  Then, if the second 
       slowest machine is twice as fast give it the index 2.  If you have a 
       machine that is 10 times faster, give it the index 10.
    4. Note that it is often very useful to have multiple machines have the 
       same power index.  In this case, dmucs will randomly assign 
       compilation tasks to the cpus for hosts with the same power index.
 
## ![DMUCS](images/dmucs-logo-small.gif) Details on How DMUCS Selects Hosts

* dmucs stores information about each compilation host in a tier that 
  equals the host’s power index.  Thus, if there are 3 machines all with 
  power index 6, dmucs will put all machines at tier 6.
 
* When dmucs gets a host request, it randomly selects one of the cpus from 
  one of the hosts in the highest tier.

* When dmucs gets a load average message from a compilation host indicating 
  that the load average on the machine is nearing 1.0 (or, more precisely, 
  `number-of-cpus * 1.0`), dmucs moves the cpus for that machine from tier 
  `power-index` to tier `power-index – 1`.  Thus, if a machine at tier 6 is 
  getting overloaded, it is moved to tier 5.  Subsequent gethost requests 
  will result in allocations from remaining hosts at tier 6, thus lightening 
  the load on the machine that was “downgraded”.
 
* If a machine’s load average stays above its `power-index * 1.0` for more 
  than 5 minutes, dmucs will move the machine to the “overloaded” state in 
  its internal database.  No more compilations will be assigned to the host 
  until its load average comes back down, and dmucs moves the machine back to 
  its original tier.
 
Original Author: Victor Norman
