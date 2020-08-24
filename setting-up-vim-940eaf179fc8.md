Unknown markup type 10 { type: [33m10[39m, start: [33m16[39m, end: [33m24[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m8[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m181[39m, end: [33m189[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m43[39m, end: [33m48[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m58[39m, end: [33m69[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m74[39m, end: [33m81[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m26[39m, end: [33m31[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m34[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m49[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m81[39m, end: [33m102[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m109[39m, end: [33m122[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m41[39m, end: [33m47[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m77[39m, end: [33m89[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m58[39m }

# Setting up Vim

A short, but sweet .vimrc file

![](https://cdn-images-1.medium.com/max/8000/1*Q-ff7kgorIzm9VJX0MuLTw.jpeg)

When did you first learn about Vim? Were you one of those unfortunate souls, who just wanted to run a git command, but ended up trapped in vim? Many of us have been there. Ending up in vim accidentally is like being accidentally teleported to an unknown planet. How are you supposed to google your way out if you do not know where you even are?

I’m still not sure, is vim something everybody knows and uses or maybe just “knows and not-uses” or is it actually really obscure. I only started using it, when I took a deep dive into git commands. Once you start using it, you suddenly realize that GUIs might be overrated. Many things are easier in a GUI, but many things are just as easy in the terminal.

## How to first approach vim?

Just go through vimtutor.

vimtutor is a great vim instruction manual and exercise book. It is probably already installed on your computer, it comes together with the vim installation. To start it, just type vimtutor in your terminal:

<iframe src="https://medium.com/media/cd17b838d4fbc97afc1790970fdfdb18" frameborder=0></iframe>

and you are transported into the vim editor with lessons and instructions and examples to practice commands on.

<iframe src="https://medium.com/media/a83700a411ec2435e51fad8ab108ec45" frameborder=0></iframe>

Once you go through the whole tutorial or at least the first few lessons, using vim becomes child’s play (including closing it). The next step is to modify vim to make it suit your needs. A vast number of settings and plugins is available for vim. But by default, most settings are turned off.

## Pimp my vim

Curiously, lots of essential vim settings are disabled by default. A good example is showing line numbers:

![](https://cdn-images-1.medium.com/max/3058/0*EPmn_-KztUimR_SX.png)

To enable them in an open vim editor, type [ESC] and then :set number and [ENTER]. This will make the line numbers immediately appear.

![](https://cdn-images-1.medium.com/max/2000/0*08uXUx9E1Flvrscu.png)

To disable it again, type [ESC] + :set number! + [ENTER].

![](https://cdn-images-1.medium.com/max/2000/0*okXORYL2a1gPdNZy.png)

Sometimes you might want to know the value of a setting, for this the command is :echo &<setting_name>, i.e. :echo &number.

![](https://cdn-images-1.medium.com/max/2000/0*6rxHRWBzfgQfDlBw.png)

To change the default behavior, create a .vimrc file in your home directory: vim ~/.vimrc and add any number of settings.

Here is a list of default vim settings I have in my .vimrc file:

<iframe src="https://medium.com/media/3828354f3d0bf100afcb650c728bbb38" frameborder=0></iframe>

Have fun testing out vim.

### External sources

* [Vim Tips Wiki: Indenting source code](https://vim.fandom.com/wiki/Indenting_source_code)

* [A Good Vimrc](https://dougblack.io/words/a-good-vimrc.html)

* [Setting up Vim](https://inesp.github.io/2019/10/20/vimrc.html)
