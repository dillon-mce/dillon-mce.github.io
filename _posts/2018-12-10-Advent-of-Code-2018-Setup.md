---
title: "AoC 2018 - Set up"
tags: adventofcode
---
### Original Set up
The first (meta) problem with Advent of Code is to figure out how you’re going to organize your code for these challenges. How get the input into a usable format, how to keep things in order and minimize the time it takes, etc.

At first, I decided to code my solutions in Swift in an Xcode playground. The organization I landed on for that was to make a blank source file and to declare a `public let day1Input: String` in there. I then set it to a multiline literal marked with `"""` on each end and pasted in the input given.

In my playground page I defined a function called something relevant like `calculateFrequency(_ string: String) -> Int`, added a constant `let testData = "+1, -2, +3, +1"` for my tests and then called `calculateFrequency(testData)` while I was writing the function and `calculateFrequency(day1Input)` when I was ready to get my answer. As I went through the days I would typically write the function to return the answer value, but as I went on (and the  number of computations for each function went up) I found that it would print the answer way faster than the playgrounds interface would catch up and show the returned answer, so I started printing it as well.

### Current Setup
Around day 9 I ran into a wall with playgrounds. It was eating memory at an unbelievable rate and taking **way too long** to calculate. That is when I started looking for other options.

I am only tentatively comfortable with the command line, but I tried just running `swift Day-9.swift` and found that it worked. It compiled and ran and printed the print statements out to the terminal. I’m sure that probably sound obvious to you CLI experts, but I don’t really know how that stuff works. So it was cool to see.

My hurdle then was getting the input data passed to the file. I tried just putting it all into the same file, but some of the inputs are really long and because of the way it compiles, you have to put the constant above where you call the function with it, which was just ugly. (I’m sure there’s a flag you can add to the `swift` command that would fix that, but I couldn’t figure it out, so I moved on.) After some googling, I found that you can get a reference to the command line arguments with “CommandLine.arguments”, the first being the file itself and the second being whatever is passed in. So I could save my input data in a separate file and run the code by calling `swift Day-1.swift "$(cat Day-1-Input.txt)"`. Problem solved!

### Getting Into Swift Scripting
That is when I started down the rabbit hole of scipting with Swift. My first discovery was that you can add a shebang to the beginning of the file and run it as it’s own script. For swift it is `#!/usr/bin/swift`. You also need to modify the permissions with `chmod +x Day-1.swift` then all you need to run it is  `./Day-1.swift "$(cat Day-1-Input.txt)"`. That was a lot better but it still involved a lot of typing and relied on my typing everything correctly.

So I decided to see if I could write a simple bash script that would run this command for me. After reading through [this very helpful website about shell scripts](http://linuxcommand.org/lc3_writing_shell_scripts.php)  I added a script called “adventofcode” to my `usr/local/bin` directory (I don’t know if that is the right place to put these things, but it seems to work.) I declared it as a bash script with `#!/bin/bash` and wrote:

```
if [ $1 != ""]; then
	./Day-$1.swift "$(cat. Day-$1-Input.txt)"
	exit 0
else
	echo "You need to pass in a number parameter"
	exit 1
fi
```

This basically says, if we’re given an input, try to call that long and messy swift script with the associated input data. Otherwise, tell the user they are doing it wrong.

Of course, I had to modify the permissions on this new adventofcode script as well, but after I did, it works exactly as I had hoped. (One thing to note is that it is using a relative path to the swift script, so it only works if you call it from the directory where that Swift script lives.)

I quickly realized that during testing I would want to call my script without passing input data so I added a case statement to my bash script so that it looked  like this:
```
if [ "$1" != "" ]; then
  case $1 in
    -t | --test )   if [ "$2" != "" ]; then
                      ./Day-$2.swift
                      exit 0
                    else
                      echo "You need to pass in a number parameter"
                      exit 1
                    fi
                    ;;
    * )             ./Day-$1.swift "$(cat Day-$1-Input.txt)"
                    exit 0
  esac
else
  echo "You need to pass in a number parameter"
  exit 1
fi
```
This says, if I call adventofcode with a `-t` or `--test` flag, it will run the script with no input, otherwise it will run it with the associated input. (Again, I know very little about bash, so there is probably a better way to organize this.)

### Cleaning Things Up
Now I had a nice (and relatively pretty) way to run my code once I had something to run. But I found as I was moving things over to individual swift files that I generally wanted the same structure for each of these files:
- I wanted to declare it as a Swift script
- I wanted to import Cocoa
- I wanted to set my inputData as the command line argument or an empty string
- I wanted some code up top that gave me a header/reference point for the stuff printing out to the console.
- I wanted an area to put my code for part 1
- I wanted an area to do some testing for part 1
- I wanted the same for part 2
- And then at the bottom I wanted a function to run both parts and log the time it took.

So I pulled out all of those pieces and made an `_template.swift` file. Then, when I was ready to start a new day I ran this series of commands:
- `cp _template.swift Day-13.swift`
- `chmod +x Day-13.swift`
- `touch Day-13-Input.txt`

After running all those commands once, I realized I could probably just add it to my adventofcode script.  So I added a `-n` or `--new` case to my script that looks like this:
```
-n | --new )   if [ "$2" != "" ]; then
                 cp -n _template.swift Day-$2.swift
                 chmod +x Day-$2.swift
                 touch Day-$2-Input.txt
                 exit 0
               else
                 echo "You need to pass in a number parameter"
                 exit 1
               fi
               ;;
```
This makes both files for me and modifies the permissions, and all I have to do is type `adventofcode -n` and a number. I added the -n flag to cp because then it will only copy if the file you’re copying to doesn’t already exist. That let me run this same script if I already had my swift file for the day, but needed to add the input file or modify the permissions. It also makes it safer if I accidentally run it on a day that I already have code for.

After all that, I have a pretty solid little set up that lets me start coding quickly and makes my life easier without getting in the way.

You can find my full adventofcode script, my template file and all of my attempts so far in [my advent of code 2018 repository on GitHub](https://github.com/dillon-mce/advent-of-code-2018).
