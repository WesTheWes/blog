---
title: "New Project: Practice App"
date: 2020-03-06T18:00:38-08:00
draft: false
---

I've been trying to think of a fun project to help me expand my programming skills. The other day I was practicing piano and I realized a large barrier I face when trying to practice is that it's hard for me to know exactly what I should be practicing. I don't have a piano teacher so I have to make up a practice regemin for myself. So of course I go online to see what I should practice and am bombarded with information. I'm going to make a little app to help with that.

## End goal
My dream is for this app be something that a musician can look at every single time they sit down to practice. It will tell them exactly what to practice, for how long, what to focus on. It should be able to take input from the practicer on how well they perform at a certain speeds and use that to adjust the future practice plan. It will also act as a log of what has been practiced and for how long.

Basically, I want it to create a tool to help people get the most out of their practice by trying to enforce **deliberate** practice and to make it fun by making sure the dificulty level allows the musician to stay in a flow state.

## First steps

Obviously this is a giant project so I'm going to start by making something that does one small thing, and then expand upon that so I don't get frustrated (I've made that mistake too many times).

I haven't planned out a roadmap or done any product design for this things but I'm feeling like writing some code right now so I came up with the following spec:

I want a command line tool that when run, goes through a loop of the following:
1. Asks what specific exercise the practicer wants to work on
2. Asks for a starting tempo and time limit for how long they want to practice this
```
What would you like to work on first?
> C Major Scale hands together
About what speed do you think you can play this at? (quarter notes)
> 110 bpm
How long do you want to work on this (minutes)?
> 5
```
3. After exercise is complete, the practicer is asked how it felt and given the following options:
4. Depending on their answer, the tempo is adjusted
```
How did that feel?
1. Impossible
2. Lots of Mistakes
3. A few mistakes
4. No mistakes but challenging (this is ideal)
5. No mistakes and too easy
> 5
Let's try it at 112 bpm
How did that feel?
...
```
5. After repeating the exercise a few times and giving `No mistakes but challenging`, increase the tempo
6. When the time limit is over, ask if they'd like to continue practicing, to move on to another topic or to end the practice
```
It's been 5 minutes, would you like to add time, move on to another exercise, or end your practice?
```
7. When they end the practice, show them a summary like this:

```
Topic                Time Practiced   Comfortable Speed
--------------------------------------------------------
C Major Scale
hands together       5 minutes        120 bpm

Left hand rootless
voicings through
12 keys              10 minutes       50 bpm

3 note voicings
measures 1-4 of
Autumn Leaves        10 minutes       57 bpm
```
8. Save the summary to a csv file with the above information and a date column

## Next steps
Nice! This actually seems like a tool I would use.

Unfortunately it took more time than expected to come up with this simple product design, and I don't have time to start coding now!

But this tool seems like it will be a great first step, plus it's complicated enough to not be trivial. In fact I have a feeling it will be a nice little challenge!

Until next time, have a beautiful day!
