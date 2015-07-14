This writeup is cross-posted from the
[sudoroom blog](https://sudoroom.org/serial-over-webaudio/).

# serial over webaudio

At the last
[hardware hack night](https://sudoroom.org/events/hardware-hack-night-2015-07-14/),
[Jake](https://github.com/jerkey) and I got a web page to transmit uart serial
data to an arduino at 9600 baud using the webaudio browser api.

Sending serial data from an ordinary web page is interesting because it means
you can communicate with hardware without installing any special software onto a
device. This makes setting up a system faster and easier, which is really useful
for annoying computers like phones where it is very difficult to install and run
new software. Any device with a modern web browser will do: android, iOS, linux,
windows, MacOSX, whatever!

For a quick demo, check out this web page:
[http://substack.neocities.org/serial.html](http://substack.neocities.org/serial.html)

Here is the final project in action:

<iframe width="420" height="315" src="https://www.youtube.com/embed/a8VLnap86pA"
frameborder="0" allowfullscreen></iframe>

# uart

Serial uses a protocol called UART
([Universal Asynchronous Receiver/Transmitter](https://en.wikipedia.org/wiki/Universal_asynchronous_receiver/transmitter))
with a very simple
[data framing](https://en.wikipedia.org/wiki/Universal_asynchronous_receiver/transmitter#Data_framing)
scheme.

UART begins with a low start bit (0), then a character (5-8 bits, configurable
but just use 8 bits) with the least significant bit first, followed by 1 or more
high stop bits (1s).

For example, to send the message "ABC", we would generate the binary data:

```
| start |    character    | stop |
|-------|-----------------|------|
| 0     | 1 0 0 0 0 0 1 0 | 1    |
| 0     | 0 1 0 0 0 0 1 0 | 1    |
| 0     | 1 1 0 0 0 0 1 0 | 1    |
```

Written another way, this data is the binary string:
`0100000101 0010000101 0110000101`.

Under UART we can send as many stop bits as we want, so if there is no data
available, we can keep sending stop bits until there is more data we want to
send.

For example, we can send the same "ABC" message from before with more stop bits
to control the timing of A, B, and C:

```
| start |    character    | stop                   |
|-------|-----------------|------------------------|
| 0     | 1 0 0 0 0 0 1 0 | 11111111111111111      |
| 0     | 0 1 0 0 0 0 1 0 | 11111                  |
| 0     | 1 1 0 0 0 0 1 0 | 1111111111111111111111 |
```

Written another way, this data is the binary string:
`01000001011111111111111111 00100001011111 0110000101111111111111111111111`.

I wrote a javascript library called
[uart-pack-frame](https://github.com/substack/uart-pack-frame) to implement UART
framing with `.write()` and `.read()` methods.

# webaudio

Now that we have a way to frame characters for UART, we can send the framed data
through the browser's webaudio API.

To do this, we can use the webaudio API's script processor node:

``` js
var Context = window.AudioContext || window.webkitAudioContext;
var context = new Context;

var sp = context.createScriptProcessor(2048, 1, 1);
sp.addEventListener('audioprocess', function (ev) {
    // ...
});
sp.connect(context.destination);
```

Inside the `'audioprocess'` event, we can populate an output buffer with the
data we want to send. For example, to send an alternating pattern of `-1` and
`1` we can do:

``` js
sp.addEventListener('audioprocess', function (ev) {
    var output = ev.outputBuffer.getChannelData(0);
    for (var i = 0; i < output.length; i++) {
        output[i] = i % 2 ? -1 : 1;
    }
});
```

The output buffer expects floating point values from `-1` to `+1`, inclusive.

These values map to the voltage coming out of the audio jack, which is exactly
what we'll need to send our UART data.

The other important piece of information we need is the number of samples per
second, which is available as `context.sampleRate`. The sample rate depends on
the system, but some common values are `44100` and `48000`. This sample rate
sets the theoretical ceiling on how fast we can transmit.

Next we'll need to pick a baud rate that will work with the capacitors that
filter out frequencies that are too high or too low. In practice `9600` and
`4800` baud seem to work reliably, depending on the device.

To send audio data, we take the audio sample rate and divide based on the baud
rate. This ratio defines a window size for how many audio samples to keep each
UART bit set.

``` js
var baudRate = 9600;
var windowSize = context.sampleRate / baudRate;
var bits = [ 0, 1, 0, 0, 0, 0, 0, 1, 0, 1]; // the letter A with UART framing

// repeat each bit by the window size number of times
var nbits = [];
for (var i = 0; i < bits.length; i++) {
    for (var j = 0; j < windowSize; j++) {
        nbits.push(bits[i]);
    }
}

sp.addEventListener('audioprocess', function (ev) {
    var output = ev.outputBuffer.getChannelData(0);
    for (var i = 0; i < output.length; i++) {
        var b = nbits.shift();
        if (b === undefined) output[i] = 0
        else output[i] = b ? -1 : 1
    }
});
```

I've wrapped up all of these concepts into a library called
[webaudio-serial-tx](https://github.com/substack/webaudio-serial-tx)
that sets up the webaudio context and encodes input with
[uart-pack-frame](https://github.com/substack/uart-pack-frame).

With [webaudio-serial-tx](https://github.com/substack/webaudio-serial-tx)
you can write a program to speak serial over the audio output in the browser:

``` js
var serial = require('webaudio-serial-tx');
var port = serial({ baud: 9600 });

port.write('HACK THE PLANET');
port.start();
```

Save this program as `serial.js`, then install install
[node.js](https://nodejs.org/) which comes with [npm](https://npmjs.org) and
then do:

```
$ sudo npm install -g browserify
$ npm install webaudio-serial-tx
$ browserify serial.js > bundle.js
$ echo '<script src=bundle.js></script>' > index.html
```

Then open `index.html` in your browser to hear a short blip of data.

You can load a simple demo of webaudio-serial-tx at:
[http://substack.neocities.org/serial.html](http://substack.neocities.org/serial.html)

# circuit

Finally we'll need a simple circuit to interface the audio jack

![circuit](http://scratch.substack.net/serial-webaudio/circuit.jpg)

Circuit diagram by [Jake](https://github.com/jerkey).

# finally

With all of those pieces together, check out the final product in action:
[https://www.youtube.com/watch?v=a8VLnap86pA](https://www.youtube.com/watch?v=a8VLnap86pA)

The baud rate settings are fussy and specific to the device, but from what I can
gather online, other people have the same issues sending serial over audio.

My laptop and Jake's phone worked well at 9600 baud, but my phone worked best at
4800 baud.

A good next step would be to get the browser microphone API working for serial
input for full duplex serial communication using only a web page.

![workspace](http://scratch.substack.net/serial-webaudio/workspace.jpg)
