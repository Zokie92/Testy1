Quake 1 Multiplayer Setup på Mac (M1) med Docker

Översikt

Denna guide dokumenterar hela processen för att få Quake 1 multiplayer att fungera på en Mac M1 med Docker. 
Den innehåller alla steg, felsökningar och lösningar vi gick igenom. Guiden täcker både server och klient.

⸻

1. Miljö och krav
	•	OS: macOS (M1)
	•	Docker & Docker Compose installerat
	•	Quake 1 pak-filer: pak0.pak och pak1.pak
	•	Klient: DarkPlaces 3504 (Mac build)
	•	Server image: moisesber/quaken:latest (Docker)
	•	Ports: 26000 UDP (Quake), 51820 UDP (WireGuard om VPN används)

2. Problem och felsökning

2.1 Klientproblem med Host_Error
	•	Symptom:
Klienten (VkQuake) fick felmeddelande:
Host_Error: Server returned version 3504, not 15 or 666 or 999.
Unknown command "cl_serverextension_download"

	•	Lösning:
Klienten och servern måste ha samma version. Vi bytte klient till DarkPlaces desktop build för Mac (version 3504).

⸻

2.2 DarkPlaces kunde inte hitta filer
	•	Symptom:
W_LoadWadFile: couldn't load gfx.wad
Check that this has an id1 subdirectory containing pak0.pak and pak1.pak
	•	Åtgärder:
	1.	Skapa en mappstruktur:
    ~/Documents/Quake/id1/pak0.pak
    ~/Documents/Quake/id1/pak1.pak

2.	Starta DarkPlaces med basdir:
~/Documents/DarkPlaces.app/Contents/MacOS/darkplaces-sdl -basedir ~/Documents/Quake

2.3 Docker-compose problem på M1
	•	Symptom:
    The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8)

	•	Lösning:
Lägg till platform: linux/amd64 i serversektionen i docker-compose.yml för kompatibilitet med M1.

2.4 Inga konfigurationsfiler hittades
	•	Symptom:
no configuration file provided: not found

	•	Åtgärd:
Navigera till rätt katalog där docker-compose.yml finns och kör:
docker-compose -f ./docker-compose.yml up -d

3. Server Setup

3.1 Docker-compose fil
services:
  quake_server:
    image: moisesber/quaken:latest
    container_name: quake_server
    platform: linux/amd64
    ports:
      - "26000:26000/udp"
    volumes:
      - ./quake_data:/nquakesv/id1
      - ./quake_data/maps:/nquakesv/modules
    environment:
      - GAME_DIR=id1
      - CONFIG_FILE=server.cfg
    restart: unless-stopped

networks:
  default:
    driver: bridge

3.2 Server konfiguration (server.cfg)
hostname "Mattias Quake Server"
deathmatch 1
fraglimit 20
timelimit 15
map e1m1
sv_maxclients 8

3.3 Starta server
docker-compose up -d
docker exec -it quake_server /bin/sh
cd /nquakesv
./quake_server -basedir /nquakesv -config server.cfg

4. Klient Setup
	1.	Ladda ner DarkPlaces 3504 för Mac.
	2.	Placera .app i t.ex. ~/Documents/.
	3.	Se till att pak-filer ligger i ~/Documents/Quake/id1/.
	4.	Starta klienten:
    ~/Documents/DarkPlaces.app/Contents/MacOS/darkplaces-sdl -basedir ~/Documents/Quake

	5.	Anslut till servern:
    connect 127.0.0.1:26000


    5. Multiplayer Tips
	•	Kontrollera att port 26000 UDP är öppen i firewall/router.
	•	Lägg till fler kartor i quake_data/maps/.
	•	Testa med korta kartor först innan större banor läggs till.
	•	Max antal klienter styrs av sv_maxclients i server.cfg.

⸻

6. Felsökningslogg
Datum
Problem
Lösning
2025-11-19
DarkPlaces kunde inte hitta pak0.pak
Lade pak-filer i ~/Documents/Quake/id1/ och använde -basedir
2025-11-19
Klienten host_error version mismatch
Bytte klient till DarkPlaces 3504
2025-11-19
Docker M1 plattformsproblem
La till platform: linux/amd64 i compose
2025-11-19
No configuration file
Navigerade till rätt katalog innan docker-compose up

7. Tidslinje
	•	Start: ca 2025-11-18
	•	Test och felsökning klient/server: 2025-11-19
	•	Docker setup och port mapping: 2025-11-19
	•	Multiplayer anslutning testad: 2025-11-19
Tidsåtgång:
- Totalt ca 8 timmar, inklusive felsökning och testning.


    https://www.macsourceports.com/game/quake