
# Fuzzy76's personal server setup

## What is this?

This is the docker-compose setup responsible for my personal server. It isn't
pretty and is full of hacks. Don't judge. :)

### Folder structure

`data_configs` contains configuration files for services.

`data_env` contains .env files for services that require them. Not committed to git.

`data_seeded` contains folders that are seeded with content, but not committed
to github because of their nature:
  - `bia_old` - An archive of the old website for the game clan Brothers In Arms.
  - `coduo` - The game folder for the Call of Duty : United Offensive server.
  - `thelounge` - All data for thelounge. I just migrate the entire folder.
  - `ultrastats` - A _really_ old statistics software for Call of Duty. Version 0.3.16.

`prod.env` is a docker environment file. Variables are:
  - `mysql_root_password`

## Service: certbot

Software: https://certbot.eff.org
Documentation: https://certbot.eff.org/docs/using.html
Image: https://hub.docker.com/r/certbot/certbot

## Service: coduo

Software: https://en.wikipedia.org/wiki/Call_of_Duty:_United_Offensive
Docker image: https://hub.docker.com/_/debian/

This is a game server for "Call of Duty: United Offensive", an expansion pack
for the original Call of Duty released in 2004. It's a 32-bit binary and for
some reason will _not_ run on a Mac host (it won't find any pk3 files). The
docker image is homemade, and quite frankly a mess. :)

It exposes a handfull of ports (28960, 20500, 20510, 20600, 20610). It does
everything in a single, huge mount.

The entrypoint is set to /coduo/start.sh - so make sure you have a shell script
there with your launch options.

### Game folder setup

First copy the game files of the retail version of the game into the "coduo"
folder. The CoD:UO Linux dedicated server README says to copy the files from
Setup/Data on disk 1, then the files from the same directory on disk 2. In my
case, I found files from a folder called "program files" on the CD instead. But
that _might_ have been a custom installer. :) When this is done correctly the
"coduo" folder contains "localization.txt", "Main", "uo" among with other
folders and files. **Important!** Rename the folder "Main" to the lower-case
"main" before proceeding.

Next, get the Call of Duty United Offensive dedicated Linux server v1.51 large
version (I found a copy on
http://planetcallofduty.gamespy.com/View6fa9.html?view=CODUOFiles.Detail&id=18).
Extract and move the files into the "coduo" folder, overwriting any duplicate
files (there are some inside "uo").

Install any mods you'd like (follow their installation instructions).

My config setup involves a main config at `uo/dedicateduo.cfg` and a
mod-specific config at `BrothersInArms_mod/awe.cfg`.

You can optionally download cod_pb.zip from
http://www.callofdutyview.net/downloads/call-of-duty-punkbuster-downloads/ and
replace the pb folder in your server install with the one inside the archive.
This will require your players to do the same, and if they don't, punkbuster
will kick them. Yes, it's a pain. But punkbuster hasn't been updated for years,
and it will block more cheaters than the default. 

Create a start.sh with something like this:

```
#!/bin/sh
cd /coduo/
./coduo_lnxded +set gamestartup \"`date +"%D %T"`\" +set com_hunkmegs 512 +set fs_homepath /coduo +set fs_game BrothersInArms_mod +set dedicated 2 +exec awe.cfg +exec dedicateduo.cfg +set net_port 28960
```

## Service: csgo

Software: http://blog.counter-strike.net/ 
Docker image: https://github.com/CM2Walki/CSGO

Gameserver for Counter Strike : Global Offensive. Should not need an
introduction.

Uses an env file for settings. Exposes ports 27015 and 27020 (both tcp and udp).
Mounts a data_storage folder for game data.

Notes:
* Edit serverconfig and comment out weird defaults, ESL config and change the Steam group id.

## Service: mysql

Software: https://www.mysql.com/ 
Docker image: https://hub.docker.com/_/mysql/

Uses an env file for settings. Mounts a folder for data storage.

## Service: nginx

Software: https://nginx.org
Docker image: https://hub.docker.com/_/nginx/

Mounts a plethora of folders. Exposes port 80 and 443 for HTTP and HTTPS.

## Service: thelounge

Software: https://thelounge.chat
Docker image: https://hub.docker.com/r/thelounge/thelounge/

Mounts a folder for storage.

## Service: ultrastats

Docker image: https://hub.docker.com/_/php/

A really old statistics software that happens to be the last to support Call of
Duty : United Offensive.

Mounts the log file from coduo and its application files.


## TODO

### Før launch

- [x] databasedump fra ultrastats
- [x] thelounge produksjonsdata
- [x] Change mounts to volumes
- [x] certbot stuff

### Etter launch

- [x] csgo (starter ikke)
- [x] coduomaps (filer laster ikke)
- [x] stats.bia.no (white page of death)
- [x] old.bia.no (http istedenfor https på stilark tipper jeg)

- [ ] ultrastats crontab `0 5 * * * /home/bia/ultrastats/ultrastats-0.3.16/src/contrib/bia_runparser.sh >> /tmp/ultrastats.log 2>&1`
- [ ] coduo crontab restart
- [x] Se om noe fra matilda.fuzzy76.net/~fuzzy76/ skal over
- [ ] crontab certbot renew
- [x] Test nedlasting av maps i coduo (test også nedlasting av config)
- [x] Test serveren med en ren ip-request
- [x] git commit og push


---

```
docker-compose run certbot certonly --webroot -w /var/www/bia_old -d old.brothersinarms.no 
docker-compose run certbot certonly --webroot -w /var/www/chat -d chat.fuzzy76.net
docker-compose run certbot certonly --webroot -w /var/www/matilda -d matilda.fuzzy76.net
docker-compose run certbot certonly --webroot -w /var/www/bia_stats -d stats.brothersinarms.no
docker-compose restart nginx
```
