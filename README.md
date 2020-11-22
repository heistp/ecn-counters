# ecn-counters

The attached script adds an iptables chain containing ECN related counters.

## Sample Run

The included file
[freenet_counters_20200514.txt](freenet_counters_20200514.txt) contains output
from the `status` command after an 11 day run at a cooperative ISP in the
Czech Republic from 5/4/2020 - 5/14/2020.

Please take these numbers with a grain of salt. The script has only been
tested at a basic level, so it may have bugs, or there may be unexpected
sources of counter activity in real-world environments. File an issue or pull
request if you feel there are improvements that could be made.

### Results Table

| Packets | Bytes | Gigabytes | Proto/Dir | What |
| ------- | ----- | --------- | --------- | ---- |
| 81157996 | 4667985921 | 4.35 | TCP out | tcp flags:0x17/0x02 /* ECN: All SYN out */ |
| 1902227 | 121047437 | 0.113 | TCP out | tcp flags:0x17/0x02 ECN match ECE CWR /* ECN: ECN SYN out */ |
| 288344 | 15207086 | 0.014 | TCP out | tcp flags:!0x17/0x02 ECN match ECE /* ECN: ECE out */ |
| 121544 | 49921253 | 0.046 | TCP out | tcp flags:!0x17/0x02 ECN match CWR /* ECN: CWR out */ |
| 11494614191 | 2373487139381 | 2,210.48 | IP out | ECN match ECT=0 /* ECN: Not ECT out */ |
| 880921 | 1235251974 | 1.15 | IP out | ECN match ECT=1 /* ECN: ECT(1) out */ |
| 19772274 | 18040388132 | 16.80 | IP out | ECN match ECT=2 /* ECN: ECT(0) out */ |
| 591218 | 829255429 | 0.772 | IP out | ECN match ECT=3 /* ECN: CE out */ |
| 11515858604 | 2393592034916 | 2,229.21 | IP out | IP total out |
| 24393708 | 1293247903 | 1.20 | TCP in | tcp flags:0x17/0x02 /* ECN: All SYN in */ |
| 363412 | 18965484 | 0.018 | TCP in | tcp flags:0x17/0x02 ECN match ECE CWR /* ECN: ECN SYN in */ |
| 1276889 | 77063893 | 0.072 | TCP in | tcp flags:!0x17/0x02 ECN match ECE /* ECN: ECE in */ |
| 438550 | 757936446 | 0.706 | TCP in | tcp flags:!0x17/0x02 ECN match CWR /* ECN: CWR in */ |
| 11713607403 | 30199838014685 | 28,125.79 | IP in | ECN match ECT=0 /* ECN: Not ECT in */ |
| 129034 | 152053523 | 0.142 | IP in | ECN match ECT=1 /* ECN: ECT(1) in */ |
| 131199589 | 385599433770 | 359.12 | IP in | ECN match ECT=2 /* ECN: ECT(0) in */ |
| 307597 | 141859857 | 0.132 | IP in | ECN match ECT=3 /* ECN: CE in */ |
| 11845243623 | 30585731361835 | 28,485.18 | IP in | IP total in |

### Interpretation

A few statistics:

| Stat | Out | In |
| ---- | --- | -- |
| Percent TCP SYNs with ECT(0) set | 2.34%<br/>(1902227/81157996) | 1.49%<br/>(363412/24393708) |
| Percent IP packets with ECT(0) set | 0.172%<br/>(19772274/11515858604) | 1.108%<br/>(131199589/11845243623) |
| Percent IP packets with CE set | 0.00513%<br/>(591218/11515858604) | 0.00260%<br/>(307597/11845243623) |

The lack of a stateful analysis makes counters challenging to interpret, but:
* there are a small but nonzero percentage of ECN initiating endpoints
* there appears to be congestion signaling with CE, both from the inside and
  outside of the network
* there are a nonzero number of packets with ECT(1) set (not me!)

A few of the ISP's backhaul links use fq_codel for queue management, but it's
unknown how much this contributes to the statistics. The CE marks on ingress
appear to be from some external source.

### Perspective

To put into perspective the percentage of CE marked packets we might typically
observe on single flows, several tests with CUBIC and Prague through fq_codel at
50Mbps are
[here](http://sce.dnsmgr.net/results/l4s-2020-11-11T120000-final/l4s-s7-oneflow/).
The CE marking percentage is based on the BDP and the CC algo in use.

From the results linked to above, here is a table of counters put together from
the [scetrace](https://github.com/heistp/scetrace) json output, and the
resulting percentage of CE marks on the total number of segments sent:

| CC | RTT | Segments | CE Marks | Percent CE |
| -- | --- | -------- | -------- | ---------- |
| CUBIC | 20ms | 244292 | 46 | 0.01883% |
| CUBIC | 80ms | 226654 | 14 | 0.00618% |
| Prague | 20ms | 248790 | 5977 | 2.402% |
| Prague | 80ms | 247715 | 2101 | 0.848% |
