---
title: "Due: A Kanban Board Task Management Script"
date: 2021-02-23T19:37:18-06:00
draft: false
tags: ["development", "script"]
---

TL;DR - The due script in my [script repository](https://github.com/hamersaw/scripts) is a shell script kanban board implementation.

TODO list management is an integral component in any workflow. There is absolutely no shortage of tools in this space, ranging from integrated online suite extensions to local text files. My UI, and broadly defined work environment, thrives on minimalist solutions. I tend to favor simplicity over robust functionality. To succinctly explain my methodology "the more things going on, the more than can (and typically do) break". This has driven many of my development choices including transitioning to Debian stable on both my laptop and home server machines rather than running a sexier alternative.

In my particular use-case I am looking for a specific set of functionality:

- Ability to logically partition tasks between home, school, work, etc

- Prioritization / scheduling of tasks

- Flexibility to attach metadata to each individual task

There are many proven solutions, probably backed by profound psychologic studies, that increase productivity x-fold by rewarding task completion and not-overwhelming users. For the last few years I relied on a simple text file, creating lists with assorted metadata. However, my increased task-load deems this solution untenable, difficult to parse, and overwhelming when confronted with the ever-growing collection of TODO items.

## Kanban Boards

Recently, kanban boards have seen a significant rise in popularity. They work by partitioning tasks using a progression of task lists. Each task is defined by a single card which transitions between lists as it iterates through the life-cycle. An typical example of the list collection for a software development cycle is "backlog" -> "todo" -> "in progress" -> "testing" -> "done". A software feature may begin in the "todo" list, move to "in progress" during development, reach the "testing" list after, transition back to "todo" after testing failures, and so-on until reaching the "done" list. In my elementary understanding, physically moving cards between task lists helps reinforce the notion that something is completing, a difficulty which plagues the tech community. An example kanban board is depicted below.

<p align="center">
  <img width="100%" src="/posts/20210223-due-a-kanban-board-task-management-script/kanban-board.png">
</p>

The rise in popularity may be largely attributed in the tech-space by applications in Agile programming. In just the last week I have used [Zenhub](https://www.zenhub.com/), which focuses on the github ecosystem tracking issues and the development cycle and [Trello](https://trello.com/) which is ran by Atlassian (think BitBucket) servicing a similar purpose for their platform. Although these solutions have relatively mature terminal-based clients (e.x. [trello-cli](https://github.com/mheap/trello-cli)) they tend to be too bulky for my simple application. Additionally, online syncing and sharing my TODO list with other users is far beyond the scope of my functionality requirements.


## Due - Shell Script

Therefore, integrated into my [scripts repository](https://github.com/hamersaw/scripts) is a tool called [due](https://github.com/hamersaw/scripts/blob/master/due). This script provides a simple shell-based kanban board implementation. Elements (boards, lists, and cards) are assigned hierarchical number identifiers which are not statically defined, but change as elements are added / removed.

To begin, we start from scratch. Below, we initialize a "home" board with 4 lists, namely "backlog", "todo", "in-progress", and "done". We use the 'add' command which takes a numeric id and title. The hierarchical length of the id determines if the title is added as a board or a list.

    hamersaw@nightcrawler:~$ due add 1 home
    [+] added board '1' 'home'
    hamersaw@nightcrawler:~$ due add 1.1 'backlog'
    [+] added list '1.1' 'backlog'
    hamersaw@nightcrawler:~$ due add 1.2 'todo'
    [+] added list '1.2' 'todo'
    hamersaw@nightcrawler:~$ due add 1.3 'in-progress'
    [+] added list '1.3' 'in-progress'
    hamersaw@nightcrawler:~$ due add 1.4 'done'
    [+] added list '1.4' 'done'
    hamersaw@nightcrawler:~$ due list 1
    1 home
        1.1 backlog
        1.2 todo
        1.3 in-progress
        1.4 done

We can add tasks in the form of cards by similarity using the 'add' command with a 3 level hierarchy.

    hamersaw@nightcrawler:~$ due add 1.1.1 'take over the world'
    [+] added card '1.1.1' 'take over the world'
    hamersaw@nightcrawler:~$ due add 1.2.1 'grocery shop'
    [+] added card '1.2.1' 'grocery shop'
    hamersaw@nightcrawler:~$ due list 1
    1 home
        1.1 backlog
            1.1.1 take over the world
        1.2 todo
            1.2.1 grocery shop
        1.3 in-progress
        1.4 done

Each task can have associated metadata by using the 'edit' command, attaching a "note" to the task (denoted by the "note" tag following the task). For example we can use vim to add grocery items to the "grocery shop" card as follows:

    hamersaw@nightcrawler:~$ due edit 1.2.1
	hamersaw@nightcrawler:~$ due list 1
    1 home
        1.1 backlog
            1.1.1 take over the world
        1.2 todo
            1.2.1 grocery shop [note]
        1.3 in-progress
        1.4 done

We use the 'move' command to transition tasks between lists.

    hamersaw@nightcrawler:~$ due move 1.2.1 1.3.1
	[|] moved card '1.2.1' to '1.3.1'
	hamersaw@nightcrawler:~$ due list 1
	1 home
		1.1 backlog
			1.1.1 take over the world
		1.2 todo
		1.3 in-progress
			1.3.1 grocery shop [note]
		1.4 done
    hamersaw@nightcrawler:~$ due move 1.3.1 1.4.1
	[|] moved card '1.3.1' to '1.4.1'
	hamersaw@nightcrawler:~$ due list 1
	1 home
		1.1 backlog
			1.1.1 take over the world
		1.2 todo
		1.3 in-progress
		1.4 done
			1.4.1 grocery shop [note]

The ability to "hide" a list has been included. I have found this is useful for the "backlog" or "done" lists to remove items which my overwhelm the user, allowing them to focus better on tasks at hand.

    hamersaw@nightcrawler:~$ due hide 1.4
    [+] hide list '1.4'
    hamersaw@nightcrawler:~$ due list 1
    1 home
        1.1 backlog
            1.1.1 take over the world
        1.2 todo
        1.3 in-progress
        1.4 done [1 hidden]

Finally, removing cards is performed using the 'remove' command.

    hamersaw@nightcrawler:~$ due remove 1.4.1
    [-] removed card '1.4.1'
    hamersaw@nightcrawler:~$ due list 1
    1 home
        1.1 backlog
            1.1.1 take over the world
        1.2 todo
        1.3 in-progress
        1.4 done [0 hidden]

There it is, a script-based kanban board implementation. The actual functionality is a bit more robust than the previous examples, providing error checking on numeric ids and support for multiple boards (e.x. home, school, work, etc).
