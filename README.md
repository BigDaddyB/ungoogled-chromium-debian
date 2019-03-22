# ungoogled-chromium-debian

This repository contains files to build Debian packages of [ungoogled-chromium](//github.com/Eloston/ungoogled-chromium).

This branch contains the code to build packages for: **Debian 9 (stretch)**

## Downloads

**Binaries**: [Get them from the Contributor Binaries website](//ungoogled-software.github.io/ungoogled-chromium-binaries/).

**Source Code**: Use the tags labeled with `stretch`. The branches are for development and may not be stable.

## Building

```sh
# Install essential requirements
sudo apt install git python3 packaging-dev

# Setup build tree under build/
mkdir -p build/src
git clone https://github.com/ungoogled-software/ungoogled-chromium-debian
git checkout debian_stretch
cp -r ungoogled-chromium-debian/debian build/src/
cd build/src

# We now need the files from the ungoogled-chromium repo
# There are two options to get it:

# Option 1 (RECOMMENDED): Download it from GitHub automatically
./debian/rules download-ungoogled-chromium
# Option 2 (ADVANCED USERS ONLY): Use an existing git clone of ungoogled-chromium
ln -s path/to/ungoogled-chromium  debian/ungoogled-upstream/ungoogled-chromium
./debian/rules checkout-ungoogled-chromium

# Final setup steps for debian/ directory
./debian/rules setup-debian

# Install remaining requirements to build Chromium
sudo mk-build-deps -i debian/control
rm ungoogled-chromium-build-deps_*.deb

# Download and unpack Chromium sources (this will take some time)
./debian/rules setup-local-src

# Start building
dpkg-buildpackage -b -uc
```

### Restarting the build

If the build aborts during the last command, it can be restarted with this command:

```sh
dpkg-buildpackage -b -uc -nc
```

If it still fails, then try this command (this will clear any intermediate build outputs):

```sh
dpkg-buildpackage -b -uc
```

If all else fails, delete the entire build tree and start again.

## Developer info

This section contains information for those contributing to ungoogled-chromium-debian.

### First-time setup

1. Clone this repo
2. Add the remote for Debian as `upstream`:

```sh
git remote add upstream https://salsa.debian.org/chromium-team/chromium.git
```

### Pull new changes from Debian

These instructions will pull in changes from Debian's `chromium` package into `debian_stretch`:

```sh
git checkout debian_stretch
git pull upstream stretch
# Complete the git merge
git merge debian_buster
# Complete the git merge
# Update patches via instructions below
```

### Pull new changes from ungoogled-chromium

These are to update the base ungoogled-chromium files

```sh
# First, get the *full* commit hash from the ungoogled-chromium repo
# We will assume this is COMMIT_HASH in the instructions
printf '%s' COMMIT_HASH > debian/ungoogled-upstream/version.txt
git add debian/ungoogled-upstream/version.txt
# Update patches via instructions below
```

### Updating patches

```sh
# Assuming we are using an existing git clone of the ungoogled-chromium repo
ln -s path/to/ungoogled-chromium  debian/ungoogled-upstream/ungoogled-chromium
./debian/rules checkout-ungoogled-chromium

./debian/devutils/update_patches.sh merge
source debian/devutils/set_quilt_vars.sh

# Setup Chromium source
mkdir -p build/{src,download_cache}
./debian/ungoogled-upstream/ungoogled-chromium/utils/downloads.py retrieve -i debian/ungoogled-upstream/ungoogled-chromium/downloads.ini -c build/download_cache
./debian/ungoogled-upstream/ungoogled-chromium/utils/downloads.py unpack -i debian/ungoogled-upstream/ungoogled-chromium/downloads.ini -c build/download_cache build/src

cd build/src
# Use quilt to refresh patches. See ungoogled-chromium's docs/developing.md section "Updating patches" for more details
quilt pop -a

# Remove all patches introduced by ungoogled-chromium
./debian/devutils/update_patches.sh unmerge
# Ensure debian/patches/series is formatted correctly, e.g. blank lines

# Sanity checking for consistency in series file
./debian/devutils/check_patch_files.sh

# Use git to add changes and commit
```

### Adding a new branch

To add either a primary or secondary branch:

1. Create a new branch that forks off an existing branch with code that is closest to the desired code.
2. Give the branch a name of the format `DISTRO_CODENAME`. For example, Ubuntu 18.10 (cosmic) should have a branch name `ubuntu_cosmic`.
3. Make the necessary changes and commit
4. Submit a Pull Request for your new branch to the branch it is based off of. In the Pull Request, specify the new branch name that should be created. (This is necessary because GitHub doesn't support the creation of branches via PRs)

### Tagging a new version

1. Update the commit or tag version of the ungoogled-chromium repository in `debian/ungoogled-upstream/version.txt` as necessary.
2. Increment the revision in `debian/distro_revision.txt`. However, if the upstream version was changed in Step 1, reset the revision to `1`.
3. Use git to add a new tag in the format `{chromium_version}-{revision}.stretch{distro_revision}` where
	* `chromium_version` is the Chromium version (from the ungoogled-chromium repo)
	* `revision` is the ungoogled-chromium revision (from the ungoogled-chromium repo)
	* `distro_revision` is the revision from `debian/distro_revision.txt`

### Contributing

Contribution guidelines are the same as ungoogled-chromium.

Submit PRs to this repository for every packaging type that should be updated.
