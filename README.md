Hubot Bitbucket Listener
========================

[![Build Status](https://travis-ci.org/andrewtarry/hubot-bitbucket.svg?branch=master)](https://travis-ci.org/andrewtarry/hubot-bitbucket)

The Hubot Bitbucket listener allows custom Bitbucket Post hooks be set up that will inform Hubot when events occur. The hook data will then be parsed and an event will be fired that custom scripts can listen for.

## Installation

Install the package with `npm install hubot-bitbucket --save` and add it to your `external-scripts.json`.

```json
["hubot-bitbucket"]
```

## Configuration

There are two optional environment variables that can be set to customise the listener.

Variable | Default | Effect
---------|---------|-------
`HUBOT_BITBUCKET_PUSH_URL` | /bitbucket/push | The route bitbucket should post to
`HUBOT_BITBUCKET_PUSH_EVENT` | bitbucketPushReceived | The event that will emitted when the push is received

## Usage

The Hubot Bitbucket Listener will not do anything with the data by default so there will be nothing added to a room. The listener will only parse the request and emit and event that you can listen for. If you want to have `Hubot` notify a room then you can easily do that.

```coffeescript

robot.on "bitbucketPushReceived", (pushEvent) ->
    push = pushEvent.push

    # Do things
```

The `push` variable is an object that will with data from the Bitbucket hook.

```coffeescript
repository = push.repo
repository_url = repository.url # e.g. https://bitbucket.org/marcus/project-x/
repository_name = repository.name # e.g. Project X
repository_slug = repository.slug # e.g. project-x

for commit in push.commits
	author = commit.author # bitbucket username
	branch = commit.master # e.g. master
	message = commit.message # A really clear message

	for file in commit.files
		file = file.filename # e.g. somefile.py
		type = file.type # i.e. modified, added etc

```
You can listen for the event and use as much of the data as you want.

## Event example

If you want hubot to write to your chat room a message like this:

> hubot: [repo_owner/repo] <#sha|link> commit message
> commit description
> - commit author

Add `["hubot-bitbucket-commit"]` to `hubot-scripts.json` (or whatever you want) and add an event listener like this

```coffeescript
# Description:
#   Hubot will listen for bitbucket webhooks!
#
# Configuration:
#   HUBOT_BITBUCKET_PUSH_ROOM room where to post
#   HUBOT_BITBUCKET_PUSH_EVENT event to emit when a push is made
#
# Commands:
#   None
#
# Author:
#   GabLeRoux
#

# https://github.com/slackhq/hubot-slack/issues/114 wtf slack...
module.exports = (robot) ->
  bitbucketPushEvent = process.env.HUBOT_BITBUCKET_PUSH_EVENT or 'bitbucketPushReceived'
  room = process.env.HUBOT_BITBUCKET_PUSH_ROOM or 'general'

  robot.on bitbucketPushEvent, (data) ->
    res = data.res # using json data directly here
    repo_name = res.repository.full_name
    repo_url = res.repository.links.html.href
    commits_url = res.push.changes[0].links.html.href

    response = "[#{repo_name}] #{repo_url}\n"

    commit = res.push.changes[0].new
    new_commit_author_username = commit.target.author.user.username
    new_commit_author_display_name = commit.target.author.user.display_name
    new_commit_author_url = commit.target.author.user.links.html.href
    new_commit_hash = commit.target.hash
    new_commit_hash_short = new_commit_hash.substring 0,7
    new_commit_message = commit.target.message

    new_commit_type = commit.type
    new_commit_name = commit.name
    new_commit_on = new_commit_type + " " + new_commit_name

    response += "New commit(s) on #{new_commit_on}\n"
    response += "#{commits_url}\n"
    response += "#{new_commit_hash_short} #{new_commit_message}\n"
    response += " - #{new_commit_author_display_name} (#{new_commit_author_username})\n"
    
    robot.messageRoom room, response

    return
  return
```

It should display something like this:
```
[gableroux/some-repository] https://bitbucket.org/gableroux/some-repository
New commit(s) on branch master
https://bitbucket.org/gableroux/some-repository/branches/compare/2ad9d60ef4f9369c5668c9dd9e3798c91e4f30cd..14c2888e3c6678d26c1b65c9f45cd5bad79b8370
2ad9d60 Some commit message

* Support for multiline commit message
* Some other content in commit message, etc.

- Gabriel Le Breton (gableroux)
```
