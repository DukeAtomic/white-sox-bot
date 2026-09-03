# Build your own Bluesky mirror bot 🦋

This is a template repo for building a [Bluesky](https://bsky.app/) bot that mirrors an X/Twitter account by pulling from existing [sports mirror bots on Mastodon](sportsbots.xyz) as a work around for lack of access Twitter's API.

## Table of Contents
* [Credits](https://github.com/AceOfBens/sports-mirror-bot-bsky/blob/main/README.md#Credits)
* [Instructions](https://github.com/AceOfBens/sports-mirror-bot-bsky/blob/main/README.md#Instructions)
* [FAQ](https://github.com/AceOfBens/sports-mirror-bot-bsky/blob/main/README.md#FAQ)

## Credits
* [Phil Nash](https://github.com/philnash) for providing the code for building a bot that posts on its own schedule. [Phil Nash's Bluesky bot template](https://github.com/philnash/bsky-bot)
* [acarters](https://github.com/acarters) for providing the code for building a bot that mirrors a X/Twitter account by pulling from existing mirror bots on Mastodon, due to the access restrictions on X's API. Most of the notes in the code that explain what each line does are from him.
* I ([Ben Ace](https://bsky.app/profile/aceofbens.com/)) cannot stress enough how much I didn't do much to this code and can't take credit for any the building of this repo, but I did edit this template and outline the instructions below! I am not a developer but a [graphic designer](https://aceofbens.com/) whose coding experience ends at tinkering with HTML and CSS on WordPress once in a while.

## Instructions

Below, I outline the lines in each file where you will need to plug in information to tell the Bluesky account you have set up for your mirror bot to pull from the correct Mastodon account, as well as a few features that do not need to be changed for the bot to operate but are helpful to know about in case you want to change the settings. Not all files need to edited (only 3 in total!), and, in fact, most of them should not be touched unless you are a developer who knows what you are doing!

### [/ .github / workflow / post.yml](.github/workflow/post.yml)

#### 🔷 Line 4
`- cron: "* * * */12 *`

**Purpose:** This line tells the bot how often to run.

**What to change:** This format can be confusing at first, but an easy way to help you visualize it is to use [crontab guru](https://crontab.guru/). Typically, I keep this setting at every 20 minutes, which would be `*/20 * * * *` by this format.

Whatever you set is what Github will use as a guideline, but from experience, I will tell you that no matter what you set, Github will run on its own timeline, meaning that if you schedule it to run every 20 minutes, then during the day (Eastern Time), it will run every 10 to 30 minutes, and on many evenings it will run every 30 minutes to 2 hours.

If the bot is slow to catch up, you can go to the Actions tab, select the most recent workflow (Post to Bluesky), and re-run all jobs to push the bot to run again, if you so wish.

#### 🔷 Lines 14-15
```
- run: npm install axios
- run: npm install tsl-mastodon-api
```

**Purpose:** These must be installed for the bot to run properly.

**What to change:** After the bot runs successfully the first time, these two lines can be removed as they do not need to be installed on every workflow.

#### 🔷 Lines 21-22
```
BSKY_HANDLE: ${{ secrets.BSKY_HANDLE }}
BSKY_PASSWORD: ${{ secrets.BSKY_PASSWORD }}
```

**Purpose:** This gives the bot the login info for the Bluesky account you've set up for it.

**What to change:** DO NOT edit these lines directly!! Instead of adding the username and password directly to the code, go to the repository's Settings > Secrets and variables (under the Security heading) > Actions > New repository secret

For **Name**, type in `BSKY_HANDLE` or `BSKY_PASSWORD`
For **Secret**, type in the handle (ex. @notphillies.bsky.social) or the password (can be the password you set up on Bluesky or an app password).

### [/ src / lib / bot.ts](src/lib/bot.ts)

#### 🔷 Line 29
`dryRun: true,`

**Purpose:** Toggle whether the bot is in test mode.

**What to change:** For the sake of organization, I included this step here, but **this should be done last** upon set up. When you are ready for the bot to begin running, change `true` to `false`.

This can be toggled on and off (`false` or `true`, respectively) for any reason in the future, such as if you are editing the code and want to avoid the possibility of the bot automatically running before you've fixed the bug or changed whatever settings you're trying to update.

#### 🔷 Line 83 (optional/if needed)
`var postNum = 20;`

**Purpose:** Tells the bot how many posts to check on the Mastodon account when checking for updates.

**What <ins>can</ins> change:** I (Ben Ace) have temporarily lowered this number to 5 (or even lower) when I've decided to update other settings in a way that would cause the bot to double post. 

For example, I add settings for the bot to substitute players' X/Twitter handles with their first and last name in the Bluesky post. When a new player joins the team, and the team account tags them, I will add them to the list of substitutions. If the Bluesky post has already received engagement (quotes, reposts, replies), typically, I will leave it up, and if it is in the 20 most recent posts from the Mastodon mirror bot, this Bluesky bot will double post with the newly added substitution.

To work around this, I change this setting to a lower number (ex. if the post I don't want it to double post is the third most recent, I will change this setting to 2). Then, the next day, I will change it back to 20. Keep in mind that if the account you're mirroring posts frequently, setting this number too low will potentially result in missed posts.

#### 🔷 Line 84
`var bskyFeedAwait = await this.userAgent.app.bsky.feed.getAuthorFeed({actor: "CHANGETHIS.bsky.social", limit: postNum,});`

**Purpose:** Tells the bot which Bluesky account's feed to compare the posts from the specified Mastodon account to.

**What to change:** Put the Bluesky handle of the account you have set up for your mirror bot (ex. notphillies.bsky.social) in place of `CHANGETHIS.bsky.social`.

#### 🔷 Line 149 (Change if/when Bluesky updates their video length limit from 3 minutes)
`if (urls[0].slice(-3) == "mp4" && parseFloat(alts[0].split("@#*")[2]) < 180 && limits.canUpload == true)`

**Purpose:** Gonna be so real with y'all, I (Ben Ace) am not entirely sure what the purpose of the line is, but it has something to do with the time limits on Bluesky videos.

**What to change:** When Bluesky updated their limits on video from 1 minute to 3 minutes, I noticed that the bots were still pulling the thumbnail of videos longer than a minute and adding a note in the alt text that the video exceeds Bluesky's limits. After digging around the code for a bit, I noticed that where it currently says `180` in the line above, it had said `60`. So, if Bluesky increases the time limit on videos again, update this line with the new time limit in number of seconds.

#### 🔷 Line 175
`cardResponse = await axios.get("https://media.d3.nhle.com/image/private/t_ratio16_9-size50/v1697721957/prd/assets/flyers/assets/phi_2568x1144.png", { responseType: 'arraybuffer'});`

**Purpose:** If the bot has trouble pulling a card from the source post and buffers for too long, this is the image it will post in place of the card.

**What to change:** Currently, there is an image file with the Philadelphia Flyers logo on it in this setting. I have yet to see the bot post this photo, but just in case your bot does, change this photo to avoid embarrassment.

#### 🔷 Line 218 (optional/if needed)
`var postNum = 20`

**Purpose:** Tells the bot how many posts to check on the Bluesky account when checking for updates.

**What <ins>can</ins> change:** Same as for Line 83, this can be edited if you're trying something out. Personally, I haven't found a need to change Line 218 yet, but I wanted to point it out in case it's useful.

### [/ src / lib / getPostText.ts](src/lib/getPostText.ts)

#### 🔷 Line 2
`const mastodon = new Mastodon.API({access_token: 'paste_access_token_here', api_url: 'https://mastodon.social/api/v1/'});`

**Purpose:** Allows the bot to access Mastodon's API. In other words, it allows the bot to view posts on Mastodon.

**What to change:** To obtain an access token, first create an account on [Mastodon](https://mastodon.social/). Then go to Settings > Development > New application.

Set the Application name as whatever will help you remember this is the access token you're using for your Bluesky mirror bot. The scopes I allowed on mine are `read:accounts`, `read:statuses`, and `profile`.

It will then give you your access token which will look like 40+ characters of keyboard smashing, and you will copy and paste that where it says `paste_access_token_here` in Line 2. Make sure to keep the apostrophes around it.

If you have your repository set to public, this may be another thing you keep secret, in which case, you would put `${{ secrets.ACCESS_TOKEN }}` in place of the access token and follow the same steps you did to [set up the login info](https://github.com/AceOfBens/sports-mirror-bot-bsky/blob/main/README.md#-lines-21-22).

#### 🔷 Line 25
`var nhlflyersReg = new RegExp("@nhlflyers@sportsbots.xyz", "g");`

**Purpose:** Replaces the handle of the Mastodon account the bot is mirroring with the handle of the Bluesky bot.

**What to change:** Rename the regex `nhlflyerReg` to anything you want in the format of `nameReg`, then replace `@nhlflyers@sportsbots.xyz` with the Mastodon handle your bot is mirroring. It is likely the same as a the X/Twitter handle with "@sportsbots.xyz" at the end.

#### 🔷 Line 87
`contentString = contentString.replace(nhlflyersReg, "notflyers.bsky.social");`

**Purpose:** Tells the bot what to replace the text specified by `nhlflyersReg` with

**What to change:** After editing Line 25, go to Line 87. Replace `nhlflyersReg` with whatever you renamed your regex in Line 25. Replace `notflyers.bsky.social` with the handle of your Bluesky mirror bot account.

#### 🔷 Line 28 (Optional)
`var nbcsphillyReg = new RegExp("@nbcsphilly", "g");`

**Purpose:** Tells the bot to look for this term/hashtag/handle/other text in the posts it pulls from Mastodon.

**What to change:** This is an example I left in for if you want the bot to change X/Twitter handles that it commonly tags to the person's or organization's name. For example, I have these set up for all the players' usernames for the Phillies and Flyers mirror bots. (See: [Phillies mirror bot code](https://github.com/AceOfBens/notphillies-bsky/blob/main/src/lib/getPostText.ts) [Lines 27-53 and 120-146], [Flyers mirror bot code](https://github.com/AceOfBens/notflyers-bsky/blob/main/src/lib/getPostText.ts) [Lines 25-60 and 123-156]).

In this specific example, I've replaced NBC Sports Philadelphia's X/Twitter handle with "NBCSP" because the Flyers will tag the TV station (which is NBCSP for most games) and radio station where you can tune into the game at the beginning of all 3 periods. In most cases, Bluesky users will still understand who is being referenced if a Twitter handle is left in the post, but to ensure the text of the Bluesky posts remain accessible (especially since users can't click on the tagged username and go to a profile to see what is tagged), I chose to create these text substitutions.

**To create your own**, the only two things you want to change from this example are:
`nbcsphillyReg` - This names the regex and connects it to the next step. It really doesn't matter what you name it, but it helps you remember why you added it.
`@nbcsphilly` - You'll need to change the first term in the parentheses to whatever text you want replaced. Make sure you keep quotation marks around this text.

#### 🔷 Line 89 (Optional, but necessary if you did the previous step)
`contentString = contentString.replace(nbcsphillyReg, "NBCSP");`

**Purpose:** Tells the bot what to replace the text from the source post with.

**What to change:** You'll need to replace both terms in the parentheses in the format of (exampleReg, "Text you wish to replace whatever's in the source post"). Everything else in this line stays the same.

**Reminder!** If you haven't yet, go back to `/ src / lib / bot.ts`, Line 29 to [toggle testing mode off](https://github.com/AceOfBens/sports-mirror-bot-bsky/blob/main/README.md#-line-29) so your bot can run on Bluesky!

### No Change Needed

The following files in this bot **do not** need to be edited to run properly, and unless you are well versed in Typescript, it is highly recommended that you do not change anything in them.

* / src / index.ts
* / src / lib / config.ts
* / .env.example
* / .gitignore
* / .nvmrc
* / LICENSE
  * Note: The license details from Phil Nash's Bluesky bot repository is included in this template because his work is the basis for this repository and the license he specified applies.
* / README.md
  * Note: As this is a text file meant to give you the necessary information to set up your Bluesky mirror bot, technically, you can change anything in here and the bot will still run properly. That said, if you make your repository public or make it private and intend to link to it anywhere, **do not** remove the credits section at the top.
* / eslint.config.js
* / package-lock.json
* / package.json
* / tsconfig.json
* / yarn.lock


## FAQ

### Does this cost anything to make?

Generally, no. The purpose of pulling from a Mastodon mirror bot instead of directly from the X/Twitter account is because accessing X/Twitter's API is a hassle for several reasons. So, that part of it costs nothing.

However, depending on how many bots you have running through your Github account and how often you schedule them to run, you may hit [Github Actions' monthly limit](https://docs.github.com/en/billing/managing-billing-for-your-products/managing-billing-for-github-actions/about-billing-for-github-actions). Most workflows for this are billed as 1 minute, meaning each time one of these bots run, it counts toward 1 minute of the 2,000 minutes per month you're allowed on a free account. Currently, I'm paying the $4/month for Github Pro to run @notflyers.bsky.social and @notphillies.bsky.social because I can spare a few bucks a month and I feel it's a worthwhile project. If you're looking to keep a free account, this may factor into how many bots you set up and [how often you schedule the bot to run](https://github.com/AceOfBens/sports-mirror-bot-bsky/blob/main/README.md#-github--workflow--postyml).

### I've noticed that [@notflyers.bsky.social](https://bsky.app/profile/notflyers.bsky.social/) often links to a YouTube video from the Philadelphia Flyers' channel. Why isn't my bot doing that?

Whenever @notflyers.bsky.social or @notphillies.bsky.social has a YouTube video embedded, I (Ben Ace) posted that manually. I'm not sure how to set up the bot to search a YouTube channel for a corresponding video when the video they've posted to X/Twitter exceeds Bluesky's 3-minute video limit, so, if I'm available before the bot has caught up, I will post manually with the YouTube embed. If I catch it after the bot has already posted it, I'll typically link to the corresponding YouTube video in a reply.

Yes, I have too much time on my hands. It's called un(der)employment.

### I've noticed that [@notphillies.bsky.social](https://bsky.app/profile/notphillies.bsky.social/)'s profile picture and banner update for Thursday and Friday home games like it does on X/Twitter. Why isn't my bot's profile picture or banner updating when the X/Twitter account it's mirroring updates them?

That's another thing I do manually. Again, I know I have too much time on my hands.

### I'm confused about something in the instructions. How can I contact you?

Honestly, I'm probably most reachable on [Bluesky](https://bsky.app/profile/aceofbens.com). My DMs are currently open; do not make me regret that.
