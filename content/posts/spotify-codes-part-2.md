---
layout: post
title: "Spotify Codes - Part 2"
author: Peter Boone
tags: ["tech", "spotify", "python"]
date: 2020-11-19
draft: true
---

![spotify code with heights labeled](/imgs/spotify-2/spotify_track_6vQN2a9QSgWcm74KEZYfDL_labeled.png)

I recently wrote [a post](/posts/2020-11-10-spotify-codes) about the technical details behind Spotify Codes which I [shared to reddit](https://www.reddit.com/r/programming/comments/jvrpvj/how_spotify_codes_work/). The response was quite surprising! [Ludvig Strigeus](https://en.wikipedia.org/wiki/Ludvig_Strigeus) (a key early developer at Spotify and the inventor of Spotify Codes) even stopped by and shared some more information on their creation and the rational behind them!

You should go read that previous post if you haven't already. Towards the end of the post, I metaphorically waved my hands and said "I don't know exactly how Spotify encodes the media references into barcodes". I knew there was some forward error correction involved, but I really didn't know much about [convolutional codes][1] and I didn't try to guess what Spotify used.

This article is going to be a little bit more technical than the last as I try to explain _exactly_ how Spotify encodes their barcodes.

## Convolutional codes

When you send data to someone, sometimes that data can get corrupted. Let's say I want to send you the following data:

```python
101010
```

I could repeat every bit twice and send that to you:

```python
111 000 111 000 111 000
```

Then, if some of the bits got distorted and flipped during transmission, you could still guess what the message was meant to be. You could examine every triplet and assume that the majority represented the correct bit.

```python
 _    _         _     _
101 001 111 000 011 001
 1   0   1   0   1   0
```

That is a very rudimentary convolutional code.




We can also represent that code like this:

```python
    [1, 1, 1]
    [0, 0, 0]
    [1, 1, 1]
    [0, 0, 0]
```

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

Virterbi Decoder: https://doi.org/10.1109/TIT.1967.1054010

High-Rate Punctured Convolutional Codes for Soft Decision Viterbi Decoding https://doi.org/10.1109/TCOM.1984.1096047

https://www.reddit.com/r/programming/comments/jvrpvj/how_spotify_codes_work/

http://web.mit.edu/6.02/www/s2012/handouts/8.pdf

https://www.mathworks.com/help/comm/ug/punctured-convolutional-encoding-in-simulink.html

Performance of concatenated codes for deep space missions: https://ipnpr.jpl.nasa.gov/progress_report/42-63/63H.PDF

https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-02-introduction-to-eecs-ii-digital-communication-systems-fall-2012/lecture-slides/MIT6_02F12_lec06.pdf

[1]: https://en.wikipedia.org/wiki/Convolutional_code "Convolutional codes"

Further reading:

https://en.wikipedia.org/wiki/Turbo_code

[Source](https://github.com/boonepeter/boonepeter.github.io/tree/master/content/posts/2020-11-10-spotify-codes.md)