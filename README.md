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
the working directory to the top level of the git working tree, and run one of
the commands below.

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

### Install

To automatically store metadata before a commit and restore metadata after a
checkout, run:

    .git/hooks/git-store-meta.pl --install

This will generate a `pre-commit` and a `post-checkout` hook for the current git
repo. Of course you can modify the hooks afterwards to fit your needs better.

Note that the installation is skipped if there's already an existed `pre-commit`
or `post-checkout` hook to avoid a dangerous overwrite. In this case you'd have
to rename your existed hook files, run the installation, and merge the hook
contents manually.

Note that since git doesn't provide a "post-reset" hook, git-store-meta doesn't
run after a successful `git reset --hard`. To restore file metadata after a
reset, an `--apply` must be run manually.

### Help

For available options and a detail description, run:

    .git/hooks/git-store-meta.pl --help

Caveats:
-------------------------------------------------------------------------------

* git-store-meta cannot restore the metadata for symbolic links without the
  support of system command `chown -h`, `chgrp -h`, and `touch -h`. In those
  systems metadata of symbolic links are never restored.
  
* git-store-meta does not record the time more than seconds precision, as it's
  not supported widely enough and many systems simply ignore it.
