# Signal Generation and Capture

In order to conform to FCC emission regulations
([Section 15.205](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-C/section-15.205),
[Section 15.209](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-C/section-15.209),
[Section 15.247](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-C/subject-group-ECFRf45d172319b1f81/section-15.247)),
one main design consideration is to control the spectral shape of the
TX signal. This is typically done by carefully design the **pulse
shape** of the digital symbols. We will discuss a standard pulse
shaping design here. In addition, we are primarily interested in
implementing a packet-switched communication system. In a
packet-switched system, the transmission is neither continuous nor
regular. Hence, we must determine at the RX whether the RX samples
contain the TX signal. This is typical done by an energy detector. We
will discuss different energy detectors and a related processing
step called **automatic gain control**.


