---
title: "Hacking For Free Red Bull"
date: 2021-05-04
tags: ["hacking"]
categories: ["cool-stuff"]
---

I've been getting this Red Bull ad on about every 9 out of 10 YouTube videos recently.
![](/images/redbullhacking/redbulladd.png)

So I decided to check it out. Heading to the [site](https://redbull.co.uk/exams) you'll find a 'study puzzle' which awaits you. The puzzle consists of connecting all the dots together, without using the same dot. There are 10 rounds, with each puzzle increasing in difficulty. After a little bit of grinding out the game, my friend Jules managed to top the leaderboard with a lovely score of **1731** (I only managed to reach 6th place). We thought, wouldn't it be cool to write a little path finding algorithm and control the browser using [Selenium](https://www.selenium.dev/) to solve the puzzles as quickly as possible?

![](/images/redbullhacking/puzzle.png)

The next day I started off by going back through my graphs and algorithms notes from last year. The problem is known as the [Hamiltonian path problem](https://en.wikipedia.org/wiki/Hamiltonian_path). I wrote out the solution in Python to the problem and then started to look at the site, to see if it would be possible, and if so how, to interact with the game via Selenium.

Inspecting the page I found that the points were in the DOM of the page, which looked hopeful. However, I'm not that well-versed with Selenium and wondered if there would be an easier way to beating the puzzle - maybe I could access some value which was used to submit my score to the leaderboard and alter it before it was sent? So that's what I set about looking for.

I wondered round the source until I found the games JavaScript file, then copied it and put it into [jsnice.org](https://jsnice.org) to go searching some more. The game was submitting the score to an api endpoint with a few different requests in order to generate some ids which were used to identify the current 'game' (which I'm assuming is the current week, the leaderboard resets every week), and also the 'run' which was your attempt at the puzzle.

Then, monitoring the network traffic in the developer tab of Chrome I looked for all traffic to '/api*'. I found that when you initially load the page a request was made to an endpoint which registered your new run and inside the response was an id. Allowing the game to timeout then caused another request to a leaderboard endpoint. I noted the run id and then took a look at what the leaderboard endpoint gave you. Here is a snippet of the response which is actually Jules' entry!

```json
{
    "data":[
    {
	"attributes":{
            "finished_at":"2021-05-03T20:12:07Z",
            "id":"992a0348-91ac-4249-91d1-011ad53ce047",
            "name":"Icy Morning",
            "position":1,
            "score":1731,
            "yours":false
	},
         "id":"992a0348-91ac-4249-91d1-011ad53ce047",
	 "links":{
            "self":"https://rb-prjct-crtns-gamebackend-eu.herokuapp.com/leaderboard_entries/992a0348-91ac-4249-91d1-011ad53ce047"
	 }
    }
    ]
}
```

Playing around with the request actually changed the position returned for my own score, but didn't seem to affect the leaderboard when playing again. This endpoint was only used to generate the little leaderboard preview once you had finished the game, no submitting of scores was done here. Moving on...

Upon pressing the submit button a request is made to another endpoint where a payload is passed in. Lucky for me, this payload contained my score! The url was made up of the game id and run id which could both be found from the initial runs request. Lovely. Now time to try and post a request of my own.

I refreshed the page and copied down the new run id without pressing the start button. Simply using curl and copying some of the request headers, I sent a post request to the correct endpoint with my ids and a score of 2000 (thinking that was the maximum as there were 10 rounds of 20 seconds each with Jules scoring 1731 while rarely spending more than 4 seconds per round). And sure enough, after pressing start and letting the game timeout, the leaderboard displayed with me at the top, score 2000. Awesome.

But was 2000 really the maximum? Of course it wasn't, why not really put myself at the top with a lovely score of 123456789. Even better. Now, let's just hope I still get those 2 cans of Red Bull.

![](/images/redbullhacking/leaderboard.png)
