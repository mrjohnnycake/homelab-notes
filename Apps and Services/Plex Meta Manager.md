---
created: Tue 2023-05-02 @ 06:33 PM
modified: Wed 2023-12-27 @ 07:55 PM
---
*These are my actual configs (except for my config.yml file as that contains sensitive info) and will be updated as I get to them.*

For more info see my GitHub "plex-meta-manager" repo

## Updating Git Repo
*this header is up top on this note so that I don't forget to do it*

I have my PMM files synced with git so that I can edit them on my computer and then push them to my repo on GitHub. I then log on to my server to fetch or pull the new changes so that everything is ready to go.

If you're using git to update your PMM files you'll need to push changes from your server to your git repo server (GitHub, etc.) every once in a while. This is because PMM creates new files (mostly assets like posters, etc.) and if you don't add and push those files back to the master repo then some of the purpose of using git is defeated.

Run these commands on your server from time to time to keep everything updated:
```
git status

git add --all

git commit -m 'type out your commit message here'

git push
```

- I have added `git pull` to crontab so that changes that I make on my computer and push to GitHub are pulled to the server automatically once a day. See my [[Backup and Maintenance]] note for that info.


# Docker Install

## Initial Setup

This docker install is different that your average container install as you need to set up the directories and configs first AND THEN run them with a docker command. It'll make sense as you read along.

```
sudo mkdir /docker/appdata/plex-meta-manager

cd /docker/appdata/plex-meta-manager

sudo mkdir config

sudo mkdir config/assets
```

- Option 1: If this is the first time you are installing this and want to start from scratch:
```
sudo curl -fLvo config/config.yml https://raw.githubusercontent.com/meisnate12/Plex-Meta-Manager/master/config/config.yml.template

sudo nano config/config.yml
```

- Option 2: If you want to copy a pre-existing temple (like mine found below):
```
sudo vim /docker/appdata/plex-meta-manager/config/config.yml
```
* Then paste in your config

Go [here](https://github.com/meisnate12/Plex-Meta-Manager-Configs) and pick out a template that looks good. You can always change it later.


## Running

#### One Time

- To run the entire normal run AND you see the output, run this:
```
sudo docker run --rm -it -v "/docker/appdata/plex-meta-manager/config:/config:rw" meisnate12/plex-meta-manager --run
```
- If you get an error where it just won't run but you're sure your setup is good, update the docker image and then try again
- The container will show up in Portainer but will be stopped after running once

To run a specific YAML file only (and skip the rest):
```
sudo docker run --rm -it -v "/docker/appdata/plex-meta-manager/config:/config:rw" meisnate12/plex-meta-manager --run-metadata-files "A-C.yml"
```

#### Scheduling

- To keep the container running all the time and have PMM run every day at 5a (that's by it's design), run this command once:
```
sudo docker run -d --restart=unless-stopped -e TZ=America/Los_Angeles -v "/docker/appdata/plex-meta-manager/config:/config:rw" --name plex-meta-manager meisnate12/plex-meta-manager
```
- You will see the container running in Portainer
- "-d" makes it run in the background and not show a console window (with all the output)


Further notes on installation:
https://metamanager.wiki/en/latest/home/guides/docker.html



# Configuring #

## File Structure ##

	config
		assets
			movies
			music
			shows
		collections
			movies
			shows
		logs
		metadata
			movies
				Your Movie (2017).yml
				OR your-movies-a-thru-c.yml (I used this method)
			music
			shows
		overlays
			movies
			shows
		playlists
			video.yml
		----------
		config.cache
		config.yml
		UUID


# Configs #

## Notes ##

- To show genre collections in the main library view, set movie collections in Plex (Movies->Advanced->Collections) to "Show collections and their items". This will make all collections show up in the library so for ones you don't want to see (other than in the Collections tab), add "collection_mode: hide" to the yml files as needed.


## config.yml ##

config/config.yml
```yaml
libraries:

#######################################################
#   LIBRARY NAMES MUST MATCH THE PLEX LIBRARY NAMES   #
#######################################################

  Movies:
    metadata_path:
    - folder: //config/collections/movies/
    - folder: //config/metadata/movies/
    settings:
      asset_directory: //config/assets/movies/
      overlay_path: //config/overlays/movies/
      remove_overlays: true
    operations:
      mass_content_rating_update: mdb_commonsense
      content_rating_mapper:
        PG: 12
        PG-13: 13
        TV-14: 15
        NR: 17
        Not Rated: 17
        R: 18
    report_path: /config/reports/Movies.yml

  Music:
    metadata_path:
    - folder: //config/metadata/music/
    settings:
      asset_directory: //config/assets/music/
    report_path: /config/reports/Music.yml

  Shows:
    metadata_path:
    - folder: //config/collections/shows/
    - folder: //config/metadata/shows/
    - pmm: country
      template_variables:
        use_other: false
        use_separator: false
        sep_style: purple
        collection_mode: hide
        include:
        - gb
        - kr
        sort_by: title.asc
    - pmm: emmy
      template_variables:
        collection_mode: hide
        collection_order: alpha
        data:
          starting: current_year-5
          ending: current_year
    - pmm: trakt
      template_variables:
        collection_mode: hide
        use_collected: false
        use_popular: false
        use_recommended: false
        use_watched: false
        limit: 20
        visible_library_trending: true
        visible_home_trending: false
        visible_shared_trending: true
    - pmm: basic
      template_variables:
        collection_mode: hide
        use_episodes: false
        use_released: true
        in_the_last_released: 120
        visible_library_released: true
        visible_home_released: false
        visible_shared_released: false
      settings:
        asset_directory: //config/assets/shows/
        overlay_path: //config/overlays/shows/
        remove_overlays: true
    operations:
      mass_content_rating_update: mdb_commonsense
      content_rating_mapper:
        TV-Y: 4
        TV-Y7: 7
        TV-G: 7
        TV-PG: 12
        TV-14: 14
        TV-MA: 18
    report_path: /config/reports/Shows.yml

playlist_files:
  - folder: //config/playlists/

settings:
  cache: true
  cache_expiration: 60
  asset_directory:
  asset_folders: true
  asset_depth: 1
  create_asset_folders: true
  prioritize_assets: true
  dimensional_asset_rename: false
  download_url_assets: true
  show_missing_season_assets: true
  show_missing_episode_assets: false
  show_asset_not_needed: true
  sync_mode: append
  minimum_items: 2
  default_collection_order:
  delete_below_minimum: true
  delete_not_scheduled: false
  run_again_delay: 2
  missing_only_released: false
  only_filter_missing: false
  show_unmanaged: true
  show_filtered: false
  show_options: false
  show_missing: false
  show_missing_assets: true
  save_report: true
  tvdb_language: eng
  ignore_ids:
  ignore_imdb_ids:
  item_refresh_delay: 0
  playlist_sync_to_user:
  playlist_report: false
  verify_ssl: true
  custom_repo:
  check_nightly: false
  show_unconfigured: true
  playlist_exclude_users:
webhooks:                            # Can be individually specified per library as well
  error:
  version:
  run_start:
  run_end:
  changes:
  delete:

plex:
  url: http://192.168.70.70:32400
  token: RmpEjdfhdfhdfftEQqpzkHY6
  timeout: 60
  clean_bundles: false
  empty_trash: false
  optimize: false

tmdb:                                                   # REQUIRED for the script to run
  apikey: b2cb2917asdgsd7f151
  language: en
  cache_expiration: 60
  region:

trakt:
  client_id: 0f2f2ac682fbb7d4bb86ecd91313c3a4311dae25b51099236aa12c1bb516
  client_secret: fa28266c8e710e82bf1ad8a21b2ba47938f3b413b938b5702f65e224ea8dd
  pin:
  authorization:                          # everything below is autofilled by the script
    access_token: c5dc968c87316516742b1a7f576c6d4699b03e37964925ccce1018226026
    token_type: Bearer
    expires_in: 7889237
    refresh_token: e1e3bbc6b6a54413a113541592e92cccf2bfcc62c0d36fccbfe4fc168cd
    scope: public
    created_at: 1694735079
mdblist:
  apikey: 3aq477s316514baunimjnygz
  cache_expiration: 60

# radarr:
#   url: http://192.168.40.45:7878
#   token: f21559376516514c31c013e1e6ac
#   add_missing: false
#   add_existing: false
#   root_folder_path: S:/movies
#   monitor: true
#   availability: announced
#   quality_profile: HD-1080p
#   tag:
#   search: false
#   radarr_path:
#   plex_path:
#   upgrade_existing: false

# sonarr:
#   url: http://192.168.40.45:8989
#   token: 0f9e7a33543242e1f43ae7d86fe
#   add_missing: false
#   add_existing: false
#   root_folder_path: S:/shows
#   monitor: all
#   quality_profile: HD-1080p
#   language_profile: English
#   series_type: standard
#   season_folder: true
#   tag:
#   search: false
#   cutoff_search: false
#   sonarr_path:
#   plex_path:
#   upgrade_existing: false
```
- All tokens, keys and IDs have been changed to share this config with the internet and not expose my personal info. Add the correct ones from your services as needed.
* Google "find plex token" to learn how to get your specific token
* For Trakt to work, enter the Client ID and Client Secret found [here](https://trakt.tv/oauth/applications) and then run the docker script. The Trakt part will fail but when you look at the output it will give you a link to go to. Copy the PIN from that site and enter in the config, save and run the docker script again. It will succeed this time and when you check the config again it will have removed the PIN and entered the information (as is show above).


## Movies

#### charts

config/collections/movies/Charts.yml
```
collections:

############################
##        TRENDING        ##
############################
  
  New Releases:
    trakt_list: https://trakt.tv/users/giladg/lists/latest-releases
    url_poster: https://i.ibb.co/VNwwdZQ/New-Releases.jpg
    url_background: https://i.imgur.com/juAQnOG.jpg
    collection_mode: hide
    sort_title: "*101a"
    schedule: daily
    sync_mode: sync
    smart_label: critic_rating.desc

  Trending:
    trakt_trending: 100
    sync_mode: sync
    collection_mode: hide
    collection_order: custom
    sort_title: "*101b"
    schedule: daily
    url_poster: https://i.imgur.com/1qbVXLi.png

############################
##        BEST OF         ##
############################

  Best Picture:
    imdb_list: https://www.imdb.com/search/title/?title_type=feature&groups=best_picture_winner,oscar_best_picture_nominees
    sync_mode: sync
    collection_mode: hide
    collection_order: custom
    sort_title: "*101c"
    schedule: daily
    url_poster: https://i.imgur.com/nNTPdYS.png
    url_background: https://i.imgur.com/juAQnOG.jpg

  Oscar Winners:
    imdb_list: https://www.imdb.com/search/title/?title_type=feature,documentary&groups=oscar_winner
    url_poster: https://theposterdb.com/api/assets/192593
    url_background: https://i.imgur.com/rVa7jiJ.jpg
    summary: Oscar Winning Movies
    sync_mode: sync
    collection_mode: hide
    collection_order: custom
    sort_title: "*101d"
    schedule: daily

############################
##         POPULAR        ##
############################

  1001 Movies You Must See Before You Die:
    trakt_list: https://trakt.tv/users/sp1ti/lists/1001-movies-you-must-see-before-you-die
    summary: "1001 Movies You Must See Before You Die is a film reference book edited by Steven Jay Schneider with original essays on each film contributed by over 70 film critics."
    url_poster: https://i.imgur.com/WmgioMY.png
    url_background: https://i.imgur.com/rVa7jiJ.jpg
    sync_mode: sync
    collection_mode: hide
    collection_order: custom
    sort_title: "*101e"
    schedule: daily
```


#### countries ####

config/collections/movies/Countries.yml
```
#######################
##     Templates     ##
#######################

templates:

  Country:
    url_poster: <<poster>>
    sort_title: <<collection_name>>
    collection_order: release
    # schedule: weekly(sunday)
    delete_not_scheduled: true
    run_again: true
    visible_home: false
    visible_shared: true
    sync_mode: sync

#######################
##     Countries     ##
#######################

collections:

  British:
   template: {name: Country, poster: https://theposterdb.com/api/assets/158488}
   smart_filter:
     all:
       country: United Kingdom
     sort_by: release.desc
   summary: A collection of British Films.

  French:
   template: {name: Country, poster: https://theposterdb.com/api/assets/158488}
   smart_filter:
     all:
       country: France
     sort_by: release.desc
   summary: A collection of French Films.

  German:
   template: {name: Country, poster: https://theposterdb.com/api/assets/158488}
   smart_filter:
     all:
       country: Germany
     sort_by: release.desc
   summary: A collection of German Films.

  Japanese:
   template: {name: Country, poster: https://theposterdb.com/api/assets/158488}
   smart_filter:
     all:
       country: Japan
     sort_by: release.desc
   summary: A collection of Japanese Films.

  Korean:
    template: {name: Country, poster: https://theposterdb.com/api/assets/158483}
    smart_filter:
      all:
        country: Republic of Korea
      sort_by: release.desc
    summary: A collection of Korean Films.

  Spanish Language:
   template: {name: Country, poster: https://theposterdb.com/api/assets/158488}
   plex_all: true
   filters:
     original_language: es
   summary: A collection of Spanish language films.
```


#### franchises ####

config/collections/movies/Franchises.yml
```yaml
#######################
##     Templates     ##
#######################

templates:

    Franchise:
        url_poster: <<poster>>
        sort_title: ++++_<<collection_name>>
        collection_order: release
        delete_not_scheduled: true
        run_again: true
        visible_home: false
        visible_shared: true
        sync_mode: sync

########################
##     Franchises     ##
########################

collections:

  Alien:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/}
    trakt_list: https://trakt.tv/users/barffy194/lists/aliens?sort=released,desc
    summary: The Alien franchise is a science fiction horror franchise, consisting primarily of a series of films focusing on the voracious extraterrestrial endoparasitoid species Xenomorph XX121, commonly referred to simply as "the Alien".

  Body Snatchers:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/}
    trakt_list: https://trakt.tv/users/mrjohnnycake/lists/body-snatchers-copy?sort=released,desc
    summary: Based on Jack Finney’s novel about an insidious and silent alien invasion that threatens the world’s population, The Invasion of the Body Snatchers is one of the most prolific stories in all of horror, and, which employs a handful of standard noirish elements and leaves the story open to countless social and political interpretations.

  Bourne:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/5420}
    imdb_list: https://www.imdb.com/list/ls026772377/
    summary: The Bourne franchise consists of action-thriller installments based on the character Jason Bourne, created by author Robert Ludlum. The franchise includes five theatrical films, and a spin-off prequel television series.

  Cloverfield:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/}
    trakt_list: https://trakt.tv/users/mateoxw10/lists/saga-cloverfield?sort=rank,asc
    summary: Cloverfield is an American science fiction anthology film series created and by J. J. Abrams consisting of three films, all set in a shared fictional universe.

  Die Hard:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/}
    trakt_list: https://trakt.tv/users/theriskoftime/lists/die-hard?sort=released,desc
    summary: An action film series centered around the character of John McClane, a New York City police detective who finds himself fighting a group of terrorists in each film.

  Eastrail 177:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/}
    trakt_list: https://trakt.tv/users/jmpichardo/lists/eastrail-177?sort=released,desc
    summary: The Eastrail 177 Trilogy is an American superhero thriller and psychological horror film series written, produced, and directed by M. Night Shyamalan. The series has been noted for its differences from more traditional superhero movies, with Shyamalan's work referred to as "the first auteur shared superhero universe".

  Hannibal Lector:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/}
    trakt_list: https://trakt.tv/users/ireallylovevids/lists/hannibal-lecter?sort=released,desc
    summary: Hannibal Lecter is an American psychological thriller film series, adapted from the Thomas Harris novel of the same name about a serial killer named Hannibal Lecter.

  Hunger Games:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/69294}
    imdb_list: https://www.imdb.com/list/ls069064647/
    summary: A science fiction film series based on the novel of the same name by Suzanne Collins. The film series takes place in a dystopian post-apocalyptic future in the nation of Panem, featuring the protagonist, Katniss Everdeen.

  Indiana Jones:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/105678}
    imdb_list: https://www.imdb.com/list/ls027779939/?sort=release_date,asc&st_dt=&mode=detail&page=1
    summary: The Adventure films series from the Director George Lucas and Steven Spielberg, starring Harrison Ford as the archaeologist Dr. Henry Walton "Indiana" Jones.

  Jack Ryan:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/}
    trakt_list: https://trakt.tv/users/magalop/lists/jack-ryan?sort=released,desc
    summary: A collection of independent movies that do not need to be watched in order all featuring John Patrick "Jack" Ryan, a fictional character created by Tom Clancy.

  James Bond:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/115662}
    trakt_list: https://trakt.tv/users/any/lists/james-bond?sort=released,desc
    summary: The James Bond film series is a British series of spy films based on the fictional character of MI6 agent James Bond, codename "007". With all of the action, adventure, gadgetry & film scores that Bond is famous for.

  Marvel Cinematic Universe:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/162885}
    imdb_list: https://www.imdb.com/list/ls039269245/
    summary: An American media franchise and shared fictional universe that is centered on a series of superhero films, independently produced by Marvel Studios and based on characters that appear in publications by Marvel Comics. The franchise has expanded to include comic books, short films, and television series.

  Middle Earth:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/115119}
    imdb_list: https://www.imdb.com/list/ls055713151/?sort=release_date,asc&st_dt=&mode=detail&page=1
    summary: The Lord of the Rings is an epic high-fantasy novel by English author and scholar J. R. R. Tolkien. Set in Middle-earth, intended to be Earth at some distant time in the past, the story began as a sequel to Tolkien's 1937 children's book The Hobbit, but eventually developed into a much larger work.

  Mission Impossible:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/}
    trakt_list: https://trakt.tv/users/user71/lists/mission-impossible?sort=released,asc
    summary: Mission Impossible is a series of a secret agent thriller films based on the popular television series. They chronicle the missions of a team of secret government agents known as the Impossible Missions Force (IMF) under the leadership of Ethan Hunt.

  The Muppets:
    template: {name: Franchise, poster: "https://theposterdb.com/api/assets/205"}
    trakt_list: https://trakt.tv/users/mrjohnnycake/lists/the-muppets?sort=released,desc
    summary: The Muppets are an American ensemble cast of puppet characters known for an absurdist, burlesque, and self-referential style of variety-sketch comedy.

  Planet of the Apes:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/21715}
    trakt_list: https://trakt.tv/users/bmonomad/lists/planet-of-the-apes?sort=released,desc
    summary: Planet of the Apes is an American science fiction franchise about a world in which humans and intelligent apes clash for control and is based on the 1963 novel by French author Pierre Boulle.

  Predator:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/}
    trakt_list: https://trakt.tv/users/wdvhucb/lists/predator?sort=rank,asc
    summary: A science fiction action film series centered on a warrior class extraterrestrial species with technologically advanced weaponry that travel to Earth to trophy hunt human beings.

  Spider-Man:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/}
    trakt_list: https://trakt.tv/users/hitsquid/lists/spider-man?sort=released,desc
    summary: Spider-Man centers on student Peter Parker who, after being bitten by a genetically-altered spider, gains superhuman strength and the spider-like ability to cling to any surface. He vows to use his abilities to fight crime, coming to understand the words of his beloved Uncle Ben- "With great power comes great responsibility."

  Star Trek:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/}
    trakt_list: https://trakt.tv/users/strangerer/lists/star-trek?sort=released,desc
    summary: Star Trek is a science fiction media franchise originating from the 1960s television series Star Trek, created by Gene Roddenberry.

  Star Wars:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/58758}
    trakt_list: https://trakt.tv/users/zorge88/lists/star-wars?sort=released,desc
    summary: Star Wars is an American epic space-opera multimedia franchise created by George Lucas, which began with the eponymous 1977 film and quickly became a worldwide pop-culture phenomenon.

  Three Flavours Cornetto:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/234}
    imdb_list: https://www.imdb.com/list/ls068623110/
    summary: An anthology series of British comedic genre films directed by Edgar Wright, written by Wright and Simon Pegg, produced by Nira Park, and starring Pegg and Nick Frost. The trilogy consists of Shaun of the Dead (2004), Hot Fuzz (2007), and The World's End (2013).

  Wizarding World:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/85944}
    trakt_list: https://trakt.tv/users/malvado/lists/wizarding-world?sort=released,desc
    summary: The Wizarding World is a fantasy media franchise and shared fictional universe centred on a series of films, based on the Harry Potter novel series by J. K. Rowling.

  X-Men:
    template: {name: Franchise, poster: https://theposterdb.com/api/assets/163839}
    imdb_list: https://www.imdb.com/list/ls026465600/
    summary: X-Men is an American superhero film series based on the fictional superhero team of the same name, who originally appeared in a series of comic books created by Stan Lee and Jack Kirby and published by Marvel Comics.
```



#### genres ####

config/collections/movies/Genres.yml
```yaml
#######################
##     Templates     ##
#######################

templates:

    Genre:
        plex_search:
          genre: <<genre>>
        url_poster: <<poster>>
        sort_title: +++++_<<collection_name>>
        collection_order: alpha

#######################
##      Genres       ##
#######################

collections:

  Action:
    template: {name: Genre, genre: Action, poster: https://theposterdb.com/api/assets/52018}
    summary: Action film is a genre wherein physical action takes precedence in the storytelling. The film will often have continuous motion and action including physical stunts, chases, fights, battles, and races. The story usually revolves around a hero that has a goal, but is facing incredible odds to obtain it.

  Adventure:
    template: {name: Genre, genre: Adventure, poster: https://theposterdb.com/api/assets/52218}
    summary: Adventure film is a genre that revolves around the conquests and explorations of a protagonist. The purpose of the conquest can be to retrieve a person or treasure, but often the main focus is simply the pursuit of the unknown. These films generally take place in exotic locations and play on historical myths. Adventure films incorporate suspenseful puzzles and intricate obstacles that the protagonist must overcome in order to achieve the end goal.  

  Animation:
    template: {name: Genre, genre: Animation, poster: https://theposterdb.com/api/assets/120090}
    summary: Animated film is a collection of illustrations that are photographed frame-by-frame and then played in a quick succession. Since its inception, animation has had a creative and imaginative tendency. Being able to bring animals and objects to life, this genre has catered towards fairy tales and children’s stories. However, animation has long been a genre enjoyed by all ages. As of recent, there has even been an influx of animation geared towards adults. Animation is commonly thought of as a technique, thus it’s ability to span over many different genres.

  Anime:
    template: {name: Genre, genre: Anime, poster: https://theposterdb.com/api/assets/126743}
    summary: A collection of Anime movies

  Biography:
    template: {name: Genre, genre: Biography, poster: https://theposterdb.com/api/assets/60369}
    summary: A collection of Biography movies

  Comedy:
    template: {name: Genre, genre: Comedy, poster: https://theposterdb.com/api/assets/51397}
    summary: Comedy is a genre of film that uses humor as a driving force. The aim of a comedy film is to illicit laughter from the audience through entertaining stories and characters. Although the comedy film may take on some serious material, most have a happy ending. Comedy film has the tendency to become a hybrid sub-genre because humor can be incorporated into many other genres. Comedies are more likely than other films to fall back on the success and popularity of an individual star.

  Crime:
    template: {name: Genre, genre: Crime, poster: https://theposterdb.com/api/assets/53057}
    summary: Crime film is a genre that revolves around the action of a criminal mastermind. A Crime film will often revolve around the criminal himself, chronicling his rise and fall. Some Crime films will have a storyline that follows the criminal's victim, yet others follow the person in pursuit of the criminal. This genre tends to be fast paced with an air of mystery – this mystery can come from the plot or from the characters themselves.

  Documentary:
    template: {name: Genre, genre: Documentary, poster: https://theposterdb.com/api/assets/51430}
    summary: Documentary film is a non-fiction genre intended to document reality primarily for the purposes of instruction, education, or maintaining a historical record.

  Drama:
    template: {name: Genre, genre: Drama, poster: https://theposterdb.com/api/assets/52016}
    summary: Drama film is a genre that relies on the emotional and relational development of realistic characters. While Drama film relies heavily on this kind of development, dramatic themes play a large role in the plot as well. Often, these dramatic themes are taken from intense, real life issues. Whether heroes or heroines are facing a conflict from the outside or a conflict within themselves, Drama film aims to tell an honest story of human struggles.

  Family:
    template: {name: Genre, genre: Family, poster: https://theposterdb.com/api/assets/53059}
    summary: Fantasy film is a genre that incorporates imaginative and fantastic themes. These themes usually involve magic, supernatural events, or fantasy worlds. Although it is its own distinct genre, these films can overlap into the horror and science fiction genres. Unlike science fiction, a fantasy film does not need to be rooted in fact. This element allows the audience to be transported into a new and unique world. Often, these films center on an ordinary hero in an extraordinary situation.

  History:
    template: {name: Genre, genre: History, poster: https://theposterdb.com/api/assets/58022}
    summary: History film is a genre that takes historical events and people and interprets them in a larger scale. Historical accuracy is not the main focus, but rather the telling of a grandiose story. The drama of an History film is often accentuated by a sweeping musical score, lavish costumes, and high production value.

  Horror:
    template: {name: Genre, genre: Horror, poster: https://theposterdb.com/api/assets/51475}
    summary: Horror film is a genre that aims to create a sense of fear, panic, alarm, and dread for the audience. These films are often unsettling and rely on scaring the audience through a portrayal of their worst fears and nightmares. Horror films usually center on the arrival of an evil force, person, or event. Many Horror films include mythical creatures such as ghosts, vampires, and zombies. Traditionally, Horror films incorporate a large amount of violence and gore into the plot. Though it has its own style, Horror film often overlaps into Fantasy, Thriller, and Science-Fiction genres.

  Musical:
    template: {name: Genre, genre: Musical, poster: https://theposterdb.com/api/assets/51427}
    summary: A Musical interweaves vocal and dance performances into the narrative of the film. The songs of a film can either be used to further the story or simply enhance the experience of the audience. These films are often done on a grand scale and incorporate lavish costumes and sets. Traditional musicals center on a well-known star, famous for their dancing or singing skills (i.e. Fred Astaire, Gene Kelly, Judy Garland). These films explore concepts such are love and success, allowing the audience to escape from reality.

  Mystery:
    template: {name: Genre, genre: Mystery, poster: https://theposterdb.com/api/assets/53060}
    summary: A Mystery film centers on a person of authority, usually a detective, that is trying to solve a mysterious crime. The main protagonist uses clues, investigation, and logical reasoning. The biggest element in these films is a sense of “whodunit” suspense, usually created through visual cues and unusual plot twists.

  Romance:
    template: {name: Genre, genre: Romance, poster: https://theposterdb.com/api/assets/53062}
    summary: "Romance film can be defined as a genre wherein the plot revolves around the love between two protagonists. This genre usually has a theme that explores an issue within love, including but not limited to: love at first sight, forbidden love, love triangles, and sacrificial love. The tone of Romance film can vary greatly. Whether the end is happy or tragic, Romance film aims to evoke strong emotions in the audience."

  Science Fiction:
    template: {name: Genre, genre: Science Fiction, poster: https://theposterdb.com/api/assets/51772}
    summary: Science Fiction (Sci-Fi) film is a genre that incorporates hypothetical, science-based themes into the plot of the film. Often, this genre incorporates futuristic elements and technologies to explore social, political, and philosophical issues. The film itself is usually set in the future, either on earth or in space. Traditionally, a Science Fiction film will incorporate heroes, villains, unexplored locations, fantastical quests, and advanced technology.

  Short:
    template: {name: Genre, genre: Short, poster: https://theposterdb.com/api/assets/53063}
    summary: A collection of Short movies

  Thriller:
    template: {name: Genre, genre: Thriller, poster: https://theposterdb.com/api/assets/52019}
    summary: Thriller Film is a genre that revolves around anticipation and suspense. The aim for Thrillers is to keep the audience alert and on the edge of their seats. The protagonist in these films is set against a problem – an escape, a mission, or a mystery. No matter what sub-genre a Thriller film falls into, it will emphasize the danger that the protagonist faces. The tension with the main problem is built on throughout the film and leads to a highly stressful climax.

  War:
    template: {name: Genre, genre: War, poster: https://theposterdb.com/api/assets/51477}
    summary: War Film is a genre that looks at the reality of war on a grand scale. They often focus on landmark battles as well as political issues within war. This genre usually focuses on a main character and his team of support, giving the audience an inside look into the gritty reality of war.

  Western:
    template: {name: Genre, genre: Western, poster: https://theposterdb.com/api/assets/51774}
    summary: "Western Film is a genre that revolves around stories primarily set in the late 19th century in the American Old West. Most Westerns are set between the American Civil War (1865) and the early 1900s. Common themes within Western Film include: the conquest of the wild west, the cultural separation of the East and the West, the West’s resistance to modern change, the conflict between Cowboys and Indians, outlaws, and treasure/gold hunting. American Western Film usually revolves around a stoic hero and emphasizes the importance of honor and sacrifice."
```



#### holidays ####

config/collections/movies/Holidays.yml
```yaml
#######################
##     Templates     ##
#######################

templates:
    Holiday:
        sort_title: +++++_<<collection_name>>
        url_poster: <<poster>>
        collection_order: release.desc
        collection_mode: hide
        delete_not_scheduled: false
        run_again: true
        visible_home: true
        visible_shared: true
        sync_mode: sync

#################################
##     Holiday Collections     ##
#################################

# Note: Show collections only during their period by uncommenting "schedule"

collections:

  Christmas: #IMDB lists have been removed as they have all been merged into christmas-static-list
    template: {name: Holiday, holiday: "Christmas", poster: https://theposterdb.com/api/assets/212635}
    visible_home: range(12/01-12/31)
    visible_shared: range(12/01-12/31)
    #schedule: range(12/01-12/31)
    #sort_title: +++++++_Christmas
    trakt_list:
      - https://trakt.tv/users/jjjonesjr33/lists/christmas
      - https://trakt.tv/users/jjjonesjr33/lists/christmas-static-list
    summary: This collection revolves around the plot involving Christmas.

  Easter Movies:
    template: {name: Holiday, holiday: Easter, poster: https://theposterdb.com/api/assets/212636}
    visible_home: range(3/22-4/25)
    visible_shared: range(3/22-4/25)
    schedule: range(3/22-4/25)
    #sort_title: +++++++_Easter
    imdb_list: 
      - https://www.imdb.com/list/ls062665509/
      - https://www.imdb.com/list/ls051733651/
    summary: This collection revolves around the plot involving Easter.

  Halloween:
    template: {name: Holiday, holiday: "Halloween", poster: https://theposterdb.com/api/assets/212637}
    visible_home: range(10/01-10/31)
    visible_shared: range(10/01-10/31)
    #schedule: range(10/01-10/31)
    #sort_title: +++++++_Halloween
    trakt_list:
      - https://trakt.tv/users/jjjonesjr33/lists/halloween
    summary: This collection revolves around the plot involving Halloween.

  Independence Day:
    template: {name: Holiday, holiday: "Independence Day", poster: https://theposterdb.com/api/assets/240645}
    visible_home: range(07/01-07/05)
    visible_shared: range(07/01-07/05)
    schedule: range(07/01-07/05)
    #sort_title: +++++++_Independence Day
    imdb_list: 
      - https://www.imdb.com/list/ls552600367/
    summary: This collection revolves around the plot involving the Fourth of July.

  New Year's Eve Movies:
    template: {name: Holiday, holiday: "New Year's Eve", poster: https://i.imgur.com/YCYXhAX.png}
    visible_home: range(12/26-01/05)
    visible_shared: range(12/26-01/05)
    schedule: range(12/26-01/05)
    #sort_title: +++++++_New Year
    imdb_list: 
      - https://www.imdb.com/list/ls066838460/
    summary: This collection revolves around the plot involving New Year's Eve.

  St. Patrick's Day Movies:
    template: {name: Holiday, holiday: "St. Patrick's Day", poster: https://theposterdb.com/api/assets/240644}
    visible_home: range(03/01-03/17)
    visible_shared: range(03/01-03/17)
    schedule: range(03/01-03/17)
    #sort_title: +++++++_St Patrick
    imdb_list:
      - https://www.imdb.com/list/ls063934595/
    summary: This collection revolves around the plot involving St. Patrick's Day.

  Thanksgiving Movies:
    template: {name: Holiday, holiday: Thanksgiving, poster: https://theposterdb.com/api/assets/212638}
    visible_home: range(11/01-11/31)
    visible_shared: range(11/01-11/31)
    schedule: range(11/01-11/31)
    #sort_title: +++++++_Thanksgiving
    imdb_list: 
      - https://www.imdb.com/list/ls000835734/
      - https://www.imdb.com/list/ls091597850/
    summary: This collection revolves around the plot involving Thanksgiving.

  Valentine's Day Movies:
    template: {name: Holiday, holiday: "Valentine's Day", poster: https://theposterdb.com/api/assets/212641}
    visible_home: range(02/01-02/14)
    visible_shared: range(02/01-02/14)
    schedule: range(02/01-02/14)
    #sort_title: +++++++_Valentine
    imdb_list:
      - https://www.imdb.com/list/ls000094398/
      - https://www.imdb.com/list/ls057783436/
      - https://www.imdb.com/list/ls064427905/
    summary: This collection revolves around the plot involving Valentine's Day.
```



#### people ####

config/collections/movies/People.yml
```yaml
#######################
##     Templates     ##
#######################

templates:

  People:
    actor: tmdb
    tmdb_person: <<person>>
    sort_title: ++_<<collection_name>>
    collection_order: release

####################
##     People     ##
####################

collections:

  Alfred Hitchcock:
    director: tmdb
    template: {name: People, person: 2636}
    # trakt_list: https://trakt.tv/users/movistapp/lists/hitchcock
    summary: Sir Alfred Joseph Hitchcock, KBE (13 August 1899 – 29 April 1980), was an English director and producer. He became known for thrillers, earning him the nickname 'Master of Suspense'. After a successful career in his native country, Hitchcock moved to Hollywood. Over a career spanning more than half a century, Hitchcock fashioned for himself a distinctive and recognizable directorial style. He pioneered the use of a camera made to move in a way that mimics a person's gaze, forcing viewers to engage in a form of voyeurism. He framed shots to maximize anxiety, fear, or empathy, and used innovative film editing.

  Christopher Guest:
    director: tmdb
    template: {name: People, person: 13524}
    summary: The Rt. Hon. Christopher Haden-Guest, 5th Baron Haden-Guest (born February 5, 1948), better known as Christopher Guest, is an American screenwriter, composer, musician, director, actor and comedian. He is most widely known in Hollywood for having written, directed and starred in several improvisational "mockumentary" films that feature a repertory-like ensemble cast, such as This is Spinal Tap.

  Dave Chappelle:
    actor: tmdb
    template: {name: People, person: 4169}
    summary: David Khari Webber "Dave" Chappelle was born on August 24, 1973 in Washington, D.C. He is a comedian, screenwriter, television/film producer and actor.

  Denzel Washington:
    actor: tmdb
    template: {name: People, person: 5292}
    summary: Denzel Hayes Washington, Jr. is an American actor, screenwriter, director and film producer.

  J.J. Abrams:
    director: tmdb
    producer: tmdb
    template: {name: People, person: 15344}
    summary: Jeffrey Jacob Abrams (born June 27, 1966) is an American filmmaker. He is best known for his work in the genres of action, drama, and science fiction.

  Jim Carrey:
    actor: tmdb
    template: {name: People, person: 206}
    summary: James Eugene Carrey (born January 17, 1962) is a Canadian and American actor, comedian, writer, and producer. Known for his energetic slapstick performances, Carrey first gained recognition in 1990, after landing a recurring role in the American sketch comedy television series In Living Color (1990–1994).

  Stephen King:
    writer: tmdb
    template: {name: People, person: 3027}
    summary: An American author of contemporary horror, suspense, science fiction and fantasy fiction. His books have sold more than 350 million copies, which have been adapted into a number of feature films, television movies and comic books. As of 2011, King has written and published 49 novels, including seven under the pen name Richard Bachman, five non-fiction books, and nine collections of short stories.

  Taika Waititi:
    director: tmdb
    template: {name: People, person: 55934}
    summary: Taika David Cohen ONZM (born 16 August 1975), known professionally as Taika Waititi, is a New Zealand film and television director, producer, screenwriter, actor, and comedian. He is the recipient of an Academy Award, BAFTA Award and a Grammy Award, and has been nominated for two Primetime Emmy Awards.

  Christopher Nolan:
    summary: Christopher Edward Nolan, CBE (born 30 July 1970) is a British-American film director, screenwriter, and producer. He was born in Westminster, London, England and holds both British and American citizenship due to his American mother. Nolan is the founder of the production company Syncopy Films. He often collaborates with his wife, producer Emma Thomas, and his brother, screenwriter Jonathan Nolan.

  Coen Brothers:
    summary: Joel Coen and Ethan Coen, collectively referred to as the Coen Brothers, are American film directors, producers, screenwriters, and editors. Their films span many genres and styles, which they frequently subvert or parody.

  Philip K Dick:
    summary: American short story writer, novelist, and essayist. Dick has been hailed as one of the most original and thought-provoking writers of science fiction. He is regarded as one of the most prolific writers of the form during the mid-twentieth century. Since his death, Dick's work has been the subject of numerous critical studies and cinematic adaptations. Critics praise his short stories as innovative and provocative, contending that Dick's fiction cleverly explores scientific, social, and metaphysical issues of concern to post-World War II America.

  Roland Emmerich:
    summary: Roland Emmerich is a German film director, screenwriter, and producer who is known for his work on disaster and action flicks. He co-founded Centropolis Entertainment in 1985 with his sister.

  Shakespeare:
    summary: William Shakespeare (26 April 1564 (baptised) – 23 April 1616) was an English poet, playwright, and actor, widely regarded as the greatest writer in the English language and the world's pre-eminent dramatist. He is often called England's national poet and the "Bard of Avon". His extant works, including some collaborations, consist of around 38 plays, 154 sonnets, two long narrative poems, and a few other verses, of which the authorship of some is uncertain. His plays have been translated into every major living language and are performed more often than those of any other playwright.
```


#### services ####

config/collections/movies/Services.yml
```yaml
collections:

############################
#         SERVICES         #
############################

  Apple TV+:
    imdb_list:
      url: https://www.imdb.com/search/title/?title_type=feature,tv_movie,documentary,video,&release_date=2015-01-01,2022-12-31&genres=%21animation&companies=co0546168&countries=us&languages=en&sort=release_date,asc
    summary: All Apple TV Originals From 2015-2025
    sync_mode: sync
    collection_mode: hide
    collection_order: release
    sort_title: +++++++_Apple
    schedule: daily
    url_poster: https://theposterdb.com/api/assets/163303
    url_background: https://wallpaper-house.com/data/out/8/wallpaper2you_228774.png

  HBO Max:
    imdb_list:
      url: https://www.imdb.com/search/title/?title_type=movie&release_date=2015-01-01,2025-12-31&countries=us&languages=en&sort=release_date,desc&count=250&companies=co0754095
    summary: All HBO Max Originals From 2015-2025
    url_poster: https://theposterdb.com/api/assets/163885
    url_background: https://wallpapercave.com/wp/wp6402755.png
    sync_mode: sync
    collection_mode: hide
    collection_order: release
    sort_title: +++++++_HBO
    schedule: daily

  Hulu:
    imdb_list:
      url: https://www.imdb.com/search/title/?title_type=movie&release_date=2015-01-01,2025-12-31&countries=us&languages=en&sort=release_date,desc&count=250&companies=co0218858
    summary: All HULU Originals From 20185-2025
    url_poster: https://theposterdb.com/api/assets/163302
    url_background: https://cdn.vox-cdn.com/thumbor/oR4hqrmTxbX_O4gdJ6np8h-PxFk=/0x439:750x861/1600x900/cdn.vox-cdn.com/uploads/chorus_image/image/56311701/Image_uploaded_from_iOS__8_.1503433270.jpg
    sync_mode: sync
    collection_mode: hide
    collection_order: release
    sort_title: +++++++_Hulu
    schedule: daily

  Netflix:
    imdb_list:
      url: https://trakt.tv/users/faria-luisfilipe/lists/netflix-originals?display=movie&hide=unreleased&sort=percentage,asc
    summary: All Netflix Originals
    url_poster: https://theposterdb.com/api/assets/163448
    url_background: https://img5.goodfon.com/original/1920x1080/1/61/fon-netflix-logo-raduga-tsvet-fon-background-skachat-oboi-sk.jpg
    sync_mode: sync
    collection_mode: hide
    collection_order: release
    sort_title: +++++++_Netflix
    schedule: daily

  Prime Video:
    imdb_list:
      url: https://www.imdb.com/search/title/?title_type=movie&release_date=2015-01-01,2025-12-31&countries=us&languages=en&sort=release_date,desc&count=250&companies=co0319272
    summary: All Amazon Originals From 2015-2025
    url_poster: https://theposterdb.com/api/assets/163301
    url_background: https://i.imgur.com/2jfs7oS.png
    sync_mode: sync
    collection_mode: hide
    collection_order: release
    sort_title: +++++++_Prime
    schedule: daily
```



#### specialty genres ####

config/collections/movies/Specialty.yml
```yaml
#######################
##     Templates     ##
#######################

templates:

    Specialty:
        url_poster: <<poster>>
        sort_title: +++++_<<collection_name>>
        collection_order: release
        delete_not_scheduled: true
        run_again: true
        visible_home: false
        visible_shared: true
        sync_mode: sync

##########################
##   Specialty Genres   ##
##########################

collections:

  Apocalyptic:
    template: {name: Specialty, poster: https://theposterdb.com/images/defaults/missing_poster.jpg}
    trakt_list: https://trakt.tv/users/mccaffe/lists/apocalyptic-dystopia?sort=released,asc
    summary: This list of movies about the end of the world includes apocalypse movies and doomsday films.

  Based on Books:
    template: {name: Specialty, poster: https://theposterdb.com/api/assets/235323}
    tmdb_keyword: 818 #https://www.themoviedb.org/keyword/818-based-on-novel-or-book/movie
    summary: Movies based on books or novels

  Based on True Events:
    template: {name: Specialty, poster: https://theposterdb.com/images/defaults/missing_poster.jpg}
    trakt_list: https://trakt.tv/users/manu101/lists/based-inspired-on-actual-events
    summary: Movies based on true events.

  Black Experience:
    template: {name: Specialty, poster: https://theposterdb.com/images/defaults/missing_poster.jpg}
    trakt_list: https://trakt.tv/users/loriejcall/lists/110-important-films-about-the-black-experience-11784433?sort=released,asc
    summary: These films will teach you something the history books don’t.

  Cult Classics:
    template: {name: Specialty, poster: https://theposterdb.com/images/defaults/missing_poster.jpg}
    trakt_list:
      - https://trakt.tv/users/mr-mc86/lists/cult-classics?sort=rank,asc
      - https://trakt.tv/users/mekwall/lists/cult-films?sort=rank,asc
    summary: Judge these movies not by their quality but by their cult following

  Disaster:
    template: {name: Specialty, poster: https://theposterdb.com/api/assets/164980}
    trakt_list: https://trakt.tv/users/29zombies/lists/disaster?sort=released,asc
    summary: dis·as·ter- a sudden event, such as an accident or a natural catastrophe, that causes great damage or loss of life. ie flood, earthquake, hurricanes.

  Dystopian:
    template: {name: Specialty, poster: https://theposterdb.com/images/defaults/missing_poster.jpg}
    trakt_list: https://trakt.tv/users/vagnerr/lists/dystopia
    summary: A dystopia is a speculated community or society that is undesirable or frightening.

  Famous Detectives:
    template: {name: Specialty, poster: https://theposterdb.com/images/defaults/missing_poster.jpg}
    trakt_list:
      - https://trakt.tv/users/goape/lists/sherlock-holmes?display=movie&sort=rank,asc
      - https://trakt.tv/users/grelipe/lists/hercule-poirot?sort=released,desc
    summary: Television and movies are dominated by detectives, with some of the best media following these iconic agents and sleuths as they investigate crime.

  Fantasy:
    template: {name: Specialty, poster: https://theposterdb.com/api/assets/51476}
    imdb_list:
    - url: https://www.imdb.com/search/title/?title_type=feature&release_date=1990-01-01,&user_rating=5.0,10.0&num_votes=100000,&genres=fantasy
      limit: 100
    - url: https://www.imdb.com/search/title/?title_type=feature&release_date=1990-01-01,&user_rating=5.0,10.0&num_votes=100000,&genres=fantasy&sort=user_rating,desc
      limit: 100 
    summary: Fantasy film is a genre that incorporates imaginative and fantastic themes. These themes usually involve magic, supernatural events, or fantasy worlds. Although it is its own distinct genre, these films can overlap into the horror and science fiction genres. Unlike science fiction, a fantasy film does not need to be rooted in fact. This element allows the audience to be transported into a new and unique world. Often, these films center on an ordinary hero in an extraordinary situation.

  Good Bad Movies:
    template: {name: Genre, genre: Animation, poster: https://theposterdb.com/images/defaults/missing_poster.jpg}
    trakt_list:
      - https://trakt.tv/users/philrivers/lists/the-50-best-good-bad-movies?sort=rank,asc
      - https://trakt.tv/users/acelove/lists/good-bad-movies?sort=title,asc
      - https://trakt.tv/users/gristlemctough/lists/bad-movies?sort=title,asc
      - https://trakt.tv/users/maxwelldeux/lists/badmovies-org-best-b-movies?sort=rank,asc
    summary: The bad special effects, the awful acting, the nonsensical plots — there’s something enchanting about a movie that’s hopelessly bad.

  Heist Movies:
    template: {name: Specialty, poster: https://theposterdb.com/api/assets/164791}
    imdb_list:
      - https://www.imdb.com/list/ls068224634/
      - https://www.imdb.com/list/ls009794682/ 
    summary: Movies detailing the preparation for, execution of and aftermath of a (well-)planned and often daring heist.

  Lovecraftian:
    template: {name: Specialty, poster: https://theposterdb.com/images/defaults/missing_poster.jpg}
    trakt_list: https://trakt.tv/users/twentywashere/lists/lovecraft-lovecraftian?sort=released,asc
    summary: Movies and shorts based on the works of HP Lovecraft (some only loosely), have Lovecraftian themes or strong references to Lovecraft or are clearly Lovecraft inspired. Some might be fringe cases and not considered Lovecraftian by purists but are included because the atmosphere is suggestive of Lovecraft's work, there are clear nods to Lovecraft and in some case it is just because of name dropping or big tentacle monsters.

  Martial Arts:
    template: {name: Specialty, poster: https://theposterdb.com/api/assets/254748}
    imdb_list:
      - https://www.imdb.com/list/ls000099643/
      - https://www.imdb.com/list/ls068611186/
      - https://www.imdb.com/list/ls068378513/
      - https://www.imdb.com/list/ls090404120/
    summary: Martial Arts film is a sub-genre of action films that feature numerous martial arts combat between characters. These combats are usually the films' primary appeal and entertainment value, and often are a method of storytelling and character expression and development. Martial Arts are frequently featured in training scenes and other sequences in addition to fights. Martial Arts films commonly include other types of action, such as hand-to-hand combat, stuntwork, chases, and gunfights.

  Mindfuck Movies:
    template: {name: Specialty, poster: https://i.imgur.com/Tl6QMgA.png}
    trakt_list:
      - https://trakt.tv/users/hdlists/lists/mindfuck-movies
      - https://trakt.tv/users/benfranklin/lists/best-mindfucks
    summary: There’s nothing greater than a film that makes you think. A film that lingers on your mind long after the credits have rolled and is the subject of many conversation and discussion. Trying to piece together the puzzle of what you saw and coming to a conclusion that makes sense or still doesn’t add up.

  Music:
    template: {name: Specialty, poster: https://theposterdb.com/api/assets/69217}
    trakt_list: https://trakt.tv/users/movistapp/lists/music?sort=released,asc
    summary: Music film is genre that revolves around music being an integral part of the characters lives.

  Pandemic:
    template: {name: Specialty, poster: https://theposterdb.com/api/assets/55837}
    imdb_list: https://www.imdb.com/list/ls092321048/
    summary: A Pandemic film resolves around widespread viruses, plagues, and diseases.

  Revenge:
    template: {name: Specialty, poster: https://theposterdb.com/images/defaults/missing_poster.jpg}
    trakt_list: https://trakt.tv/users/jtucci/lists/revenge?sort=released,asc
    summary: Who doesn’t love watching someone finally get their sweet, sweet, revenge?

  Sports:
    template: {name: Specialty, poster: https://theposterdb.com/api/assets/167979}
    trakt_list: https://trakt.tv/users/reafthompson/lists/sports?sort=released,asc
    summary: A Sports Film revolves around a sport setting, event, or an athlete. Often, these films will center on a single sporting event that carries significant importance. Sports films traditionally have a simple plot that builds up to the significant sporting event. This genre is known for incorporating film techniques to build anticipation and intensity. Sport films have a large range of sub-genres, from comedies to dramas, and are more likely than other genres to be based true-life events.

  Stand Up Comedy:
    template: {name: Specialty, poster: https://theposterdb.com/api/assets/166533}
    imdb_list:
      - https://www.imdb.com/list/ls070221411/
      - https://www.imdb.com/list/ls086584751/
      - https://www.imdb.com/list/ls086022668/
      - https://www.imdb.com/list/ls049792208/
    summary: Stand-up comedy is a comedic style in which a comedian performs in front of a live audience, speaking directly to them through a microphone. Comedians give the illusion that they are dialoguing, but in actuality, they are monologuing a grouping of humorous stories, jokes and one-liners, typically called a shtick, routine, act, or set. Some stand-up comedians use props, music or magic tricks to enhance their acts. Stand-up comedians perform quasi-autobiographical and fictionalized extensions of their offstage selves.

  Time Travel:
    template: {name: Specialty, poster: https://theposterdb.com/api/assets/160794}
    trakt_list: https://trakt.tv/users/vagnerr/lists/time-travel?sort=released,asc
    summary: What would you do with the power to travel through time? Even though we're stuck in the present, humans can't stop dreaming about what we would do if we could change the past, tell the future, and influence our current reality. Luckily, the following cinematic masterpieces have done all the thinking for us. These movies showcase the challenges, pitfalls, and miracles that come with using time travel to change the course of history—if it was possible.

#  Writing:
#    template: {name: Specialty, poster: https://theposterdb.com/images/defaults/missing_poster.jpg}
#    trakt_list: 
#    summary: 
```



#### studios ####

config/collections/movies/Studios.yml
```yaml
#######################
##     Templates     ##
#######################

templates:
    Studio:
        url_poster: <<poster>>
        sort_title: ++++++++_<<collection_name>>
        collection_order: release
        delete_not_scheduled: true
        run_again: true
        visible_home: true
        visible_shared: true
        sync_mode: sync

################################
##     Studio Collections     ##
################################

collections:

  Pixar Animation Studios:
    template: {name: Studio, poster: https://theposterdb.com/api/assets/189763}
    trakt_list:
      - https://trakt.tv/users/selkcer/lists/pixar?sort=released,desc

  Studio Ghibli:
    template: {name: Studio, poster: https://theposterdb.com/api/assets/193453}
    trakt_list:
      - https://trakt.tv/users/hitsquid/lists/studio-ghibli?sort=released,desc

  Walt Disney Animation Studios:
    template: {name: Studio, poster: https://theposterdb.com/api/assets/191971}
    trakt_list:
      - https://trakt.tv/users/pauloguerra/lists/disney-animations?sort=released,desc
```



#### Name-Fixes ####

config/metadata/movies/Name-Fixes.yml
```yaml
#####################
##     Example     ##
#####################

#  [TMDb number]: # [Name of movie]
#    title: [Name of movie]
#    url_poster: https://theposterdb.com/api/assets/[poster-number]

####################################
##     Plex Auto-naming Fixes     ##
####################################

 metadata:

  913290: # Barbarian
    title: Barbarian
    url_poster: https://theposterdb.com/api/assets/267452

  20504: # The Book of Eli
    title: The Book of Eli
    url_poster: https://theposterdb.com/api/assets/165333

  9276: # The Faculty
    title: The Faculty
    url_poster: https://theposterdb.com/api/assets/33949

  5994: # The Family Man
    title: The Family Man

  436305: # The Farthest
    title: The Farthest
    url_poster: https://theposterdb.com/api/assets/21494

  8738: # The Final Countdown
    title: The Final Countdown
    url_poster: https://theposterdb.com/api/assets/179552

  65754: # The Girl with the Dragon Tattoo
    title: The Girl with the Dragon Tattoo
    url_poster: https://theposterdb.com/api/assets/39706

  9504: # Glengarry Glen Ross
    title: Glengarry Glen Ross
    url_poster: https://theposterdb.com/api/assets/108248

  497: # The Green Mile
    title: The Green Mile
    url_poster: https://theposterdb.com/api/assets/31065

  1581: # The Holiday
    title: The Holiday
    url_poster: https://theposterdb.com/api/assets/14838

  11287: # A League of Their Own (1992)
    title: A League of Their Own
    url_poster: https://theposterdb.com/api/assets/87818

  334: # Magnolia
    title: Magnolia
    url_poster: https://theposterdb.com/api/assets/188510

  668: # On Her Majesty's Secret Service
    title: On Her Majesty's Secret Service
    url_poster: https://theposterdb.com/api/assets/2080

  319: # True Romance
    title: True Romance
    url_poster: https://theposterdb.com/api/assets/166546
```



## Shows ##



## Playlists ##

config/playlists/video.yml
```yaml
playlists:
  Dave's Nod Off List:
    sync_to_users:
    sync_mode: sync
    libraries: Movies, Shows
    trakt_list: https://trakt.tv/users/mrjohnnycake/lists/dave-s-nod-off-list?sort=title,asc
    summary: For some reason, Dave can fall asleep to these videos easier than other media. Probably because he's seen each so many times.
    url_poster: https://m.media-amazon.com/images/I/710nZSbneDL._SS500_.jpg
```
