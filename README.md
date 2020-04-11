Test repository to evaluate impact of .gitattributes

# Initialisation
```
git init
git config --local core.autocrlf false // to make sure git commit as-is on windows
git add lorem.crlf
git add lorem.lf
git add README.md
git commit -m "initial commit"
```

# Verifying git setup
```
$ git rm -r --cached .
rm 'README.md'
rm 'lorem.crlf'
rm 'lorem.lf'
$ git reset --hard
HEAD is now at 13d463a initial commit
$ file lorem.crlf 
lorem.crlf: ASCII text, with CRLF line terminators
$ file lorem.lf
lorem.lf: ASCII text
$ git branch before-normalisation
```

# Enabling end-of-lines normalisation
Text files will be stored with LF eol in repository index.
These files will be automatically converted to platform eol during `checkout/reset` operations.
New text files will be automatically converted to LF eol during `git add` irrespective to eol convention used in working tree.

```
$ echo "* text=auto" > .gitattributes
$ git add --renormalize .
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   lorem.crlf

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	.gitattributes

$ git add .gitattributes
$ file lorem.crlf 
lorem.crlf: ASCII text, with CRLF line terminators // before commit, crlf file still contains CRLF eol
$ git commit -m "enable normalization of end of lines for text files"
```

# After renormalisation
Post renormalisation, working tree has been untouched by normalisation.

Example on linux:
```
$ file lorem.crlf 
lorem.crlf: ASCII text, with CRLF line terminators
$ file lorem.lf
lorem.lf: ASCII text
```

We have to force git to regenerate files in working tree to have proper line endings:
```
$ git rm -r --cached .
rm '.gitattributes'
rm 'README.md'
rm 'lorem.crlf'
rm 'lorem.lf'
$ git reset --hard
HEAD is now at 3ab96d4 enable normalization of end of lines for text files
$ file lorem.crlf 
lorem.crlf: ASCII text
$ file lorem.lf
lorem.lf: ASCII text
```

Below table gives the expected EOL per platform

| Platform | Expected EOL on text files |
 ----------|---------------------------
| Linux    | LF                         |
| Mac      | LF                         |
| Windows + gitforwindows | CRLF        |
| Windows + git from cygwin | LF        |


# Impact of renormalisation for other contributors of the repository

1st case: contributors don't have pending commits not yet merged in central repository

* Linux, Mac, Windows/Cygwin users:
When they will receive the commit containing the normalisation of end of lines, git will automatically rewrite impacted files in their working tree. (true for platform where line endings is LF as only impacted files by normalisation are CRLF ones and they will be part of the commit).

* Windows / gitforwindows:
    * Text files that were initially in CRLF in repository will stay in CRLF in working tree even if they have been renormalised in repository index.
    * Text file that were initially in LF in repository should switch to CRLF in working tree but git won't do it by itself (technically they haven't been touched by the normalization commit).

    Users will have to clean their local working directory with the following commands:
    ```
    $ git rm -r --cached .
    rm 'README.md'
    rm 'lorem.crlf'
    rm 'lorem.lf'
    $ git reset --hard
    HEAD is now at 13d463a initial commit
    $ file lorem.crlf 
    lorem.crlf: ASCII text, with CRLF line terminators
    $ file lorem.lf
    lorem.lf: ASCII text, with CRLF line terminators
    ```

    // TODO see what happen if users don't do it.



2nd case:contributors have pending commits not yet merged in central repository

* Linux, Mac, Windows/Cygwin users:
During rebase / merge, users will face conflicts due to difference of line endings on files that where initially with CRLF eol. These conflicts can be
ignored automatically by passing `-s recursive -X renormalize` to `git rebase` or `git merge` commands. 

* Windows / gitforwindows users: TODO
