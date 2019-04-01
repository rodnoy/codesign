# Start
If you use a smart card or a flash card to confirm your authenticity on your Mac, then you may have problems deploying the program from Xcode to the real device. In my everyday life, it happens that I work remotely through a VPN connection. I use a flash stick on which keys are stored for my authentication. It should be plugged into the connector almost constantly.
Xcode building for devices was failing each time. 
The error was `specified item could not be found in the keychain.` To be honest, this message does not seem exhaustive to me.

TL;DR the solution was found [here ](https://forums.developer.apple.com/thread/106938#333777)

There is one way to get around this problem, remove the flash drive with certificates before compiling the program. But this is an extremely inconvenient solution, you quickly get tired of doing that.

And so, why is this happening...
The problem is you need several code sign operations for the build process . One operation does not have time to complete successfully as the others already begin. Other operations simply end their work with an error.
So as **cstar** said kindly, we need to create a wrapper over `/usr/bin/codesign` to be sure that there is only one codesign operation running at a time.

Here are the step-by-step instructions on how to do this:

## Step one  
You need to temporarily disable System Integrity Protection. We need that to copy /usr/bin/codesign file to /usr/bin/codesign.orig and change it's rights.

Restart your mac and keep holding `Ctr+R` to enter to recovery mode, then select in the top menu **Utilities** and then **Terminal**.

Type terminal command : `csrutil disable`.
Reboot you computer again.

If you want more information about SIP on your Mac, you can read it [here ](https://www.imore.com/how-turn-system-integrity-protection-macos).

## Step two

Open Terminal and go to `/usr/bin`. For example by typing `cd /usr/bin`. You will need to rename **codesign** utility to, for example **codesign.orig**. Type in the terminal `sudo cp codesign codesign.orig`. Exactly what to do this you had to turn off SIP on your Mac. 

## Step three

Remove **codesign** utility by doing `sudo rm codesign`. Go ahead, you have already copied the original file to **codesign.orig**.

Instead of a deleted file, create an empty codesign file. Using your favorite text editor, copy the proposed [script](https://github.com/rodnoy/codesign/blob/master/codesign). Don't forget to **sudo** when opening the file with your text editor `sudo emacs codesign`. 

## Step four

We should make executable the created file. To check its current permissions you can type something like `ls -l codesign`. And the add `-rwxr-xr-x` by doing sudo `chmod 751 codesing`.

If you want read more about how to set file permission here is a good [atricle ](http://www.macinstruct.com/node/415). 

## Step five

The SIP can be activated again. Repeat the step one but this time type in terminal `csrutil enable`.

Voila! 

# Troubleshooting

If you got something like a
`command /bin/sh failed with exit code 126`, it probably means you forgot step four, so just apply to the **codesign** file `chmod +x`

