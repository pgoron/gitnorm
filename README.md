Test repository to evaluate impact of .gitattributes

# Initialisation from a linux host
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
Post renormalisation, working directory have been untouched by normalisation.

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

