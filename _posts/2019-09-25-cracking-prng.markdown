---
layout: post
title:  "Cracking PRNG used in non-secure manner"
date:   2019-09-25 22:55:41 +0200
categories: security
---
If you ever wondered how does PRNG based attacks work - here is a simple example with breakdown. It's nothing fancy or advanced but it's a cool experiment.

## Naive poker machine
Let's suppose Bob decides to create a standalone poker machine (Texas Hold'em). Because he's an honest person and doesn't want to rig the game, he wants it to be random. Because he's oblivious to problems in random number generation he decides to use System.Random class provided by .NET.

## When problem begins?
So... Why Bob can get out of business? After all System.Random does a pretty good job generating random numbers. If you generate large amount of random numbers you will end up with pretty uniform distribution.


``` C#
static void Main(string[] args)
{
    var generatedCounts = new int[10];
    var rand = new Random();
    for(int i = 0; i < 1000000; i++)
    {
        var randomNumber = rand.Next(0,10);
        generatedCounts[randomNumber]++;
    }
    Console.WriteLine(string.Join(" ",generatedCounts));
}
```


Example output:

>100220 100003 100497 100166 99911 99973 100021 99939 99293 99977

So let's assume that player and computer players have equal chance of getting 2♣ and A♠️. But distribution is not all that matters...

A single deck of 52 cards have 52! possible states. However System.Randowm have a seed of 32 bits. So if we create new instance of System.Random it can have only 2^32 states. That's a big problem.

## Bad usage of System.Random

It's easier to use an example, so let's assume that Bob's machine gives you the information on the order the cards were dealt (eg by a cool animation) and at the beggining of each game the machine creates new instance of System.Random to shuffle cards using some algorithm like Fisher–Yates shuffle.

Let's assume that you invest in one game just enough to see the flop - you now have infomration about 5 cards with order they were dealt. 

If you enumerate all possible seeds of System.Random and shuffle cards using same algorithm as machine uses, then match what you got on machine you'll end up with just a handfull of possible states the deck was when cards were dealt. If you continue to play and wait for another card to apper (turn) you'll know what card will apper on table next (river) and what card computer players have.

How long does it take to check all shuffled decks generated using new System.Random instance on modern computer? About 20 minutes. That's long but it's also easiest possible scenario to scale - you can distribute this job across multiple machines or use scalable serveless solutions offered by cloud and get result in under a minute.

If waiting or scaling is not for you there is also another possibility:

If the machine uses .NET Framework or .NET Core < 2.2 default Random seed is Environment.TickCount. That means that if you crack the seed once you get the miliseconds the machine is up. Now you can return to the same machine, add the time you were gone (in miliseconds) and try to brute force seeds around that value. If you give yourself 1 hour of margin it should take no more than 2 seconds and you'll probably know all cards after the flop. And that's how you win with poorly designed poker machine.