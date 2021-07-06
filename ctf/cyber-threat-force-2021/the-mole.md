---
title: The Mole - CyberThreatForce CTF 2021
---

# The Mole
## CyberThreatForce CTF 2021
#### Category: Network
#### 3 Jul 2021

description:

> Hello CTF-Agent,
We got, thanks to one of our contact, the acces on one of the APT-403's computer.
Your mission will be to decrypt the encrypted communication.
We trust you.
H.Q.

The challenge comes with two pcap files - each containing a single TCP session.

If we open the first pcap in wireshark and follow TCP stream on the only session, we'll see a user running shell commands to list some files. One of these is an RSA private key, which I copied to a file.

![tcp session](images/the-mole/pcap-1.png)

```
-----BEGIN RSA PRIVATE KEY-----
MIIBOQIBAAJAdA6LYrTnWaPJ5DSg03uaDH3biN4jfnH1mEqFuZILfHov4/SEdEqu
Gj5/gJ3ypGTbgWUFe9VQGY0LnXN3j6773wIDAQABAkBui0wJAPc8Ut6DF/34cssR
CvCJNc3pKvMb1B/72jhGn24mgbyHyNdxK7xaY0ultnQ/3hmFuT0i/UWXdfGlOTEB
AiEAzqTk8qn/OZ9dfGOUd27JZJxucxvyB9TlY9xH5mMxS1ECIQCPxsE2N5avydWj
UnMhecYCGuwqfTTt7kMH9fGO5aWoLwIgf/2CEQtaGcarkK/c9VyZQMfjYUid0Fv8
+K0nm3s0vQECIFt+Rqvi2hCJp1skd8GAxaHHUiyDuvACZEOnng2qVC3fAiEAkjsZ
bWWZnzPMxFCUq54kXCyqHom1XRAKClNFchnNORA=
-----END RSA PRIVATE KEY-----
```

The second pcap only contains these base64 lines which I also copied to a file.

```
CPrYwDhZMg+jxAWhK3Tu2LCzDPKteChdeXkO72T7BHhpPRtqbXHEcyvmhsmsPyHDjhflEV/BQwD42kYeO9EkfQ==
BfwWKiRIDj5VzWRYck6NUQdF52thWaeLwAdwHXvM/vVd5tZGF0YVWslOr9u/CyZQd3N552Mxtp4o9CaBaD657Q==
YuqH7THelhN0rNwYmBak+bZv20sqTX10O1Kmpc6ncmt91kYm1jxLLsQa157IBqbaSRI205D0/2LLPnoAkgCMGg==
LBKjeMcgIYaQx5cUX9JBQTab6bqFTP7dzTcRMcAy6jle64Q22Zr+JbwwJTvWlhZKT82pbGU/Ux/o2+L5bghhSA==
WHYuGXboak1lY2lKXsrjrurvNcsdbCebk8ebUc4e3eBuoAxlOoHfheYHzKfiC4bibBUmS+YLn5eA/kg/OWkh5w==
DPTFnoyKxcMl4DsfNlKL49msE8Sp7hgbIP1v9P3fe071BEMAl2VgxqCez2R2XvCZo7u8VAJFFmNNTfiO+MJdMw==
```

Then I wrote some bash to base64 decode each line and split them into their own files.

```bash
i=0
for line in `cat base64-data`; do
    echo -n $line | base64 -d > data$i
    i=$(expr $i + 1)
done
```

The split files now contained the decoded, encrypted data.

```
$ ls data*
data0  data1  data2  data3  data4  data5

$ hexdump -C data0
00000000  08 fa d8 c0 38 59 32 0f  a3 c4 05 a1 2b 74 ee d8  |....8Y2.....+t..|
00000010  b0 b3 0c f2 ad 78 28 5d  79 79 0e ef 64 fb 04 78  |.....x(]yy..d..x|
00000020  69 3d 1b 6a 6d 71 c4 73  2b e6 86 c9 ac 3f 21 c3  |i=.jmq.s+....?!.|
00000030  8e 17 e5 11 5f c1 43 00  f8 da 46 1e 3b d1 24 7d  |...._.C...F.;.$}|
```

Then I looped over each file and decrypted them with `openssl`.

```bash
for file in `ls data*`; do
    openssl rsautl -decrypt -inkey key -in $file
done
```

> Cyberthreatforce is in.Hurry up we have to kick them out.Please take this code.CYBERTF{D3c0d3_TcP_FL0w}Log out now !We have to be fast.
