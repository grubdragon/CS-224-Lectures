Physical layer: Overview
============================

* The following are some functions that are performed by the link
  layer and physical layer:

- Once a set of bits are received from the higher layers, the
  lik/physical layer adds a few extra bits to the frame in order to
  enable error detection and correction

- The link layer adds some special bits at the start/end of a frame to
  enable the receiver to identify the frame boundaries.

- The bits of the frame are mapped to some underlying characteristic
  of the signal (e.g., 0 is mapped to a low amplitude of the wave and
  1 is mapped to a high amplitude) using an encoding scheme.

- Finally, an electromagnetic wave is imprinted with the data in the
  packet by a process called modulation and is transmitted over the
  physical channel (a copper wire, a fiber optic cable, a wireless
  channel etc.)

* The steps error detection/correction and framing are usually done in
  the part of the link layer that resides in hardware, and is tightly
  coupled with the physical layer that does the modulation.

* What are error detecting/correcting codes? Some examples are as
  follows:

- Simple parity bits. Add a parity bit to the end of a group of bits
  such that the number of 1s is even (or odd). The parity can be added
  once per frame or more frequently (e.g., one bit for every 7
  bits). Parity bits can detect single bit errors, but cannot be used
  to correct the errors since they cannot pinpoint the error
  location. Further, if two bits are in error, the parity can fail to
  detect it.

- Two dimensional parity bits: add one parity bit for every 7 bits of
  data, and add one parity bit for each bit position across bytes in
  the frame. That is, compute parity bits for every row of 7 bits, and
  a parity byte across all columns of bit positions. Such parity bits
  can detect a larger number of bit errors.

- Checksum: a sum of groups of bits of the frame can be stored as a
  checksum. For example, divide the frame into groups of 16 bits, add
  them all up, and store a 16 bit checksum (how is carry handled?
  There are rules for bit arithmatic). The sender embeds the sum as
  part of the packet header. The receiver independently computes the
  sum on the received packet. If they do not match, error is
  detected. In the internet protocols, checksum is used at the
  transport layer.

- CRC (cyclic redundancy check) is a more complex form of error
  detection as compared to checksum. All the bits in a message are
  treated as coefficients of a polynomial, and a few extra bits are
  added to make the polynomial divisible by a special generator
  polynomial. At the receiver, if the received bits are not exactly
  divisible by the generator, an error is suspected. The Ethernet
  header adds a 32-bit CRC over a 1500 byte packet, which can detect
  most types of bit error patterns.

- Repetetive codes: another simple idea is to repeat the same bit
  multiple times. For example, if every bit is repeated 3 times (send
  111 instead of 1), up to two bit errors can be detected. For
  example, if 000 or 111 is corrupted to 011 or 001 or something like
  that, an error can be detected. However, if 000 is changed to 111,
  it cannot be detected.

Note that if only one bit errors can occur, then the above code can
not only detect but also correct errors. For example, if 001 is
received, it can be corrected to 000. This simple idea can be
generalized to more complex error correcting codes.

* An error correcting code takes a m-bit message, adds a few parity
  bits, to create a n-bit code word. Out of the 2^n possible
  codewords, only 2^m correspond to valid messages and the rest
  indicate error states. For the repetetive code above, m=1 and n=3.

* Hamming distance between two code words = the number of bits in
  which the code words differ. For example, in the repetetive code
  above, the code words 000 and 111 have a Hamming distance of 3.

* Let d be the minimum distance between any two valid code words in an
  error correcting code. Then, the code can be used to detect up to
  d-1 bit errors, OR can be used to correct (d-1)/2 bit errors. For
  example, the above repetetive code can detect 3-1=2 bit errors or
  correct (3-1)/2 = 1 bit errors.

* What is the best way to construct code words for a given value of m
  and n? Several possible codes exist, e.g., Hamming codes.

* Next topic: how are bits converted into high/low voltages? Several
  encoding schemes exist.

- NRZ (non return to zero): bit 0 is encoded with low voltage and 1
  with high voltage. Problem: long series of 0s or 1s cause the
  baseline average to be distorted (the receiver uses the baseline
  average to calibrate what is high and low). Another problem: the
  receiver can lose sync with the sender about where the bit
  boundaries are if there are not enough transitions and their clocks
  drift.

Some systems use scrambling, i.e., exor the data with a random
sequence to avoid long series of 0s and 1s.

- NRZI (NRZ inverted): transition from current signal to encode 1 and
  stay at same signal to encode 0. Long strings of 0s are still a
  problem.

- Manchester encoding: 0 is encoded with a low-to-high transition, and
  1 with a high-to-low transition. Problem: now the receiver has to
  sample twice to get one bit of information. Efficiency = 50%. That
  is, the rate at which information is transmitted is halved compared
  to NRZ and NRZI. (Baud rate = rate of sending symbols, bit rate =
  rate of sending bits. With Manchester, bit rate is half of baud
  rate)

- 4B/5B: a set of 4 bits are mapped to codewords of 5 bits, such that
  there are no consecutive 0s. Then, the resulting 5 bit codewords are
  sent via NRZI. The efficiency is 80%.

* Framing: how does the receiver identify start and end of packets?

- Special start and end symbols (a special pattern of bits) can be
  placed to let the receiver know when it should start/stop sampling
  voltages on the channel. For example, in the 4B/5B code, some 5-bit
  codewords serve as start/end indicators. What if these bit patterns
  appear inside the payload? They must be 'escaped' with character
  stuffing.

- Byte counting approaches: the sender sends a fixed length payload,
  or informs the receiver of the payload length as part of the header,
  so that the receiver knows when to stop decoding. What if the count
  value in the header is corrupted because of a bit error? Framing
  error: frame boundaries are incorrectly inferred.
