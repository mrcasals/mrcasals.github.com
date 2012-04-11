---
layout: post
title: "My first Octopress plugin: Spotify Play button"
date: 2012-04-11 18:02
comments: true
categories: 
---

I'm a big fan of Spotify. I use it everyday, I pay my monthly fee to get rid of
those annoying ads (I don't know what ads do people get in other countries, but
Spanish ones are really annoying... *really*). I was really happy when they
launched their [Metadata API][metadata-api], but [today's
announcement][play-blogpost] is even better.

## Embed Spotify everywhere... even Octopress!

Spotify's new feature [Play Button][play-button] lets you embed a single track,
an album or a playlist to any web page, giving you the possibility to put a
soundtrack anywhere. So I decided to make an Octopress plugin for this. You can
find [its source on GitHub][spotify-play-repo].

The plugin is pretty simple. You have to use the `spotify` liquid tag and, as
param, one of the following examples. 

In the next release it will let you specify the height and the width of the
widget. I hope it won't take very long :)

### A single track
Example input:
{% codeblock %}
spotify spotify:track:4vVE8YDntxjT0gld4bdd4x
{% endcodeblock %}

Example output:
{% spotify spotify:track:4vVE8YDntxjT0gld4bdd4x %}

### An album
Example input:
{% codeblock %}
spotify spotify:album:2B9q4KPjOEYu885Keo9dfX
{% endcodeblock %}

Example output:
{% spotify spotify:album:2B9q4KPjOEYu885Keo9dfX %}

### A playlist
Example input:
{% codeblock %}
spotify spotify:user:mrc2407:playlist:1O9JUUKiMqA8LXahz61fCH
{% endcodeblock %}

Example output:
{% spotify spotify:user:mrc2407:playlist:1O9JUUKiMqA8LXahz61fCH %}

### Some independent tracks
Example input:
{% codeblock %}
spotify spotify:trackset:PREFEREDTITLE:5Z7ygHQo02SUrFmcgpwsKW,1x6ACsKV4UdWS2FMuPFUiT,4bi73jCM02fMpkI11Lqmfe
{% endcodeblock %}

Example output:
{% spotify spotify:trackset:PREFEREDTITLE:2fdMAIsNH4wY7RRda8aCfL,4ik6sbfwyVd9QoU3nUHvKI %}

[metadata-api]: https://developer.spotify.com/technologies/web-api/
[play-blogpost]: http://www.spotify.com/se/blog/archives/2012/04/11/introducing-the-spotify-play-button/
[play-button]: https://developer.spotify.com/technologies/spotify-play-button/
[spotify-play-repo]: https://github.com/mrcasals/octopress_spotify_play_plugin
