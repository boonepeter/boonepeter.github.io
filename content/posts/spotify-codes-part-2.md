---
layout: post
title: "Spotify Codes - Part 2"
author: Peter Boone
tags: ["tech", "spotify", "python"]
date: 2020-11-19
draft: true
---

![spotify code with heights labeled](/imgs/spotify-2/spotify_track_6vQN2a9QSgWcm74KEZYfDL_labeled.png)

In [part 1](/posts/2020-11-10-spotify-codes) I dove into Spotify Codes and explained the general technical concepts of how they work. If you haven't read that post you should check it out. I [shared it to reddit](https://www.reddit.com/r/programming/comments/jvrpvj/how_spotify_codes_work/) where it generated a lot of interesting discussion. [Ludvig Strigeus](https://en.wikipedia.org/wiki/Ludvig_Strigeus), a key early developer at Spotify and the guy who invented Spotify Codes, even stopped by and shared some more information on their creation and the rational behind them!

At the end of part 1, I wrote that I wasn't sure about some of the details, so I was not able to implement my own barcode to URI converter. Thanks to a little more digging and a [lot of help from someone on Stack Overflow](https://stackoverflow.com/a/64950150/10703868) I can now do that conversion. If you want to just look at the code you can [find it here](https://github.com/boonepeter/boonepeter.github.io-code/tree/main/spotify-codes-part-2)

This article is going to be a little bit more technical than part 1 as I try to explain _exactly_ how Spotify encodes their barcodes. I will include some more resources if you want to keep learning.

## Media reference

The media reference integer is expressed as a 37 bit binary number.

```python
57639171874 -> 0100010011101111111100011101011010110
```

## CRC Calculation

The CRC-8 calculation uses the following generator:

```text
x^8 + x^2 + 1
```

The CRC is calculated by following these steps:

```text
Pad with 3 bits on the right:
01000100 11101111 11110001 11010110 10110000
Reverse bytes:
00100010 11110111 10001111 01101011 00001101
Calculate CRC as normal (highest order degree on the left):
-> 11001100
Reverse CRC:
-> 00110011
Invert check:
-> 11001100
Finally append to step 1 result:
01000100 11101111 11110001 11010110 10110110 01100
```

## Convolutional encoding

1. Convolutionally encode the 45 bits using the common generator
polynomials (1011011, 1111001) in binary with puncture pattern 
110110 (or 101, 110 on each stream). The result of step 2 is 
encoded using tail-biting, meaning we begin the shift register 
in the state of the last 6 bits of the 45 long input vector. 

  Prepend stream with last 6 bits of data:
  001100 01000100 11101111 11110001 11010110 10110110 01100
  Encode using first generator:
  (a) 100011100111110100110011110100000010001001011
  Encode using 2nd generator:
  (b) 110011100010110110110100101101011100110011011
  Interleave bits (abab...):
  11010000111111000010111011110011010011110001...
  1010111001110001000101011000010110000111001111
  Puncture every third bit:
  111000111100101111101110111001011100110000100100011100110011

4. Permute data by choosing indices 0, 7, 14, 21, 28, 35, 42, 49, 
56, 3, 10..., i.e. incrementing 7 modulo 60. (Note: unpermute by 
incrementing 43 mod 60).

  The encoded sequence after permuting is
  111100110001110101101000011110010110101100111111101000111000

5. The final step is to map back to bar lengths 0 to 7 using the
gray map (000,001,011,010,110,111,101,100). This gives the 20 bar 
encoding. As noted before, add three bars: short one on each end 
and a long one in the middle. 


> Shortly afterward, in a paper submitted in May 1968 [23], Jim Omura observed that the VA was
simply the __standard forward dynamic programming solution to maximum-likelihood decoding
of a discrete-time, finite-state dynamical system observed in memoryless noise__

[Convolutional codes](https://en.wikipedia.org/wiki/Convolutional_code) are a technique used to send redundant information to help

## Examples



uri: `spotify:track:1ykrctzPhcSS9GS3aHdtMt`

media ref: `26560102031`

`[0, 6, 6, 0, 7, 6, 0, 2, 2, 3, 1, 7, 0, 7, 6, 4, 6, 1, 4, 7, 4, 1, 0]`

![joker presents](../imgs/spotify-2/spotify_track_1ykrctzPhcSS9GS3aHdtMt.jpeg)


media ref: `67775490487`

uri: `spotify:user:jimmylavallin:playlist:2hXLRTDrNa4rG1XyM0ngT1`

`[0, 2, 6, 7, 1, 7, 0, 0, 0, 0, 4, 7, 1, 7, 3, 4, 2, 7, 5, 6, 5, 6, 0]`

![jimmy's stuff](../imgs/spotify-2/spotify_user_jimmylavallin_playlist_2hXLRTDrNa4rG1XyM0ngT1.jpeg)


media ref: `57639171874`

uri: `spotify:user:spotify:playlist:37i9dQZF1DWZq91oLsHZvy`

`[0, 5, 7, 4, 1, 4, 6, 6, 0, 2, 4, 7, 3, 4, 6, 7, 5, 5, 6, 0, 5, 0, 0]`

![indie kicks](../imgs/spotify-2/spotify_user_spotify_playlist_37i9dQZF1DWZq91oLsHZvy.jpeg)


## Media references

<iframe src="https://open.spotify.com/embed/playlist/1QmDaiCAo3YmH23UQyV0FN" width="300" height="380" frameborder="0" allowtransparency="true" allow="encrypted-media"></iframe>

References:

## Further Reading

### CRCs

[CRC Rocksoft](https://zlib.net/crc_v3.txt)

### Convolutional codes

[High-Rate Punctured Convolutional Codes for Soft Decision Viterbi Decoding](https://doi.org/10.1109/TCOM.1984.1096047)

[Viterbi Decoding of Convolutional Codes (MIT handout)](http://web.mit.edu/6.02/www/s2012/handouts/8.pdf)

[Virterbi Decoder](https://doi.org/10.1109/TIT.1967.1054010)


[Punctured Convolutional Encoding](https://www.mathworks.com/help/comm/ug/punctured-convolutional-encoding-in-simulink.html)

[Performance of concatenated codes for deep space missions](https://ipnpr.jpl.nasa.gov/progress_report/42-63/63H.PDF)

[Convolutional Codes and State Machine View/Trellis (MIT Slides)](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-02-introduction-to-eecs-ii-digital-communication-systems-fall-2012/lecture-slides/MIT6_02F12_lec06.pdf)
