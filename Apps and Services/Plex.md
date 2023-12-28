---
created: Tue 2023-05-02 @ 06:32 PM
modified: Wed 2023-12-27 @ 03:45 PM
---
# Install

### CLI

```
docker create --name='Plex' --net='media' -e TZ="America/Los_Angeles" -e 'PLEX_CLAIM'='claim-tUXXX9QP-76ggRELExz' -e 'PLEX_UID'='99' -e 'PLEX_GID'='100' -e 'VERSION'='latest' -p '32400:32400/tcp' -v '/tmp/':'/transcode':'rw' -v '/mnt/user/data/media/':'/data/media':'rw' -v '/mnt/user/appdata/Plex-Media-Server':'/config':'rw' 'plexinc/pms-docker'
```

### Portainer Stack / Compose

From [this](https://docs.linuxserver.io/images/docker-plex/) page

```yaml
version: "2.1"
services:
  plex:
    image: lscr.io/linuxserver/plex:latest
    container_name: plex-media-server
    network_mode: host
    environment:
      - PUID=1010
      - PGID=1000
      - TZ=America/Los_Angeles
      - VERSION=docker
      - PLEX_CLAIM= #optional - go to https://plex.tv/claim for code
    volumes:
      - /mnt/SSD/plex-media-server:/config
      - /mnt/SSD/transcodes:/transcodes
      - /mnt/Media:/media
    ports:
      - 32400:32400
    restart: unless-stopped
```


# Settings

*These settings are noted here because PMM had issues until things were set a certain way. So these are the settings that have been working for me both with PMM and with how I want the server to work.*

#### My-Plex-Username
*anything not mentioned is assumed to be intuitive*

- Account
	- Audio & Subtitles
		- Automatically select audio and subtitle tracks is checked
		- Preferred Audio Language - English
		- Preferred Subtitle Language - English
		- Auto-Select Subtitle Mode - Shown with Foreign Audio
		- Subtitles for the Deaf or Hard-of-Hearing Searches - Prefer non-SDH subtitles
		- Forced Subtitle Searches - Prefer non-forced subtitles
	- Experimental Features - Enabled

- Online Media Sources
	- Live TV, Movies & TV - Disabled for Managed Users
	- Music - Disabled
	- Discover Source - Disabled for  Managed Users
	- Display "More Ways to Watch" search results - Disabled for Managed Users
	- Display availabilities from other streaming services on detail pages - Enabled

- Streaming Services
	- I selected all of the most popular services

- Plex Home
	- Guest is disabled
	- WifesName
		- Access to Movies, Shows
		- Restriction Profile - none
		- Allow Downloads - Enabled
		- Live TV & DVR Access - Allow Live TV only
		- No restrictions on Movies, TV Shows or Music
	- Daycare Kids
		- Access to Movies, Shows
		- Restriction Profile - none
		- Allow Downloads - Disabled
		- Live TV & DVR Access - No Live TV and DVR access
		- Movies - Allow only 3, 4, 5
		- TV Shows - Allow only 3, 4, 5, Daycare
		- Music - no restrictions
	-  MyOlderKidsName (has his own account that is added to our Home)
		- Access to Movies, Shows
		- Allow Downloads - Enabled
		- Live TV & DVR Access - Allow Live TV only
		- Movies - Allow only 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, MyYoungerKidsName, MyOlderKidsName
			- Exclude "Not-MyOlderKidsName"
		- TV Shows - Allow only 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, MyOlderKidsName
			- Exclude "Not-MyOlderKidsName"
		- Music - no restrictions
	- MyYoungerKidsName
		- Access to Movies, Shows
		- Restriction Profile - none
		- Allow Downloads - Enabled
		- Live TV & DVR Access - Allow Live TV only
		- Movies - Allow only 3, 4, 5, 6, 7, 8, 9, 10, 11, MyYoungerKidsName
			- Exclude "Not-MyYoungerKidsName"
		- TV Shows - Allow only 3, 4, 5, 6, 7, 8, 9, 10, 11, MyYoungerKidsName
			- Exclude "Not-"MyYoungerKidsName"
		- Music - no restrictions

- Manage Library Access
	- Just do what is intuitive


#### Plex Web
*make sure you check "Show Advanced" at the top of the first settings page you go to*

- General
	- Language - English
	- Automatically Sign In - Not checked
	- Remember Selected Tab - Checked
	- Play Theme Music - Not checked
	- Allow Fallback to Insecure Connections - On same network as server
	- Time Format - 12 hour
	- Setting Sync is turned on

- Quality
	- Automatically Adjust Quality - not checked
	- Internet Streaming Video Quality - 2 Mbps, 720p
	- Play smaller videos at original quality - checked
	- Home Streaming / Use recommended settings - checked

- Debug
	- Debug level - disabled
	- Direct Play - checked
	- Direct Stream - checked
	- Use alternate streaming protocol for video playback - not checked

- Player
	- Normalize Multi-channel Audio - checked
	- Subtitles - White, bottom, normal
	- Burn Subtitles - Automatic
	- Cinema trailers to Play before Movies - none
	- Force multi-channel audio support - not checked

#### Status
- nothing to do here


#### Settings

- General
	- Send crash reports to Plex - checked
	- Push notifications - checked
	- Enable Plex Media Server debug logging - checked
	- Enable Plex Media Server verbose logging - not checked
	- Server update channel - Public

- Remote Access
	- Enable Remote Access as per instructions
	- Manually specify public port - checked
		- 32400
	- Internet upload speed - 20 Mbps
	- Limit remote stream bitrate - 8 Mbps

- Agents (anything not mentioned is unchecked)
	- Movies
		- Personal Media
			- Personal Media is checked by default
			- Local Media Assets (Movies) - checked
		- Plex Movie (Legacy)
			- Local Media Assets (Movies) - checked
			- Plex Movie (Legacy) - check by default
		- The Movie Database
			- Local Media Assets (Movies) - checked
			- The Movie Database - checked by default
	- Shows
		- Personal Media Shows
			- Personal Media Shows - checked by default
			- Local Media Assets (TV) - checked
		- TheTVDB
			- Local Media Assets (TV) - checked
			- TheTVDB - checked by default
			- Plex Theme Music - checked
		- The Movie Database
			- The Movie Database - checked by default
	- Artists
		- Personal Media Artists
			- Personal Media Artists - checked by default
			- Local Media Assets (Artists) - checked
		- Last.fm
			- Local Media Assets (Artists) - checked
			- Last.fm - checked by default
			- Home Theatre Backdrops - checked
	- Albums
		- Personal Media Albums
			- Personal Media Albums - checked by default
			- Local Media Assets (Albums) - checked
		- Last.fm
			- Local Media Assets (Albums) - checked
			- Last.fm - checked by default
			- LyricFind - checked
	- Photos
		- Photos
			- Photos - checked by default

- Library
	- Scan my library automatically - unchecked
	- Run a partial scan when changes are detected - checked
	- Include music libraries in automatic updates - unchecked
	- Scan my library periodically - checked
		- Interval - hourly
	- Empty trash automatically after every scan - checked
	- Allow media deletion - checked
	- Weeks to consider for Continue Watching - 12
	- Maximum number of Continue Watching items which will appear - 20
	- Include season premieres in Continue Watching - checked
	- Video played threshold - 90%
	- Video play completion behavior - at final credits marker position
	- Enable smart shuffling on artists and smart music playlists - checked
	- Group albums by type - enabled
	- Run scanner tasks at a lower priority - checked
	- Marker source - "both, try online first"
	- Generate video preview thumbnails - as a scheduled task and when media is added
	- Generate intro video markers - as a scheduled task and when media is added
	- Generate credits video markers - as a scheduled task and when media is added
	- Generate chapter thumbnails - as a scheduled task and when media is added
	- Analyze audio tracks for loudness - as a scheduled task and when media is added
	- Analyze audio tracks for sonic features - as a scheduled task and when media is added
	- Database Cache Size (MB) - 4096

- Plugins
	- Region - United States
	- Everything else unchecked

- Network
	- Enable server support for IPv6 - unchecked
	- Secure connections - preferred
	- Certificate boxes are empty
	- Preferred network interface - pick the service LAN interface you use
	- Strict TLS configuration - checked
	- Enable local network discovery (GDM) - checked
	- Remote streams allowed per user - 1
	- LAN Networks - 192.168.10.0/28, 192.168.20.0/24, 192.168.30.0/29, 192.168.50.0/24, 192.168.70.0/24
	- Terminate Sessions paused for longer than - 20
	- Treat WAN IP as LAN bandwidth - unchecked
	- Enable Relay - unchecked
	- URL and IP boxes are empty
	- Webhooks - checked

- Transcoder
	- Transcoder quality - Make my CPU hurt
	- ==Transcoder temporary directory - ______________________==
	- Transcoder default throttle buffer - 60
	- Background transcoding x264 preset - fast
	- Enable HDR tone mapping - checked
	- Disable video stream transcoding - unchecked
	- Use hardware acceleration when available - unchecked
	- Use hardware-accelerated video encoding - checked
	- Maximum simultaneous video transcode - 4

- Languages
	- Automatically select audio and subtitle tracks - checked
	- Prefer audio tracks in - English
	- Subtitle mode - Shown with foreign audio
	- Prefer subtitles in - English

- DLNA
	- Enable the DLNA server - unchecked
	- It is recommended by Plex not to mess with any of the advanced settings here.

- Scheduled Tasks
	- Time at which tasks start to run - 2:00
	- Time at which tasks stop running - 5:00
	- Backup database every three days - checked
	- Backup directory - "/var/lib/plexmediaserver/Library/Application Support/Plex Media Server/Plug-in Support/Databases"
	- Optimize database every week - checked
	- Remove old bundles every week - checked
	- Remove old cache files every week - checked
	- Refresh local metadata every three days - checked
	- Update all libraries during maintenance - unchecked
	- Upgrade media analysis during maintenance - checked
	- Refresh library metadata periodically - checked
	- Perform extensive media analysis during maintenance - checked

 - Extras
	 - Choose Cinema Trailers from - All movies
	 - Include Cinema Trailers from movies in my library - checked
	 - Include Cinema Trailers from new and upcoming movies in theaters - unchecked
	 - Include Cinema Trailers from new and upcoming movies on Blu-ray - unchecked
	 - Movie pre-roll video - empty
	 - Global music videos path - empty

#### Manage

- Libraries
	
	 - Movies
		- General
			- Library Type - Movies
			- Name - Movies
			- Language - English
		- Add Folders
			- `/mnt/Media/Video/movies`
		- Advanced
			- Scanner - Plex Movie
			- Agent - Plex Movie
			- Visibility - Include in home screen and global search
			- Enable Cinema Trailers - checked
			- Certification Country - United States
			- Use original titles - unchecked
			- Prefer artwork based on library language - checked
			- Use local assets - checked
			- Prefer local metadata - unchecked
			- Find extras - checked
			- Only show trailers - unchecked
			- Allow red band trailers - unchecked
			- Localized subtitles - checked
			- Include adult content - unchecked
			- Minimum automatic collection size - Disabled
			- Ratings Source - Rotten Tomatoes
			- Enable video preview thumbnails - checked
			- Collections - Show collections and their items
			- Enable credits detection - checked
	
	 - Shows
		- General
			- Library Type - TV Shows
			- Name - Shows
			- Language - English
		- Add Folders
			- `/mnt/Media/Video/shows`
		- Advanced
			- Scanner - Plex TV Series
			- Agent - Plex Series
			- Visibility - Include in home screen and global search
			- Episode sorting - Library default
			- Certification Country - United States
			- Episode Ordering - TheTVDB
			- Use season titles - checked
			- Use original titles - unchecked
			- Prefer artwork based on library language - checked
			- Use local assets - checked
			- Prefer local metadata - unchecked
			- Find extras - checked
			- Only show trailers - unchecked
			- Allow red band trailers - unchecked
			- Localized subtitles - checked
			- Include adult content - unchecked
			- Enable video preview thumbnails - checked
			- Collections - Show collections and their items
			- Seasons - Hide for single season series
			- Enable intro detection - checked
			- Enable credits detection - checked

	 - Music
		- General
			- Library Type - Music
			- Name - Music
			- Language - English
		- Add Folders
			- `/mnt/Media/Audio/music`
		- Advanced
			- Scanner - Plex Music
			- Agent - Plex Music
			- Visibility - Include in home screen and global search
			- Album sorting - Library default
			- Sonic Analysis - checked
			- Prefer local metadata - unchecked
			- Store track progress - unchecked
			- Include related content from shared libraries - unchecked
			- Artist Bios - checked
			- Album Reviews and Critic Ratings - unchecked
			- Popular Tracks - checked
			- Find Lyrics - checked
			- Genres - Plex Music
			- Album Art - Both Plex Music and Local Files

- Optimized Versions
	- nothing to change here

- Live TV & DVR
	- I don't use this so I leave it not set up

- Troubleshooting
	- nothing to change here

- Console
	- nothing to do here

- Plugins
	- nothing to change here
