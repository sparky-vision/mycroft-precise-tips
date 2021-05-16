I have recently become enamored with an open-source virtual assistant project, name of Mycroft. One of the advantages of this system, besides its focus on privacy and security, is that you can train your own wake word, as well as perform various other customizations. However, the documentation for precise was, I thought, slightly lacking, and so that motivated me, once I figured out the process, to write it down, so that others may have a guide to help them.

First, it's important to know that while the process isn't particularly difficult, but it is a little time-consuming. You'll want to set aside some time to make your recordings in a quiet place. Further, this guide will assume a level of familiarity with basic Linux / Unix / *nix commands, some familiarity with using a command line, and knowledge of how to use a computer. None of this gets programmer-y or difficult, but some people see a command line and head for the hills. Embrace it, it's fun.

Things you will need!
<ul>
 	<li>A Raspberry Pi (obviously) running Mycroft. I'm using core 20.8.1 (Buster Keaton). If they update things in the future, I'll try to remember this exists and update it as well.</li>
 	<li>Another computer to do the modeling on. I'm using a Macbook Pro, but it will work on anything because we're going to run things on a virtual machine (VM). If you have a machine with Ubuntu 18.04 installed on it, that will probably work as well.</li>
 	<li>A virtual machine. I recommend <a href="https://www.virtualbox.org/">VirtualBox from Oracle</a>, for its price (none) and usability across just about every operating system that exists. Of course, you can run this straight on Linux, too, but I'm recommending a <em>specific</em> version of Ubuntu, so the VM is a good place to start unless you're experienced enough to know exactly what you're doing.</li>
 	<li>An iso of <a href="https://ubuntu.com/download/desktop">Linux Ubuntu 18.04</a></li>
 	<li>A fairly decent USB microphone. This doesn't have to be expensive, but it's a bit of an investment. Expect to spend at least $80 for a decent one. Under that the quality can get a bit shoddy. This <a href="https://www.amazon.com/FIFINE-Mute-Button-Monitor-Headphone-Jack-Four-Pickup-Patterns-K690/dp/B08KD5NHKV/ref=sr_1_17?dchild=1&amp;keywords=usb+microphone&amp;qid=1621117988&amp;sr=8-17">one is fine</a>. If you're going to use a USB mic with VirtualBox, remember to select Devices-&gt;USB-&gt;The USB mic from the Devices menu, or the device will not show up in the VM due to the OSs inability to "share" the USB device. Make sure you're getting good audio from the sound settings section of the control panel.</li>
</ul>
Second: I DO NOT RECOMMEND doing this on the Raspberry Pi, for a lot of reasons, but if you have a laptop or other computer, it will likely run faster on that. I had some trouble getting this to work on Ubuntu 20, so I went back to 18.04, which has worked very well for me so far. Let's go through how I did all this.<br><br>

Step zero: If you know all of what I'm about to say, and want to skip to why precise won't work, scroll down to step whatever.

Step one: prepare your environment. I did this by getting VirtualBox installed, and then setting up a Ubuntu 18.04 virtual environment. You'll want...quite a lot of disk space for this. I ended up allocating 10GB for the image to use, and that turned out to be too small. 20GB is ideal. Why 18.04? Well, because it's what I used, it's fairly recent, and it worked. I tried on 20.04, and ended up with a <a href="https://en.wikipedia.org/wiki/Dependency_hell"><em>lot </em>of issues</a> getting precise to model and run correctly. Your milage may vary, but since this is a guide for the confused and frustrated, it's what I'm recommending that you use. Get it installed in your VM with all the default software. You probably don't need any add-ons.

Step two: you'll want to install, well, a bunch of things. Starting with updating apt. It doesn't really matter <em>where</em> you install this, but the home directory ("cd ~") is where I installed my copy, and that works just fine.

```
sudo apt-get update
```

That will run, now install git

```
sudo apt install git
```

Congratulations, you have installed the world's premiere file change-tracking software. That was fun. Let's move on.

For some reason, we have to add another repository to git before we're allowed to install all of the software we plan to install.

```
sudo add-apt-repository universe
```

Step three: We may now advance to the reason for our exercise - getting precise installed. Git makes it easy to grab a copy.

```
git clone https://github.com/MycroftAI/mycroft-precise.git
```

It will download a copy of precise 0.3.0, and you will now see a new directory in your home folder, "mycroft-precise". Change into that directory, and edit setup.py with the following command:

```
sudo pico setup.py
```

Why pico and not nano? Because I learned Linux on SuSE 9 a loooong time ago, and it will open nano for you anyway. ;)

Okay, you're going to have a lot of text here. But there's one line we need to change in this file, way down at the bottom.

```
'numpy==1.16',
'tensorflow&gt;=1.13,&lt;1.14', # Must be on piwheels
'sonopy',
'pyaudio',
'keras&lt;=2.1.5',
'h5py',
'wavio',
'typing',
'prettyparse&gt;=1.1.0',
'precise-runner',
'attrs',
'fitipy&lt;1.0',
'speechpy-fast',
'pyache'
```

See the line that says 'h5py'? Change that to read

```
'h5py&lt;3.0.0',
```

Save the file, and exit pico or nano or whatever you're using. (Vi? Emacs? Vim? Editing the inodes with a magnetic needle?) Hooray, we can now install precise. This is done by running

```
./setup.sh
```

Step four: This will take some time to complete. It will download everything for you, do all sorts of magic, and eventually, return you (hopefully) to a command prompt. Congratulations, you have now installed precise. Make it usable by running

```
source .venv/bin/activate
```

From within the mycroft-precise directory. This will tell Python to include the precise libraries.

Step five: and one of the most important ones. Download the zip of mycroft-precise binary data, <a href="https://github.com/mycroftai/precise-data/tree/dist">from this page</a>. The file you need will change depending on the architecture you're running Mycroft on. For a Picroft, it's the <a class="js-navigation-open Link--primary" title="armv7l" href="https://github.com/MycroftAI/precise-data/tree/dist/armv7l" data-pjax="#repo-content-pjax-container">armv7l</a> files. Grab precise-engine.tar.gz, and unzip it. Within, you will find a folder called "precise-engine". SSH (or SFTP, or whatever) into your Picroft, and navigate to

```
/home/pi/.mycroft/precise
```

And replace the existing precise-engine folder there with the one you just downloaded. Also, I've found you need to do the following bit of dumbassery:

```
chmod 777/home/pi/.mycroft/precise/precise-engine/precise-engine
```

Which allows all users and all groups the ability to read, write, and execute that file. I tried with lesser permissions and ran into errors. Probably not a big deal, you <em>are</em> behind a decent firewall and not exposing your Pi to unknown incoming connections, right? Right?? Reboot your pi. (Strictly necessary? ...probably?)

Okay, modeling. There are several guides to running this software. The <a href="https://github.com/MycroftAI/mycroft-precise/wiki/Training-your-own-wake-word#how-to-train-your-own-wake-word">official Mycroft one is here</a>, and some additional thoughts by <a href="https://github.com/el-tocino/localcroft/blob/master/precise/Precise.md">contributor ElTocino are here</a>. Here are <em>my</em> thoughts, having messed with this thing for a while.

The official Mycroft-precise training page is a good start, but some additional information is needed. I say this because once I got the hang of creating models, I was able to do so without using almost any of the tools other than precise-train, precise-listen, precise-test, and precise-collect. You should also go ahead and make all of the directories that it asks you to. In my case, I called my model tng-computer (I'm a Trek fan, can you tell?) and so my dirs were
<ul>
 	<li>tng-computer
<ul>
 	<li>not-wake-word</li>
 	<li>test
<ul>
 	<li>not-wake-word</li>
 	<li>wake-word</li>
</ul>
</li>
 	<li>wake-word</li>
</ul>
</li>
</ul>

Let's cover the data you'll need first.<br>

The official Mycroft-precise instructions tell you that you need around twelve samples. In my experience, this is too little, and indeed, ElTocino would agree with us here. For my model, I recorded about 300 samples of the wake word, which in my case, was "computer". Half of these are me, half are my partner, and I sprinkled in all my kids, too, to help get a precise model. Everyone you want to be able to use Mycroft should contribute. It will help. Try to get lots of different inflections - rising, falling, neutral, happy, sad, disgusted. Think of any pronouncations that might be considered alternate. For instance, when saying "computer", a lot of United States central midwest accents tend to drop the explicit "t", so it sounds more like "compu-er", without the explicit tapped "t" on the hard palate. I realized after I recorded that I needed to update my wake-words to include that.

Now, the not-wake-words...

You need a <em>lot</em> of not-wake data. Oodles of it. In addition to the <a href="http://download.tensorflow.org/data/speech_commands_v0.01.tar.gz">Google Speech Commands</a> (side note: it's like they're trying to camouflage that link) and the <a href="http://pdsounds.tuxfamily.org/">Public Domain Sounds Backup</a>, you should record many, many instances of yourself saying every conceivable rhyme or word that <em>might sound like your wake word</em>. For my model, that was:
<ul>
 	<li>intruder, abuser, bruiser, hooter, cooter, looter, scooter, shooter, suiter, commuter, consumer, confuser, polluter, recruiter, persecutor, prosecutor, you hear her, trouble shooter, sewer, stupor, etc.</li>
</ul>
Record lots of those. In addition, you will find that random noises that you didn't expect will activate a poorly-trained model. To combat this, I recommend at least an hour of room noise, and possibly more, depending on all the activities that take place in your room. Have kids? Record an hour of them horsing around in the room. Record some of their television shows. Record conversations with your partner. For some reason, my daughter's whistling tunes at the Mycroft set it off after I thought I had made a pretty good model. Add some of that and try again.<br><br>

As for how to do this...

The downloadable sound sets are pretty self-explanatory. Download and unzip them into your not-wake-word directory. (Precise-train looks in directories recursively, so don't worry about having folders within folders.) Recordings of the actual wake-word, and rhymes, are best done with the precise-collect tool. Just give them descriptive names so you can remember what they are, and start slappin' that spacebar. Don't leave silence at the beginning or end of the recording, this will mess up the training. For room noise recordings, you can use arecord:

```
arecord -f S16_LE -t wav -r 16000 --max-file-time 30 roomnoise_may16.wav
```

arecord is a little dumb in that it doesn't give you feedback that it's saving additional files, but it is. This command tells arecord to start recording in 30-second increments. This is useful because if someone says the wake-word accidentally, you can go back and pull that one file without losing the rest of your data.<br>

With the Google Speech Commands, and Public Domain sounds, and all my room noise and rhymes, I had about ~51K not-wake-sounds. This is probably sufficient. You can now run precise-train. I didn't bother with the really long commands that ElTocino posted - in part because I couldn't find documentation as to what those switches do. (Maybe they'll chime in here...) For my purposes, it was enough to do

```
precise-train tng-computer.net tng-computer/ -e 150
```

It's not many epochs (cycles of training) and that's okay! I started hitting val_acc of 1.0 <em>really</em> quickly after about 100 epochs, and I think that if you have a high-quality dataset, I suspect you'll see the same results. You'll want to be hitting in the upper .9s, or you won't have a good model.

If you get to those numbers, congrats, you can now do the exciting part and use:

```
precise-convert tng-computer.net
```

Obviously replacing the .net file name with your own. You'll get a .pb and .pb.params file. Dump those into your home dir on your Picroft. Follow the <a href="https://mycroft-ai.gitbook.io/docs/using-mycroft-ai/customizations/wake-word">instructions here</a> to edit your Mycroft config to tell it about the new file. I found that for me, the trigger_level and sensitivity had to be set quite permissively, at 1 and 0.9, respectively. It unintentionally activates a few times a day, at present, but that's because this is a new model and I need to add some more not-wake-word data. It's quite usable!

You may see an error that the .params file isn't working. This isn't a problem...because reasons. Enjoy your new wake word, and LLAP. 🖖🏻 (Many thanks to ElTocino for their help in getting this software working.)
