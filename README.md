# Chapter 37

Do the same requests above, but change the seek rate to different values: -S 2,-S 4,-S 8,-S 10,-S 40,-S 0.1. How do the times change?

$ ./disk.py -a 7,30,8 -G -S 2
$ ./disk.py -a 7,30,8 -G -S 4
Seek time is shorter. Default value is 1.

Do the same requests above, but change the rotation rate: -R 0.1,-R 0.5,-R 0.01. How do the times change?

Default value is 1. Rotate time and transfer time are longer.

FIFO is not always best, e.g., with the request stream -a 7,30,8, what order should the requests be processed in? Run the shortest seek-time first (SSTF) scheduler (-p SSTF) on this workload; how long should it take (seek, rotation, transfer) for each request to be served?

➜  CS5600-disk ./disk.py -a 7,30,8 -c -p FIFO
OPTIONS seed 0
OPTIONS addr 7,30,8
OPTIONS addrDesc 5,-1,0
OPTIONS seekSpeed 1
OPTIONS rotateSpeed 1
OPTIONS skew 0
OPTIONS window -1
OPTIONS policy FIFO
OPTIONS compute True
OPTIONS graphics False
OPTIONS zoning 30,30,30
OPTIONS lateAddr -1
OPTIONS lateAddrDesc 0,-1,0

z 0 30
z 1 30
z 2 30
0 30 0
0 30 1
0 30 2
0 30 3
0 30 4
0 30 5
0 30 6
0 30 7
0 30 8
0 30 9
0 30 10
0 30 11
1 0 30 12
1 0 30 13
1 0 30 14
1 0 30 15
1 0 30 16
1 0 30 17
1 0 30 18
1 0 30 19
1 0 30 20
1 0 30 21
1 0 30 22
1 0 30 23
2 0 30 24
2 0 30 25
2 0 30 26
2 0 30 27
2 0 30 28
2 0 30 29
2 0 30 30
2 0 30 31
2 0 30 32
2 0 30 33
2 0 30 34
2 0 30 35
REQUESTS ['7', '30', '8']

Block:   7  Seek:  0  Rotate: 15  Transfer: 30  Total:  45
Block:  30  Seek: 80  Rotate:220  Transfer: 30  Total: 330
Block:   8  Seek: 80  Rotate:310  Transfer: 30  Total: 420

TOTALS      Seek:160  Rotate:545  Transfer: 90  Total: 795

➜  CS5600-disk ./disk.py -a 7,30,8 -c -p SSTF
OPTIONS seed 0
OPTIONS addr 7,30,8
OPTIONS addrDesc 5,-1,0
OPTIONS seekSpeed 1
OPTIONS rotateSpeed 1
OPTIONS skew 0
OPTIONS window -1
OPTIONS policy SSTF
OPTIONS compute True
OPTIONS graphics False
OPTIONS zoning 30,30,30
OPTIONS lateAddr -1
OPTIONS lateAddrDesc 0,-1,0

z 0 30
z 1 30
z 2 30
0 30 0
0 30 1
0 30 2
0 30 3
0 30 4
0 30 5
0 30 6
0 30 7
0 30 8
0 30 9
0 30 10
0 30 11
1 0 30 12
1 0 30 13
1 0 30 14
1 0 30 15
1 0 30 16
1 0 30 17
1 0 30 18
1 0 30 19
1 0 30 20
1 0 30 21
1 0 30 22
1 0 30 23
2 0 30 24
2 0 30 25
2 0 30 26
2 0 30 27
2 0 30 28
2 0 30 29
2 0 30 30
2 0 30 31
2 0 30 32
2 0 30 33
2 0 30 34
2 0 30 35
REQUESTS ['7', '30', '8']

Block:   7  Seek:  0  Rotate: 15  Transfer: 30  Total:  45
Block:   8  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
Block:  30  Seek: 80  Rotate:190  Transfer: 30  Total: 300

TOTALS      Seek: 80  Rotate:205  Transfer: 90  Total: 375

➜  CS5600-disk 



Now use the shortest access-time first(SATF) scheduler(-p SATF). Does it make any difference for -a 7,30,8 workload? Find a set of requests where SATF outperforms SSTF; more generally, when is SATF better than SSTF?

No difference.


SATF is better when seek time is shorter then rotate time.


Here is a request stream to try: -a 10,11,12,13. What goes poorly when it runs? Try adding track skew to address this problem (-o skew). Given the default seek rate, what should the skew be to maximize performance? What about for different seek rates (e.g., -S 2, -S 4)? In general, could you write a formula to figure out the skew?

$ ./disk.py -a 10,11,12,13 -c
$ ./disk.py -a 10,11,12,13 -o 2 -c
skew = (track-distance(40) / seek-speed) * rotation-speed / rotational-space-degrees(360 / 12) = math.ceil((40 / 1) * 1 / 30) = 2

-S 2: math.ceil((40 / 2) * 1 / 30) = 1

-S 4: math.ceil((40 / 4) * 1 / 30) = 1

-R 2: math.ceil((40 / 1) * 2 / 30) = 3

Specify a disk with different density per zone, e.g., -z 10,20,30, which specifies the angular difference between blocks on the outer, middle, and inner tracks. Run some random requests (e.g., -a -1 -A 5,-1,0, which specifies that random requests should be used via the -a -1 flag and that five requests ranging from 0 to the max be generated), and compute the seek, rotation, and transfer times. Use different random seeds. What is the bandwidth (in sectors per unit time) on the outer, middle, and inner tracks?

$ ./disk.py -z 10,20,30 -a -1 -A 5,-1,0 -c
outer: 3/(135+270+140)=0.0055
middle: 2/(370+260)=0.0032

$ ./disk.py -z 10,20,30 -a -1 -A 5,-1,0 -s 1 -c
outer: 3/(255+385+130)=0.0039
middle: 2/(115+280)=0.0051

$ ./disk.py -z 10,20,30 -a -1 -A 5,-1,0 -s 2 -c
outer: 2/(85+10)=0.0211
middle: 3/(130+360+145)=0.0047

$ ./disk.py -z 10,20,30 -a -1 -A 5,-1,0 -s 3 -c
outer: 5/875=0.0057



A scheduling window determines how many requests the disk can examine at once. Generate random workloads (e.g., -A 1000,-1,0, with different seeds) and see how long the SATF scheduler takes when the scheduling window is changed from 1 up to the number of requests. How big of a window is needed to maximize performance? Hint: use the -c flag and don’t turn on graphics (-G) to run these quickly. When the scheduling window is set to 1, does it matter which policy you are using?

Set to 1 is equal to FIFO. Maximize performance needs size of the disk(-w -1).

$ ./disk.py -A 1000,-1,0 -p SATF -w 1 -c      // 220125
$ ./disk.py -A 1000,-1,0 -p FIFO -w 1 -c      // 220125
$ ./disk.py -A 1000,-1,0 -p SSTF -w 1 -c      // 220125
$ ./disk.py -A 1000,-1,0 -p BSATF -w 1 -c     // 220125
$ ./disk.py -A 1000,-1,0 -p SATF -w 1000 -c   // 35475
Create a series of requests to starve a particular request, assuming an SATF policy. Given that sequence, how does it perform if you use a bounded SATF (BSATF) scheduling approach? In this approach, you specify the scheduling window (e.g., -w 4); the scheduler only moves onto the next window of requests when all requests in the current window have been serviced. Does this solve starvation? How does it perform, as compared to SATF? In general, how should a disk make this trade-off between performance and starvation avoidance?

$ ./disk.py -a 12,7,8,9,10,11 -p SATF -c          // 7,8,9,10,11,12 Total: 555
$ ./disk.py -a 12,7,8,9,10,11 -p BSATF -w 4 -c    // 7,8,9,12,10,11 Total: 525
All the scheduling policies we have looked at thus far are greedy; they pick the next best option instead of looking for an optimal schedule. Can you find a set of requests in which greedy is not optimal?

$ ./disk.py -a 9,20 -c            // 435
./disk.py -a 9,20 -c -p SATF  



# Chapter 38

Use the simulator to perform some basic RAID mapping tests. Run with different levels (0, 1, 4, 5) and see if you can figure out the mappings of a set of requests. For RAID-5, see if you can figure out the difference between left-symmetric and left-asymmetric layouts. Use some different random seeds to generate different problems than above.

$ ./raid.py -L 5 -5 LS -c -W seq
$ ./raid.py -L 5 -5 LA -c -W seq

left-symmetric    left-asymmetric
0 1 2 P           0 1 2 P
4 5 P 3           3 4 P 5
8 P 6 7           6 P 7 8
Do the same as the first problem, but this time vary the chunk size with -C. How does chunk size change the mappings?

$ ./raid.py -L 5 -5 LS -c -W seq -C 8K -n 12

0  2  4  P
1  3  5  P
8 10  P  6
9 11  P  7
Do the same as above, but use the -r flag to reverse the nature of each problem.

$ ./raid.py -L 5 -5 LS -W seq -C 8K -n 12 -r
Now use the reverse flag but increase the size of each request with the -S flag. Try specifying sizes of 8k, 12k, and 16k, while varying the RAID level. What happens to the underlying I/O pattern when the size of the request increases? Make sure to try this with the sequential workload too (-W sequential); for what request sizes are RAID-4 and RAID-5 much more I/O efficient?

$ ./raid.py -L 4 -S 4k -c -W seq
$ ./raid.py -L 4 -S 8k -c -W seq
$ ./raid.py -L 4 -S 12k -c -W seq
$ ./raid.py -L 4 -S 16k -c -W seq
$ ./raid.py -L 5 -S 4k -c -W seq
$ ./raid.py -L 5 -S 8k -c -W seq
$ ./raid.py -L 5 -S 12k -c -W seq
$ ./raid.py -L 5 -S 16k -c -W seq
16k

Use the timing mode of the simulator (-t) to estimate the performance of 100 random reads to the RAID, while varying the RAID levels, using 4 disks.

$ ./raid.py -L 0 -t -n 100 -c    // 275.7
$ ./raid.py -L 1 -t -n 100 -c    // 278.7
$ ./raid.py -L 4 -t -n 100 -c    // 386.1
$ ./raid.py -L 5 -t -n 100 -c    // 276.5
Do the same as above, but increase the number of disks. How does the performance of each RAID level scale as the number of disks increases?

$ ./raid.py -L 0 -t -n 100 -c -D 8   // 275.7 / 156.5 = 1.76
$ ./raid.py -L 1 -t -n 100 -c -D 8   // 278.7 / 167.8 = 1.66
$ ./raid.py -L 4 -t -n 100 -c -D 8   // 386.1 / 165.0 = 2.34
$ ./raid.py -L 5 -t -n 100 -c -D 8   // 276.5 / 158.6 = 1.74
Do the same as above, but use all writes (-w 100) instead of reads. How does the performance of each RAID level scale now? Can you do a rough estimate of the time it will take to complete the workload of 100 random writes?

$ ./raid.py -L 0 -t -n 100 -c -w 100       // 275.7    100 * 10 / 4
$ ./raid.py -L 1 -t -n 100 -c -w 100       // 509.8    100 * 10 / (4 / 2)
$ ./raid.py -L 4 -t -n 100 -c -w 100       // 982.5
$ ./raid.py -L 5 -t -n 100 -c -w 100       // 497.4
$ ./raid.py -L 0 -t -n 100 -c -D 8 -w 100  // 275.7 / 156.5 = 1.76    100 * 10 / 8
$ ./raid.py -L 1 -t -n 100 -c -D 8 -w 100  // 509.8 / 275.7 = 1.85    100 * 10 / (8 / 2)
$ ./raid.py -L 4 -t -n 100 -c -D 8 -w 100  // 982.5 / 937.8 = 1.05
$ ./raid.py -L 5 -t -n 100 -c -D 8 -w 100  // 497.4 / 290.9 = 1.71
Run the timing mode one last time, but this time with a sequential workload(-W sequential). How does the performance vary with RAID level, and when doing reads versus writes? How about when varying the size of each request? What size should you write to a RAID when using RAID-4 or RAID-5?

$ ./raid.py -L 0 -t -n 100 -c -w 100 -W seq    // 275.7 / 12.5 = 22
$ ./raid.py -L 1 -t -n 100 -c -w 100 -W seq    // 509.8 / 15 = 34
$ ./raid.py -L 4 -t -n 100 -c -w 100 -W seq    // 982.5 / 13.4 = 73
$ ./raid.py -L 5 -t -n 100 -c -w 100 -W seq    // 497.4 / 13.4 = 37
12k.