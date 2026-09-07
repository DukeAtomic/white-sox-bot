## Credits
* [Phil Nash](https://github.com/philnash) for providing the code for building a bot that posts on its own schedule. [Phil Nash's Bluesky bot template](https://github.com/philnash/bsky-bot)
* [acarters](https://github.com/acarters) for providing the code for building a bot that mirrors a X/Twitter account by pulling from existing mirror bots on Mastodon, due to the access restrictions on X's API. Most of the notes in the code that explain what each line does are from him.
* I ([Ben Ace](https://bsky.app/profile/aceofbens.com/)) cannot stress enough how much I didn't do much to this code and can't take credit for any the building of this repo, but I did edit this template and outline the instructions below! I am not a developer but a [graphic designer](https://aceofbens.com/) whose coding experience ends at tinkering with HTML and CSS on WordPress once in a while.

Made possible by [AceOfBens](https://github.com/AceOfBens/sports-mirror-bot-bsky/tree/main).

The tutorial Ben made is missing a VERY important step also. You must get the API ID for the bot you are trying to mirror.

https://mastodon.social/api/v2/search?resolve=true&q=https://mastodon.social/@whitesox@sportsbots.xyz

Replace @whitesox.@sportsbots.xyz with the handle you want to search for and you can find the Account ID. 
Take that ID and replace the ID in quotes that is written in mastodon.GetStatuses on Line 30 of getPostText.