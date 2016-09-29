# Repology

![Example report](docs/screenshot.png)

Repology analyzes multiple package repositories and compares versions
of packages in them.

The report it produces shows which repositories contain or lack
which packages, and whether packages need updating.

## Uses

- **Users**:
  - Compare completeness and freshness of package repositories,
    choose most up to date distro
  - Find out what repositories contain newest versions of packages
    you need
- **Package/port maintainers**:
  - Another way to track new releases of software you package
  - Compete with other distros in keeping up to date
  - Find fellow maintainers to resolve packaging problems together
  - Keep package naming and versioning schemes in sync to other
    repos for your and your user's convenience
- **Software authors**:
  - Keep track of how well your project is packaged
  - Keep in touch with your product package maintainers
  - If you're not using [semantic versioning](http://semver.org/)
    yet, see how useful it is

# Status

Repology is currently in an early phase of development, with a goal
of creating usable utility in a quick and dirty way. For now, it is
usable in two modes: as a command line generator of single HTML
report and a static website generator for [repology.org](repology.org).

## Howto

### Dependencies

- [python3](https://www.python.org/), with the following modules
  - [pyyaml](http://pyyaml.org/) YAML parser
  - [setuptools](https://pypi.python.org/pypi/setuptools)
  - [jinja2](https://jinja.pocoo.org/) template engine
  - [requests](http://python-requests.org/) (for fetching some repository data)
- [wget](https://www.gnu.org/software/wget/) (for fetching some repository data)
- [git](https://git-scm.com/) (for fetching some repository data)

### Usage

Repology operation is split into two distinct steps: repository
data processing and producing the report. Separate utilities exist
for these two phases.

#### Fetching data

First, you need to fetch repository data with ```repology-update.py``` utility.

```
./repology-update.py --statedir repology.state --tag production --verbose --fetch --update --parse
```

```--statedir``` is mandatory for all utilities and specifies path
to directory where repository data will be stored. Directory is
created by ```repology-update.py``` if it doesn't exist.

```--tag``` and ```--repository``` specify which repositories to
update. ```--tag``` is most useful as it allows to quickly pick
repository groups. Tags include ```all``` to process all supported
repositories, ```production``` to only process repositories listed
on [repology.org](repology.org). Multiple tags have AND logic, e.g.
you get repositories which have ALL the tags you specify.

```--verbose``` is useful for tracking progress of utility operation

Remaining arguments specify which actions to perform.

```--fetch``` and ```--update``` control whether downloading of
repository data is allowed. Specify none to disallow downloading,
specify ```--fetch``` so allow only downloading for repository
data which hasn't been downloaded yet, or specify both options
to always download data, e.g. keep all your repository data up
to date.

```--parse``` enables parsing repositories and processing serialized
data other utilities may use.

#### Generating HTML reports

Next, you need to produce HTML page with report. The following
command generates complete (huge!) report on a single page:

```
./repology-report.py --statedir repology.state --tag production --output full.html
```

```--statedir``` and ```--tag``` have the same meaning as with
fetching and should generally match arguments you've specified
to ```repology-update.py```. But you are free to produce report
for smaller subset of repositories.

Since the full report is really huge any may hang your browser
and hard to read, you may use multiple filtering options:

- ```-m``` filter by maintainer (e.g. ```amdmi3@FreeBSD.org```)
- ```-c``` filter by category (e.g. ```games```; note that this
	       matches substring, e.g. ```games-action``` from Gentoo
	       as well)
- ```-n``` filter only packages present in this many repos
- ```-i``` matches only packages present in specified repo
- ```-x``` matches only packages not present in specified repo

Note, that generated report uses ```repology.css``` stylesheet,
so if you want to copy report to another directory, you'll want to
copy css file along with it.

#### Examples

Packages which likely are needed to be added to pkgsrc (because
all other repositories have them) and their versions:

```
./repology-report.py -s repology.state -t production -x pkgsrc -n 3 -o missing-pkgsrc.html
```

State of FreeBSD ports maintained by me:

```
./repology-report.py -s repology.state -t production -m amdmi3@FreeBSD.org -i FreeBSD -o maintainer-amdmi3.html
```

State of games along all repos, with irrelevant packages filtered out:

```
./repology-report.py -s repology.state -t production -c games -n 2 -o games.html
```

#### Generating static website

Finally, if you want to produce your own instance of
[repology.org](http://repology.org) website, run

```
./repology-gensite.py --statedir repology.state --tag production --output /usr/www/repology.org
```

This generates self-contained website in ```/usr/www/repology.org```
directory which is created if it doesn't exist.

## Repository support

### FreeBSD

Links:
- [FreshPorts](http://freshports.org)
- [SVN](https://svnweb.freebsd.org/ports/head)

Parses INDEX-11.bz2 file, available data:
- name
- version
- comment
- maintainer
- category

TODO: Parsing port Makefiles may extract additional info such as
licenses and options.

### Debian/Ubuntu

Links:
- [https://www.debian.org/distrib/packages#search_packages](Debian Packages)
- [http://packages.ubuntu.com/](Ubuntu Packages)

Parses Sources.gz, available data:
- name
- version
- maintainer
- category
- homepage

TODO: May need to parse ```*.debian.tar.xz``` for additional info for
each package. Heavy?

### Gentoo

Links:
- [Gentoo packages](https://packages.gentoo.org/)

Parses git mirror of portage repository (file tree only, doesn't
look into ebuilds), available data:
- name
- version
- category

TODO: Parse metadata.xml for maintainer. Parse ebuilds for more info.

### pkgsrc

Links:
- [pkgsrc.org](http://pkgsrc.org/)
- [CVS](http://cvsweb.netbsd.org/bsdweb.cgi/pkgsrc/)
- [pkgsrc.se](http://www.pkgsrc.se/)

Is capable of generating INDEX file similar to that of FreeBSD, but
it takes too much time (hours). For now, we parse README-all list
from NetBSD, which only contains:
- name
- version
- category
- comment

TODO: Generate and parse INDEX, even if it's done infrequently.

### OpenBSD

- [CVS](http://cvsweb.openbsd.org/cgi-bin/cvsweb/ports/)
- [openports.se](http://openports.se/)

Parses INDEX file (format similar to FreeBSD), available data:
- name
- version
- comment
- maintainer
- category

### Arch

- [Package Search](https://www.archlinux.org/packages/)

Parses package databases (core, extra, community), available data:
- name
- version
- comment
- homepage
- maintainer (packager, actually)
- license

## Reading the report

Report is HTML table, columns correspond to repositories and rows
correspond to packages. Cells hold package versions, highlighted
as follows:

- ```cyan```: package is only present in a single repo: nothing to
              compare version to, and may be local artifact
- ```green```: package up to date
- ```red```: package outdated (there's newer version in some other repo)
- ```yellow```: there are multiple packages some of which are up to date
                and others are outdated
- ```gray```: version was manually ignored, likely because of broken
              versioning scheme

Note that there may be multiple packages of a same name in a single repo
(either naturally, or because of name transformations).

## Package matching

Some packages are named differently across separate repositories,
so package name transformations are performed to merge differently
named packages into a single entity. [rules.yaml](rules.yaml) contains
rules for such transformation, is in YAML format containing array
of rules. Each rule may contain:

- Matching conditions:
  - ```repo```: only apply rule to repositories of specific type
    (currently ```freebsd```, ```debian```, ```gentoo```, ```pkgsrc```,
     or ```openbsd``` note that these are NOT repository names)
  - ```name```: apply to package of specified name
  - ```namepat```: apply to package with name matching specified regexp
  - ```verpat```: apply to package with version matching specified regexp

- Actions:
  - ```ignore```: completely ignore the package
  - ```setname```: set package name to an argument. ```$0``` is replaced
    with old name, ```$1```, ```$2``` and so on may be used to refer
    capture groups in ```namepat``` regexp, if it's present
  - ```pass```: leave package as is. Useful to prevent following rules
    from matching

Only first matching rule applies. If no rule applies (or ```pass```
rule applies), package data is not modified. Afterwards, regardless
of whether any rule matched, name is always converted to lowercase
and underscores (```_```) replaced with hyphens (```-```).

Note that transformation may merge several packages into single
entry. If they are unrelated this may be a problem (solved by giving
them unique names by rules based on e.g category), otherwise it is
useful and may be done intentionally to merge variants or parts of
specific project into single entity (e.g. merge ```freeciv-client```
and ```freeciv-server``` into ```freeciv``` in pkgsrc, to match
them both against ```freeciv``` in other repos, or match ```fltk15```,
```fltk16```, ```fltk17``` into just ```fltk``` and look at newest
version).

### Example

Debian has ```botan1.10```, FreeBSD has ```botan110``` and other
repos have ```botan```. To merge these into single entity:

```
- { repo: debian, name: botan1.10, setname: botan }
- { repo: freebsd, name: botan110, setname: botan }
```

FreeBSD names Perl module ports as ```p5-My-Module```, Debian does
```libmy-module-perl```, and Gentoo just places ```My-Module``` under
```dev-perl``` category. To merge all these perl modules into single
synthetic ```perl:my-module``` entry (and actually do the same to
all perl modules at once):

```
- { repo: freebsd, namepat: "p5-(.*)",      setname: "perl:$1" }
- { repo: debian,  namepat: "lib(.*)-perl", setname: "perl:$1" }
- { repo: gentoo,  category: dev-perl,      setname: "perl:$0" }
```

## Author

* [Dmitry Marakasov](https://github.com/AMDmi3) <amdmi3@amdmi3.ru>

## License

GPLv3 or later, see [COPYING](COPYING).
