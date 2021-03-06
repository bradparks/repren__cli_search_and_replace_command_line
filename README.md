repren
======

## Multi-pattern string replacement and file renaming

**A command-line search-and-replace swiss army knife**

It's the usual deal. Naming things is one of the hard problems in computer science.
Frobinators should be called Glurps, and WhizzleSticks are being replaced
by AcmeExtrudedPlasticFunProviders. But now there are 2000 Java files, some
JavaScript, a little C++ and YAML, and of course documentation, that all have
to be updated at once. And there are a few different naming conventions in
each format. And the files and directories all have old names that should be
changed, too.

I really don't know why there isn't a single well-known tool for this already.
The standard tools like *sed*, *perl*, *awk*, *Vim* macros, or even IDE refactoring
tools, each only solve part of the problem.  Inevitably you end up digging through
the dark corners of the sed man page and writing bash scripts that would scare your mother.

Repren is a simple upgrade that tries to cover a lot of use cases, and to do it simply.


## Installation

No dependencies except Python 2.7+. It's easiest to install with pip:

    sudo pip install repren

Or, since it's just one file, you can copy the
[repren](https://raw.githubusercontent.com/jlevy/repren/master/repren)
script somewhere (perhaps within your own project) and make it executable.


## Try it

Let's try a simple replacement in our working directory (which has a few source
files):

    bash-3.2$ repren --from frobinator-server --to glurp-server --full --dry-run .
    Dry run: No files will be changed
    Using 1 patterns:
      'frobinator-server' -> 'glurp-server'
    Found 102 files in: .
    - modify: ./site.yml: 1 matches
    - rename: ./roles/frobinator-server/defaults/main.yml -> ./roles/glurp-server/defaults/main.yml
    - rename: ./roles/frobinator-server/files/deploy-frobinator-server.sh -> ./roles/glurp-server/files/deploy-frobinator-server.sh
    - rename: ./roles/frobinator-server/files/install-graphviz.sh -> ./roles/glurp-server/files/install-graphviz.sh
    - rename: ./roles/frobinator-server/files/frobinator-purge-old-deployments -> ./roles/glurp-server/files/frobinator-purge-old-deployments
    - rename: ./roles/frobinator-server/handlers/main.yml -> ./roles/glurp-server/handlers/main.yml
    - rename: ./roles/frobinator-server/tasks/main.yml -> ./roles/glurp-server/tasks/main.yml
    - rename: ./roles/frobinator-server/templates/frobinator-webservice.conf.j2 -> ./roles/glurp-server/templates/frobinator-webservice.conf.j2
    - rename: ./roles/frobinator-server/templates/frobinator-webui.conf.j2 -> ./roles/glurp-server/templates/frobinator-webui.conf.j2
    Read 102 files (190382 chars), found 2 matches (0 skipped due to overlaps)
    Dry run: Would have changed 2 files, including 0 renames

Looks good. Now actually do it.

    bash-3.2$ repren --from frobinator-server --to glurp-server --full .
    Using 1 patterns:
      'frobinator-server' -> 'glurp-server'
    Found 102 files in: .
    - modify: ./site.yml: 1 matches
    - rename: ./roles/frobinator-server/defaults/main.yml -> ./roles/glurp-server/defaults/main.yml
    - rename: ./roles/frobinator-server/files/deploy-frobinator-server.sh -> ./roles/glurp-server/files/deploy-frobinator-server.sh
    - rename: ./roles/frobinator-server/files/install-graphviz.sh -> ./roles/glurp-server/files/install-graphviz.sh
    - rename: ./roles/frobinator-server/files/frobinator-purge-old-deployments -> ./roles/glurp-server/files/frobinator-purge-old-deployments
    - rename: ./roles/frobinator-server/handlers/main.yml -> ./roles/glurp-server/handlers/main.yml
    - rename: ./roles/frobinator-server/tasks/main.yml -> ./roles/glurp-server/tasks/main.yml
    - rename: ./roles/frobinator-server/templates/frobinator-webservice.conf.j2 -> ./roles/glurp-server/templates/frobinator-webservice.conf.j2
    - rename: ./roles/frobinator-server/templates/frobinator-webui.conf.j2 -> ./roles/glurp-server/templates/frobinator-webui.conf.j2
    Read 102 files (190382 chars), found 2 matches (0 skipped due to overlaps)
    Changed 2 files, including 0 renames

All done. If this is in git, do a git diff to verify, test, then commit it all.


## What else?

    Usage: repren -p <pattern-file> [options] [path ...]

    repren: Multi-pattern string replacement and file renaming

    Options:
      --version             show program's version number and exit
      -h, --help            show this help message and exit
      --from=FROM_PAT       single replacement: match string
      --to=TO_PAT           single replacement: replacement string
      -p PAT_FILE, --patterns=PAT_FILE
                            file with multiple replacement patterns (see below)
      --full                do file renames and search/replace on file contents
      --renames             do file renames only; do not modify file contents
      --literal             use exact string matching, rather than regular
                            expresion matching
      -i, --insensitive     match case-insensitively
      --preserve-case       do case-preserving magic to transform all case
                            variants (see below)
      -b, --word-breaks     require word breaks (regex \b) around all matches
      --exclude=EXCLUDE_PAT
                            file/directory name regex to exclude
      --at-once             transform each file's contents at once, instead of
                            line by line
      -t, --parse-only      parse and show patterns only
      -n, --dry-run         dry run: just log matches without changing files


Repren is a simple but flexible command-line tool for rewriting file contents
according to a set of regular expression patterns, and to rename or move files
according to patterns. Essentially, it is a general-purpose, brute-force text
file refactoring tool. For example, repren could rename all occurrences of
certain class and variable names in a set of Java source files, while
simultaneously renaming the Java files according to the same pattern. It's more
powerful than usual options like `perl -pie`, `rpl`, or `sed`:

- It can also rename files, including moving files and creating directories.
- It supports fully expressive regular expression substitutions.
- It performs group renamings, i.e. rename "foo" as "bar", and "bar" as "foo"
  at once, without requiring a temporary intermediate rename.
- It is careful. It has a nondestructive mode, and prints clear stats on its
  changes. It leaves backups. File operations are done atomically, so
  interruptions never leave a previously existing file truncated or partly
  edited.
- It supports "magic" case-preserving renames that let you find and rename
  identifiers with case variants (lowerCamel, UpperCamel, lower_underscore, and
  UPPER_UNDERSCORE) consistently.
- It has this nice documentaion!

If file paths are provided, repren replaces those files in place, leaving a
backup with extension ".orig". If directory paths are provided, it applies
replacements recursively to all files in the supplied paths that are not in the
exclude pattern. If no arguments are supplied, it reads from stdin and writes
to stdout.

Patterns:

Patterns can be supplied in a text file, with one or more replacements consisting
of regular expression and replacement. For example:

    # Sample pattern file
    frobinator<tab>glurp
    WhizzleStick<tab>AcmeExtrudedPlasticFunProvider
    figure ([0-9+])<tab>Figure \1

(Where `<tab>` is an actual tab character.) Each line is a replacement.
Empty lines and #-prefixed comments are ignored.

As a short-cut, a single replacemet can be specified on the command line using
`--from` (match) and `--to` (replacement).

Examples:

    # Here `patfile` is a patterns file.
    # Rewrite stdin:
    repren -p patfile < input > output

    # Shortcut with a single pattern replacement (replace foo with bar):
    repren --from foo --to bar < input > output

    # Rewrite a few files in place, also requiring matches be on word breaks:
    repren -p patfile --word-breaks myfile1 myfile2 myfile3

    # Rewrite whole directory trees. Since this is a big operation, we use
    # `-n` to do a dry run that only prints what would be done:
    repren -n -p patfile --word-breaks --full mydir1

    # Now actually do it:
    repren -p patfile --word-breaks --full mydir1

    # Same as above, for all case variants:
    repren -p patfile --word-breaks --preserve-case --full mydir1

Notes:

- As with sed, replacements are made line by line by default. Memory
  permitting, replacements may be done on entire files using `--at-once`.
- As with sed, replacement text may include backreferences to groups within the
  regular expression, using the usual syntax: \1, \2, etc.
- In the pattern file, both the regular expression and the replacement may
  contain the usual escapes `\n`, `\t`, etc. (To match a multi-line pattern,
  containing `\n`, you must must use `--at-once`.)
- Replacements are all matched on each input file, then all replaced, so it's
  possible to swap or otherwise change names in ways that would require
  multiple steps if done one replacement at at a time.
- If two patterns have matches that overlap, only one replacement is applied,
  with preference to the pattern appearing first in the patterns file.
- If one pattern is a subset of another, consider if `--word-breaks` will help.
- If patterns have special charaters, `--literal` may help.
- The case-preserving option works by adding all case variants to the pattern
  replacements, e.g. if the pattern file has foo_bar -> xxx_yyy, the
  replacements fooBar -> xxxYyy, FooBar -> XxxYyy, FOO_BAR -> XXX_YYY are also
  made. Assumes each pattern has one casing convention. (Plain ASCII names only.)
- The same logic applies to filenames, with patterns applied to the full file
  path with slashes replaced and then and parent directories created as needed,
  e.g. `my/path/to/filename` can be rewritten to `my/other/path/to/otherfile`.
  (Use caution and test with `-n`, especially when using absolute path
  arguments!)
- Files are never clobbered by renames. If a target already exists, or multiple
  files are renamed to the same target, numeric suffixes will be added to make
  the files distinct (".1", ".2", etc.).
- Files are created at a temporary location, then renamed, so original files are
  left intact in case of unexpected errors. File permissions are preserved.
- Backups are created of all modified files, with the suffix ".orig".
- By default, recursive searching omits paths starting with ".". This may be
  adjusted with `--exclude`. Files ending in `.orig` are always ignored.
- Data can be in any encoding, as it is treated as binary, and not interpreted
  in a specific encoding like UTF-8. This is less error prone in real-life
  situations where files have encoding inconsistencies. However, note the
  `--case-preserving` logic only handles casing conversions correctly for plain
  ASCII letters `[a-zA-Z]`.


## License

Apache 2


