# Scrap & Jag(g)

A pair of tools I wrote in an ADHD hyperfocus session, based around trying to make some CLI stuff less of a pain. YMMV

## Building/installation

If you want to run these scripts, grab a [Janet release](https://github.com/janet-lang/janet/releases).

If you want to compile th

## Scrap 

Scrap is a cli-style clibboard. Set it up by defining the `SCRAP_DIR` environment variable. Usage is shown below.

### Scrap usage


```cmd
  scrap: pipe data to/from a junk dir

  subcommands
  scrap put <name>: Saves stdin to name
  scrap get <name>: Cats <name> to stdout
  scrap add <name>: Appends stdin to name
  scrap map <name> <expr>: Passes each line of <name> through expr
  scrap ls <name?>: Lists the files in <name> scrap workspace, the current one if not provided
  scrap rm <name>: Deletes the <name> file in the current workspace
  scrap update <name> <expr>: Passes each line of <name> through the Janet expr <expr>, nil lines are removed
  scrap current: Gives the current scrap workspace
  scrap ws save <name?>: Saves the current set of scrap files to a backup folder. If <name>
  scrap ws load <name>: Clears the current scrap workspace, and then loads the workspace from <name>
  scrap ws ls: Lists the workspaces
```


## Jag(g): Janet Aggregate

This is a bit of an awk-alike, but using Janet

It works in 3 phases

#### Beginning

You can pass `-s <delim>` if the contents of stdin will be using a delimiter other than newline

#### Middle

Using `-m <Janet Expr>` adds `<Janet Expr>` to the row mappers  
Using `-f <Janet Expr>` adds `<Janet Expr>` as predicate to the row mappers
Using `-%` specifies that each row will be returning an array that should be flattened.

#### End

The mapped rows are based on the rows deterimined from `-s` and modifed and filtered by `-m`/`-f`.

Using `-r <Janet Expr>` takes all the mapped rows, runs a function on the resuling array (assigned to `r` in the expr), and then appends the result to the output array
Using `-r* <Janet Expr>` takes all of the mapped rows, and use `<Janet Expr>` as a reducer with the accumuulator assigned to `acc` and the curernt element assigned to `el`. It then appends the result to the output array
Using `-M <Janet Expr>` runs `<Janet Expr>` as a mapping function over the current output array, usually you'll want to use it *after* any `-r`/`-r*` calls
Using `-p <Prefix String>` prepends `<Prefix String>` to the pre-output array, which is printed before the output array.
Using `-j <string>` sets the output row joiner to `<string>`.

### Special functions

The following functions in Jag have been set up to use the current row implicitly

- `split: (split delimeter &opt val)` Splits the row as a string using delimeter. If `val` is passed, it is used instead of the current row.
- `has: (has patt &opt val)` Predicate to test if the current row contains `pat`. If `val` is passed, it is used instead of the current row.
- `regex: (patt &op val)` Performs a global spork/regex match on the current row. If `val` is passed, it is used instead of the current row.
- `columns: (& cols)` Wrapper around zipcoll, it treats `cols` as a list of keys, and the current row as a list of values, and turns the key/value pairs into a struct

### Jag examples

#### Summing numbers
```cmd
echo 1,2,3 | jag -s , -m "(scan-number (string/trim r))" -r* "(+ acc el)" 
# => 6
```


