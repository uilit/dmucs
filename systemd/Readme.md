# Setting up DMUCS as services

## On system wide startup using systemd (requires system admin)

To setup the DMUCS request server and loadavg process to startup when the 
system boots, you need system administrative privileges.

### DMUCS load average monitoring on a compilation host

On any machine which will provide compilation services, copy the file 
`dmucs-defaults` in this directory to the file `/etc/default/dmucs` by 
typing:

```
    sudo cp dmucs-defaults /etc/default/dmucs
```

Edit the `/etc/default/dmucs` file to ensure the `LOADAVG_OPTIONS=` line 
contains the host name of the server which runs the dmucs request server. 
The `dmucs-defaults` file itself has an example.

Again, on any compilation host copy the file `dmucs-loadavg.service` to the 
file `/etc/systemd/system/dmucs-loadavg.service` by typing:

```
    sudo cp dmucs-loadavg.service /etc/systemd/system
```

Then enable this new service by typing:

```
    sudo systemctl enable dmucs-loadavg
```

then type:

```
    sudo systemctl start dmucs-loadavg
```

The load average service will then start whenever the machine is rebooted.

### DMUCS request server (only needed on one machine)

The DMUCS request server is only run on one 'well known' machine which is 
visible over the (local area) network to all compilation hosts and/or the 
machines your users will use to start a compilation.

On the machine which will run the DMUCS request server, copy the file 
`dmucs-defaults` in this directory to the file `/etc/default/dmucs` by 
typing:

```
    sudo cp dmucs-defaults /etc/default/dmucs
```

If you are planning on having your DMUCS request server listen on a port 
other than the 'standard' 9714 port, edit the `/etc/default/dmucs` file to 
ensure the `DMUCS_OPTIONS=` line contains the port on which you want to 
have your DMUCS request server listen for host requests. The 
`dmucs-defaults` file itself has an example.

**NOTE:** If you do use a port other than 9714, you MUST also make sure ALL 
of the compilation host's loadavg processes know which port to use by 
editing the `LOADAVG_OPTIONS=` line in each compilation hosts' 
`/etc/default/dmucs` file.

Again, on the DMUCS request server machine copy the file 
`dmucs-server.service` to the file 
`/etc/systemd/system/dmucs-server.service` by typing:

```
    sudo cp dmucs-server.service /ets/systemd/system
```

Then enable this new service by typing:

```
    sudo systemctl enable dmucs-server
```

then type:

```
    sudo systemctl start dmucs-server
```

The request server service will then start whenever the machine is 
rebooted.

**NOTE:** On the machine which runs the DMUCS request server, make sure any 
firewall actually allows inbound TCP traffic on the port 9714 (or the 
alternative port on which the DMUCS request server listens).

## On personal login (does not require system admin)

You can setup the dmucs request server and/or loadavg process to startup 
when you login with out needing system administrative privileges.

Place the following commands in either your login shell's startup files 
(often `$HOME/.bashrc` or `$HOME/.profile`) or your Desktop's Graphical 
Environment's 'Session and Startup' collection of 'Application Autostart' 
definitions.

We will cover addition to your `$HOME/.bashrc` file here, addition to your 
Desktop's Graphical Environment Session and Startup configuration or other 
shells will vary between Desktop environments or shells, but is, in detail, 
roughly the same.

If your personal machine is a compilation host, then you need to have the 
DMUCS loadavg process running on your machine. To do this add the following 
line to your `$HOME/.bashrc` file:

```
    /usr/local/bin/loadavg --server <DMUCS-request-server-host-name>  &
```

Where `<DMUCS-request-server-host-name>` is replaced by the network host 
name for the DMUCS request server you use. 
Note the use of `&` at the end 
of the line, this will run the loadavg process in the background (and let 
you get on with other work).

If your personal machine is running the DMUCS request server, then you need 
to have the following line in your `$HOME/.bashrc` file:

```
    /usr/local/bin/dmucs &
```

Note, again, the use of `&` at the end of the line, this will run the 
loadavg process in the background (and let you get on with other work).

IF you want the DMUCS request server to listen on a port other than 9714, 
then the line above should instead look like:

```
    /usr/local/bin/dmucs --port <request-service-port> &
```

Where `<request-service-port>` is the port number on which you want the 
DMUCS request server to listen. Note that you MUST also ensure all of the 
loadavg processes on every compilation host has the `--port 
<request-service-port>` added to their startup line.
