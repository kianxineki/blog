I spent this winter break down on the Kenai peninsula with the folks.
A few days before the new year, my father, little brother, and I
took my little brother's cheap telescope out to look at the full moon.

I had forgotten how difficult it is to actually use a telescope to look at
distant objects.
For one, it's really difficult to use a telescope while wearing glasses, and
without glasses the focus needs to be changed if someone else wants to look into
the eyepiece.
It's also quite difficult to position a cheap telescope by hand while bending
over sideways or kneeling on the ground to use the eyepiece.

These sorts of manual adjustments are best left to machines, I thought,
so I decided to use my little brother's
[Lego NXT brick](http://en.wikipedia.org/wiki/Lego_Mindstorms_NXT),
webcam, and a USB game controller I had recently bought at
a thrift shop for a few bucks to put together a robotic control system for the
telescope.

![gamepad](/images/robot-telescope/gamepad.jpg)

With a bucket of LEGOs, NXT parts, and lots of rubber bands, I set to work
building this robotic telescope control system.

To swivel the telescope from side to side,
I built a double-spool to wind up string in one direction while unwinding in the other.
This string was tied to some legos attached to the telescope body, so the
telescope would swivel about the pivot when the spool pulled it in a direction.

![double spool](/images/robot-telescope/double-spool.jpg)

To lift the telescope, I first dangled a plastic cup from some rubber bands on
the end with the eyepiece to act as a counter-balance.
I stuck a motor to a simple gear system that lifted the bigger end of the
telescope.
By balancing the telescope so that the bigger end didn't take much force to
lift, the motor needed to do much less work to adjust the telescope pitch.

![pitch motor](/images/robot-telescope/pitch-motor.jpg)
![telescope](/images/robot-telescope/telescope.jpg)

With the hardware in place, I wrote some code to drive the NXT directly with my
laptop over the NXT's USB interface. Using
[nxt-python](http://code.google.com/p/nxt-python/),
this task was pretty straight-forward.
On the laptop, I used [pygame](http://www.pygame.org/) to read events
from the USB gamepad.
I had used this model of gamepad before to control an 
underwater ROV for a contest in California, so I knew it would be pretty simple
to get it working for this telescope project.
I've also put
[the controller script that ran on the laptop](http://github.com/substack/telescope-controller/blob/master/telescope.py)
up on github for others to build upon.

Construction and programming took about a day, but I spent altogether too much
time getting the first webcam I found lying around the house to work with Linux
to no avail. Luckily there was another webcam around that worked without any
fuss in [cheese](http://projects.gnome.org/cheese/).

![webcam](/images/robot-telescope/webcam.jpg)

I took the whole setup outside in the cold and used the gamepad and laptop
screen to hunt for the moon.
Unfortunately, when I found the moon, none of the moon's features were
distinguishable in the video output.

![outside setup](/images/robot-telescope/outside-setup.jpg)
![moon](/images/robot-telescope/moon.jpg)

At first I thought this was due to the focus of the webcam.
I had read some posts online about people mounting webcams to telescope who took
off the webcam's lens in order to focus properly.
The webcam had no obvious way to take apart and I didn't want to risk damaging
it since it didn't belong to me anyways, so the next day I tinkered with putting
other camera lenses in between the webcam and the eyepiece and a number of other
things that didn't work.

However, in the process of trying to figure out how to focus the camera during
the day, I pointed the robotic telescope out a window facing some trees and
captured these images. This was the same camera setup that I'd used previously
to look at the moon.

![](/images/robot-telescope/cheese-thumbs/2010-01-01-142703.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-142824.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-142840.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-143124.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-153506.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-153622.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-153650.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-153715.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-153751.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-153758.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-153845.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-153914.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-153935.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-154011.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-154038.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-154052.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-154219.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-154321.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-154341.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-154412.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-154416.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-154630.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-154700.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-154728.jpg)
![](/images/robot-telescope/cheese-thumbs/2010-01-01-154752.jpg)

I've uploaded a video of footage from the webcam and of the telescope in
action. Download or watch below.

[telescope.ogv](/video/telescope.ogv) [Ogg Theora, 13 M]

<object width="425" height="344">
    <param
        name="movie"
        value="http://www.youtube.com/v/ZmRET6z0XWg&amp;hl=en_US&fs=1&amp;"
    >
    </param>
    <param name="allowFullScreen" value="true"></param>
    <param name="allowscriptaccess" value="always"></param>
    <embed
        src="http://www.youtube.com/v/ZmRET6z0XWg&amp;hl=en_US&amp;fs=1&amp;"
        type="application/x-shockwave-flash"
        allowscriptaccess="always"
        allowfullscreen="true"
        width="425" height="344"
    ></embed>
</object>

I showed both of my little brothers how to get the code I wrote and the
necessary software onto their eee-pcs so that they could drive the telescope
around too. The older of my two little brothers, Ross, who is 11, dismantled the
telescope control system and built a robot car with the parts. He figured out
how to modify the telescope script to drive the car around pretty well and
after that he read through
[Invent With Python](http://inventwithpython.com/)
and is having all sorts of fun building games and ciphers.

The telescope may not have been very useful for looking at the moon or other
astronomical objects, but it was a fun little project that could at least
capture some interesting daytime images.
However, as a motivational tool to get my siblings more interested in robotics
and computer programming, this project was very successful.
