# How to set extra mouse buttons on Linux


If you have a mouse with more than the usual 3 buttons, I have found that it can be a pretty difficult process to configure them to work as you like under a Linux system. I will explain below the easiest method I found.

The first things you need to do is install a software package called xautomation that allows us to map commands to button presses. To do this in a Debian based system you just have to enter the following command:

```bash
  sudo apt-get install xbindkeys
```

After this you have to create the configuration file in your home directory:

```bash
  xbindkeys --defaults > /root/.xbindkeysrc
```

Now you must find what is the number of your extra mouse buttons, just type xev and a white window will open. Move your mouse over it and press the buttons you are interested in. You will see something like:

```
  ButtonPress event, serial 34, synthetic NO, window 0x2400001,
      root 0x11d, subw 0x0, time 1192699, (99,59), root:(693,358),
      state 0x0, button 8, same_screen YES
```

Now you know that you need to configure button 8. 

Just open the .xbindkeysrc with your favourite text editor and start the configuration. There you will find what format you will use for the commands. 

Next I will show you how you can map keyboard keys or combinations to your mouse buttons (for things like copy and paste for example). For this you will need a package called xautomation.

```bash
  sudo apt-get install xautomation
```

By typing `xte --help` you can see some examples of commands you can use.

For my copy-paste example I will put the following in my .xbindkeys file:

```
  # Copy
  "xte 'keydown Control_L' 'key c' 'keyup Control_L'"
          b:8

  # Paste
  "xte 'keydown Control_L' 'key v' 'keyup Control_L'"
          b:9
```

After this, all you have to do is start (or restart) xbindkeys by typing `xbindkeys` (or `killall xbindkeys && xbindkeys` for a restart).
