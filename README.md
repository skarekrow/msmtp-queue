# A simple set of scripts/units for queuing and sending messages with msmtp.

* `msmtpq` is a drop-in replacement for `msmtp` to queue an outgoing
  message. It never blocks and never actually sends messages.
* `msmtpq-flush` sends reliably queued messages.
* `msmtpq-queue` shows information about queued messages.

## To use:

1. Send messages with `msmtpq` instead of `msmtp`.
2. Install and enable the provided systemd units to automatically flush queued
   messages.
3. Use `msmtmpq-queue` to see information about the messages.
   - `-h` help / usage
   - `-c` print count of queued messages and exit
   - `-f` include the full message file path

   
## Systemd Units:

* `msmtp-queue.service` - Invokes msmtpq-flush to flush queued messages, if any.
* `msmtp-queue.timer` - Invokes msmtp-queue.service every 10 minutes. This will
  allow you to send mail while offline.
* `msmtp-queue.path` - Invokes msmtp-queue.service immediately when a message is
  queued. This is the fast-path for sending mail immediately you're online.
  
These scripts really shouldn't ever *eat* your mail although you may end up
sending the same email twice on very rare occasions. However, if you send a
message while offline, they'll only re-try once every 10 minutes.

Also note, these scripts look for/put configs/logs in non-standard directories:

* GONFIG (msmtp config): `$XDG_CONFIG_HOME/msmtp/config`.
* LOG: `$XDG_LOG_HOME/msmtp.log` (`$XDG_LOG_HOME` defaults to `~/.local/log`).
* QUEUE_DIR (mail queue): `$XDG_DATA_HOME/mail.queue`.

These are defined as variables at the top of the provided scripts.

### MacOS launchd
* `launchd/name.neuhalfen.msmtpq-watcher.plist` - launchd service that watches the spool directory

## Install:

This is a highly simplified script assuming you already have the above-mentioned 
msmtp config file and set up XDG variables.


### Linux
```
git clone https://github.com/neuhalje/msmtp-queue.git /tmp/msmtpq
sudo cp /tmp/msmtpq/msmtpq* /usr/local/bin
mkdir -p ~/.config/systemd/user $XDG_DATA_HOME/mail.queue
cp /tmp/msmtpq/systemd/msmtp-queue.* ~/.config/systemd/user
systemctl --user enable msmtp-queue.path msmtp-queue.timer
```
Afterwards, update your mutt configuration to use msmtpq instead of msmtp.

### MacOS
MacOS needs a current bash from homebrew, else the stone age bash (3.x) from MacOS will err on the scripts.

```
brew install flock bash msmtp

git clone https://github.com/neuhalje/msmtp-queue.git /tmp/msmtpq
sudo cp /tmp/msmtpq/msmtpq* /usr/local/bin
mkdir -p ~/.config/systemd/user $XDG_DATA_HOME/mail.queue

cp launchd/*  ~/Library/LaunchAgents/

launchctl unload ~/Library/LaunchAgents/name.neuhalfen.msmtpq-watcher.plist 
launchctl load ~/Library/LaunchAgents/name.neuhalfen.msmtpq-watcher.plist 
launchctl start name.neuhalfen.msmtpq-watcher
```


Afterwards, update your mutt configuration to use msmtpq instead of msmtp.
---

WHY? Because it's pretty much-bullet proof and the default scripts were NIH.
