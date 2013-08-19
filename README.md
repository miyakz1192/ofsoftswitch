# OpenFlow 1.3 Software Switch

This is an [OpenFlow 1.3][ofp13] compatible user-space software switch implementation. The code is based on the [Ericsson TrafficLab 1.1 softswitch
implementation][ericssonsw11], with changes in the forwarding plane to support
OpenFlow 1.3.

The following components are available in this package:
* `ofdatapath`: the switch implementation
* `ofprotocol`: secure channel for connecting the switch to the controller
* `oflib`: a library for converting to/from 1.3 wire format
* `dpctl`: a tool for configuring the switch from the console

# Getting Started

These instructions have been verified on Ubuntu 12.04 LTS. Other distributions or versions may need different steps.

## Before building
The switch makes use of the NetBee library to parse packets, so we need to install it first.  
Goto https://github.com/MeshSr/netbee-lite for more detailed instructions.  
We have provided pre-compiled library files in ofsoftswitch/lib. If you want to update them with new versions, just follow step 2, or you can skip it.  

1. Get ofsoftswtich sorce code

    ```
    $ git clone https://github.com/MeshSr/ofsoftswitch.git
    ```

2. After successfully compiling the netbee-lite project, you should add its shared libraries built in `<your-path-to>/netbee-lite/bin/` to `ofsoftswitch/lib` directory.  

    ```
    $ cp <your-path-to>/netbee-lite/bin/libn*.so ofsoftswitch/lib
    ```

## Building
Run the following commands in the `of13softswitch` directory to build and install everything:

    $ ./boot.sh
    $ ./configure  --build=i686-pc-linux --host=arm-xilinx-linux-gnueabi --target=i686-linux LIBS="-L./lib -lnbee -lnbprotodb -lnbnetvm -lnbpflcompiler -lnbsockutils -lpcap -lpcre -lxerces-c -licuuc -licudata"
    $ make
    $ sudo make install

## Running
1. Start the datapath:

    ```
    $ sudo udatapath/ofdatapath --datapath-id=<dpid> --interfaces=<if-list> ptcp:<port>
    ```

    This will start the datapath, with the given datapath ID, using the interaces listed. It will open a passive TCP connection on the given port. For a complete list of options, use the `--help` argument.

2. Start the secure channel, which will connect the datapath to the controller:

    ```
    $ secchan/ofprotocol tcp:<switch-host>:<switch-port> tcp:<ctrl-host>:<ctrl-port>
    ```

    This will open TCP connections to both the switch and the controller, relaying OpenFlow protocol messages between them. For a complete list of options, use the `--help` argument.

## Configuring
You can send requests to the switch using the `dpctl` utility.

* Check the flow statistics for table 0.

    ```
    $ utilities/dpctl tcp:<switch-host>:<switch-port> stats-flow table=0
    ```

* Install a flow to match IPv6 packets with extension headers hop by hop and destination and coming from port 1.

    ```
    $ utilities/dpctl tcp:<switch-host>:<switch-port> flow-mod table=0,cmd=add in_port=1,eth_type=0x86dd,ext_hdr=hop+dest apply:output=2
    ```

* Add a meter:

    ```
    $ utilities/dpctl tcp:<switch-host>:<switch-port> meter-mod cmd=add,meter=1 drop:rate=50
    ```

* Send flow to meter table

    ```
    $ utilities/dpctl tcp:<switch-host>:<switch-port> flow-mod table=0,cmd=add in_port=1 meter:1
    ```

For a complete list of commands and arguments, use the `--help` argument.

The `dpctl` utility has some limitations at the moment:
* No support for OXM masks
* No support for multipart messages
* Some set_field action fields are not present


# Contribute
Please submit your bug reports, fixes and suggestions as pull requests on
GitHub, or by contacting us directly.

# License
OpenFlow 1.3 Software Switch is released under the BSD license (BSD-like for
code from the original Stanford switch).

[ofp13]: https://www.opennetworking.org/images/stories/downloads/specification/openflow-spec-v1.3.0.pdf
[ericssonsw11]: https://github.com/TrafficLab/of11softswitch
