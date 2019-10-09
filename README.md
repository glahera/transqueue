# transqueue
1. Original Problem:
  I use Radarr and Sonarr to download movies and TV Shows from torrent trackers then send over Plex to stream to my family, but over the time, problems surface. One of the biggest problems is that I encountered is that I want the best quality but my family is scattered all over the world, and not all members have access to Fiber connection, so not everyone can stream a full fat Bluray Remux from Plex server.
2. Solution:
  I decided to transcode from Radarr and Sonarr to a more "bite-sized" video file that's easier to digest for everyone. I found scarthe's PlexEncode code along the way and I've used it for a long time, but after a while, I realized problems with it. The script runs locally within each docker containers (I run them in Docker environment), but then they each called their own ffmpeg process and it was killing the server's performance. So I decided to create these scripts and Docker images to resolve that problem. Introducing Transqueue, a solution for nobody's problem (I guess).
  These docker images are:
  - Modified Radarr and Sonarr images
  - A vanilla Redis server image
  - A transcode worker image
  This is how they work:
  - Radarr/Sonarr receives "Downloaded" from torrent client
  - Radarr/Sonarr imports the file into designated location, whether it is locally or rclone mount
  - After importing, Radarr/Sonarr runs queue.sh script. This script will analyze bitrate and date of the episode. If the file has lower bitrate (both video and audio) than designated limit, it will do nothing to it. Otherwise, it will push the file's location and transcode argument to Redis server. Top or bottom of the list would depend on the date of the episode's air time. You can set all of these settings in the script, in "Define Variables" section.
  - Transcode Worker runs transcode.sh on loop, and always waiting for job from Redis server (by using BLPOP command). When a job is available on Redis' list, it will pick up and begin processing.
3. Upcoming feature:
  I'm also trying to make the Transcode Worker to be Docker Swarm friendly so that multiple nodes can process multiple videos simultaneously. And further than that is making this Transqueue to be more generic and multi purpose, not just for Sonarr/Radarr.
