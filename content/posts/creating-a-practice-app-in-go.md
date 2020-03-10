---
title: "Creating a Basic Piano Practice Command-Line App in Go"
date: 2020-03-09T12:19:55-07:00
draft: false
---
Based on the spec laid out in my [previous post]({{< ref "posts/new-project-practice-app.md" >}}), I'm creating as simple of a golang app as possible to satisfy everything.

First and foremost I am trying to make a working product, so I'm resisting an urge to optimize and make everything DRY. Of course if I notice I'm reusing a ton of the same code I'll wrap it in a function to make things readable.

## Representing each exercise

I'm going to create a little struct to represent exercises. All I care about is what is being practiced, the total time practiced, and the final bpm.

Thinking ahead, I also think I might need an end time to know when to move on to the next exercise, so instead of a "time practiced" field, I'm creating a starting and ending time.

```golang
type exercise struct {
	name      string
	startTime time.Time
	endTime   time.Time
	maxBPM    int
}
```

## Creating the flow of the whole app

First I want to get an overview of what the app is doing, THEN worry about user inputs... So I created some functions to abstract getting user inputs, then worked out the main loop of my app in `func main()`.

After some trial and error, this is what I came up with.

```golang
func main() {
	var exercises []exercise
	for {
		name, minutes, bpm := getExerciseInfo()
		startTime := time.Now()
		endTime := startTime.Add(time.Duration(minutes) * time.Minute)
		maxBPM := 0
		for {
			for {
                // Practice once then rate difficulty
                fmt.Printf("Repeat this exercise at %v bpm\n", bpm)
				difficulty := getDifficulty()
				if difficulty >= 4 {
					maxBPM := bpm
				}
				if time.Now().After(endTime) {
                    // If practiced longer than time, then continue
					break
                }
				// If still time in practice, set the new bpm and try again
				bpm = getNextBPM(bpm, difficulty)
            }
            continueExercise := continueExercise()
			if continueExercise {
                // If continuing, get length of extension from user and add it to end time and total minutes
				moreMinutes := getMinutes()
				minutes += minutes + moreMinutes
				endTime = endTime.Add(time.Duration(moreMinutes) * time.Minute)
			} else {
                // Done practicing this exercise!
				fmt.Printf("Great job practicing %v!\nYou practiced for %v minutes, and your most recent bpm was %v\n", name, minutes, bpm)
				break
			}
        }
        // Add exercise to slice of exercises, and ask if they'd like to start a new one
		newExercise := exercise{name: name, startTime: startTime, endTime: endTime, maxBPM: maxBPM}
		exercises = append(exercises, newExercise)
		continuePractice := continuePractice()
		if continuePractice {
			break
		}
	}
	fmt.Println(exercises)
}
```

## Getting user input

This works great! Okay now to figure out how to get user input. Stack Overflow provided me with this, which is basically just creating a `bufio.Scanner` using standard input and then using `Scan`

```golang
scanner := bufio.NewScanner(os.Stdin)
fmt.Print("What would you like to work on first?\n>")
for {
    s.Scan()
    name = s.Text()
    if len(name) == 0 {
        fmt.Print("\nExercise name cannot be blank\n>")
    } else {
        break
    }
}
```

Seems like a fine way to process inputs to me! 

I know I'll be using this over and over again to prompt the user so I made a function to abstract the above. It takes two parameters: a string to display to the user and a validator function to check that the user has valid input. If the validation function returns an error, it displays that error to the user and loops until the user provides a valid input.

```golang
func getUserInput(inputString string, isValidInput func(string) error) string {
	scanner := bufio.NewScanner(os.Stdin)
	for {
		fmt.Println(inputString)
		fmt.Print(">")
		scanner.Scan()
		input := scanner.Text()
		if err := isValidInput(input); err != nil {
			fmt.Printf("%v\n", err)
		} else {
			return input
		}
	}
}
```

And here's an example of a validation function:

```golang
func testPostitveInteger(input string) error {
	if convertedInt, err := strconv.Atoi(input); err != nil || convertedInt < 1 {
		return errors.New("Input must be a positive integer")
	}
	return nil
}
```

Using that I could start filling out the function that I used in the original loop

```golang
func getExerciseInfo() (name string, minutes int, bpm int) {
	name = getUserInput("What would you like to work on?", testStringLength)
	bpmInput := getUserInput("About what speed do you think you can play this at? (quarter notes)?", testPostitveInteger)
	bpm, _ = strconv.Atoi(bpmInput)
	minutesInput := getUserInput("How long do you want to work on this (minutes)?", testPostitveInteger)
	minutes, _ = strconv.Atoi(minutesInput)
	return
}
```

I'm not checking for an error when converting to an int because that's already being tested in `testPositiveInteger` (there is probably a nicer way to handle this but I'm not going to worry about that now).

## Putting it all together

Okay I finished! Here's what the entire file looks like (I wrote everything in `main.go`, don't judge me I have no idea what I'm doing!)

```golang
package main

import (
	"bufio"
	"errors"
	"fmt"
	"math"
	"os"
	"strconv"
	"strings"
	"time"
)

type exercise struct {
	name      string
	startTime time.Time
	endTime   time.Time
	maxBPM    int
}

func main() {
	var exercises []exercise
	for {
		name, minutes, bpm := getExerciseInfo()
		startTime := time.Now()
		endTime := startTime.Add(time.Duration(minutes) * time.Minute)
		for {
			for {
				fmt.Printf("Repeat this exercise at %v bpm\n", bpm)
				difficulty := getDifficulty()
				newBPM := getNextBPM(bpm, difficulty)
				if endTime.Before(time.Now()) {
					break
				}
				bpm = newBPM
			}
			continueExercise := continueExercise(minutes)
			if continueExercise {
				minutesInput := getUserInput("How long do you want to work on this (minutes)?", testPostitveInteger)
				moreMinutes, _ := strconv.Atoi(minutesInput)
				minutes += minutes + moreMinutes
				endTime = endTime.Add(time.Duration(moreMinutes) * time.Minute)
			} else {
				fmt.Printf("Great job practicing %v!\nYou practiced for %v minutes, and your most recent bpm was %v\n", name, minutes, bpm)
				break
			}
		}
		newExercise := exercise{name: name, startTime: startTime, endTime: endTime}
		exercises = append(exercises, newExercise)
		continuePractice := continuePractice()
		if !continuePractice {
			break
		}
	}
	fmt.Println(exercises)
}

func getUserInput(inputString string, isValidInput func(string) error) string {
	scanner := bufio.NewScanner(os.Stdin)
	for {
		fmt.Println(inputString)
		fmt.Print(">")
		scanner.Scan()
		input := scanner.Text()
		if err := isValidInput(input); err != nil {
			fmt.Printf("%v\n", err)
		} else {
			return input
		}
	}
}

func testStringLength(input string) error {
	if len(input) < 1 {
		return errors.New("Input cannot be blank")
	}
	return nil
}

func testPostitveInteger(input string) error {
	if convertedInt, err := strconv.Atoi(input); err != nil || convertedInt < 1 {
		return errors.New("Input must be a positive integer")
	}
	return nil
}

func testYesOrNo(input string) error {
	if !strings.EqualFold(input, "n") && !strings.EqualFold(input, "y") {
		return errors.New("Please enter y or n")
	}
	return nil
}

func testInArray(inputArray []string) func(string) error {
	return func(input string) error {
		for _, v := range inputArray {
			if v == input {
				return nil
			}
		}
		return fmt.Errorf("Input needs to be one of the following: %v", inputArray)
	}
}

func getExerciseInfo() (name string, minutes int, bpm int) {
	name = getUserInput("What would you like to work on?", testStringLength)
	bpmInput := getUserInput("About what speed do you think you can play this at? (quarter notes)?", testPostitveInteger)
	bpm, _ = strconv.Atoi(bpmInput)
	minutesInput := getUserInput("How long do you want to work on this (minutes)?", testPostitveInteger)
	minutes, _ = strconv.Atoi(minutesInput)
	return
}

func getDifficulty() int {
	testFunction := testInArray([]string{"1", "2", "3", "4", "5"})
	question := "On a scale of 1 to 5, how difficult was that?\n"
	question += "1 - No mistakes and too easy\n"
	question += "2 - No mistakes but challenging\n"
	question += "3 - A few mistakes\n"
	question += "4 - Lots of Mistakes\n"
	question += "5 - Impossible"
	difficultyString := getUserInput(question, testFunction)
	difficulty, _ := strconv.Atoi(difficultyString)
	return difficulty
}

func continueExercise(minutesElapsed int) bool {
	continueQuestion := fmt.Sprintf("%v minutes have passed. Would you like to continue? [y,n]", minutesElapsed)
	yesOrNo := getUserInput(continueQuestion, testYesOrNo)
	return yesOrNo == "y"
}

func continuePractice() bool {
	continuePractice := getUserInput("Would you like to continue practice with another exercise? [y,n]", testYesOrNo)
	return continuePractice == "y"
}

func getNextBPM(bpm int, difficulty int) (newBPM int) {
	switch difficulty {
	case 5:
		newBPM = int(math.Round(float64(bpm) / 3))
	case 4:
		newBPM = int(math.Round(float64(bpm) / 2))
	case 3:
		newBPM = bpm - 5
	case 2:
		newBPM = bpm
	case 1:
		newBPM = bpm + 2
	}
	return
}
```

## Actual output
And it works! Here's a me actually using it to practice: (well I only practiced for 1 minute but I'm counting it)

```
$ go run ./
What would you like to work on?
>C major scales, 2 octaves, hands together            
About what speed do you think you can play this at? (quarter notes)?
>120
How long do you want to work on this (minutes)?
>1
Repeat this exercise at 120 bpm
On a scale of 1 to 5, how difficult was that?
1 - No mistakes and too easy
2 - No mistakes but challenging
3 - A few mistakes
4 - Lots of Mistakes
5 - Impossible
>3
Repeat this exercise at 115 bpm
On a scale of 1 to 5, how difficult was that?
1 - No mistakes and too easy
2 - No mistakes but challenging
3 - A few mistakes
4 - Lots of Mistakes
5 - Impossible
>3
Repeat this exercise at 110 bpm
On a scale of 1 to 5, how difficult was that?
1 - No mistakes and too easy
2 - No mistakes but challenging
3 - A few mistakes
4 - Lots of Mistakes
5 - Impossible
>2
Repeat this exercise at 110 bpm
On a scale of 1 to 5, how difficult was that?
1 - No mistakes and too easy
2 - No mistakes but challenging
3 - A few mistakes
4 - Lots of Mistakes
5 - Impossible
>2
1 minutes have passed. Would you like to continue? [y,n]
>n
Great job practicing C major scales, 2 octaves, hands together!
You practiced for 1 minutes, and your most recent bpm was 110
Would you like to continue practice with another exercise? [y,n]
>n
[{C major scales, 2 octaves, hands together {13804052221729533920 82969520521 0x1187420} {13804052286154043360 142969520521 0x1187420} 0}]
```

Okay the summary is not up to spec...definitely can't tell my bpm or time practiced using that. But I'll fix that next time!

Using this was actually kind of nice! I like the simplicity of having a machine tell me exactly what to practice and to know there is a time limit set. Feels like playing a video game!

## Ideas for next time

Obviously I'd like to clean up the summary, but as I was writing this I had a ton of more ideas of what to do with this app. Here's a few:

- Making the `getNextBPM` function a little bit more complex, maybe it can use more than just the very last difficulty rating and bpm to calculate the next speed to practice at
- Creating a list of exercises before the practice starts
- Allowing user to create a list of predetermined exercises to pick from
- Remembering the previous BPMs of exercises
- Displaying a graph of the practicer's progress over time.
- Organizing exercise by type (warmup/technique, repitoire, language)
- Randomizing exercises to keep things fresh and unexpected
- Optimizing to focus on weak areas (like having a higher percentage chance of picking a scale that you aren't as good at)

Eventually I would also like to allow users to interact with an API which would obviously involve making all the input a lot more independent of stdin. Maybe I could give the exercise `struct` some methods, and maybe a more abstract interface.

The end goal is to have the input come through an API, but I'd like to keep the command line too available too, so I'll need to separate the behavior of the practice session from the command line. I can create a slick frontend and life will be beautiful!

But I'm getting waaaaay ahead of myself. There are so many things to do, but if I start looking too big picture I tend to get get frustrated with all the small things. So I'm going to continue expanding this project in the smallest possible ways I can think of, and I can't wait! Until next time!
