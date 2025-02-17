# Dockerfiles

**DEPRECATED**. This repository is no longer actively maintained. It has been replaced with the following:

- [Databio/dockerfiles](https://github.com/databio/dockerfiles) is the new source for lab dockerfiles
- [Bulker](bulker.databio.org/) is a polished, updated version of my 'containerized executables' that is now much more approachable than this initial version.


I'll leave this repository here just for reference.

---

This repository contains a complete solution to dockerizing common commands. It includes both  Dockerfiles for building various docker containers, as well as a `bin` directory with executables that are run in their respective container.

## Getting started: An example

Just clone this repository:

```
git clone https://github.com/nsheff/docker.git
```

Then, build any containers you want. For example, to build the pandoc container:

```
cd docker
make pandoc
```

Now, you can run this containerized `pandoc` command like this:

```
./bin/pandoc
```

## Building images

In this repo is a [Makefile](Makefile) with recipes for building each image.  Type `make IMAGE`, where `IMAGE` is one of the values in the `Dockerfile_IMAGE`. You can also tab-complete to see which images can be built. You can also re-build any container from scratch (don't use caches) by adding '-nocache' to the target name, *e.g.* `make pandoc-nocache`.

## Containerized executables 

In the [/bin](/bin) folder are little shell wrappers that execute commands in containers. These wrappers will do all the magic so that the commands will: 1) run in your current working directory; 2) mount your filesystem appropriately, and 3) run as the current user/group to preserve permissions. Thus, each of the command in the `bin` is a drop-in replacement for the corresponding utility, so that you don't have to install any of this stuff natively. I add this `bin` folder to my `PATH` and then immediately have access to each of these tools using the respective containers:

```
export PATH='$PATH:path/to/docker/bin'
```


## `shny` - running a shiny server

I use the `bin/shny` executable. Run an app in the shiny server container like so:

```
shny ~/code/LOLAweb/apps/LOLAweb
```

Since this is going to `docker exec` the `shiny::runApp()` function in `R`, the container needs to be already running at the moment (start it with `dR`).

## `rxgn` - Roxygenize in container

```
rxgn ~/code/LOLA
```

## `jekyll` - build and serve a Jekyll website from a container
My `jekyll` container has Ruby, Jekyll, and friends, so I can serve up local jekyll sites for testing without installing Ruby and so forth. 

Serve up a jekyll site with:

```
cd path/to/blog
jekyll serve 
```

Then visit the site at `http://localhost:4000/`

## liquify

This container combines Liquid templates and YAML data to build a simple, command-line templating system. The container just has `ruby` and `liquid` and a simply ruby script. The executable, `bin/liquify`, uses the container to populate a template with data from a yaml file, and spit the output into an outfile. That's it! You can see the command-line options with `liquify -h`:

```
    -t, --template TEMPLATE          Liquid template file
    -d, --data DATA                  YAML data file
    -o, --outfile OUTFILE            Output file
```

## `pandocker` - `pandoc` in docker

`pandocker` is a containerized drop-in replacement for the `pandoc` executable. I build a container with `pandoc` and a bunch of `latex` prereqs, and then you can just use the `pandocker` executable and it will run your stuff through `pandoc` in the container. I softlinked `pandoc` to `pandocker` and now I don't have to install pandoc or any latex stuff on any of my computers.

## `jabref`

I use `bin/jabref` as an executable to run containerized Jabref. This is more convenient than using the Java CLI (`java -jar blah blah`), and also lets me run multiple `jabrefs` at the same time, which is nice, because I can run exports even when `jabref` is already running.


## libreoffice

Run containerized libreoffice with:
```
libre
```

# Non-executable container descriptions

## rim
My R production environment, based on the bioconductor Docker containers, then installs a bunch of packages I use regularly (the packages lists are, for example, in [Rsetup/rpack_basic.txt](Rsetup/rpack_basic.txt)).

Currently this is the same as rdev.

## rdev

My R development environment, for building packages. Comes with everything you need to run `R CMD check` and `R CMD BiocCheck` -- and a couple of helper scripts in `bin/`:

This will build, RCheck, and BiocCheck the package, all in a docker container with all dependencies already required.
```
dockrpack /path/to/R/package
```
This will run your unit tests in a docker container:
```
dockrtest /path/to/R/package
```

## `vis`

My docker container to convert Inkscape SVG figures into PDF, and other graphics processing tasks. Under construction.

TODO:
* put svg2pdfpng in here
* write a wrapper script (a la djserve) to run the conversion in a container


# Using dpipe

The `dpipe` script in `/bin` lets you create slick containerized version of software that will behave in the host OS as if the software were installed locally (it will just handle permissions, setting userID, etc), so that you don't have to install them. As an example, look at `pandocker`, which can replace the `pandoc` executable if you stick `/bin` in your `PATH`.

I should produce other things using the same system.

# TO DO:

I'm working on a script to be able to run any of these `script/drun` followed by the name of the image.



# Installing an R environment 

Originally, I was install packages *inside* my R containers. This seems to be the preferred way to do this from the rocker and bioconductor containers. After a few years of doing it this way, I've decided I changed my mind: I think it makes more sense to just containerize the basic requirements and then install and host the packages locally. The reason is that otherwise the containers become huge, and it gives more control over which packages I want to hold on which systems, while still providing all the system prerequisites and R versions.

You can also use these to install all these nice R packages in one shot.

```
gclo nsheff/docker
cd code/docker
Rscript Rsetup/install_bioconductor.R
Rscript Rsetup/install_fonts.R
Rscript Rsetup/Rsetup.R
Rscript Rsetup/Rsetup.R --packages=Rsetup/rpack_basic.txt
Rscript Rsetup/Rsetup.R --packages=Rsetup/rpack_bio.txt
```


## Some useful commands:

* `docker rm $(docker ps -a -q)`: cleans all stopped containers
* `docker image prune`: removes all dangling images



