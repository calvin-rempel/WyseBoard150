# WyseBoard

This is a simple converter to allow use of a PS/2 style keyboard with a Wyse 150 serial terminal. It is a fork of [this project](https://github.com/bryanc806/WyseBoard), though when I built it I found that on my Wyse 150 that the keymapping was completely messed up. After a lot of trial and error, I got MOST of the keys correctly mapped, although I was unable to find keys matching up to the down arrow key (which, when connecting the terminal to Linux and trying to use a text editor, is a bit of a problem). I mapped that and a few of the other keys to some of the F-Keys (oddly, there is another F4 that doesn't seem to correspond to F4 when in the terminal menu - I'm not sure what is up with that), and at some point may map the ESC key to one of these function keys - the hope being that I can find a way to get Linux to re-map these keys to the keys I need them to be.

I have also found the key output to be a bit jittery at times (ie, I suddenly wind up with extra characters). I tried lowering the baud rate on the keyboard adapter, although that does not appear to have made much difference. That being said, if I can get the key mapping sorted out, it is infrequent enough that - while a bit annoying - it could probably be lived with (though I may see if I can figure that out down the road. Or not. We will see).

## Connections on an Arduino Pro Mini:

    Pro-Mini Pin	Connection
    ------------    ----------
    3 --> PS/2 Clock
    5 --> PS/2 Data
    9 --> Pro-Mini 10 (SPI SS)
    12 --> Wyse Data (Pin 1)
    13 --> Wyse Clock (Pin 3)
    +5v --> Wyse +5v (Pin 2), PS/2 +5v
    GND --> Wyse Gnd (Pin 4), PS/2 GND


## Operation

The converter uses the SPI interface in slave mode to shift keyboard
data out.  The Wyse protocol is a simple bit array that is shifted into
the terminal while the terminal sends the Clock signal.

To ensure syncronization, the SPI interface is reset if the Clock signal is low for a number of calls to loop().

See https://terminals-wiki.org/wiki/index.php/Wyse_WY-60

The PS2Advanced Library (https://github.com/techpaul/PS2KeyAdvanced) is needed for the PS2 side of things.

## Use With Linux and Terminfo

Because I have been unable to get the key mapping perfect on the Arduino side of things, I am compensating for that in part by using terminfo.

### Install Dependencies
Per [Heamogoblin](https://bytemyvdu.wordpress.com/2016/09/03/tandy-trs80-m100-serial-terminal/), assuming you are on Debian, run:
```
sudo apt-get install build-essential libncurses5-dev libncursesw5-dev
```
and then, in the directory you downloaded vt100wyse to, run:
```
tic vt100wyse
```

Then, copy the resultant file so it can be used system wide:
```
sudo cp $HOME/.terminfo/v/vt100-wyse /usr/share/terminfo/v/
```
and reboot:
```
sudo reboot
```

To use the new terminfo file, run:
```
export TERM=vt100-wyse
```
I added this into my bashrc so it would run upon login.

I got the original source for the VT100 terminfo file from: http://www.catb.org/~esr/terminfo/
And some much needed details for adding the end and escape key from here: https://www.ibm.com/docs/en/zos/2.3.0?topic=syntax-defined-capabilities

Note: The escape key only SORT-OF works. Testing in VIM, I can escape to command mode, but it also overwrites a character and does something funny - so until such a time
as I can figure that one out, VIM would not really be a usable text editor with this terminal (though, perhaps VIM supports custom key bindings which might be a suitable work around).


