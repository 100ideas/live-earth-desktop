# live-earth-desktop

## INSTALL LOG (100ideas)

using system default python2.7, osx 11.6

**install listed and unlisted deps**
```shell
# get pip
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py | python

# install deps to global python2 location... gross
python -m pip install requests lxml html5lib Pillow pytz tzlocal bs4

# need opencv (for cv2 python package, see details below)
brew install opencv@2
python -m pip install scikit-image
```
*note*: see full shell log if I forgot one of the packages that slipped by... `./CMD_LOG 2022-04-13_python_setup.log`

**ok.... now we need cv2 python package**
... its pythong opencv bindings... so we need opencv that works w python2.7...
    https://gist.github.com/shinsumicco/52bfcabeccb0290348feee48cd5dcdb9

... the gist suggests an involved compile from src w/ flags for oldy python...

... **gonna just try `brew install opencv@2` instead**...
```shell 
brew install opencv@2
python -m pip pip install opencv-python==4.2.0.32
```

edit output paths in `goes-east.py` as described by original readme below, then test install with
`python goes-east.py`

consider putting these requirements in a `requirements.txt` for pip... see here
- https://pip.pypa.io/en/stable/user_guide/#requirements-files
- ok generated it

```shell
# generate requirements file
~/dev/etc/willwhitney-live-earth-desktop ❯ python -m pip freeze --all > requirements.txt

# then install from it in another environment.
python -m pip install -r requirements.txt
```

on my system the script worked (eventually), but the system complained a lot on stderr about dynamic linking going on in qtkit libs (ughhh why do we even install qtkit for the love of god). It was bothering, found a fix that works for my system, at least, and placates the system linker that was complaining on security grounds...

fixing dyld warnings, backround reading:
- https://medium.com/@donblas/fun-with-rpath-otool-and-install-name-tool-e3e41ae86172
- https://itwenty.me/2020/07/understanding-dyld-executable_path-loader_path-and-rpath/

```
# shell log in `CMD_LOG - patch Qt binaries to avoid using relative rpath in dynamic linker.log`

~/dev ❯ for pa in QtGui QtCore QtTest QtWidgets QtConcurrent
              echo $pa # install_name_tool -rpath '@executable_path/../Frameworks' /Users/100ideas/Library/Python/2.7/lib/python/site-packages/cv2/.dylibs/ /Users/100ideas/Library/Python/2.7/lib/python/site-packages/cv2/.dylibs/$pa
        end
QtGui
QtCore
QtTest
QtWidgets
QtConcurrent

~/dev ❯ for pa in QtGui QtCore QtTest QtConcurrent QtWidgets; install_name_tool -rpath '@executable_path/../Frameworks' /Users/100ideas/Library/Python/2.7/lib/python/site-packages/cv2/.dylibs/ /Users/100ideas/Library/Python/2.7/lib/python/site-packages/cv2/.dylibs/$pa; end
error: /Library/Developer/CommandLineTools/usr/bin/install_name_tool: no LC_RPATH load command with path: @executable_path/../Frameworks found in: /Users/100ideas/Library/Python/2.7/lib/python/site-packages/cv2/.dylibs/QtGui (for architecture x86_64), required for specified option "-rpath @executable_path/../Frameworks /Users/100ideas/Library/Python/2.7/lib/python/site-packages/cv2/.dylibs/"
error: /Library/Developer/CommandLineTools/usr/bin/install_name_tool: no LC_RPATH load command with path: @executable_path/../Frameworks found in: /Users/100ideas/Library/Python/2.7/lib/python/site-packages/cv2/.dylibs/QtCore (for architecture x86_64), required for specified option "-rpath @executable_path/../Frameworks /Users/100ideas/Library/Python/2.7/lib/python/site-packages/cv2/.dylibs/"
error: /Library/Developer/CommandLineTools/usr/bin/install_name_tool: no LC_RPATH load command with path: @executable_path/../Frameworks found in: /Users/100ideas/Library/Python/2.7/lib/python/site-packages/cv2/.dylibs/QtTest (for architecture x86_64), required for specified option "-rpath @executable_path/../Frameworks /Users/100ideas/Library/Python/2.7/lib/python/site-packages/cv2/.dylibs/"
error: /Library/Developer/CommandLineTools/usr/bin/install_name_tool: no LC_RPATH load command with path: @executable_path/../Frameworks found in: /Users/100ideas/Library/Python/2.7/lib/python/site-packages/cv2/.dylibs/QtConcurrent (for architecture x86_64), required for specified option "-rpath @executable_path/../Frameworks /Users/100ideas/Library/Python/2.7/lib/python/site-packages/cv2/.dylibs/"

# now try python goes-east.py and check `err.log` to see if complaints have been silenced
```


## TODOs 
- switch to python3
  - tidy up install process and use virtualenv 
  - ditch/replace bloat deps like cv2 (opencv for jpg stitching and saving ugh)
- or deno/node 
  - https://github.com/jeremiak/noaa-goes-east-image-scraper/blob/master/scrape.js
- use apple automations / applescript / or some system api to directly control screensaver / background
- refactor hardcoded config strings (paths) into config vars
- maybe port over to serverless architecture + s3, provide proxy?
- 

---

**UPDATE: GOES-East now supported instead of Himawari.** Use https://github.com/jakiestfu/himawari.js for Himawari images instead of this. Something has changed about where the Himawari images are saved, and I don't want to spend time tracking it down when there's a perfectly great package already. You can use his script via `launchctl` the same way you can this one in order to get continually-updated images for your desktop. GOES-East is still supported in this package.

There's a satellite called Himawari-8 which is geostationary over approximately Papua New Guinea. The very excellent people who run this satellite have set up a [live stream](http://himawari8.nict.go.jp/) of the ultra-high-res images that it takes. They are gorgeous.

Similarly, there's another satellite called GOES-East. It's above South America at the equator, and its photos are just as amazing as the Himawari ones. Because the NOAA is excellent people too, they also have [live images available](https://www.star.nesdis.noaa.gov/GOES/GOES16_FullDisk.php).

Inspired by [someone awesome on Reddit](https://www.reddit.com/r/programming/comments/441do9/i_made_a_windows_powershell_script_that_puts_a/), and based on a script by [celoyd](https://github.com/celoyd), I built a script that downloads the latest photo. With a `plist` file for `launchd` on OS X, I can run this script every ten minutes and always have the latest image on my machine. And then by setting my OS X desktop to a slideshow of the images inside a folder, the latest Himawari-8 or GOES-East photo is always set as my desktop image.

![](himawari-example.png)
![](goes-example.png)

## Instructions

1. Clone this repo
2. `pip install Pillow requests pytz tzlocal`
1. Pick whether you want images of Australia and Southeast Asia (Himawari) or the Americas (GOES-East).
3. Change the paths set in `tmp` and `out` and `os.system("rm ...")` in `himawari.py` or `goes-east.py` and those in `himawari.plist` or `goes-east.plist` to paths inside this directory.
4. Try the Python script by running `python himawari.py` or `python goes-east.py` just to make sure everything's kosher. It should download an image.
5. `ln -s <this-dir>/himawari.plist /Users/<you>/Library/LaunchAgents/` or `ln -s <this-dir>/goes-east.plist /Users/<you>/Library/LaunchAgents/`
6. `launchctl load -w /Users/<you>/Library/LaunchAgents/himawari.plist` or `launchctl /Users/<you>/Library/LaunchAgents/goes-east.plist` to start it running every 10 minutes
7. Go to OS X Preferences > Desktop and Screen Saver and set your desktop to rotate through the images contained in the `images` directory that you're writing these images to (whatever directory you made `out` point to).
8. Enjoy!
