git-store-meta
===============================================================================

git-store-meta is a light-weight tool for file metadata storing and applying
for Git.

Features:
-------------------------------------------------------------------------------

* Light dependency, cross-platform consistent behavior, desirable performance.

* Data files are in plain text format and can be easily revisioned, diffed, or
  manually modified as needed.

* Supported metadata: mtime, atime, mode, user, uid, group, gid, acl.

* Can store the metadata of git-revisioned files into a data file.

* Can apply the metadata stored in the data file to the working copy.

* Can update the metadata for changed files quickly.

* Can easily pick which metadata fields to store, update, or apply.

* Can determine whether to store, update, or apply directory metadata.

Dependency:
-------------------------------------------------------------------------------

- git (any version)
- *nix platform with basic shell environment (required by Git)
- perl >=5.8 with built-in modules (required by Git)
- sort (any version) (required by Git)
- setfacl, getfacl (optional, for manipulating ACL metadata)
- touch, chown, chgrp (optional, for applying metadata to links)

Usage:
-------------------------------------------------------------------------------

Copy the `git-store-meta.pl` file to `/path/to/your/repo/.git/hooks/`, add
executable permission to it, change the working directory to the Git working
tree, and run `.git/hooks/git-store-meta.pl`.

For a centralized installation, place `git-store-meta.pl` at a desired path,
add executable permission to it, add its directory to the `PATH` environment
variable, and run `git-store-meta.pl` instead.

`git-store-meta.pl` can be run with the subcommands below:

### Store

To store the metadata of all git-revisioned files, run:

    git-store-meta.pl --store -f user,group,mode,mtime,atime

And a data file named `.git_store_meta` (by default) will be created in your
repo, `git add` it so that the metadata is revisioned.

The `--field` (`-f`) option determines which fields are to be stored. Their
order matters partially since it affects the order of fields in the data
file, but it doesn't affect applying. For example the above command creates
a data file with these fields:

    <file> <type> <user> <group> <mode> <mtime> <atime>

If `--field` is not provided, git-store-meta takes the fields defined in the
current data file, and thus running `git-store-meta.pl --store` works in most
usual cases.

The `--directory` (`-d`) option can be provided so that all directories under
Git revision control have their metadata stored, too.

    git-store-meta.pl --store -d

### Apply

To apply (restore) the metadata recorded in the data file, run:

    git-store-meta.pl --apply

And all recorded metadata will be applied to the working tree files.

Fields can be selectively applied. For example, to apply mtime only:

    git-store-meta.pl --apply -f mtime

For a similar reason it'd be preferable to add `--directory` (`-d`) option so
that metadata of directories are restored if they have been stored:

    git-store-meta.pl --apply -d

Furthermore, the `--verbose` (`-v`) option can be used to info what exactly are
being applied.

An `--apply` can not be run when the working tree or index is dirty, since the
data is in an inconsistent state in such case and the apply could be an
irreversible mistake. The `--force` option can be added to skip the check and
force the applying.

For a special use case such as to apply metadata for files checked out from a
bare repo `/path/to/foo` to an isolated directory `/path/to/bar`, related Git
environment variables must be explicitly provided to pass the check, such as:

    GIT_DIR=../foo GIT_WORK_TREE=. ../foo/hooks/git-store-meta.pl --apply

### Update

After a `--store` and commit, `--update` can be run to update metadata only
for changed files, which is much faster than to re-scan all revisioned files:

    git-store-meta.pl --update

`--directory` (`-d`) can be added so that changed directories (those with
added, deleted, or renamed files) are also updated:

    git-store-meta.pl --update -d

Note that files or directories not considered changed by Git are not updated,
even if their metadata have been changed, and a `--store` is required to record
their metadata changes in such cases. Though, conversely, this avoids
unintentional metadata changes from being included, and could be preferrable in
many cases.

### Install

To automatically store metadata before a commit and restore metadata after a
checkout or merge for files, run:

    git-store-meta.pl --install

to generate `pre-commit`, `post-checkout`, and `post-merge` hooks for the
current Git repo. Of course you can modify the hooks afterwards to fit your
needs better.

The `--directory` (`-d`) option can be provided so that metadata of directories
are also handled:

    git-store-meta.pl --install -d

Note that the installation is skipped to avoid a dangerous overwrite if there
are existing hooks. In this case you can rename the existing hook files, run
the installation again, and merge the hook contents manually. The `--force`
option can be added to overwrite existing hooks if desired.

Note that since Git doesn't provide a "post-reset" hook, git-store-meta doesn't
run after a successful `git reset --hard`. To restore file metadata after a
reset, an `--apply` must be run manually.

### Help

For available options and a detail description, run:

    git-store-meta.pl --help

Caveats:
-------------------------------------------------------------------------------

* git-store-meta cannot restore the metadata for symbolic links without the
  support of system command `chown -h`, `chgrp -h`, and `touch -h`. In those
  systems metadata of symbolic links are never restored.

* git-store-meta does not record the time more than seconds precision, as it's
  not supported widely enough and many systems simply ignore it.
