git-store-meta
===============================================================================

git-store-meta is a light-weight tool for file metadata storing and applying
for git.

Features:
-------------------------------------------------------------------------------

* Light dependency, cross-platform consistent behavior, desirable performance.

* Can store the metadata of git-revisioned files into a data file.

* Can apply the metadata stored in the data file to the working copy.

* Can update the metadata for changed files quickly.

* Can easily pick which metadata fields to store, update, or apply.

* Can determine whether to store, update, or apply directory metadata.

Dependency:
-------------------------------------------------------------------------------

- git (any version)
- sort (any version)
- *nix platform with basic shell environment (required by Git)
- perl >=5.8 with built-in modules (required by Git)
- setfacl, getfacl (optional, for manipulating ACL metadata)
- touch, chown, chgrp (optional, for applying metadata for links)

Usage:
-------------------------------------------------------------------------------

Copy the `git-store-meta.pl` file to `path/to/your/repo/.git/hooks/`, change
the working directory to the top level of git working tree, and run the
commands.

### Store

To store the metadata of all git-revisioned files, run:

    .git/hooks/git-store-meta.pl --store -f user,group,mode,mtime,atime

And a data file named `.git_store_meta` will be created in your repo,
`git add` it so that the metadata of your files are recorded.

The `-f` (`--field`) option determines which fields are to be stored. Their
order matters partially since it affects the order of fields in the data
file, but it doesn't affect applying. For example the above command creates
a data file with these fields:

    <file> <type> <user> <group> <mode> <mtime> <atime>

If `-f` is not provided, git-store-meta will look in the current
`.git_store_meta` and take its fields definition. And therefore running
`.git/hooks/git-store-meta.pl --store` works in most usual cases.

`-d` (`--directory`) option can be provided so that all directories under git
revision control have their metadata stored, too.

    .git/hooks/git-store-meta.pl --store -d

### Apply

To apply (restore) the metadata recorded in the data file, run:

    .git/hooks/git-store-meta.pl --apply

And all recorded metadata will be applied to the working copy.

Fields can be selectively applied. For example, to apply mtime only:

    .git/hooks/git-store-meta.pl --apply -f mtime

For a similar reason it'd be preferable to add `-d` option so that directory
metadata are restored if they have been stored.

    .git/hooks/git-store-meta.pl --apply -d

Furthermore, `--verbose` (`-v`) option can be used to info what exactly are
being applied.

### Update

After the first `--store` and commit, `--update` can be run to re-scan 
metadata for changed files, which is much faster than to re-scan all revisioned
files. `--directory` (`-d`) can be added so that each changed file makes its
ancestor folders re-scanned.

Note that files or directories whose metadata have been changed without any
content (or modes git cares) change will not be awared in this way, and a
`--store` will be required in these cases. Though, conversely, this prevents
unintentional metadata changes from being added, and could be preferrable in
some cases.

### Help

For a more available options or a more detail description, run:

    .git/hooks/git-store-meta.pl --help

Automation:
-------------------------------------------------------------------------------

Copy or add the `pre-commit` and/or `post-checkout` hook files to
`path/to/your/repo/.git/hooks`, make sure they have "x" permission, and now
file metadata will be stored during a commit and be restored during a checkout
automatically.

Sample code for the pre-commit hook:

    #!/bin/sh
    # when running the hook, cwd is the top level of working tree

    # update (or store if failed)
    perl $(dirname "$0")/git-store-meta.pl --update ||
    perl $(dirname "$0")/git-store-meta.pl --store ||
    exit 1

    # remember to add the updated cache file
    git add .git_store_meta

Sample code for the post-checkout hook:

    #!/bin/sh
    # when running the hook, cwd is the top level of working tree

    sha_old=$1
    sha_new=$2
    change_br=$3

    # apply metadata only when the HEAD is changed
    if [ ${sha_new} != ${sha_old} ]; then
        perl $(dirname "$0")/git-store-meta.pl --apply -d
    fi

Of course you can adjust the options to fit your requirements.

Note that since git doesn't provide a "post-reset" hook, git-store-meta doesn't
run after a successful `git reset --hard`. To restore file metadata after that,
an `--apply` must be run manually.

Caveats:
-------------------------------------------------------------------------------

* git-store-meta cannot restore the metadata for symbolic links without the
  support of system command `chown -h`, `chgrp -h`, and `touch -h`. In those
  systems metadata of symbolic links cannot be restored.
  
* Also, git-store-meta does not record the time more than seconds precision, as
  it's not supported widely enough, and many systems simply ignore it.
