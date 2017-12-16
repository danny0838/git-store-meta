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
- sort (any version)
- *nix platform with basic shell environment (required by Git)
- perl >=5.8 with built-in modules (required by Git)
- setfacl, getfacl (optional, for manipulating ACL metadata)
- touch, chown, chgrp (optional, for applying metadata to links)

Usage:
-------------------------------------------------------------------------------

Copy the `git-store-meta.pl` file to `path/to/your/repo/.git/hooks/`, change
the working directory to the Git working tree, and run one of the commands
below.

### Store

To store the metadata of all git-revisioned files, run:

    .git/hooks/git-store-meta.pl --store -f user,group,mode,mtime,atime

And a data file named `.git_store_meta` (by default) will be created in your
repo, `git add` it so that the metadata is revisioned.

The `--field` (`-f`) option determines which fields are to be stored. Their
order matters partially since it affects the order of fields in the data
file, but it doesn't affect applying. For example the above command creates
a data file with these fields:

    <file> <type> <user> <group> <mode> <mtime> <atime>

If `--field` is not provided, git-store-meta takes the fields defined in the
current data file, and thus running `.git/hooks/git-store-meta.pl --store`
works in most usual cases.

The `--directory` (`-d`) option can be provided so that all directories under
Git revision control have their metadata stored, too.

    .git/hooks/git-store-meta.pl --store -d

### Apply

To apply (restore) the metadata recorded in the data file, run:

    .git/hooks/git-store-meta.pl --apply

And all recorded metadata will be applied to the working tree files.

Fields can be selectively applied. For example, to apply mtime only:

    .git/hooks/git-store-meta.pl --apply -f mtime

For a similar reason it'd be preferable to add `--directory` option so that
directory metadata are restored if they have been stored.

    .git/hooks/git-store-meta.pl --apply -d

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

After a `--store` and commit, `--update` can be run to re-scan metadata only
for changed files, which is much faster than to re-scan all revisioned files.
`--directory` (`-d`) can be added so that each changed file makes its ancestor
folders re-scanned.

Note that files or directories whose metadata have been changed without any
content (or modes Git cares) change will not be awared, and a `--store` is
required if you want to record these metadata changes. Though, conversely,
this avoids unintentional metadata changes from being added, and could be
preferrable in some cases.

### Install

To automatically store metadata before a commit and restore metadata after a
checkout, run:

    .git/hooks/git-store-meta.pl --install

This will generate a `pre-commit` and a `post-checkout` hook for the current Git
repo. Of course you can modify the hooks afterwards to fit your needs better.

Note that the installation is skipped if there's already an existed `pre-commit`
or `post-checkout` hook to avoid a dangerous overwrite. In this case you'd have
to rename your existed hook files, run the installation, and merge the hook
contents manually.

Note that since Git doesn't provide a "post-reset" hook, git-store-meta doesn't
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
