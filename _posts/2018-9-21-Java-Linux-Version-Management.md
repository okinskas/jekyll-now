---
layout: post
title: Java Linux Version Management
---

I have several Java versions and JDKs installed on my linux machine. With the changes to
Java's versioning system and the rise of OpenJDK, it is increasingly important to be able to jump from one
environment to the next during development.

While looking for a suitable solution to managing the JDKs and JREs on my machine, I came across some that
fell short of my needs and others that were overly verbose. While exploring all the avenues I could find,
I created several key requirements I wanted with this system:

- I **should** be able to add new versions without much hassle.
- I **should not** have to interfere with the $JAVA_HOME variable.
- I **should not** have to remember overly-verbose syntax to swap versions.

# Java Version Management
Here are the steps I came up with:

- Make $JAVA_HOME dynamic.
- Use 'update-alternatives' to change versions.
- Create bash functions to easily execute commands.

## Dynamic Java Home

To create a dynamic $JAVA_HOME variable, I looked at the versions that existed on my machine and how the required
path was determined. I found that executing `which javac` would produce a symlink. Using `readlink $(which javac)`
determines where that symlink points. Stepping up two directories from that would produce where $JAVA_HOME should be.
As an added bonus, the dynamic $JAVA_HOME variable allowed me to set $JRE_HOME dynamically too.

I added the following exports to my `~/.bashrc`:

```bash
## Dynamic JAVA_HOME and JRE_HOME
export JAVA_HOME=$(dirname "$(dirname "$(readlink -f "$(which javac || which java)")")")
export JRE_HOME=$JAVA_HOME/jre
```

## Using update-alternatives

A great way of switching java versions is using the update-alternatives command. This can be done for java and javac
respectively.

Executing `~$ update-alternatives --config java` displays a list of java paths on the machine and requests user input:

`Press <enter> to keep the current choice[*], or type selection number: `

Given the command would have to be executed for java and javac, both of which would require user input, I want
to speed up this process.

## Creating Bash Functions

Now the only thing left to do was to delegate everything into functions. The exact sequence would be:
- update-alternatives for java.
- update-alternatives for javac.
- source ~/.bashrc to ensure everything $JAVA_HOME and $JRE_HOME are updated.
- print everything to confirm changes have been made.

The end result in my `~/.bashrc` looks as follows:

```bash
# Dynamic JAVA_HOME and JRE_HOME
export JAVA_HOME=$(dirname "$(dirname "$(readlink -f "$(which javac || which java)")")")
export JRE_HOME=$JAVA_HOME/jre

# Functions

## Set to Oracle Java 8
function oracle8() {
	echo 2 | sudo update-alternatives --config java >/dev/null
	echo 1 | sudo update-alternatives --config javac >/dev/null
	source ~/.bashrc
	printf "\n------------------------------------------\n\n"
	java -version
	javac -version
	echo "JAVA_HOME: $JAVA_HOME"
	printf "\n------------------------------------------\n\n"
}

## Set to OpenJDK/JRE 10
function open10() {
	echo 3 | sudo update-alternatives --config java >/dev/null
	echo 2 | sudo update-alternatives --config javac >/dev/null
	source ~/.bashrc
	printf "\n------------------------------------------\n\n"
	java -version
	javac -version
	echo "JAVA_HOME: $JAVA_HOME"
	printf "\n------------------------------------------\n\n"
}
```
It is worth noting that I have hard-coded the numeric values in update-alternatives that apply to my machine.
These would need to be adjusted to whatever the environment requires.

I also added `>/dev/null` for the update-alternatives commands to prevent the table and input request being
displayed to the user. The functions can be refactored further to avoid duplication.

Now if I want to change java versions, I can simply type `oracle8` or `open10` to switch to
Oracle JDK 1.8 and OpenJDK 10 respectively.