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
# scripts/hubot-bitbucket-commit.coffee
module.exports = (robot) ->
  bitbucketPushEvent = process.env.HUBOT_BITBUCKET_PUSH_EVENT or 'bitbucketPushReceived'
  room = process.env.HUBOT_BITBUCKET_PUSH_ROOM or 'general'

  robot.on bitbucketPushEvent, (data) ->
    response = ""
    push = data.res # using json data directly here
    prefix = "[#{push.canon_url}#{push.repository.absolute_url}|#{push.repository.absolute_url}]"
    if (push.commits.length > 1)
      response += robot.messageRoom "general", "#{prefix} #{push.commits.length} new commits"

    for commit in push.commits
      sha_link = "<#{push.canon_url}#{push.repository.absolute_url}commits/#{commit.raw_node}|#{commit.node}>"
      modified = if commit.files.length > 1 then " - #{commit.files.length} files modified" else " - #{commit.files.length} file modified"
      author = " - #{commit.author}"
      response += "#{prefix} #{sha_link} #{commit.message}\n#{modified}\n#{author}\n"

    robot.messageRoom room, response

# Below is using hubot-bitbucket's Push object, but it's lacking informations

#    push = data.push # using validated push object
#    prefix = "[<#{push.repo.url}|#{push.repo.name}>]"
#
#    if (push.commits.length > 1)
#      response += robot.messageRoom room, "#{prefix} #{push.commits.length} new commits"
#
#    for commit in push.commits
#      console.log commit
#      modified = if commit.files.length > 1 then "#{commit.files.length} files modified" else "#{commit.files.length} file modified"
#      author = " - #{commit.author}"
#      response += "#{prefix} #{commit.message}\n#{modified}\n#{author}\n"
#
#    robot.messageRoom room, response
```
