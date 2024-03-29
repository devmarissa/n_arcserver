# n_arcserver

ROBLOX user admin shell deployed in-game via JavaScript

by n_arc

[![n_arc's Projects](https://img.shields.io/badge/n_arc%27s-Projects-red.svg?style=flat-square)](https://discord.gg/h8kTjvZ)

## Instructions

Go to settings.json and set `username` and `password` to the username and password of the ROBLOX account you want to use. The `key` field is essentially a password for the site (to prevent strangers from accessing account functions). There is also an optional setting `maximumRank` which can be used to prevent attacks. User's above this rank are immune from having their rank changed and attempts to change a user's rank to something above this will be rejected. I recommend generating a random string or just smashing your keyboard since this will typically be accessed by another script that doesn't have to memorize said key.

## Free Host Tutorial
1. Go to [heroku.com](https://heroku.com/) and sign up for an account.
2. Install [heroku command line tools](https://devcenter.heroku.com/articles/heroku-command-line#download-and-install).
3. [Download](https://github.com/n-arc/n_arclogger/archive/master.zip) this repository.
4. Open the settings.json file and fill in the fields "username", "password", and "key" in the quotes after each.
5. Place all files within a folder.
6. Open a terminal or command prompt and type `cd `, then drag the folder into the window and release. It should fill in the path name afterwards. Hit enter.
7. Type in `heroku login` and type in the username and password of the account you made in step 1.
8. Type in `git init` [Enter] followed by `git add --all` [Enter] and then `git commit -am "Initial"` [Enter]
9. Type in `heroku create` [Enter]; you can put in a custom name after the command as well. eg. `heroku create roblox-js-server`
10. Finally type `git push heroku master` [Enter] and let it go through. If all goes well it will deploy after a minute or two and will tell you the url of your server around the end of the process.

## Updating
To update the server files on heroku (esp. n_arcserver.js):

1. Go to your original n_arcserver folder and delete all files EXCEPT settings.json (unless you want to reenter info)
2. [Redownload](https://github.com/n-arc/n_arclogger/archive/master.zip) the repository.
3. Drag all the files in the new folder you downloaded into the old one EXCEPT for settings.json
4. Open a terminal or command prompt and type `cd `, then drag the folder into the window and release. Hit enter.
5. Type in `git add --all` [Enter]
6. Type in `git commit -am "Update"` [Enter]
7. Type in `git push heroku master` [Enter] and let it run.

Sometimes you also have to update dependency files like roblox-js which the module requires:

1. Open a terminal or command prompt and type `cd `, then drag the folder into the window and release. Hit enter.
2. Type in `heroku config:set NODE_MODULES_CACHE=false` [Enter]
3. Type in `git commit --allow-empty -m "Rebuild"` [Enter]
4. Type in `git push heroku master` [Enter] and let it run
5. Type in `heroku config:unset NODE_MODULES_CACHE` [Enter]

## Lua Example

A module script is available in [n_arcserver.mod.lua](/lua/n_arcserver.mod.lua) that allows you to use functions to send commands to the server. An initializer function is returned by the module and requires the arguments `domain` and `key`. If the third `group` argument is provided, the returned table will automatically call group-related functions with that groupId, otherwise it has to be the first argument.

The commands `promote`, `demote`, `setRank`, `shout`, `post`, `handleJoinRequest`, `forumPostNew`, `forumPostReply`, and `message` are available, all arguments are in the same order as they are in the documentation, with parameters first and then each body argument in order (excluding key). The return value of the function is the decoded table that the API returns.

Example usage, assuming ModuleScript is named "n_arcserver" and is in the same directory as the script (eg. both ServerScriptService):
```lua
local server = require(script.Parent.n_arcserver)
local domain = 'n_arc.herokuapp.com' -- Make sure there is no http:// in here!
local key = '/UAO9lTOYapr8ecV8cs/t3cP9c7na6rKHfRn7M6GDct+PdJyQJ40Jebe+CKZDgKV8TRLtbBqfhJc/eHNC7RHA8BCKkrFOkaIKC9/ripy34QzLq3m2qqy/GdyCg/5KHFUPbsuRNetr52ZP+6E2puKWrR9XvuAMG9bq+X02luwmID6aU7YBpq7sALl21Pv0OB4wy43VhuI3esN8w/Rl0ZC3LiJWwMv8PnwCKqgmq9L9UXLVBEPNJ9Plcv73+QqArHqiZ/qtrJO88='
local groupId = 18
local userId = game:GetService'Players':GetUserIdFromNameAsync'n_arc'
local api = server(domain, key, groupId)
print(api.promote(userId).message)
print(api.message(userId, 'Subject', 'Body').message)
print(api.getBlurb(1).data.blurb)
```

## Documentation

### METHOD /example/{argument1: type}/{argument2: type}/[optional argument: type]?[optional_argument: type]&[optional_argument2: type]
```http
Example usage
```

{argument3: type}

Description

Post data is under the header, in this example is starts with argument 3. This should be sent in json format or it will not be read correctly.
All functions respond with a json table containing the fields `error`, `message`, and `data`. The `error` field is always visible but may be null, the `message` field contains a human-readable message summarizing the pages action, and `data` contains response data from the API is applicable.

[response1: type, reponse2: type]

### POST /demote/{group: number}/{target: number}
```http
/demote/18/2470023
{"key": "hunter2"}
```

{key: string}

Sets the role of the player to the adjacent lower-rank role.

[newRankName: string, newRank: number, newRoleSetId: number]

### GET /getBlurb/{userId: number}
```http
/getBlurb/2470023
```

Gets the blurb of the user with `userId`.

[blurb: string]

### POST /getPlayers/delete/{uid: string}
```http
/getPlayers/delete/2f08e9796a
{"key": "hunter2"}
```

{key: string}

Deletes the getPlayers job with id `uid` from the filesystem if complete or the list if not. Note that if it is not complete it will still be running on the server though it cannot be accessed.

### POST /getPlayers/make/{group: number}/[rank: number]?[limit: number]&[online: boolean]
```http
/getPlayers/make/147864?limit=1&online=false
{"key": "hunter2"}
```

{key: string}

Gets the players in group with group ID `group`. If `rank` is not specified it gets all players from all ranks, otherwise it gets all players from the role with that rank. If `online` is true only online users will be collected. If `limit` is enabled users will be returned in the same order they are visible in the group pages to the set number and forever if the number is -1. Note that this slows down the function considerably. The function returns a `uid` that can eventually be used to get the resulting player list (if the page just waited it could time out since this action can take a while). The file containing players is stored on the file system in the folder `players` and is not cleared by this script, therefore the result will be usable across restarts. See below for retrieving.

[uid: number]

### GET /getPlayers/retrieve/{uid: string}
```http
/getPlayers/retrieve/2f08e9796a
```

Gets the result of the getPlayers job, returning `progress` in percent when not complete while `complete` denotes whether or not it is. The players are in json object `players`.

[progress: number,
complete: boolean,
players (object): {username (string): userId (number)}]

### POST /handleJoinRequest/{group: number}/{username: string}/{accept: boolean}
```http
/handleJoinRequest/18/Froast/true
{"key": "hunter2"}
```

{key: string}

Searches for the join request of user with username `username` in the group with group ID `group` and accepts them if `accept` is true and denies them if it is false (note that for either case you still need the parameter in the url)

### POST /message/{recipient: number}
```http
/message/2470023
{"subject": "Test", "body": "Test", "key": "hunter2"}
```

{subject: string,
body: string,
key: string}

Messages user with ID `recipient` with a message that has subject `subject` and body `body`.

### POST /promote/{group: number}/{target: number}
```http
/promote/18/2470023
{"key": "hunter2"}
```

{key: string}

Sets the role of the player to the adjacent higher-rank role.

[newRankName: string, newRank: number, newRoleSetId: number]

### POST /setRank/{group: number}/{target: number}/{rank: number}
```http
/setRank/18/2470023/2
{"key": "hunter2"}
```

{key: string}

Sets rank of player with user ID `target` to rank with rank number `rank` in group with group ID `group`. Responds with the role set ID of the user's updated rank, `newRoleSetId`.

[newRoleSetId: number]

### POST /shout/{group: number}
```http
/shout/18
{"message": "Test", "key": "hunter2"}
```


{message: string,
key: string}

Shouts in group with group ID `group` and the message `message`.

### POST /post/{group: number}
```http
/post/18
{"message": "Test", "key": "hunter2"}
```


{message: string,
key: string}

Posts a message to the wall in group with group ID `group` and the message `message`.

### POST /forumPost/new/{forumId: number}?[locked: boolean]
```http
/forumPost/new/32?locked=true
{"subject": "Test", "body": "Test", "key": "hunter2"}
```

_NOTE: Lua function name is 'forumPostNew'_

{subject: string,
body: string,
key: string}

Creates a new forumPost with subject `subject` and body `body` in forum with id `forumId`. If `locked` is true the replies to the post will be disabled.

[newPostId: number]

### POST /forumPost/reply/{postId: number}?[locked: boolean]
```http
/forumPost/reply/201983178?locked=true
{"body": "Test", "key": "hunter2"}
```

_NOTE: Lua function name is 'forumPostReply'_

{body: string,
key: string}

Replies to forumPost with postId `postId`, the reply has the body `body`. If `locked` is true the replies to the post will be disabled.

[newPostId: number]

*Updated port of [roblox-js](https://github.com/sentanos/roblox-js) library.*
