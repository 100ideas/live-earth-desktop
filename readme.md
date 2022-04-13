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
  - prior art:
    - https://github.com/jeremiak/noaa-goes-east-image-scraper/blob/master/scrape.js
    - https://github.com/maxogden/goes-16-cira-geocolor/blob/master/run.sh
- or this mature-looking GOES scraper written in kotlin / java
  - https://github.com/n9Mtq4/NOAA-GOES-Image-Scraper
- use apple automations / applescript / or some system api to directly control screensaver / background
- refactor hardcoded config strings (paths) into config vars
- maybe port over to serverless architecture + s3, provide proxy?
- fix up goes-movie.py


## feature: movie mode, *animate-earth* codebase w/ "morphing" between frames

I made this comment back in 2017 on maxogdens GOES scraper repo (https://github.com/maxogden/goes-16-cira-geocolor/issues/2). Unfortunately animate-earth codebase is 7 years old now...

> The following project may be of some interest: @dandelany[/animate-earth](https://github.com/dandelany/animate-earth)
> 
> ![](https://camo.githubusercontent.com/d13dc1418d1ebacd0e070c20e69b77b6a40d300a/687474703a2f2f692e696d6775722e636f6d2f5452454d7a684e2e676966)
> 
> > [animate-earth](https://github.com/dandelany/animate-earth) is an experiment in applying optical-flow-based motion interpolation to processed RGB satellite imagery from Himawari-8's AHI imager, with the aim of regularly producing high-quality smoothed videos from these images, and sharing them with the world. [The final products can be found on this Youtube channel](https://www.youtube.com/embed/8knZ2-cys6M?list=PLrmCQL5hELCx-9utf0HXyXuBuc1IIPIJQ), updated nearly daily.
> > 
> > Motion interpolation is accomplished with the open source [Butterflow tool by dthpham](https://github.com/dthpham/butterflow)), which uses [OpenCV’s implementation](http://docs.opencv.org/master/d7/d8b/tutorial_py_lucas_kanade.html#gsc.tab=0) of an algorithm devised by Gunnar Farnebäck.
> >
> >  > Farnebäck, Gunnar. "Two-frame motion estimation based on polynomial expansion." Image analysis (2003): 363-370. [archive.org pdf](https://web.archive.org/web/20170829054040/http://www.diva-portal.org/smash/get/diva2:273847/FULLTEXT01.pdf).

also intersting: stabilizing location of the terminator in the frame over a very long timelapse (month+): https://www.youtube.com/shorts/6GaQAitlH4M

Will Bresnahan (https://github.com/n9Mtq4)has shared some even smoother timelaps animations from goes full-disk images: https://www.youtube.com/watch?v=HEgQXcmLj-E. He wrote a mature-looking scraper in kotlin. https://github.com/n9Mtq4/NOAA-GOES-Image-Scraper

**very interesting**: https://github.com/celoyd/hi8 - repo that generates "https://glittering.blue" movies from himawari-8 raw data.

check out
- original repo https://github.com/celoyd/hi8
- https://github.com/n9Mtq4/NOAA-GOES-Image-Scraper
- https://github.com/100ideas/live-earth-desktop
- https://github.com/dandelany/animate-earth
- https://github.com/dthpham/butterflow

interesting people
- https://github.com/celoyd
- https://github.com/n9Mtq4
- https://github.com/dandelany


### a js scraper written by maxogden

```shell
# scraper written by maxogden
# https://github.com/maxogden/goes-16-cira-geocolor/blob/master/run.sh

node index.js # downloads latest
find . -name "*.png" -size -1k -delete # if last ran exited w/ half written files
ls images | xargs -I {} sh -c "if [ ! -f renders/{}.png ]; then montage images/{}/*.png -tile 3x3 -geometry +0+0 -background none renders/{}.png; fi"
for f in renders/*.png; do
  filename=$(basename $f .JPG)
  if [ ! -f overlay/$filename ]; then
    convert $f -fill white -pointsize 30 -gravity NorthWest -draw "text 10,10 '$(node date.js $filename)'" -pointsize 20 -draw "text 10,50 'CIRA GeoColor, NASA GOES-16 Satellite'" overlay/$filename
  fi;
done
rm -rf movietmp
mkdir movietmp
ls overlay | tail -n -96 | xargs -I {} cp overlay/{} movietmp # only build last 96 (last 24 hours worth)
ffmpeg -y -r 5 -pattern_type glob -i 'movietmp/*.png' -vf scale=2034:-1 -vcodec libx264 -crf 25 output/24hrs.mp4
```

---

# original readme content

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
