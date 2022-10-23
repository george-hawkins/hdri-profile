Duplicating Greg Zaal's HDRI profile
====================================

The following page describes trying to create an identical RawTherapee profile to the one Greg Zaal uses for his HDRIs.

RawTherapee UI
--------------

The RawTherapee UI is a little unusual, bits of the panels that make up the _Profile_ settings UI are greyed out when not applicable - so, that might lead you to believe that if something isn't greyed out then changing it is meaningful.

However, there are two clear cases where making changes is ignored (and any changes you make will not be saved if you save the profile):

* If you haven't first opened an image.
* If you're in a section where the section header has a little power button, and it's not active:

![img.png](images/power-off.png)

Above the power button is inactive so, any changes you make to _Radius_ etc. will be ignored unless you toggle the power button to active:

![img.png](images/power-on.png)

The HDRI profile
----------------

In his Poly Haven article on ["How to Create High Quality HDR Environments"](https://blog.polyhaven.com/how-to-create-high-quality-hdri/), Greg Zaal links to the profile he uses with RawTherapee.

I wanted to see how one would create the same profile directly in RawTherapee.

If you download and look at Greg's profile, you see:

```
[Version]
AppVersion=5.4
Version=331
```

So, the profile was created with version 5.4 of RawTherapee which you can download [here](https://www.rawtherapee.com/downloads/5.4/).

Lensfun DB location
-------------------

For whatever reason, the Linux AppImage for RawTherapee 5.4 sets the location of the Lensfun DB incorrectly when creating `~/.config/RawTherapee/options`. As a result it can't auto-detect and cameras or lenses from the EXIF data contained in images.

To fix this exit RawTherapee and edit `~/.config/RawTherapee/options`. Search for:

```
[Lensfun]
DBDirectory=
```

And replace it with:

```
[Lensfun]
DBDirectory=share/lensfun/version_1
```

Thanks to ronanM's comment [here](https://github.com/Beep6581/RawTherapee/issues/5622#issuecomment-583722775) for the necessary clue.

Note: when the AppImage starts up, you'll see something like the following in the console output:

```
LD_LIBRARY_PATH: /tmp/tmp.DLVZKDgcHE:/usr/lib:
/tmp/.mount_RawThe4FWiPw/usr/lib/librsvg-2.so
```

The `/tmp/tmp.DLVZKDgcHE` just seems to contain libraries but the `/tmp/.mount_RawThe4FWiPw` contains an entire file system. If you `cd` to this directory, you can then find the `DBDirectory` referenced above:

```
$ cd /tmp/.mount_RawThe4FWiPw
$ ls usr/share/lensfun/version_1
6x6.xml  compact-nikon.xml  contax.xml  mil-samsung.xml  slr-canon.xml  ...
```

For whatever reason, the `usr` portion is implicit when specifying the `DBDirectory` value.

Sample raw image
----------------

As noted above, we need to have an image open or else any profile changes will be ignored. From Greg's `HDRI.pp3` we can see that he used:

```
LFCameraModel=Canon EOS 600D
LFLens=Canon Canon EF-S 10-18mm f/4.5-5.6 IS STM
...
W=3457
H=5194
```

So to reproduce his `.pp3` file exactly, you'd need a photo taken with an identical setup and with the same image size. The nearest suitable image that I could find online was the Canon EOS 1200D `.CR2` file that you can find on the RawSamples Canon [page](https://www.rawsamples.ch/index.php/en/canon).

Note: there is a sample on this page for the 600D but it was taken with a lens that RawTherapee 5.4 cannot auto-detect (and the image is not large enough). The 1200D image is a suitable size and the camera and lens can both be auto-detected by RawTherapee 5.4.

Neutral profile
---------------
Start RawTherapee and open your `.CR2` sample image. Then you can create a base profile from which to work by going to the _Editor_ tab (center left) and the going to the _Processing Profiles_ panel (top right) and selecting _(Neutral)_ from the dropdown list.

![img.png](images/rawtherapee-5.4-neutral.png)

If you then select the _Save_ icon to right of the dropdown and save the profile as "neutral", you then have something to compare with:

```
$ cp ~/.config/RawTherapee/profiles/neutral.pp3 .
$ vimdiff neutral.pp3 HDRI.pp3
```

So looking at the diff, let's go through the differences between RawTherapee's neutral profile and Greg's HDRI profile such that we eventually build up an identical profile.

**Update:** it turns out that there are actually very few _real_ changes (there are various changes that don't actually take effect as the section they're in were enabled and later disabled again). And to the remaining _real_ changes, only a few of them are probably really relevant, e.g. the dead-pixel detection probably isn't make or break (unless it fixes pixels stuck high such that hey introduce rays of infinitely bright light). One can completely ignore changes in sections where `Enabled=false`.

### The Preview area

The central area containing the opened image is called the _Preview_ area.

* &#x274c; Click the _Rotate left_ icon once (it ends up as the `Rotate` value in the `Coarse Transformation` section of the profile).

This looks a little stupid with our sample image that's already correctly oriented, but we're trying to completely replicate Greg's profile.

### The Processing Profile tabs

![tabs](images/tabs.png)

### The Exposure tab

* &#x2705; Tick _Highlight reconstruction_ (this changes way more settings than just `HLRecovery` and gets us much closer to Greg's profile).
* &#x2705; Go to the _L\*a\*b\* Adjustments_ section and enable its power icon (this corresponds to the `Luminance Curve` section in the profile file).
* &#x274c; Go to the _Tone Mapping_ section, enable its power button, and change _Scale_ to 0.3.

### The Details tab

* &#x2753; Go to the _Noise Reduction_ section and enable its power icon.
  * In the _Chromiance_ subsection, change _Method_ to _Manual_ and change _Master_ to 4, _Red-Green_ to 3.6 and _Blue-Yellow_ to -1.9. And switch _Method_ back to _Automatic global_. This is another group of setting that I suspect is **redundant** because ultimately Greg switched back to _Automatic global_.
* &#x274c; Go to the _Local Contrast_ section, enable its power button, and change _Radius_ to 40 and _Amount_ to 0.

### The Color tab

* &#x2705; Go to the _Channel Mixer_ section and enable its power icon.
* &#x2705; In the _White Balance_ section, set _Method_ to _Custom_ and change _Temperature_ to 5400 and _Tint_ to 1.1.
* &#x2705; In the _Output Profile_ subsection of the _Color Management_ section, untick _Black Point Compensation_.
* &#x2705; Go to the _HSV Equalizer_ section and enable its power icon.
* &#x2705; Go to the _RGB Curves_ section and enable its power icon.

### The Transform tab

Go to the _Profiled Lens Correction_ section. As the sample image is one of the known camera and lens combinations, the _Auto-matched correction parameters_ should be automatically selected. After confirming this:

* &#x2705; Untick _Distortion correction_ and _Vignetting correction_.
* &#x2705; Tick _Chromatic aberration  correction_.

When we come to creating a profile that's suitable for the One X2, I think leaving this as _None_ is probably appropriate as the One X2 knows its own lens characteristics very well and takes them into account when stitching

* &#x274c; Go to the _Crop_ section, enable its power icon, and enter 3457 for _Width_, 5194 for _Height_ and 3:2 for _Ratio_.
* &#x274c; Go to the _Resize_ section, enable its power icon, and change _Method_ to _Nearest_, change _Specify_ to _Scale_ and then the _Scale_ value to 1.

### The RAW tab

* &#x2705; In the _Preprocessing_ section, tick _Hot pixel filter_ and _Dead pixed filter_.

Save and disable
----------------

Save all the changes in a new profile called "my-hdri".

As noted above, some settings are only there because the section they're in was enabled, the profile was saved and the section later disabled and the profile saved again (meaning the section became marked `Enabled=false` but the changes within the section were preserved). This means changing such settings was pointless (those marked with &#x274c; above) and only included here for completeness.

So:

* Go to the _Exposure_ tab and disable the _Tone Mapping_ section.
* Go to the _Details_ tab and disable the _Local Contrast_ section.
* Go to the _Transform_ tab and disable the _Crop_ and _Resize_ sections.

Now, save the updated profile as "my-hdri-final".

Remaining differences
---------------------

If you diff the resulting `my-hdri-final.pp3` files with Greg's `HDRI.pp3` the remaining differences should be just:

**TODO**

In the _Color_ tab:

* Go to the _Color Toning_ section and enable its power icon, this toggles both the sections `Enabled=true` and sets `SaturatedOpacity=1` (its `0` by default). When you toggle the power icon off `SaturatedOpacity` and `SatProtectionThreshold` get set to apparently random but irrelevant values (as the section is now again disabled). I couldn't find a way, to disable the section and leave the `SaturatedOpacity` set to `1` so, don't try this bit even if you're trying to exactly match Greg's profile.
