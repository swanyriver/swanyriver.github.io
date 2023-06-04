# How do I remember all the params/flags for the various tools?
and the answer is mostly I don't,  I use the following strategies

* learn the flags for tools that I do use over and over again like xargs, grep/rg
* learn the flags that are common to multiple commands (like -i for interactive)
* learn small reusable functionality and compose it in pipelines
  * I often use `<cmd> | rev | cut -f 1 -d " " | rev` instead of piping into awk (which has a last-field matcher) because the awk syntax is harder to remember
  * don't buy into the myth about ["Useless Use of Cat"](https://blog.sanctum.geek.nz/useless-use-of-cat/),  in fact, prefer cat into pipeline over parameter,  because piping into commands is a much more repeatable and adaptable pattern then learning specific argument placement or flags, and cat-ed files cant be modified by receiving program
* Most-Importantly,  make aliases/functions:
  * once you find a tool+param combination you are using over and over again make a short name for it.
  * Prefer `function foo {}` over `alias foo=` because functions are parametrizable, composable (you cant invoke an alias from within another alias or function)
 
### example of a bunch of commands/flags turned into shorter easier to remember functions for working with git:

```
function gitty {
  git add -A :/ && git commit -m "${1}"
}

function gatty {
  git add -A :/ && git commit --amend
}

function gnatty {
  git add -A :/ && git commit --amend --no-edit
}

# gain() defined elswhere, finds branch point from main branch
function gitnames {
  /usr/bin/env git diff --name-only $(gain)
}

# find ancestor commit from main branch
function gain {
  main_diverged_commits=$(git cherry main)
  if [ -z "$main_diverged_commits" ]; then
    /usr/bin/env git rev-parse HEAD
  else
    /usr/bin/env git rev-parse $(git cherry main | head -1 | cut -f 2 -d " ")^
  fi
}
```

### example of *extending* a cli by function wrapping
```
function git {
  if [[ $1 == "names" ]]; then
    gitnames
    return
  fi

  if [[ $1 == "duff" ]]; then
    /usr/bin/env git diff $(gain)
    return
  fi

  if [[ $1 == "ibase" ]]; then
    /usr/bin/env git rebase -i $(gain)
    return
  fi

  /usr/bin/env git "$@"
}

```

## When is it a good idea to make a function/alias for a command+params
Probably the fourth or tenth time you find yourself reading the man-page to find
the same param combination

## When is it a good idea to make a function/pipeline that takes-arguments, branches, combines multiple tools, etc?
Probably never, it often takes more time than it saves, but it can be quite fun.
If faced with some large repetitive task like transform all of X to Y and then
process them all with program Z,  and it seems it might take about as much time
to create a helper function as it would to do it manually;   then do it because
if it works well enough and the task ever comes up again you will already have
the pipeline ready.

With one caveat: make only just good enough, and don't generalize.  Make
something that works for just that purpose and next time just manually edit if
the needs change.  If you find yourself editing the same script 5 times then
that's a sign that its time to generalize it.
