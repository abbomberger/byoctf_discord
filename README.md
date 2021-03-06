# byoctf_discord - Bring Your Own [Challenge || Capture] The Flag

## TL;DR
A CTF framework that allows players to submit and complete challenges from other players. They are rewarded when players solve their challenges. 

People always want to help, but they can't play if they create challenges for us. this addresses that.

Discord provides the UI/UX 

Users can trade points amongst themselves for favors, info, etc.

I'm not a great dev so code is a hodge-podge, but it *mostly* works.

People kept asking if fs2600 was going to host Shell On The Border again. My answer internally was 'not until the byoc framework is done.' Here we are... 

---

## Features
This implements several features that are unique to SOTB or match our event's aesthetique

- User contributed challenges meaning GMs/hosts can play too. 
  - Internally validated flags (if you trust us to not look at flags)
  - Externally validated flags (if you don't trust us to not look at flags)
    - We provide a demo server for you to host on your own infrastructure. 
  - Purchasable hints
  - Reward system for user contributed challenges.
    - Some percentage of flag value
  - JSON based challenge definition.
- Flag oriented
  - Points are bound to flags (except for externally validated challenges)
  - One flag can belong to several challenges. 
    - Challenges can be composed by binding specific aspects(flags) of other challenges. 
    - Think of xbox achievements or goals; "Forensicator: solve 3 forensics flags." 
- Bonus flags 
  - flags not bound to a specific challenge. (internally they are)
  - useful for flags "found" in the environment or created in an impromptu fashion. Throw-away flags 
- Inter-player transactions via the "tip" system.
  - informal hint purchases? 
  - drug transactions?
  - provides a mechanism for players to try and social engineer points off eachother.
  - rewards for kindness?
- Team oriented
  - scores are often displayed in the context of your team.
  - hints purchased by a teammate are viewable by all teammates
  - submissions and unlocked challenges are viewable by all teammates. 
  - if you buy it, your team gets access to it.
  - You can't submit flags authored by your team, of course. 
- Public/Private mode for the scoreboard. 
  - you can see your team scores but not other teams. 
  - helps motivate teams who would give up if they felt they didn't stand a chance.
  - Can show MVPs for individual score. 
- Challenge dependencies
  - Challenges are hidden until all flags from each "parent" challenge (or dependecy) is solved. 
  - _pretty sure this is working as intended._ 
  - BYOC challenges can depend on byoc or non-byoc challenges. 
    - Allows individuals to extend a challenge that already exists. 
- Reactive points for solves
  - FirstBlood
    - Bonus for first team to solve a flag
    - default
  - Flag Decay
    - Flags become less valuable as they are solved by other teams.
    - not default, fully implemented or tested.
  - Configurable via text editor or CLI
    - decay rate
    - BYOC fees and reward 
    - firstblood rewards
    - see `settings.py` and `ctrl_ctf.py`
- User management and registration handled via discord...
  - uses account name and its descriminator e.g. `shyft#0760`
- Flag submission is ratelimited
  - tunable via settings. 
- Most commands are restricted to DM with the bot
  - prevents flag and score info leak in public channels. 
  - `!tip` is allowed in public channels to inform others that you offered up a tip. This allows you to use `@<discord_name>` if they are in the same channel. (`statewide hackery` or a dedicated CTF channel is a good place for that)
  - It can be done in the DM as well but you need the username of the recipient (like `shyft#0760`; click on their name in Discord to view that.)

## Expected Features
- ~~BYOC challenges being dependent on other BYOC challenges or Non BYOC challenges.~~ 
  - This seems to be working now.
- public http scoreboard for live events. 
- API access to challenges and solves. (removing dependecy on discord although it is integral right now; not any time soon)
- maybe an admin web gui? idk... 

## TODO / bugs
- Flag solves only report 1 challenge that it is a part of during the "congrats" message. 
- ~~You can submit a flag for a challenge that isn't unlocked yet.~~ 
  - ~~may not be a huge issue. How would you know to find it if you hadn't been prompted to search for it some how?~~
  - ~~this is more likely to occur for challenges with a lot of narrative and that build on each other.~~
    - ~~things where you are likely to have to "investigate" for perform some sort of forensics as part of a later challenge.~~
  - resolved. 100% unlock
- We chose to not show your teammates challenges or your own challenges
  - You can use `!bstat` to see your challenges. 
  - This helps prevent a teammate working on your challenges when they couldn't submit it anyway. 
---
## How to play

Key commands 
- `!reg <team_name> <team_password>` - register and join *teamname*; super case-sensitive.  
  -- wrap in quotes if you have spaces in the teamname; 
  -- if the team exists and your password is correct, you're in. 
  -- if no team exists with the name specified, the team will be created with password specified. 
  -- leading and trailing spaces are stripped from team name and password.
- `!top` - shows your score 
- `!all [tag]` - filter challenges by tag; use !tag to exclude
- `!v <challenge_id>` - detail view of a specific challenge
- `!sub <flag>` - submit a flag you find while working on a challenge
- `!esub <chall_id> <flag>` - submit an externally validated flag. (challenge should say if it's externally validated.)
- `!solves` - show all the flags your team has submitted. 
- `!unsolved` - show all of the unlocked challenges that don't have at least one submission. 
- `!log` - all transactions you particpated in (sender or recipient of a tip, BYOC rewards and fees, and solves among other things)
- `!pub` - all transactions that have happened the game. if scoreboard is private, amounts are omitted. 
- `!psol [challenge_id]` - all solves for all challenges or just challenge_id 
- `!help` - shows the long name of all of the commands. Most of the above commands are aliases or shorthand for a longer command.

---

# BYOC Challenges

## A few notes about BYOC challenges
- ### ***There is no way to edit your challenge once you commit it***
  - Make sure you don't have typos 
  - use DNS names rather than hardcoded IPs for links
  - use non-dynamic links for sharing files (google drive may yield a different link even you create a new version of the "same" file )
  - use a git repo?
  - ***Make sure it's solvable*** 
    - The bot can't prove or know that it is or isn't
  - ***ALL SALES ARE FINAL!***
- Cumulative challenge value is the sum of flag values
  - Must exceed 100 points ; configurable
- Challenge titles must be unique. 
- Flags must be globally unique.   
  - potential info leak about a flag that exists, but we just have to accept that risk... `!byoc_check` and `!byoc_commit` rate limited to help mitigate this.
- Description is limited to 1500 chars. (more of a discord UI thing. messages can't exceed 2000 chars, )
- Hints are given to user by lowest cost first. 
  - create the hint you would like to give first with the lowest cost.
  - points are required to purchase hints. (doesn't reduce value of challenge.) 
    - Admins can grant points via ctrl_ctf.py but shouldn't make a habit of it...
  - ***Question to the audience: Should authors get a portion of hint buys?*** 
- The framework doesn't (can't) host files. 
  - Link to a pastebin, google drive, github, torrent, etc. if you need storage or more space for words... 
  - Files - most are ephimeral and are deleted after 1-2 weeks
    - https://transfer.sh/ or http://jxm5d6emw5rknovg.onion/
      - 10gb files; no account; wget-able; tor service for your protection or whatever.
    - https://bashupload.com/
    - https://www.file.io/
    - https://onionshare.org/ - a good choice but requires tooling.
    - https://github.com - not ephimeral
      - **least sketch and editable by you if you make a mistake. **
      - 100mb file limit
      - maybe serve large file via transfer.sh and update the link every couple of weeks(if the event runs that long)?
  - Words
    - https://gist.github.com/
    - https://pastebin.com/
    
- By default, it costs 50% of the total challenge value (sum of flags) to post a challenge. 
- By default, your reward for the solve of a flag which is part of your challenge is 25% of that flags value. 
  - If the challenge is externally validated, it's based on the challenge value.
  - You will get a reward of 25% of the flag value everytime there is a successful solve. (not including firstblood or decayed point value)
  - Example: (assuming default settings for fees and rewards)
```
- you have a challenge with 2 flags.
  - flag 1 is worth 150 points and flag 2 is 50 points for a total of 200 points
- It will cost you 100 points to post it via `!byoc_commit` so you do.
- Now you are down `100` points
- When the first solve comes in for flag 1, you will get a reward of `37.5` points
- Now you are only down `62.5` points. 
- When the second solve for flag 1 comes in, you will get another reward of `37.5` points
- Now you are only down `25` points.
- When the first solve comes in for flag 2, you will get a reward of `12.5` points
- Now you are only down `12.5` points.
- When the third solve for flag 1 solve for flag 1 comes in, you will get another reward of `37.5` points
- At this point you have earned back your commit fee and have turned a profit of `25` points
```
  - This is only possible if your challenge is solvable. You could create some time-sink challenge that's impossible, but you'd have to pay to do so... 
  - Also, keep in mind that the BYOC fee and rewards can be tuned while the game is running. All transactions that take place after the change will reflect the newer rate and old transactions will reflect the older rate.
  - **Consider adding a rating system for BYOC challs** 
  - Admins can also prevent your challenge from being solvable if you figure out a scheme that is in violation of the spirit of the game. That depends on them and you, of course. 
- ~~This sucks to admit... but we can still find your externally validated flags if someone successfully submits it... it'll end up in the Solves table in the db (we won't know the flag before that happens though)~~
  - ~~we need to store it the flag so the `!solves` command can show you which flags you've already submitted.~~ 
  - ~~open to arguments against this or a PR to avoid it.~~ 
  - We kind of worked around this by storing a hash of the flag in the solve text rather than the flag itself. As the author you will see the hash in the solve and can hash your own flags to see which one matches. 
  - Keep in mind that we still touch the unhashed flag so you have to trust that we're not logging... 😉 good luck... 
---
## Common criticisms of the BYOC concept
- ***Creating an impossible challenge in an effort distract players.*** 
  - This is a possibility and always has been.
  - Part of developing your CTF skillset is to be able to recognize this and manage your time effectively. 
  - There is also the fact that it costs points to post a challenge. that might be enough of a deterent for most. Others, maybe not. 
  - I don't think the issue will be rampant enough to warrant scrapping the idea.
- ***Creating a trivial challenge worth a lot of points***
  - I believe that this issue will self-regulate. 
  - If it's just a simple challenge and everyone can solve it, then it doesn't have much bearing on the final outcome. 
  - If everyone's a loser, no one's a loser. 
  - There's also a task management component to this. Don't let one of the high value easy flags slip by. 
- If you don't want to risk it and avoid BYOC, use `!all !byoc` 
  - `!` like _not_ or a logical inversion. 
---
## Notes or guidance for developing challenges.
Most of the following are considerations regarding building your challenge. 
- ***Defining outcomes/objectives***
  - How much prior knowledge do you have to have in order to accomplish the task? 
    - Can you learn all of it during the amount of time that the challenge is available
  - Is this something targeting a beginner in infosec or a beginner in a certian infosec discipline
    - computer n00b vs web app pentesting n00b but 10 year network engineering vet vs ...
  - Are you validating experience or guiding a learning experience?
- ***Assigning a score to your challenge***
  - What is the range or scale of your event?
    - many are done in 25-50 point increments upto 500
    - This is arbitrary. 
  - how "off the shelf" is the attack/vector?
    - does a metasploit module already exist?
  - How "custom" is the target system or attack technique?
    - stock config = less understanding of the application required to exploit
    - intentionally misconfigured = more understanding of the application required to exploit
    - how recent is the application, exploit, or technique?
    - 
  - What skills are required to accomplish the challenge?
    - how much customization/tailoring of existing exploit code is required?
    - How much code
  - How impactful is the challenge
    - "gee whiz" factor for the uninitated
  - ? 
- ***What infrastructure do I(the author of the challenge) need to have in place?***
  - This doesn't/shouldn't have much bearing on the score. 
  - **REALLY** depends on the challenge
    - You should have one server per team if:
      - attempting to solve the challenge (attacking/exploiting the server) has a high probability of reducing other teams ability to solve.
      - this is like having someone reset your box on hackthebox just after you get a shell. (realistic-ish but frustrating) 
    - You could use a shared server if:
      - if attacking the server has a high probability of reducing other teams ability to solve. 
  - Docker is a fair middle-ground regarding hosting a suite of challenges on a single server while minimizing exposure/risk of compromising other challenges if exploited. 
    - We have some dockerfiles and control scripts to talk about that if the time comes. 
  - The external validation server (or one like it) if you choose to not share your flags with the bot.
  
---
## Submitting a challenge 
- Validate your challenge by attaching the json file in a DM to the bot with the command `!byoc_check`
  - If it's valid, an extended preview will show up showing the cost. 
- Commit your challenge (actually post it) by attaching the json file in a DM to the bot with the command `!byoc_commit`
  - If the challenge is valid, the preview will show up and you will be prompted to type `confirm` within 10 seconds.
    - If you do, you are charged the fee (again, by default 50% of the cumulative value) and the challenge is made available to others.
- Others can use `!all byoc` and `!v <chall_id>` to see it. 
  
---
A basic single flag challenge.
```json
{
    "author": "Combaticus#8292",
    "challenge_title": "r3d's challenge",
    "challenge_description": "good luck finding my flag -> https://pastebin.com/tiRMg9dR",
    "tags": ["pentest"], 
    "flags": [
        {
            "flag_title": "r3d flag", 
            "flag_value": 200,
            "flag_flag": "FLAG{this_is_a_flag_from_r3d}"

        }
    ], 
    "hints": [
        {
            "hint_cost": 10,
            "hint_text": "the flag is easy"
        }
    ]
}
```
A challenge with multiple flags.
```json
{
    "author": "Combaticus#8292",
    "challenge_title": "r3d's multi-flag challenge",
    "challenge_description": "good luck finding my flags at 3.43.54.28",
    "tags": ["pentest", "forensics"], 
    "flags": [
        {
            "flag_title": "flag 1 ", 
            "flag_value": 100,
            "flag_flag": "FLAG{this_is_flag_1_for_multiflag}"

        },
        {
            "flag_title": "flag 2", 
            "flag_value": 300,
            "flag_flag": "FLAG{this_is_flag_2_for_multiflag}"

        }
    ], 
    "hints": [
        {
            "hint_cost": 25,
            "hint_text": "Both of the the flags are easy"
        }
    ]
}
```
A challenge that depends on other challenges (by challenge ID)
```json
{
    "author": "Combaticus#8292",
    "challenge_title": "r3d's child challenge",
    "challenge_description": "good luck finding my flag",
    "tags": ["pentest"], 
    "depends_on": [6,7],
    "flags": [
        {
            "flag_title": "r3d dependent flag ", 
            "flag_value": 200,
            "flag_flag": "FLAG{solved_6_7}"

        }
    ], 
    "hints": [
        {
            "hint_cost": 10,
            "hint_text": "the flag depends on solving chall 6 and 7"
        }
    ]
}
```
An externally validated challenge.
```json
{
    "author": "Combaticus#8292",
    "challenge_title": "r3d's external challenge",
    "challenge_description": "good luck finding my flag. ",
    "tags": ["coding"], 
    "hints": [
        {
            "hint_cost": 25,
            "hint_text": "the flag is also easy"
        }
    ],
    "external_challenge_value": 250, 
    "external_validation": true, 
    "external_validation_url": "http://mydomain.com:5000/validate"
}
```
- Clone the server code -> https://github.com/ShyftXero/byoctf_ext_validation
- Start the external validation server prior to checking your challenge. part of the check is to see if it's able to validate. 
- You won't know your challenge ID to update the `flags.json` on your validation server until you actually commit your challenge and pay the fee. 
  - just use a text editor. and up date the key that corresponds to the flag. 
  - you only need one server to validate multiple challenges. 


There are a couple of other examples in the `example_challenges` folder... 


---

# Setup

add your own token to `secrets.py`

```bash
git clone https://github.com/ShyftXero/byoctf_discord
cd byoctf_discord
echo "DISCORD_TOKEN='asdfasdfasdf'" > secrets.py # https://discord.com/developers/applications; setup a bot  
./ctrl_ctf.py DEV_RESET # creates the db and fills with test data by calling populateTestData.py
# or ./ctrol_ctf.py INIT # for no test data
python byoctf_discord.py
```

---
# Info that may be redundant... 
This was built with Shell On The Border in mind so it may not be suitable for any other events. 

Whoa... this was harder to do that I thought it would be... but CTFs are fun and so is running them.

lot's of edge cases to consider for you pesky hackers...


I wanted to build this for the next iteration of SOTB or any CTF that fs2600 crew participates in running. 

This incarnation lives in the Arkansas Hackers discord server as a bot. 

It's meant to be interacted with primarily via DM, but there are a couple of reasons to interact with it in a public channel. 

One of the designs goals that I thought would be interesting was the concept of being able to trade points between users. 

We have other odd-ball games like 0x21 at SOTB and we wanted a shady grey market of info trading to be created. 

We want points to approximate a 'currency' during the CTF. You can send a `tip` to so someone for helping you or to settle some debt.  

I wanted the framework to be "flag oriented" rather than "challenge oriented". Points are associated with a flag rather than the challenge itself. 

A challenge's value is the sum of the values of its flags. this gives us partial completion. 

Also a single flag can exist as part of multiple challenges. This allows for interesting interweaving of challenges (for the GMs; not the BYOC players yet...)

One of the key features (and main reason for building this) is the ability to allow users to partcipate in the fun of creating challenges. 

They describe their challenge in a json file and send it to the automaton.

The bot doesn't/can't host any files so that is on the challenge author to figure out. The description is also limited to 1500 chars. Link to a pastebin, google drive, mega, torrent, etc. if you need storage or more space for words... 

Users have to pay a percentage of the value of their challenge before it becomes available to other players. 

This is to provide *some* incentive for it to be solvable by others (not a guarantee). 

They are given a reward for every team that solves it. this can become a money maker for a team. 

For example, player1 a challenge worth 500 points costs 250 points to post. (50% of value; this is tunable) 

player2 can submit the flag for the 500 points (and possible firstblood bonus of 10% for 550 points; again tunable) 

player1 will receive a reward of 25% (125 points; also tunable).

Two separate teams will have to submit the flag in order for player1 to break even. 

They will profit on the third solve. 

Obviously, teammates can't solve your flags. 

This framework is very team oriented.

No limit on team size. 

So, for the paranoid, you don't have to trust us with the flags. Although, this does make it harder to have a unified experience and violates the 'flag oriented' design, but... whatever. 

we provide a basic server for you to host somewhere publicly accessible. When someone tries to submit a flag, we forward the request to your server and you confirm whether it was correct or incorrect. 

You're limited on what you can do in this mode. One flag per challenge. 

This is nice because it allows the GMs to be able to compete as well. 

You just have to trust that we aren't capturing all of the requests that we forward to you for validation... we aren't... 


that's all the documentation for now... 