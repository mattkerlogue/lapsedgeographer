---
title: "Prompt-moting a custom R prompt"
date: 2020-11-21
slug: custom-r-prompt
tags:
  - r
  - console
  - prompt
---

The default prompt in the R console merely indicates it's awaiting input. A single less than sign to signify R has nothing to do and wants you to give it a task. Back in October there was a mild buzz on [#Rstats twitter](https://twitter.com/search?q=%23rstats) about customising your R prompt after [Romain Francois](https://twitter.com/romain_francois/) gave a talk at the R Addicts Paris Meetup. As documented in this [RTask blog](https://rtask.thinkr.fr/modify-rstudio-prompt-to-show-current-git-branch/) Romain's prompt informs him of the active git branch and how much memory R is using. I've had a custom prompt for a while so I thought I'd write a short post about my setup.

## The default prompt

R's default prompt is a single less than sign, `>`, I guess used to symbolise an arrow.

```
> 1+1
[1] 2
> LETTERS[round(runif(4, 1, 10),0)]
[1] "F" "H" "F" "G"
>
```

The prompt is a simple character string, stored in `.Options`, meaning we can easily inspect it and modify it. You can even use emoji, though I'd recommend using a base emoji rather than a composite emoji that requires multiple emoji as I got some strange side effects when trying things like the rainbow flag (🏳️‍🌈) which is rendered from a combination of the white flag (🏳️) and rainbow (🌈) emoji characters.

```
> getOption("prompt")
[1] "> "
> options("prompt" = "! ")
!
! 1+1
[1] 2
! options("prompt" = "R is ready> ")
R is ready>
R is ready> LETTERS[round(runif(4, 1, 10),0)]
[1] "H" "D" "C" "A"
R is ready> options("prompt" = "> ")
>
> options("prompt" = "👉 ")
👉
👉 1+1
[1] 2
👉 LETTERS[round(runif(4, 1, 10),0)]
[1] "C" "D" "F" "J"
👉 options("prompt" = "> ")
>
```


## My custom prompt

For a long time my custom prompt simply told me the time, which given how long some code can run can be a useful way of tracking how long a chunk of code/script takes without resorting to littering your code with `start <- Sys.time()` and `Sys.time() - start` to get the elapsed time down to the millisecond. Usually I just want to know when I executed the command so if it's still running I know how long it's been going. Adding the time to the R prompt seemed a pretty nifty approach. In time I then realised that it'd also be useful to know what directory I'm working in. I've then since gone on to amend the directory to show the path from my starting point (if I'm in an RStudio project) so I know if I'm in a sub-directory or not. My most recent adaptation has been to include information about git: which branch I'm in, whether there are files that have been modified since the last commit, and whether I'm ahead of the remote origin.

### Adding the time

Adding the time might seem simple, let's just add Sys.time() to the prompt option.

```
> options("prompt" = paste(Sys.time(), "> "))
2020-11-21 16:26:53 > 1+1
[1] 2
2020-11-21 16:26:53 > LETTERS[round(runif(4, 1, 10),0)]
[1] "G" "H" "F" "H"
2020-11-21 16:26:53 > 
2020-11-21 16:26:53 > 
2020-11-21 16:26:53 > 
```

Unfortunately, that doesn't work, because the prompt is stored as the character output of the `paste()` expression and so you're stuck with the time that you set the option, not the current time. Setting the prompt to just `Sys.time()` is even worse because it just gives you the raw value from `Sys.time()` and no space, can you see where the prompt ends and the command starts?.

```
2020-11-21 16:26:53 > options("prompt" = Sys.time())
1605976093
16059760931+1
[1] 2
1605976093
```

We could insert `options("prompt" = paste(Sys.time(), "> "))` into our script at regular stages, but that would be painful and doesn't help if we're working directly in the console itself. What we need to do is create a function that can modify the prompt.

```r
my_prompt <- function() {

  console_msg <- paste0("[",
                        format(Sys.time(), "%H:%M:%S"),
                        "] > ")

  options(prompt = console_msg)

  invisible(TRUE)

}
```

This function gets the current time, formats it just to the hour, minutes and seconds, wraps it in square brackets and then sets that as the prompt. It also retains the less than sign because it's something we're already used to as part of the prompt in the R console.

```
> my_prompt()
[16:51:40] > my_prompt()
[16:51:42] > my_prompt()
[16:51:42] > my_prompt()
[16:51:43] >
[16:51:43] > 1+1
[1] 2
[16:51:43] > LETTERS[round(runif(4, 1, 10),0)]
[1] "B" "B" "H" "E"]
[16:51:43] > my_prompt()
[16:52:05] > my_prompt()
[16:52:06] > my_prompt()
```

So the function nice and easily modifies the prompt but we still need to call it every time we want to update the prompt. What we need to do is get R to call this function itself every time it runs code. We do this using the [`addTaskCallback()`](https://stat.ethz.ch/R-manual/R-devel/library/base/html/taskCallback.html) function, which _"registers an R function that is to be called each time a top-level task is completed"_ - that is, every time R completes a task[^1] it will run this function.

We can't however do this interactively from the console[^2], plus we want this to happen from the start, so we need to edit our `.Rprofile` file. The `.Rprofile` is one of a number of files that R uses to configure your workspace when you load R[^3], TLDR you need a `.Rprofile` file in your home directory, `usethis::edit_r_profile()` will either open your existing `.Rprofile` or create one for you if you don't have one.

In your .`Rprofile` file you'll want to create or amend the `.First()` function, this is a function that R calls when it first launches. We can also store our prompt editing function here. You can see that I also use the `.First()` function to set the `blogdown.author` option so that that's available from the start.

```r
.First <- function() {

  my_prompt <- function() {
  
    console_msg <- paste0("[",
                          format(Sys.time(), "%H:%M:%S"),
                          "] > ")

    options(prompt = console_msg)

    invisible(TRUE)

  }

  my_prompt()
  
  addTaskCallback(my_prompt)

  options(blogdown.author = "Matt")

}
```

Save your `.Rprofile` and then restart R. And right from the get-go you'll have a prompt that tells you the time (well technically the last time R completed anything, if R hasn't done anything for a while it won't update until you execute some code).

### Adding folder information

Do you work in multiple R sessions at a time, or move up and down the file tree. I often have two or three RStudio sessions open at a time with different projects in each, and it can be easy to forget which window is which project. To solve this I first I decided to add the working directory to the prompt function by calling `basename(getwd())`, which gives you the last element of the path of the current working directory[^4]. I also decided to ditch the seconds from the time element of the prompt to keep it short.

```r
my_prompt <- function() {
  
  my_loc <- basename(getwd())

  console_msg <- paste0("[",
                        format(Sys.time(), "%H:%M"),
                        " ", my_loc,
                        "] > ")

  options(prompt = console_msg)

  invisible(TRUE)

}
```

```
[17:26 mattR] >
[17:26 mattR] > setwd("man")
[17:27 man] > setwd("..")
[17:27 mattR] >
```

But what about if you're in a large project with multiple folders and sub-folders of folders, how will you know where you are in relation to the base folder of your project. To track this, we make use of the [`{here}`](https://here.r-lib.org) package to identify the root of the project.

```r
my_prompt <- function() {
  
  proj_path <- here::here()
  my_loc <- getwd()

  if (!is.null(proj_path)) {

    if (grepl(proj_path, my_loc)) {

      my_base <- basename(proj_path)

      my_loc <- paste0(my_base, gsub(proj_path,  "", my_loc),
                       collapse = .Platform$file.sep)

    } else {

      home <- Sys.getenv("HOME")

      my_loc <- paste0("!! ", gsub(home, "~", my_loc))
    }
  }

  console_msg <- paste0("[",
                        format(Sys.time(), "%H:%M"),
                        " ", my_loc, "/",
                        "] > ")

  options(prompt = console_msg)

  invisible(TRUE)

}
```

```
[17:36 mattR/] >
[17:36 mattR/] > setwd("man")
[17:37 mattR/man/] > setwd("..")
[17:37 mattR/] > setwd("..")
[17:37 !! ~/R] > setwd("birthdayplanets")
[17:37 !! ~/R/birthdayplanets/] >
```

First let's use `here::here()` to identify where R has launched and set that as our project path. Then call our current working directory. Assuming we've picked up a project path from `here()` then let's see if `proj_path` is in `my_loc`, if so then paste together the base of the project path with the current working directory (if you're in the base folder than the `gsub()` command will remove all text). If the project path isn't in the path of your current working directory then we return the current location (after shortening anything under the user's home directory with the tilde, `~`, to represent the home directory) and also warn the user by displaying two exclamation marks between the time and the folder reference.

### Adding git info
Most of my projects use git (and GitHub) for version control. But I'm pretty bad at remembering whether I've committed code and/or pushed it. I use [Zsh](https://en.wikipedia.org/wiki/Z_shell) and [OhMyZsh](https://ohmyz.sh) in my Terminal with modifications that easily sign post for me what branch I'm in, when there are changes that need committing, and whether I'm up to date with the remote repo. But I work a lot in R and I realised it would be useful to have some information in my R console. This required a bit of figuring out, largely from StackOverflow posts about how to determine the status of a repo from the console.

```r
my_prompt <- function() {
  
  proj_path <- here::here()
  my_loc <- getwd()

  ...

  git_branch <- suppressWarnings(system("git rev-parse --abbrev-ref HEAD",
                                        ignore.stderr = TRUE, intern = TRUE))

  if (length(git_branch) != 0) {
    git_msg <- paste0(" @", git_branch)
    git_status <- suppressWarnings(system("git status -s",
                                          ignore.stderr = TRUE, intern = TRUE))
    git_ahead <- suppressWarnings(system("git status -sb",
                                         ignore.stderr = TRUE, intern = TRUE))
    git_ahead_chk <- grepl("ahead", git_ahead)

    if (length(git_status) != 0) {
      git_msg <- paste0(git_msg, " ✘")
    } else if (git_ahead_chk) {
      git_msg <- paste0(git_msg, " ⬆︎")
    }

  } else {
    git_msg <- ""
  }

  console_msg <- paste0("[",
                        format(Sys.time(), "%H:%M"),
                        " ", my_loc, "/",
                        git_msg,
                        "] > ")


  options(prompt = console_msg)

  invisible(TRUE)

}
```

```
[18:03 mattR/ @main] > usethis::use_readme_md()
✓ Setting active project to '/Users/matt/R/mattR'
✓ Writing 'README.md'
● Modify 'README.md'
[18:03 mattR/ @main ✘] > system("git add README.md")
[18:04 mattR/ @main ✘] > system("git commit -m \"Add README\"")
[main aac9773] Add README
 1 file changed, 4 insertions(+)
 create mode 100644 README.md
[18:05 mattR/ @main ⬆︎] > setwd("man")
[18:05 mattR/man/ @main ⬆︎] > 1+1
[1] 2
[18:06 mattR/man/ @main ⬆︎] > setwd("..")
[18:06 mattR/ @main ⬆︎] > system("git push origin main")
To https://github.com/mattkerlogue/mattR.git
   018f0af..aac9773  main -> main
[18:06 mattR/ @main] >

```

First, let's find out what git branch we're on, we can use `system()` to issue commands from the R console to the underlying system shell, usually that will execute the command and not return anything bar a message of the output but setting `intern = TRUE` allows us to save the output to an object that R can interrogate. If we run the command in a folder that's not within a git repo we'll get an error/warning message which we don't care about. If we're in a git branch, let's construct a message telling us which branch we're in, then let's then get some additional info: we can get the status of our repo (i.e. have we got modifications that need committing) and whether we're ahead of the origin. If we've got changes we need to commit then let's alert the user with a heavy cross (✘), if not and we're ahead let's use a heavy arrow (⬆︎) to show we need to push. I realised while writing this that I didn't have a README set up for the package I was in, so this provided a great opportunity to demonstrate how the prompt changes as a file is created, committed and pushed.

If you want to have the same console experience as me, then full code for my prompt is [here](https://github.com/mattkerlogue/mattR/blob/main/R/matt_prompt.R).

[^1]: A top-level task can be thought of as a chunk of connected code, that is if you have a a set of nested functions (e.g. `sum(is.na(c(1:5, NA_real_, 3:1)))`) that is a single task. If there is a break/separator in between code that indicates a new task. Most R code is written with a new task starting on a new line, but you can write multiple tasks on the same line using the semi-colon as a separator (e.g. `1+1; 2-2`), this is not often used, but each of those expressions represents a separate task.

[^2]: There might be a way, it just broke every time I tried.

[^3]: It's far too complicated to go into, some useful discussion [here](https://rstats.wtf/r-startup.html), and nicely summed up in this tweet. {{<tweet 961553618196418560>}}

[^4]: Yes, I have a folder called `mattR`, what's the matter with that. As a matter of fact, mattR is actually a [package](https://github.com/mattkerlogue/mattR) of some custom functions, including my custom prompt so that it's easy to deploy on both my personal and work devices.