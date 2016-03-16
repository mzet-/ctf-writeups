
## Level 1

```
$ curl -s http://ctf.infosecinstitute.com/levelone.php | grep flag
```

flag: `infosec_flagis_welcome`

## Level 2

```
$ curl -s http://ctf.infosecinstitute.com/leveltwo.php | grep img
$ curl -s http://ctf.infosecinstitute.com/img/leveltwo.jpeg | openssl base64 -d
```

flag: `infosec_flagis_wearejuststarting`

## Level 3

Dependencies:
zbar-tools
bsdgames

```
$ curl -s http://ctf.infosecinstitute.com/levelthree.php | grep img
$ wget http://ctf.infosecinstitute.com/img/qrcode.png
$ zbarimg -q --raw qrcode.png | morse -ds
```

flag: `INFOSECFLAGISMORSING`

## Level 4

```
$ alias rot13="tr '[A-Za-z]' '[N-ZA-Mn-za-m]'"; curl -vvv http://ctf.infosecinstitute.com/levelfour.php 2>&1 | grep Set-Cookie | awk '{print $3}' | rot13
```

flag: `infosec_flagis_welovecookies`

## Level 5

Dependencies:
steghide

```
$ curl -s http://ctf.infosecinstitute.com/levelfive.php | grep img
$ wget http://ctf.infosecinstitute.com/img/aliens.jpg
$ steghide --extract -q -p '' -sf aliens.jpg -xf - | python3 -c "import binascii; x=int(input(),2); print(binascii.unhexlify('%x' % x))"
```

flag: `infosec_flagis_stegaliens`

## Level 6

```
$ wget http://ctf.infosecinstitute.com/misc/sharkfin.pcap
$ strings sharkfin.pcap | head -1 | xxd -p -r
```

flag: `infosec_flagis_sniffed`

## Level 7

```
$ curl -s -v http://ctf.infosecinstitute.com/levelseven.php 2>&1 | grep 200 | awk '{print $4}' | openssl base64 -d
```

flag: `infosec_flagis_youfoundit`

## Level 8

```
$ wget http://ctf.infosecinstitute.com/misc/app.exe
$ strings app.exe | grep flag
```

flag: `infosec_flagis_0x1a`

## Level 9

```
$ lynx -dump -listonly http://www.google.com/search?q='Cisco IDS default password' | grep cisco | head -1 | awk '{print $2}' | lynx -dump - | grep 'default password'
$ curl -s --data 'username=root&password=attack' http://ctf.infosecinstitute.com/levelnine.php | grep alert | grep -o "'.*'" | sed "s/'//g" | rev
```

## Level 10

Dependencies:
audacity

```
$ wget http://ctf.infosecinstitute.com/misc/Flag.wav
$ audacity Flag.wav
(audacity GUI) Adjust 'Playback speed' to hear recognizable audio
```

flag: `infosec_flagis_found`

## Level 11

```
$ curl -s http://ctf.infosecinstitute.com/leveleleven.php | grep img
$ wget http://ctf.infosecinstitute.com/img/php-logo-virus.jpg
$ strings php-logo-virus.jpg | head -3 | tail -1 | awk -F'_' '{print $3}' | base64 -d
```

flag: `infosec_flagis_powerslide`

## Level 12

```
$ diff -u <(curl -s http://ctf.infosecinstitute.com/leveleleven.php) <(curl -s http://ctf.infosecinstitute.com/leveltwelve.php) | grep design | grep -o '".*"' | awk '{print $1}' | sed 's/"//g'
$ curl -s 'http://ctf.infosecinstitute.com/css/design.css' | grep color | awk '{print $2}' | xxd -r -p
```

flag: `infosec_flagis_heyimnotacolor`

## Level 13

```
$ curl -s ctf.infosecinstitute.com/levelthirteen.php.old
$ wget ctf.infosecinstitute.com/misc/imadecoy
$ wireshark imadecoy (GUI -> File -> Export Objects -> HTTP -> HoneyPY.PNG)
$ ristretto HoneyPY.PNG
```

flag: `infosec_flagis_morepackets`

## Level 14

```
$ curl -s http://ctf.infosecinstitute.com/misc/level14 | grep '(104' | tail -1 | awk '{print $2}' | tr -d "',\\\\u00" | xxd -p -r
```

flag: `infosec_flagis_whatsorceryisthis`

## Level 15

```
$ curl -s http://ctf.infosecinstitute.com/levelfifteen/.hey
$ firefox http://crypo.in.ua/tools/eng_atom128d.php
```

flag: `infosec_flagis_rceatomized`
