---
title: "HomePod mini 2 gets the S10, Apple TV 4K the A19, and HomeView arrives with A18"
date: 2026-10-13
summary: "The first HomePod with a Neural Engine, an Apple TV that Apple says is up to 50% faster on CPU and 2x on GPU, and a new 7-inch smart display on the iPhone 16's A18."
categories:
  [
    "s10",
    "a19-6-5",
    "a18-6-5",
    "n1",
    "homepod/mini/2",
    "homepod/mini",
    "apple-tv/4k/4",
    "apple-tv/4k",
    "homeview/apple-homeview",
    "homeview/apple-homeview-wall-mounted",
  ]
---

Apple refreshed two home products and added a third today. The [HomePod mini 2](/devices/homepod/mini/2/) ships on 30 October with the [S10](/chips/s10/). The [Apple TV 4K (4th generation)](/devices/apple-tv/4k/4/) moves to the [A19](/chips/a19/base/6-5/), and a new product line, [Apple HomeView](/devices/homeview/), arrives on the [A18](/chips/a18/base/6-5/). The Apple TV and HomeView both ship on 13 November.

### HomePod mini 2: S5 to S10

The original [HomePod mini](/devices/homepod/mini/1/) ran the [S5](/chips/s5/) from Apple Watch Series 5. Its successor gets the S10, the SiP that Apple Watch Series 10 introduced and that the [Apple Watch SE 3](/devices/watch/se/3/) still uses.

|                    | CPU             | Neural Engine | Process | Released  |
| ------------------ | --------------- | ------------- | ------- | --------- |
| [S10](/chips/s10/) | 64-bit, 2 cores | 4 cores       | 4nm     | Sept 2024 |
| [S5](/chips/s5/)   | 64-bit, 2 cores | —             | 7nm     | Sept 2019 |

That makes the HomePod mini 2 **the first HomePod with a Neural Engine**. The S5, the [S7](/chips/s7/) in [HomePod (2nd generation)](/devices/homepod/base/2/) and the A8 in the original HomePod have none. Apple says Siri AI requests on HomePod mini 2 can run on-device or through Private Cloud Compute, and Siri AI itself is coming in a later software update.

It also follows a familiar pattern: HomePods get the Watch SiP that is on its way out, as the S5 and S7 did before it. The S10 arrives a month after the [S11 replaced it in Series 12 and Ultra 4](/news/apple-watch-series-12-and-ultra-4/). The HomePod mini 2's spec sheet lists Wi-Fi 6E and an Ultra Wideband chip without calling it second-generation, so it is recorded here with the [U1](/chips/u1/), like the original. It costs $149.

### Apple TV 4K: A15 Bionic to A19

Apple doesn't publish core counts for Apple TV, so the 4th generation is recorded with the [6c CPU / 5c GPU A19](/chips/a19/base/6-5/), the same chip as the [iPhone 17](/devices/iphone/17/base/) and the [Studio Display](/devices/display/studio-display/).

|                                   | CPU (P + E) | P-core clock | GPU     | Ray tracing | AV1 decode | Process |
| --------------------------------- | ----------- | ------------ | ------- | ----------- | ---------- | ------- |
| [A19 6c/5c](/chips/a19/base/6-5/) | 2 + 4       | 4.26 GHz     | 5 cores | Yes         | Yes        | 3nm     |
| [A15 6c/5c](/chips/a15/base/6-5/) | 2 + 4       | 3.24 GHz     | 5 cores | No          | No         | 5nm     |

Apple claims up to 50% faster CPU and up to 2x faster GPU performance than the previous generation. The "new, dedicated AV1 decoder" is the A19's own, so this is **the first Apple TV with hardware AV1 decode**. None of its A15, A12, A10X or earlier chips had one.

The new model is also the first device outside iPhone, iPad and Mac with the [N1](/chips/n1/), which brings Wi-Fi 7, Bluetooth 6 and Thread. Paired with four HomePod mini 2 or HomePod (2nd generation) speakers, it supports Apple's first four-speaker surround setup. It comes in one configuration for $299.

### Apple HomeView: a new product line on A18

HomeView is a 7-inch Liquid Retina smart display in two models: [Apple HomeView](/devices/homeview/apple-homeview/), a tabletop model on a speaker base, and [Apple HomeView (Wall Mounted)](/devices/homeview/apple-homeview-wall-mounted/). Both use the same A18 and both cost $299. HomeView is not on sale in the EU.

As with the Apple TV, it is recorded with the standard [6c CPU / 5c GPU A18](/chips/a18/base/6-5/) from the [iPhone 16](/devices/iphone/16/base/), which has a 16-core Neural Engine rated at 35 TOPS. Until now, the base A18 had only ever shipped in iPhones. Apple says both HomeView's A18 and the Apple TV's A19 run Secure Exclaves, which keep microphone data (and on HomeView, camera data) apart from the rest of the system.

HomeView lists Wi-Fi 6E rather than Wi-Fi 7, so like the HomePod mini 2 it has no N1, leaving the Apple TV the only one of today's three with it.

### Discontinued

The original [HomePod mini](/devices/homepod/mini/1/) and the [Apple TV 4K (3rd generation)](/devices/apple-tv/4k/3/) are both discontinued. That leaves **no [A15](/chips/a15/) device and no S5 device on sale**. The HomePod (2nd generation) carries on with the S7, and with the HomePod mini 2 it is one of the two speakers the new Apple TV can use for surround sound.
