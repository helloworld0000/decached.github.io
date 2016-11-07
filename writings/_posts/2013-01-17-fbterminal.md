---
layout: post
title: Facebook on Terminal
category: writings
tags: [facebook, python]
---

When you are popular or you have lots of Facebook newbies in your friend list,
you tend to receive lots of messages continuously which after a point become
very annoying. People then ask you "why didn't you reply to their messages even
though you were online". As Facebook doesn't allow us to stay invisble while
knowing which of our friends are online, I thought a better way would be to not
leave the terminal I work in and fire a command to know whether a particular
friend is online.

This is a command line interface for Facebook to check notifications, who's
online, post a status, Check unread messages.

> Update: This doesn't work anymore as Facebook has updated its API
and scope of permissions. It was fun while it lasted. I, currently, have no
plans to fix this, but you are welcome to chip in a [pull
request](https://github.com/{{ site.username }}/fbterminal).


### Installation & Configuration

- Install: `$ [sudo] pip install fbterminal`

- Configure:

    - Go to [Facebook Developers](https://developers.facebook.com/apps) and
      create a new app

    - Add `App Name` of your choice

    - Deselect `Web hosting` (You won't be needing that)

    - In `App Domains` put `localhost`

    - Select `Website with Facebook Login`

    - In `Site URL` put `http://localhost:7777/` (Make sure no apps are running
    on port 7777)

    - Save the changes

    - Copy the APP ID and APP SECRET to `~/.fbterminal`

- Run: `$ fbterminal -p "Hello World"`

### License

This is [MIT](https://github.com/decached/fbterminal/blob/master/LICENCE) with no
added caveats, so feel free to use without a disclaimer or anything silly like that.
