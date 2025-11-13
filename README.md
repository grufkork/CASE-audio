# CASE Audio
Here is documentation and configs for the CASE audio system.

Files:
- `librespot-handler.sh` - Put in `/usr/local/bin/` and make executable. Handles some callbacks from raspotify
- `conf` - Raspotify config. Put at `/etc/raspotify/conf`, probably requires sudo


`raspotifyconf_old-251113` is an old config from before an update that broke streaming and updated the config.

## Lösta problem

### 2025-11-13 
pajade systemet. I raspotify-loggen kunde ses att låtarna inte var tillgängliga. apt upgrade-ade, vilket skrev över configen men efter att ha återinfört inställningarna var allt igång och snurrade igen.

### Lite tidigare
Allt ljud i multirummet var paj, det var trasig kabel efter att Bmann m.fl. varit och pillat i connectorn mellan ljudkort och pi. Nya kablar så funkade det. Dock surrade en av högtalarna och hade låg volym - typiskt för trasig kabel då högtalarna använder en balanserad signal. En av kablarna hade gått av i kontakten till ljudkortet, fixade och då funkade allt galant igen.


# Övrigt
*Nedan är lite anteckningar från när jag reverse-engineerade systemet, förhoppningsvis ger detta den som läser en lite bättre chans.*

Högtalarna kommer i par, där den ena är aktiv och tar in en signal och driver den andra högtaleren i kedjan som är passiv.




Ljudkortet som sitter i projektrummet: https://www.3e-audio.com/dsp/adau1701-2in4out/

### **Projektrummet** (case-audio.com)
Pi:n i projektrummet streamar musiken från spotify och skickar ut ljudet via snapserver som körs på samma pi.

Spotify -> Raspotify (som använder på Librespot) -> SnapServer

Raspotify använder librespot för själva streamingen. Se `conf` för hela configen.

Raspotify-configen har `LIBRESPOT_BACKEND="pipe"`

Snapserver-configen har `source = pipe:///run/snapfifo?name=default&mode=create`


Raspotify har en service som startar librespot


### **Klienter** (inklusive projekrummet)
Alla Pi:s inklusive projektrummet kör snapclient för att ta emot ljud från snapserver och spelar upp det antingen via ADAU1701-ljudkortet eller hörlursuttaget.

(INTERNET) -> Snapclient -> ALSA -> I2S -> Kontakt J5 -> ADAU1701
Oklart hur utenhet specificeras av snapclient

ALSA snackar I2S till en ADAU1701 (som sitter i lådan)


## Kommandon
- `snapclient -l` listar ljudenheter
- `snapclient -s [INDEX]` startar snapcast med vald ljudenhet 

- `alsamixer` visuell mixer för ALSA-enheter, har inga kontroller för ljudkortet dock
- `sudo systemctl snapserver/snapclient status/stop/start/restart`
- `journalctl -u snapserver/snapclient/raspotify -f` kolla loggar för att debugga

###  Configs
- /etc/snapserver.conf
- /etc/raspotify/conf (requires sudo)


## ADAU1701 J5 pinout till raspberryn
| ADAU1701 J5 | RPI                    |                      |
| ----------- | ---------------------- | -------------------- |
| 2 GND       | 39 GND                 |                      |
| 3 DATA_IN_0 | 40 PCM_DOUT (data out) |                      |
| 7 LRCLK_IN  | 35 PCM_FS (frame sync) | höger/vänster select |
| 5 BCLK_IN   | 12 PCM_CLK             | bit clock            |
| 9           | 11 GPIO17 (???)        | (indikatorlampa??)   |

## BIT CONNECT
- 2 GND <- 39 GND
- 3 DATA_IN_0 <- 40 PCM_DOUT (Data out)
- 5 BCLK_IN  <- 
- 7 LRCLK_IN
- <- 12 PCM_CLK
