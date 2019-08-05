# COMP1521 Week 9 Networking

## Fun stuffs

### ssh to cse in one line

Use local machine to ssh to cse machine is painful as every time of ssh, username and password should be provided. There is a way to solve this by typing one line in terminal to ssh to cse machine.

Add these to `~/.ssh/config` and change the zID to your zID. Generate a ssh key using `ssh-keygen`, and add the public key to cse machine. Then, you are able to ssh to cse by only typing `ssh cse`.

```text
host cse
    Hostname login.cse.unsw.edu.au
    user z5228988
    UseKeychain yes
    AddKeysToAgent yes
    IdentityFile ~/.ssh/cse
```

### traceroute

`traceroute` prints the route packets take to network host.
Connecting to USNW Library network, `traceroute google.com` gives the following resutls. Trying to open these addresses fails because they are routers, not designed for web.

```text
13:38:26@~$ traceroute google.com
traceroute to google.com (216.58.196.142), 64 hops max, 52 byte packets
 1  * * *
 2  ufw1-ae-1-3161.gw.unsw.edu.au (149.171.253.92)  9.349 ms  2.271 ms  2.601 ms
 3  libwdr1-vl-3090.gw.unsw.edu.au (149.171.253.66)  2.833 ms  2.515 ms  2.749 ms
 4  libcr1-te-4-5.gw.unsw.edu.au (149.171.255.89)  2.630 ms  2.698 ms  2.454 ms
 5  unswbr1-te-1-9.gw.unsw.edu.au (149.171.255.101)  17.350 ms  2.222 ms  2.292 ms
 6  138.44.5.0 (138.44.5.0)  2.565 ms  2.568 ms  2.478 ms
 7  et-0-1-0.bdr1.msct.nsw.aarnet.net.au (113.197.15.109)  2.931 ms  2.710 ms  2.698 ms
 8  138.44.10.41 (138.44.10.41)  2.927 ms  3.191 ms  2.746 ms
 9  * * *
10  108.170.233.194 (108.170.233.194)  9.423 ms  3.369 ms
    syd15s04-in-f14.1e100.net (216.58.196.142)  2.843 ms
```

While `traceroute google.com.au` gives longer output.

```text
traceroute to google.com.au (172.217.25.35), 64 hops max, 52 byte packets
 1  * * *
 2  ufw1-ae-1-3161.gw.unsw.edu.au (149.171.253.92)  2.057 ms  2.028 ms  2.150 ms
 3  libwdr1-vl-3090.gw.unsw.edu.au (149.171.253.66)  2.299 ms  2.341 ms  2.280 ms
 4  libcr1-te-4-5.gw.unsw.edu.au (149.171.255.89)  2.232 ms  2.321 ms  8.554 ms
 5  unswbr1-te-1-9.gw.unsw.edu.au (149.171.255.101)  59.736 ms  2.300 ms  2.265 ms
 6  138.44.5.0 (138.44.5.0)  2.464 ms  2.419 ms  2.257 ms
 7  et-0-1-0.bdr1.msct.nsw.aarnet.net.au (113.197.15.109)  18.964 ms  9.392 ms  28.721 ms
 8  gw1.xe-0-2-1.bdr1-msct-nsw.aarnet.net.au (202.158.205.150)  3.982 ms  3.236 ms  3.133 ms
 9  * * *
10  209.85.247.132 (209.85.247.132)  6.380 ms
    209.85.253.176 (209.85.253.176)  4.158 ms
    74.125.37.154 (74.125.37.154)  4.169 ms
11  172.217.25.35 (172.217.25.35)  2.712 ms  2.615 ms
    108.170.233.195 (108.170.233.195)  2.753 ms
```
`traceroute unsw.edu.au` is very long.

```text
 1  * * *
 2  ufw1-ae-1-3161.gw.unsw.edu.au (149.171.253.92)  6.051 ms  2.458 ms  2.396 ms
 3  libwdr1-vl-3090.gw.unsw.edu.au (149.171.253.66)  3.872 ms  2.164 ms  2.234 ms
 4  ombcr1-te-4-5.gw.unsw.edu.au (149.171.255.77)  2.283 ms  2.300 ms  2.271 ms
 5  unswbr1-te-2-13.gw.unsw.edu.au (149.171.255.105)  2.135 ms  2.171 ms  2.071 ms
 6  138.44.5.0 (138.44.5.0)  2.340 ms  2.378 ms  2.235 ms
 7  et-0-1-0.bdr1.msct.nsw.aarnet.net.au (113.197.15.109)  3.361 ms  2.437 ms  2.414 ms
 8  as4826.nsw.ix.asn.au (218.100.52.6)  2.913 ms  3.715 ms  3.059 ms
 9  be-108.cor01.syd11.nsw.vocus.net.au (114.31.192.84)  24.036 ms  23.951 ms
    be-108.cor02.syd04.nsw.vocus.net.au (114.31.192.86)  25.625 ms
10  be200.lsr01.dody.nsw.vocus.network (103.1.77.16)  26.405 ms
    be201.lsr01.dody.nsw.vocus.network (103.1.77.24)  26.613 ms
    be200.lsr01.dody.nsw.vocus.network (103.1.77.16)  26.366 ms
11  be802.lsr01.stck.vic.vocus.network (103.1.76.149)  26.516 ms  27.532 ms  27.049 ms
12  be801.lsr01.lndn.sa.vocus.network (103.1.76.157)  26.830 ms  26.358 ms
    be801.lsr01.adel.sa.vocus.network (103.1.76.155)  25.007 ms
13  be-200.bdr01.adl02.sa.vocus.net.au (103.1.77.129)  24.338 ms  24.587 ms
    be-201.bdr01.adl02.sa.vocus.net.au (103.1.77.153)  26.086 ms
14  static-222.44.255.49.in-addr.vocus.net.au (49.255.44.222)  26.449 ms  25.888 ms  25.758 ms
15  202.58.61.194 (202.58.61.194)  24.031 ms  26.015 ms  24.182 ms
16  * * *
17  202.58.59.102 (202.58.59.102)  30.791 ms !X * *
18  * 202.58.59.102 (202.58.59.102)  30.502 ms !X  25.826 ms !X
```

And `traceroute cse.unsw.edu.au` does not work because UNSW CSE might disabled it for security reason.

```text
 1  * * *
 2  ufw1-ae-1-3161.gw.unsw.edu.au (149.171.253.92)  8.404 ms  2.233 ms  1.816 ms
 3  libwdr1-vl-3090.gw.unsw.edu.au (149.171.253.66)  2.296 ms  2.260 ms  2.112 ms
 4  ombcr1-te-4-5.gw.unsw.edu.au (149.171.255.77)  2.119 ms  2.155 ms  2.057 ms
 5  ombudnex1-po-2.gw.unsw.edu.au (149.171.255.170)  2.524 ms  2.475 ms  2.293 ms
 6  ufw1-ae-1-3154.gw.unsw.edu.au (149.171.253.36)  2.556 ms  2.655 ms  2.446 ms
 7  129.94.39.23 (129.94.39.23)  2.512 ms  2.717 ms  2.616 ms
 8  * * *
 9  * * *
10  * * *
11  * * *
12  * * *
13  * * *
14  * * *
15  * * *
16  * * *
17  * * *
18  * * *
19  * * *
20  * * *
21  * * *
22  * * *
23  * * *
24  * * *
25  * * *
26  * * *
27  * * *
28  * * *
29  * * *
30  * * *
31  * * *
32  * * *
33  * * *
34  * * *
35  * * *
36  * * *
37  * * *
38  * * *
39  * * *
40  * * *
```