# Pebble Development Setup

After the shutdown of CloudPebble, developing apps for Pebble has been much harder, especially on Windows machines. However, using Windows Subsystem for Linux, this is entirely possible. Additionally, due to some broken links on the [developer.rebble.io](https://developer.rebble.io/) page, the general instructions for installation are hard to find and sometimes outdated, so this guide will also serve as a host for the instructions for Mac and Linux as well.

# Linux/Windows

The Windows installation is almost entirely the same as the Linux installation, but the Windows Subsystem for Linux (WSL) needs to be set up. Go to the Windows Store and download your preferred distro of Linux. This guide uses Ubuntu but any distro should work. Launch it and run the installation.

### Install Dependencies

On Ubuntu 20.04 (and some other distros) certain older packages aren't available through the default `apt` package manager. Run these commands based on your distro.

**On ≤ Ubuntu 18.04 (and other older distros)**

```bash
# Python 2.7 + pip + virtualenv
sudo apt install -y python2.7 python2.7-dev
curl https://bootstrap.pypa.io/get-pip.py --output get-pip.py
sudo python2 get-pip.py
sudo -H python2.7 -m pip install -U pip setuptools wheel virtualenv

# Node.js + npm (will take a few minutes if npm isn't already installed)
sudo apt install -y libssl1.0-dev nodejs-dev npm

# Emulator dependencies
sudo apt install -y libsdl1.2debian libfdt1 libpixman-1-0
```

**On ≥ Ubuntu 20.04 (and other newer distros)**

```bash
# Python 2.7 + pip + virtualenv
sudo apt update
sudo apt install python2
curl https://bootstrap.pypa.io/get-pip.py --output get-pip.py
sudo python2 get-pip.py
sudo -H python2.7 -m pip install -U pip setuptools wheel virtualenv
sudo apt install python2-dev

# Node.js + npm (will take a few minutes if npm isn't already installed)
sudo apt install libnode-dev npm 

# Emulator dependencies
sudo apt install -y libsdl1.2debian libfdt1 libpixman-1-0
```

### Install Pebble SDK

Create a folder called `pebble-dev/` in the location you want to store the SDK (using `mkdir pebble-dev`). This guide uses `~/pebble-dev/` (the home directory) but you can swap in any path for `~` (marked in green).

Run the following commands to extract the SDK:

```bash
cd ~/pebble-dev/
[ -e pebble-sdk-4.5-linux64.tar.bz2 ] || wget https://developer.rebble.io/s3.amazonaws.com/assets.getpebble.com/pebble-tool/pebble-sdk-4.5-linux64.tar.bz2
[ -e pebble-sdk-4.5-linux64 ] || tar -xf pebble-sdk-4.5-linux64.tar.bz2
cd pebble-sdk-4.5-linux64
```

You should now be in the directory `~/pebble-dev/pebble-sdk-4.5-linux64` with the SDK files and directories inside it.

Now add the `pebble` command to your path by running:

```bash
echo 'export PATH=~/pebble-dev/pebble-sdk-4.5-linux64/bin:$PATH' >> ~/.bashrc &&. ~/.bashrc
```

Update outdated dependencies: 

```bash
# Fix pypkjs in requirements.txt
if [ "$(grep 'pebble-sdk-homebrew' requirements.txt)" ]; then cp requirements.txt requirements.txt.OLD; grep -v pypkjs requirements.txt.OLD > requirements.txt; echo 'git+https://github.com/pebble/pypkjs.git' >> requirements.txt; fi

# Replace pebble-tool
if [ "$(grep 'sdk.getpebble.com' pebble-tool/pebble_tool/sdk/manager.py)" ]; then sudo mv pebble-tool pebble-tool.OLD; sudo git clone https://github.com/pebble-dev/pebble-tool.git; fi
```

Install the Python library dependencies locally:

```bash
# Install requirements
[ -e .env ] || virtualenv .env
. .env/bin/activate
pip install -r requirements.txt
pip install -r pebble-tool/requirements.txt
deactivate
```

Finally, install the latest SDK

```bash
pebble sdk install latest
```

**IMPORTANT**: On Windows, it's advisable to keep your *project files* (not any of the SDK files you've just installed) in the regular Windows path, since regular Windows files are accessible from WSL, but not vice versa. You can access them at `/mnt/<drive letter>/<windows>/<path>/` .

# Mac
The Mac installation uses the Nix package manager. If you don't want to install it, (possibly outdated) alternatives are in [the old Reddit post](https://www.reddit.com/r/pebble/comments/9i9aqy/developing_for_pebble_without_cloudpebble_windows/).

### Install Dependencies
```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"

# Install freetype
brew install freetype
```

### Install Nix
You can install Nix from the internet with one command, but it depends on your macOS version (due to a filesystem change on macOS 10.15 Catalina)

```bash
# if on macOS <= 10.14 (Mojave)
sh <(curl -L https://nixos.org/nix/install) --daemon

# if on macOS >= 10.15 (Catalina)
sh <(curl -L https://nixos.org/nix/install) --darwin-use-unencrypted-nix-store-volume
```

Reload your terminal by closing and reopening it, and verify your installation by running `nix-env --version`

### Install Cachix
Cachix is a Nix extension that speeds up building. Install Cachix and point it to the Pebble cache.

```bash
nix-env -i cachix
cachix use pebble
```

Reboot your system for the cache to take hold.

### Setup Nix Shell
If you haven't already, create a folder to hold your Pebble projects (this guide will use `pebble-dev` but you can call it whatever your want) with `mkdir pebble-dev`. Enter that directory using `cd pebble-dev`.

Create a file called `shell.nix` in TextEdit (or another text editor)

```nix
(import
  (builtins.fetchTarball https://github.com/Sorixelle/pebble.nix/archive/master.tar.gz)
).pebbleEnv { }
```

Run `nix-shell`to enter the Nix shell, where you're able to run the `pebble` command. The first time you run it, it might take a few seconds to download dependencies.

Once you're in the shell, install the Pebble SDK.

```bash
pebble sdk install latest
```

**IMPORTANT**: In each project directory that you create, you need to copy the `shell.nix` file into it, and activate the Nix shell with `nix-shell` before running `pebble` commands.

# Create and Run Demo App

Your Pebble development environment should now be set up! Go to the directory you want your projects to be stored in, and clone a project from github or create a new one using `pebble new-project` Test that the SDK works correctly by building the app using `pebble build`.

You can install the built app either on an emulator or on your phone.

- **Emulator**
    **Note:** On Windows, to run an emulator, it needs a display server, as WSL cannot natively display graphical windows. For this I'm using [Xming](https://sourceforge.net/projects/xming/), but you can use any server you want. Download and install it, then run it. (It won't do anything except show an icon in the system tray-- that's what we want.) Run `echo 'export DISPLAY=:0' >> ~/.bash_profile && . ~/.bash_profile` to link WSL to Xming. You might want to set Xming to launch at startup so you won't need to run it each time you boot your computer

    Run `pebble install --emulator [aplite, basalt, chalk]`, and a window with the emulator in it running the app should appear.

- **Phone**
    Turn on the Developer Connection in the Pebble app on your phone (in the Settings menu). Run `pebble install --phone <IP address in developer settings>` in the command line, and the app should appear on your Pebble.

# Writing Apps
Once all of this is set up, having a code editor makes working with these projects much easier, and is another missing feature from CloudPebble. [VSCode](https://code.visualstudio.com/) is a popular editor, but any code editor works. Also, be sure to check the #app-dev channel on the [Rebble discord](http://rebble.io/discord) if you have any issues or questions during development.

### Development Options
There are 3 main options for developing Pebble apps and watchfaces, each catering to a different use case.

#### Pebble C
Write watch apps or watchfaces in C, with a phone side server. The phone-side server is usually PebbleKit JS, where you write the server in Javascript and it runs on both iOS and Android (recommended), or if you need platform-specific features, you can use PebbleKit Android and PebbleKit iOS to interface with companion Android and iOS apps respectively.

If you want to make a watch app, this is the most flexible option, though C is a harder language to learn than the other Javascript-only options.

#### Pebble.js
Framework that runs inside of PebbleKit JS that automatically creates the C watch app based on your JavaScript code. Much easier to learn than C-based watch apps, but the UIs it can create are significantly less flexible, and it requires a connection to the phone at all times (whereas C-based apps can work even with Bluetooth disconnected).

#### Rocky.js
Framework to write watchfaces (no watch apps) in Javascript that run natively on the watch. Unlike Pebble.js, these work without a connection to the phone, but the framework itself was never fully completed, so some things might not work. 

### References
**Pebble Developer Guides**
[https://developer.rebble.io/developer.pebble.com/guides/index.html](https://developer.rebble.io/developer.pebble.com/guides/index.html)

**Pebble API Reference**
[https://developer.rebble.io/developer.pebble.com/docs/index.html](https://developer.rebble.io/developer.pebble.com/docs/index.html)

**Learning C with Pebble**
[https://pebble.gitbooks.io/learning-c-with-pebble](https://pebble.gitbooks.io/learning-c-with-pebble)

**Pebble C App Examples**
[https://github.com/pebble-examples](https://github.com/pebble-examples), [https://github.com/pebble/pebble-sdk-examples](https://github.com/pebble/pebble-sdk-examples)

---

### Other Options

If you don't want to set it up on your local system, or if something from this guide isn't working, you can use these options to get a prebuilt development environment.

**Ubuntu Virtual Machine Image**
[https://willow.systems/pebble/vm/](https://willow.systems/pebble/vm/)

**Docker Image**
[https://hub.docker.com/r/rebble/pebble-sdk](https://hub.docker.com/r/rebble/pebble-sdk)

---

Anyways, I hoped this helped some people get set up with developing for Pebble. If you have any questions or if I missed anything, let me know by filing an issue or on the [Rebble discord](http://rebble.io/discord).

This guide is mostly based off of these links:

[https://developer.rebble.io/developer.pebble.com/sdk/download/index.html](https://developer.rebble.io/developer.pebble.com/sdk/download/index.html)

[https://gist.github.com/zeevro/ce20d0d74a869d73119e9d1522b7caa7](https://gist.github.com/zeevro/ce20d0d74a869d73119e9d1522b7caa7)

[https://gist.github.com/pauleeeeee/37942295077da184000f8ac9ab1aad40](https://gist.github.com/pauleeeeee/37942295077da184000f8ac9ab1aad40)

[https://github.com/Sorixelle/pebble.nix](https://github.com/Sorixelle/pebble.nix)
