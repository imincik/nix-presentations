---
title: Advantages of Nix powered GRASS environment
theme: solarized
---

# Advantages of Nix powered GRASS environment

GRASS GIS Community Meeting, Prague, 2024

**Ivan Mincik, @imincik**, Nix Geospatial Team

---

Nix is **a huge thing** and sometimes **looks like a magic.**

---

Docker builds **ARE (usually) repeatable**, but they are **NOT designed to be reproducible !**

**Reproducibility =** the same input always returns the exactly same output.

---

## Why Nix ?

The way for **running a software on a computer**.

*(E. Dolstra, PhD theses, 2006)*

---

## Features of Nix

* **Reproducibility** as a core feature:

  * between multiple builds
  * between multiple machines
  * over the time

* **Full control** over whole **dependency graph**

* **No software conflicts**

---

## Features of Nix

* Software **deployment** and **configuration** is **declarative** (using Nix lang. and/or CLI)

* Software versions are **locked forever** or **updated only when requested**

* Runs on **all Linux, Mac, Win WSL 2**

* Dozens of other **very unique features**

---

## Advantages for Geospatial users ?

* **Isolated envs** for **all types of project components** (CLI, GUI, DBs, services, languages, scripts)

* All components are **consistently built** together (same GDAL for Python, QGIS, PostGIS, ...)

* Each project can use **different versions of software**

* Great **customization support**

---

## Advantages for GRASS users ?

**Reproducibile environments**

... should be mandatory for researchers and scientists.

---

## What is Nix ?

* **Nix** - the package manager/build system
* **Nix** - the language
* **Nixpkgs** - the largest packages repo
* **Nix modules system** - the declarative configuration management
* **NixOS** - the unique operating system

*\+ dozens of other community projects (Home Manager, ..)*

---

## What is Geospatial NIX ?

* **Geo** and **non-geo** software repository

* **Isolated** and **reproducible environment** for **geospatial** projects

* Runs on all **Linux** distros, (Mac) and in **containers**

---

## Components of Geospatial NIX ?

* **Geospatial NIX** - weekly updated repo

* **Geospatial NIX.env** - environment builder

* **https://geospatial-nix.today/** - web UI

---

# DEMO

(magic Nix one-liners)

---

## Run GRASS

* No GRASS installed

  ```bash
  $ grass

  The program 'grass' is not in your PATH.
  ```

* Run GRASS from Internet repo (geospatial-nix)

  ```bash
  $ rm -rf ~/grassdata  # note to myself

  $ nix run github:imincik/geospatial-nix#grass -- --text -c EPSG:4326 -e ~/grassdata/world_latlong_wgs84

  $ nix run github:imincik/geospatial-nix#grass -- --version

  GRASS GIS 8.3.2
  ```

---

## Run GRASS (other version)

* Run GRASS in other version

  ```bash
  $ nix run github:imincik/geospatial-nix/58d8cff#grass -- --version

  GRASS GIS 8.3.1
  ```

---

## Shell environment

* No QGIS installed

  ```bash
  $ qgis

  The program 'qgis' is not in your PATH.
  ```

* Create shell environment with GRASS and QGIS

  ```bash
  $ nix shell github:imincik/geospatial-nix#{grass,qgis}

  $ grass --version
  GRASS GIS 8.3.2

  $ qgis --version
  QGIS 3.36.3-Maidenhead 'Maidenhead' (exported)
  ```

---

## Which GRASS ?

* Which GRASS and QGIS ?

  ```bash
  $ which grass
  /nix/store/4gngq1xaiimh953d85w6kay9jgr904cm-grass-8.3.2/bin/grass

  $ ls /nix/store/<HASH>-grass/
  ```

* Show package dependencies

  ```bash
  $ nix derivation show github:imincik/geospatial-nix#grass
  ```

* Exit shell environment

  ```bash
  $ exit  # no grass anymore :(
  ```

---

## GRASS package

* [GRASS package recipe](https://github.com/imincik/geospatial-nix/blob/master/pkgs/grass/default.nix)

---

## GRASS customization 1

* Modify build dependencies (latest GDAL from master)

  ```bash
  $ nix run -L --impure --expr \
    "let \
      f = builtins.getFlake "github:imincik/geospatial-nix"; \
      p = f.packages.x86_64-linux; \
    in p.grass.override { gdal = p.gdal-master; }"

  $ g.version -e

  GRASS 8.3.2 (2024)
  PROJ: 9.4.1
  GDAL/OGR: 3.10.0dev
  GEOS: 3.12.1
  SQLite: 3.43.2
  ```

---

## GRASS customization 2

* Modify build configuration (no X support)

  ```bash
  $ nix run -L --impure --expr \
    "let \
      f = builtins.getFlake "github:imincik/geospatial-nix"; \
      p = f.packages.x86_64-linux; \
    in p.grass.overrideAttrs (old: { configureFlags = old.configureFlags ++ [ \"--without-x\" ]; })"
  ```

---

## GRASS in container

* Nix is a better Docker image builder than Docker

  ```bash
  $ nix build --impure --expr \
    "let \
      f = builtins.getFlake "github:imincik/geospatial-nix"; \
      p = f.packages.x86_64-linux; \
      np = f.inputs.nixpkgs.legacyPackages.x86_64-linux; \
    in np.dockerTools.buildImage \
      { name = \"grass\"; config.Cmd = [ \"\${p.grass}/bin/grass\" \"--version\" ]; }"

  $ docker load < ./result

  $ docker run grass:<TAG>
  ```

---

## GRASS development environment 1

* Get GRASS source code

  ```bash
  $ mkdir grass-dev && cd grass-dev

  $ git clone git@github.com:imincik/grass.git code
  $ cd code

  $ git checkout nixfile  # branch containing flake.nix
  ```

* Build GRASS from source

  ```bash
  $ nix develop

  $ ./configure

  $ make -j8
  ```

---

## GRASS development environment 2

* Automatically enable the environment

  ```bash

  $ echo "use flake" > .envrc
  
  $ direnv allow
  ```

https://direnv.net/

---

## GRASS development version

* Run latest GRASS development version

  ```bash

  $ nix run github:imincik/grass/nixfile#grass -- --text
  ```

---

## GRASS addons (current WIP)

* Launch GRASS with addons

  ```bash
  $ export GRASS_ADDON_BASE=$(\
      nix build --print-out-paths github:imincik/geospatial-nix/grass-addons-package#grass-plugin-r-hydrodem
    )
    
  $ nix run github:imincik/geospatial-nix#grass -- --text --exec r.hydrodem
  ```

---

## GRASS module (current WIP)

* GRASS Geospatial NIX module

  ```nix
    # https://github.com/imincik/grass-addons-module/blob/master/geonix.nix

    applications.grass = {
      enable = true;
      plugins = p: [ 
          geopkgs.grass-plugin-r-hydrodem
          geopkgs.grass-plugin-v-histogram
      ];
    };
  ```

---

## Geospatial NIX.today

* https://geospatial-nix.today/

---

## Nix documentation

* https://nix.dev/

