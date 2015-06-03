# freenas-stuff

# freenas-stuff
I needed to setup a vpn tunnel at boot which mounts remote filesystems. Unfortunately, in Freenas, you cannot 
automatically login a openvpn connection (no auth possible) on the jailhost and you cannot install 
(ports-)software. A workaround is to install "expect" in a jail and copy the required files to the jail host. 
When done, the login can be automated with a script (expect/sh).

make (or in) a jail
ssh to the jail
pkg install expect

note: "tempdir" in the next steps should be reachable/available for the jail host.

    mkdir /mnt/tempdir/tcl8.6
    cp /usr/local/bin/expect /mnt/tempdir
    cp /usr/local/lib/expect5.45/libexpect545.so /mnt/tempdir/ 
    cp /usr/local/lib/libtcl86.so.1 /mnt/tempdir/ 
    cp -r /usr/local/lib/tcl8.6 /mnt/tempdir/ 

on jail host:

    cp /mnt/tempdir/libexpect545.so /usr/local/lib/expect5.45/libexpect545.so
    cp /mnt/tempdir/libtcl86.so.1 /usr/local/lib/libtcl86.so.1
    mkdir /usr/local/lib/tcl8.6
    cp -r /mnt/tempdir/tcl8.6/ /usr/local/lib/tcl8.6/

test if expect is working on the jail host:

    /usr/local/bin/expect
    expect1.1>

now a script can be used to automatically login openvpn

    vi /root/startup.sh

    #!/bin/sh

    /usr/local/bin/expect <<EOD
    spawn /usr/local/sbin/openvpn openvpn.ovpn
    sleep 1
    match_max 100000
    expect -exact "sername:"
    send -- "username\r"
    expect -exact "assword:"
    send -- "password\r"
    expect eof
    EOD

    ...perform other actions

to autostart this script:

    vi /conf/base/etc/rc.d/tunnel

    #!/bin/sh
    #
    #PROVIDE: tunnel
    #REQUIRE: DAEMON
    #KEYWORD: shutdown

    . /etc/rc.subr

    name=tunnel
    rcvar=tunnel_enable

    start_cmd="${name}_start"

    tunnel_start()
    {
    /root/startup.sh
    }

    load_rc_config $name
    run_rc_command "$1"

    chmod 555 tunnel
    cp tunnel /etc/rc.d/


    vi /conf/base/etc/rc.conf

add these lines:

   #tunnel
   tunnel_enable="YES"




