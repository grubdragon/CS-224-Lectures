Resource allocation, fairness, QoS
===================================

* Basic problem: internet only provides best effort service. It does
  not guarantee any fair allocation of bandwidth or other
  resources. As a result, no bandiwdth or delay guarantees. What can
  you do to get more quality of service (QoS) guarantees from the
  network? We will discuss two main ideas - traffic shaping and router
  scheduling.

* What is fairness? Two popular definitions: Jain fairness index and
  max-min fairness.

* Jain fairness index = [sum(xi)]^2 / n sum (xi^2). Equal to 1 if all
  xi = 1. Equal to 1/n if only one xi=1 and rest 0. Measures extent of
  fairness.

* However, sometimes, some flows may not have enough traffic to send,
  in which case they cannot get equal share of bandwidth. Suppose 2
  FTP flows and 2 audio flows sharing a link of 1 Mbps. Audio call
  generates 100 kbps each. In that case, it is not possible to
  allocate 250 kbps of "fair share" to audio. What is fair allocation
  here? Since audio need lesser than fair share, give them 100 kbps
  each. Remaining 800 kbps is split between 2 FTP flows equally. This
  notion of fairness is called "max-min" fairness. How to do a max-min
  allocation? Start filling each flow until it hits its limit. Once a
  flow's allocation has reached its limit, continue with the other
  flows.

* By QoS, we mean end-to-end bandwidth and delay
  guarantees. Paricularly important for real time audio/video
  applications. Multimedia applications can withstand a small amount
  of delay by buffering incoming data and delaying playback by few
  hundreds of milliseconds. However, they cannot delay playback by
  much for interactivity. So such applications will work well if there
  is some bound on end-to-end latency. Most applications have some
  basic adaptability built into them (e.g., change resolution of video
  stream or adjust playback buffer), so are able to survive on
  Internet today even though no QoS mechanism is in place.

* Several proposals have been put up for changing the internet
  architecture to enable more fairness and QoS. These proposals
  involve flows specifying their rate, reserving resources along the
  path, and routers enforcing these specifications. None of these
  proposals has seen any implementation, but a few ideas are
  interesting.

* Two main functionalities needed to implement fairness and QoS on the
  Internet. (1) Traffic shaping: traffic from hosts should be
  regulated as per a specific rate that can be handled by the
  Internet. (2) Ruuter scheduling. Routers cannot simply use
  first-in-first-out (FIFO) but must use more sophisticated algorithms
  to guarantee fairness and QoS.

* Now to do traffic shaping? The idea of a token bucket filter (TBF) -
  used to specify traffic, as well as shape incoming traffic. Think of
  a bucket in which tokens accumulate at rate "R". The bucket can hold
  up to "B" tokens, called bucket depth. If no tokens accumulated,
  flow can send at rate R. If tokens accumulated, it can send up to a
  burst B. Optionally, the peak rate P can also be specified, so that
  the burst B doesn't go at a very high rate.

* For example, consider TBF with R = 1 packet/s. B = 20 packets. P = 5
  packets/s. Then, if the flow doesn't have any traffic for a long
  time, it can send at rate P for 4 seconds, after which all tokens in
  bucket will be empty, and it can subsequently send at rate R.

* TBF or other traffic shaping mechanisms must be used by source or by
  routers to stick to a traffic spec, QoS needs to be
  guaranteed. There is no way to guarantee service if all sources send
  unlimited traffic.

* Router scheuling high-level problem: several flows at a bottleneck
  router. We will need to allocate the bottleneck bandwidth in a
  max-min fair way between them. A flow can be a TCP flow or all
  traffic between a source-destination pair or any thing else. We will
  refer to the granularity at which fairness should be maintained as
  "flow".

* What are common router scheduling policies / queueing disciplines?

- First come first serve (FCFS or FIFO). No guarantee whatsoever.

- Priority queueing. When a flow of higher priority exists, send it
  before flow of lower priority. May end up starving some flows. Not
  really fair, but useful when you want to prioritize certain type of
  traffic. 

- Round robin (RR). Enqueue each flow separately. Send from each
  flow's queue in round robin fashion. Simple. O(1)
  complexity. Results in max-min fairness only if packet sizes are
  equal, which is not always the case.

- Fair queueing (FQ). In ideal world, we want to do bit-by-bit round
  robin (BR), so that max-min fairness is achieved even with variable
  packet sizes. However, packet is the granularity of
  transmission. The fair queueing algorithm tries to emulate
  bit-by-bit round robin over entire packets. Most FQ algorithms are
  O(log N) where N is number of "active" flows.

- Weighted fair queueing (WFQ). It is possible to give priority to
  flows. For example, we may want to send 1 bit of a flow for every
  "w" bits of another flow. Then the second flow is said to have a
  weight of "w". The WFQ algorithms lets us specify weights in this
  manner. WFQ is also usually O(log N).

- Deficit round robin (DRR) is an approximation to FQ. It is O(1) like
  RR but tries to achieve FQ type allocation in the long run.

* For bounded delay guarantees, traffic shaping using token bucket +
  guaranteed bandwidth using appropriate scheduling like fair queueing
  at every router can provide provably bounded delays. Why? Because
  your queueing delay is only influenced by your own traffic in fair
  queueing. So if your traffic is limited by token bucket, maximum
  queue size any packet will see is only bucket size, hence delays are
  bounded.

* We will study BR, FQ, WFQ, DRR in detail. Consider the following
  running example throughout the lecture. Two flows A and B are
  sharing a link that has a capacity of 1 bit/sec. A has two packets,
  say A1 and A2, of 100 bits each that arrive at t=0 and t=50. B has
  two packets (B1, B2) of 50 bits each that arrive at t=50. Let us
  understand the various scheduling policies using these example.

* With FCFS, the order of packets and the end times of transmissions
  are as follows. A1(100), A2(200), B1(250), B2(300).

* With RR, A1(100), B1(150), A2(250), B2(300). Some unfairness to B
  due t its smaller packet size.

* Let's see what ideal BR should do. When a bit is sent from one
active flow each, we count that as a "round". We will track both
rounds, time, and bits sent.

Time	Round BitsofA  BitsofB	Event
0	0     0	       0	A1 arrives
50	50    50       0	A2,B1,B2 arrive
150	100   50       50	A1,B1 done
250	150   50       50	B2 done
300	200   50       0	A2 done

So BR achieves the following order A1,B1(150), B2(250), A1(300). We
want to achieve something similar using packets. That is the goal of
FQ.

* Note the relationship between round number and number of active
  flows. In 0-50, round number grew by one each second. In 50-150, it
  grew by one every 2 seconds because of 2 active flows. This can be
  generalized as follows. dR/dt = L/N_ac. L is rate of link (faster
  link means round finish faster). N_ac is number of active flows
  (more flows means round go slower).

* Key idea of FQ: for every incoming packet, compute the round at
  which it will finish, and schedule packets based on this finish
  round number. Let SX_i and FX_i be the rounds in which packet i of
  flow X starts and finishes transmission. Then, note that FX_i = SX_i
  + P_i where P_i is simply length of packet i. What is SX_i?

* If no earlier packet of this flow in queue when packet arrives, then
  SX_i is simply the round in which the packet arrives
  AX_i. Otherwise, it is round in which previous packet finishes FX_{i-1}. SX_i
  = max(FX_{i-1}, AX_i), where AX_i is arrival round of packet i of
  flow X.

* For every flow, as long as you keep updating the round computation
  with time, the FQ algorithm works as follows. Whenever a packet
  arrives, put it in per-flow Q. Compute its finishing round number
  (either based on finishing round number of previous packet, or based
  on arrival round number). Whenever link gets free, pick the packet
  with the smallest F and transmit it. This algorithm is guaranteed to
  give max-min allocation and performance similar to BR. Complexity
  O(log N).

* Lets see how it works for the example above.
  SA_1 = 0. FA_1 = 0 + 100 = 100
  SA_2 = max(100, 50) = 100. FA_2 = 100+100 = 200
  SB_1 = 50. FB_1 = 50+50 = 100
  SB_2 = max(100, 50) = 100, FB_2 = 100+50 = 150

* Sorting packets in order of finishing rounds and transmitting them,
  we get A1(100), B1(150), B2(200), A2(300). Note that we get the
  order of bit-wise round robin, therefore we have achieved our goal.

* Preemptive vs non-preemptive. If a packet with lower F arrives when
  a packet is being transmitted, you cannot normally preempt ongoing
  transmission in practice.

* Weighted fair queueing: you send "w" bits of a flow in a certain
  round. So FX_i = SX_i + P_i/w. So flows with higher wait finish in
  earlier rounds, have lower F, and get picked faster.

* BR based on weights 1:5 for A and B for earlier example achieves the
  following order: B1, B2, A1, A2

Time	Round BitsofA  BitsofB	Event
0	0     0	       0	A1 arrives
50	50    50       0	A2,B1,B2 arrive
110	60    10       50	B1 done
170	70    10       50	B2 done
200	100   30       0	A1 done
300	200   100      0	A2 done

* WFQ calculations on the same example
  SA_1 = 0. FA_1 = 0 + 100 = 100
  SA_2 = max(100, 50) = 100. FA_2 = 100+100 = 200
  SB_1 = 50. FB_1 = 50+50/5 = 60
  SB_2 = max(60, 50) = 60, FB_2 = 60+50/5 = 70

* So order of transmission in WFQ should be B1, B2, A1, A2 much like
  BR. However, since B1 doesn't arrive till t=50, A1 will be
  sent. Depending on preemptive or non-preemptive mode, B1 may have to
  wait till A1 finishes. If no preemption A1(100), B1(150), B2(200),
  A2(300). If A1 gets prempted at 50, then we have the following
  finishing times. B1(100), B2(150), A1(200), A2(300). Phew!

* Alternate to FQ: deficit round robin (DRR). It is similar to RR,
  except for the notion of credits/deficit. Every queue maintains a
  deficit counter. In each round, every queue receives a "quantum" Q
  of credits. Once the transmission finishes, the data transmitted is
  deducted from the deficit counter DC. In every round, only those
  queues with positive DC are allowed to transmit. In some sense, the
  notion of credits ensures fairness across different apcket
  sizes. Different Q for each queue can create preferential treatment
  for some flows. In long term, leads to bandwidth allocation similar
  to FQ, though short term delays may be different.

* Lets work out DRR for earlier example with Q=60.

Round Time   DC_A 	    DC_B    	Event
1     0      0+60=60	    0		A1 arrives, starts transmission
1     100    60-100=-40     0+60=60	B1 starts
1     150    -40	    60-50=10	B1 done, one round done
2     150    -40+60=20	    10		A2 starts
2     250    20-100=-80	    10+60=70	A2 done, B2 starts
2     300    -80	    70-50=20	B2 done

* It lead to same order as RR. Why? What can be done to fix it? Let's try with quantum size of 40.

Round Time   DC_A 	    DC_B    	Event
1     0      0+40=40	    0		A1 arrives, starts transmission
1     100    40-100=-60     0+40=40	B1 starts
1     150    -60	    40-50=-10	B1 done, round 1 done
2     150    -60+40=-20	    -10+40=30	skip A, start B2
2     200    -20	    30-50=-20	B2 done, round 2 done
3     200    -20+40=20	    -20		A2 starts
3     300    20-100=-80	    -20		A2 done

* Smaller quantum size led to A1, B1, B2, A2 which is closer to what
  FQ gets. Is DRR equal to FQ for quantum size of 1 bit? For example,
  consider what will happen in the following situation. N flows, N-1
  of which have packets of size M. The Nth flow is initially
  empty. For M-1 rounds, no one sends anything, since DC<0. The Nth
  flow gets a packet just at the beginning of round M. In the Mth
  round, all N-1 flows will send their packets, and the Nth flow will
  send one bit. What would FQ have done? Is the packet delay of Nth
  flow same with DRR and FQ? Is the bandwidth allocation the same?
  Think about it.

* Other things routers can do: Weighted RED (WRED). Have different
  drop probability RED curves for different flows.

