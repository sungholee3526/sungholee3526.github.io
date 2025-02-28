---
layout: post
title: "Fixing Media Keys on Mac"
date: 2018-01-08 00:00:00 +0900
---

Back in the days when Sierra hadn't reached its "high" yet, if you pressed the
play/pause key on your keyboard, it only affected the native macOS media apps.
But since macOS High Sierra, if any webpage with multimedia content is open,
macOS forwards the media key inputs to Safari.

I didn’t like this behavior. As someone who listens to music a lot, I pause the
music when I encounter a webpage with audio or video, and resume the music I was
listening to when I’m done with the web content. Naturally, I found myself
repeatedly opening iTunes again and clicking the play button every time Safari
took over the media key inputs. Instead, I wanted my media keys to exclusively
control iTunes.

Fortunately, I wasn’t alone. A project called
[High Sierra Media Key Enabler](https://github.com/milgra/macmediakeyforwarder)
was active in the open-source community. I downloaded and installed the app and
the play/pause key began to work as I intended — only controlling iTunes.
However, I soon discovered a problem with the app.

![Screenshot of High Sierra Media Key Enabler]({{ site.baseurl }}/assets/img/fixing-media-keys-on-mac/hsmke.png)

## 🐛 Bug found!

The play/pause key functioned as desired but I couldn't skip or rewind songs. It
became apparent other users were experiencing the same problem, and an
[issue](https://github.com/milgra/macmediakeyforwarder/issues/7) was already
open on GitHub. One detail stood out to me after I read through the discussion.
People using an external keyboard were having the problem. It was a good thing
because I had an idea after realizing this.

I vaguely recalled seeing a key code table where the key codes for
fast-forwarding/rewinding and next/previous track were defined separately. It
occurred to me that perhaps the MacBook's internal keyboard uses one set of
these codes, while some external keyboards use the other.

## Let's debug & fix

I cloned the repository and began examining the code. I located the section
where the app branches based on the key code it received. And sure enough, the
app was only processing one set of the aforementioned key codes.

<!-- prettier-ignore-start -->
{% highlight objectivec %}
iTunesApplication* iTunes = [SBApplication applicationWithBundleIdentifier:@"com.apple.iTunes"];
switch (keyCode) {
    case NX_KEYTYPE_PLAY:[iTunes playpause];break;
    case NX_KEYTYPE_FAST:[iTunes nextTrack];break;
    case NX_KEYTYPE_REWIND:[iTunes backTrack];break;
}
{% endhighlight %}
<!-- prettier-ignore-end -->

On the other hand, IOKit of macOS defines both sets of track control key codes.

<!-- prettier-ignore-start -->
{% highlight objectivec %}
#define NX_KEYTYPE_PLAY			16
#define NX_KEYTYPE_NEXT			17
#define NX_KEYTYPE_PREVIOUS		18
#define NX_KEYTYPE_FAST			19
#define NX_KEYTYPE_REWIND		20
{% endhighlight %}
<!-- prettier-ignore-end -->

I confirmed my theory by setting a breakpoint at the switch statement and
inspecting the value of `keyCode`. When I pressed the track control keys on my
keyboard, the values corresponded to the other set of track control key codes,
not the ones the app was capturing.

I added the missing key codes to the code and tested again.

<!-- prettier-ignore-start -->
{% highlight objectivec %}
iTunesApplication* iTunes = [SBApplication applicationWithBundleIdentifier:@"com.apple.iTunes"];
switch (keyCode) {
    // Play/pause
    case NX_KEYTYPE_PLAY:[iTunes playpause];break;

    // Fast forward
    case NX_KEYTYPE_NEXT:
    case NX_KEYTYPE_FAST:[iTunes nextTrack];break;

    // Rewind
    case NX_KEYTYPE_PREVIOUS:
    case NX_KEYTYPE_REWIND:[iTunes backTrack];break;
}
{% endhighlight %}
<!-- prettier-ignore-end -->

Et voila! The app works with my keyboard! I submitted a
[pull request](https://github.com/milgra/macmediakeyforwarder/pull/17) and it
was accepted by the project owner. Subsequently, I noticed a
[user reporting](https://github.com/milgra/macmediakeyforwarder/issues/7#issuecomment-360548098)
that the issue was resolved for them as well.

## A little flex

_This section was added later time._

The High Sierra Media Key Enabler repository was included in the
[2020 GitHub Archive Program](https://archiveprogram.github.com/arctic-vault/).
It's just a few lines of code but I'm proud that my code is preserved in the
Arctic. Hopefully, the survivors of a societal collapse won't have problems
switching songs on their Macs.

## Notes

- The project name has been changed to "Mac Media Key Enabler".
- My former GitHub username was "sh1217sh." If you encounter this username in
  the project's README or issue thread, that refers to me.
