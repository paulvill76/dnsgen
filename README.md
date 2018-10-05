ISC DNS Packet Generator
========================

dnsgen
------

`dnsgen` is somewhat like Nominum's `dnsperf` utility, and indeed
shares many of the same command line parameters.

Unlike `dnsperf`, it uses `AF_PACKET` raw sockets and therefore only
runs under Linux.  The use of raw sockets allows for the use of a
far larger range of source ports and higher performance than using
"normal" UDP sockets.  The data file is loaded completely into
memory on start up so that disk I/O does not affect measurements.
For optimal performance `dnsgen` supports a raw input file mode where
the data file contains raw pre-compiled DNS queries.

To reduce CPU load `dnsgen` does not attempt to correlate received
packets with those it has transmitted.  It simply counts those packets
that arrive back on the network interface.  Is is therefore best used
on a network interface that is directly connected to the server under
test and not shared with any other services.

In normal operation the packet-per second value reported is the peak
rolling average of the received packet rate observed during the run.

To attempt to find this value `dnsgen` starts sending packets at the
specified sending rate (`-r` option) and then measures the rate at
which packets are received.  The sending rate is then continually
adjusted to be the midpoint between the maximum observed rate so
far and the received rate, plus the specified "increment" rate (`-R`
option).

Eventually a steady state should be achieved when the difference
between the received rate and the transmitted rate is equal to the
increment rate, and where that increment represents a small overhead
in lost packets.

In the alternative "ramp" mode (`-M` option) packets are transmitted
at the specified starting rate with the rate increasing thereafter
by the specified increment every 0.1s without regard to the inbound
received rate.

The "batch" value (`-b`) controls how many packets are transmitted at
a time via the `sendmmsg` system call.  It is important to tune this
to find the optimal value for your configuration.

dnsecho
-------

Uses `AF_PACKET` mode to receive raw (UDP) packets and immediately
return them from whence they came.

Raw File Format
===============

The raw file format looks exactly like a stream of TCP queries,
i.e. a repeated sequence of a two byte packet length (in network
order) followed by the query in wire format.

EDNS OPT RRs are not included within the file, but may be optionally
added "in memory" via the `QueryFile` API once the raw file has
been loaded.

The `dnscvt` utility should be used to convert `dnsperf` format input
files into the raw format.
