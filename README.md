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

- git >= 1.7.2
- *nix platform with basic shell environment (required by Git)
- perl >=5.8 with built-in modules (required by Git)
- sort (any version) (required by Git)
- Linux::ACL module, or getfacl/setfacl (optional, for manipulating ACL metadata)
- File::lchown module, or touch/chown (optional, for applying metadata to symbolic links)

Usage:
-------------------------------------------------------------------------------

Copy the `git-store-meta.pl` file to `/path/to/your/repo/.git/hooks/`, add
executable permission to it, change the working directory to the Git working
tree, and run `.git/hooks/git-store-meta.pl`.

For a centralized installation, place `git-store-meta.pl` at a desired path,
add executable permission to it, add its directory to the `PATH` environment
variable, and run `git-store-meta.pl` instead.

`git-store-meta.pl` can be run with the actions below:

### Store

To store the metadata of all git-revisioned files, run:

    git-store-meta.pl --store

And a data file named `.git_store_meta` (by default) will be created in your
repo, `git add` it so that the metadata is revisioned.

The `--fields` (`-f`) option determines the fields to be stored. For example,
the command:

    git-store-meta.pl --store -f user,group,mode,mtime,atime

creates a data file with these fields:

    <file> <type> <user> <group> <mode> <mtime> <atime>

The `--directory` (`-d`) option can be provided so that all directories under
Git revision control have their metadata stored, too.

    git-store-meta.pl --store -d

Oppositely, `--no-directory` can be provided to reverse the behavior.

Such settings as `--fields` and `--directory` will be recorded in the data
file, and a future run of `--store`, `--update`, or `--apply` will load the
recorded settings if not explicitly specified.

### Apply

To apply (restore) all metadata recorded in the data file to working tree
files, run:

    git-store-meta.pl --apply

Fields can be selectively applied. For example, to apply mtime only:

    git-store-meta.pl --apply -f mtime

Similarly, to not apply metadata for directories:

    git-store-meta.pl --apply --no-directory

Furthermore, the `--verbose` (`-v`) option can be used to inform what exactly
are being applied.

An `--apply` can not be run when the working tree or index is dirty, since the
data is in an inconsistent state in such case and the apply could be an
irreversible mistake. The `--force` option can be added to skip the check and
force the applying.

### Update

After a `--store` and commit, `--update` can be run to update metadata only
for changed files and directories, which is much faster than to re-scan all
revisioned ones:

    git-store-meta.pl --update

Note that files or directories not considered changed by Git are not updated,
even if their metadata have been changed, and a `--store` is required to record
their metadata changes in such cases. Though, conversely, this avoids
unintentional metadata changes from being included, and could be preferable in
many cases.

### Install

To automatically store metadata before a commit and restore metadata after a
checkout or merge for files, run:

    git-store-meta.pl --install

to generate `pre-commit`, `post-checkout`, and `post-merge` hooks for the
current Git repo. Of course you can modify the hooks afterwards to fit your
needs better.

Note that the installation is skipped to avoid a dangerous overwrite if there
are existing hooks. In this case you can rename the existing hook files, run
the installation again, and merge the hook contents manually. The `--force`
option can be added to overwrite existing hooks if desired.

Note that certain advanced git operations are not covered by hooks. For
example, no hook is run after a successful `git reset --hard`, `git rebase`,
or `git filter-branch`. Likewise, no hook is run in the submodule repositories
after a successful `git submodule update`. An `--apply` must be run manually
in such cases.

### Help

For more available options and details, run:

    git-store-meta.pl --help

Caveats:
-------------------------------------------------------------------------------

* git-store-meta does not record the time more than seconds precision, as it's
  not supported widely enough and many systems simply ignore it.

* git-store-meta is bound to a git repository. If the git directory or the
  working tree directory cannot be automatically determined, provide them
  explicitly to get it work. For example, to apply metadata for files checked
  out from a bare repo `/path/to/foo` to current directory `/path/to/bar`:

      GIT_DIR=../foo GIT_WORK_TREE=. ../foo/hooks/git-store-meta.pl --apply
