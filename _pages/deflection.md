---
permalink: /deflection/
title: "Deflection"
excerpt: "A game of ambient flow."
share: true
header:
  image: mm-deflection-feature.png
deflection_gallery:
  - url: mm-deflection-level-eddies.png
    image_path: mm-deflection-level-eddies.png
    alt: "gameplay level eddies"
  - url: mm-deflection-level-windmill.png
    image_path: mm-deflection-level-windmill.png
    alt: "gameplay level windmill"
  - url: mm-deflection-level-whirlpool.png
    image_path: mm-deflection-level-whirlpool.png
    alt: "gameplay level whirlpool"
modified: 2019-06-12T13:00:00-04:00
---

{% include base_path %}

![image-right]({{ site.url }}{{ site.baseurl }}/images/mm-deflection-icon-200.png){: .align-right}

Deflection is a visually immersive game of ambient flow in which you can try to level up in pursuit of enlightenment, or simply enjoy the moment. While leveling up gives you access to wisdom of the ages, leveling down provides somewhat more practical and earthly advice. Game play is simple; draw lines that serve as mirrors and/or lenses to direct blue sparks towards a red portal while preventing any sparks from striking a blue portal.

(Humble-brag) This is my first effort at creating a game. It's just a hobby at this point; something I work on in my spare time. And although it took a couple years to nail it, I'm quite pleased with the result...

{% include gallery id="deflection_gallery" caption="Gameplay screen captures of levels `eddies`, `windmill`, and `whirlpool`." %}

It's currently only available for Android. If there's demand, it would not be too much effort for me to port it to iOS. But I have a number of other game ideas I'm eager to get at. You can get it from [Google Play](https://play.google.com/store/apps/details?id=com.timberdig.deflection.android) (recommended) or you can download the APK [from here]({{ site.url }}{{ site.baseurl }}/assets/downloads/deflection.apk) and manually install it. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/Bi5I3TZQXlc" frameborder="0" allowfullscreen></iframe>

## Nerd Out

For those interested in some of the technical aspects, Deflection was developed in Java using [libgdx](https://libgdx.badlogicgames.com/), [box2d](http://box2d.org/), [ashley](https://github.com/libgdx/ashley), [universal tween engine](http://www.aurelienribon.com/blog/projects/universal-tween-engine/), and a number of other libraries. It's currently not making use of in-app payments, but the infrastructure is there, and I'm keeping open the option.

The signature at the start of the game serves as an asset loading progress indicator. It consists of several splines and uses a single alpha value to determine how much of each spline to draw.

Glow pulsing and particle emissions are pre-scripted in choreography to the music. Pre-scripting is a long process...

 * Use [Sonic Visualizer](http://www.sonicvisualiser.org/) to identify note onsets and durations.
 * Export to csv.
 * Merge several csv files and tweak in a spreadsheet to generate JSON syntax for "music events".
 * Create JSON file.
 * Pass JSON file through [Google Flatbuffers](https://google.github.io/flatbuffers/) utility to serialize events to a binary file.

Flatbuffers has a steep learning curve, but it is much faster and more memory efficient than reading and interpreting JSON.

I originally tried to implement the glowing lines using [Box2DLights](https://github.com/libgdx/box2dlights). You can see my contribution of ChainLight to that project. Box2D turned out to be a little too heavy for my needs, so now it's used only for ray-casting. Even then, the normals obtained from it are segmented due to flat edges of fixture geometry, resulting in uneven reflections. So after each ray intersection, I have code performing an additional parametric intersection calculation for more precise normals. The result is much more beautiful. I might have been able to get similar results by using SLERP interpolations, but by the time I realized that, what I had was already working.

The glow is implemented using a sprite with a one or two-pixel wide white to transparent gradient texture region. The animated flare texture region is white to transparent cloud/noise that is tesselated and repeats once in both U and V directions. I don't have the code in front of me, but to draw each segment I use something like SpriteBatch.drawSprite(), which allows passing in non-rectangular (isosceles trapezoid) sprite coordinates for curve segments. The duplicated pattern in the flare texture region allows me to offset the U and V coordinates all the way to the center of the region (where the pattern repeats) without displaying beyond the region's edge. To get the continuous flow effect, I steadily increase the V offset. When it gets past the center I subtract regionHeight/2 from the offset. drawSprite() is not the most efficient way to do this, because vertex data passed to gl gets duplicated. But it's good enough for my needs.

[Deflection Privacy Policy]({% page_url deflection-privacy-policy %})

{% comment %}

## Notable Features

- bullet point 1
- bullet point 2

{% endcomment %}

