# Hyperledger Fabric - Internals Documentation

[![Build Status](https://travis-ci.org/IBM/hlf-internals.svg?branch=master)](https://travis-ci.org/IBM/hlf-internals)

This repository contains documentation on the internal architecture of Hyperledger Fabric with focus on the management of chaincode and its life-cycle. This repository is the souce of <https://ibm.github.io/hlf-internals> and the main content is located in the `docs` folder.

Structure and content:

- [docs](docs): root directory for the documentation.
- [assets](assets): supporting materials to produce the documentation

The documentation is written in Markdown plus a couple of extensions. To build this repository:

```bash
    # install Python v 3.7

    pip install mkdocs mkdocs-material pygments mkdocs-img2fig-plugin
    mkdocs build
    mkdocs gh-deploy --force
```

Look at the [mkdocs.yml](mkdocs.yml) for details about the installed extensions and plugins.

**Disclaimer**: this documentation comes AS IS and there is no warranty on the correctness of the content.  The documentation has been produced by extracting design, architectural concepts and process flows by inspecting the code of Hyperledger Fabric v1.4. While I am trying to keep it updated and get validation about the content, I also have other work to do and this is primarily a side project. Use it at your risk. We leveraged to build the support for in Fabric to author smart contracts in Haskell.

**Note**: if the content of the `docs` folder is browsed directly, some of the referenced images will now be rendered in the GitHub browser. This is not an error but a result of the fact that when these pages are rendered in a static website all the Markdown documents are transformed into directories to make friendlier URLs. This transformation changes the relative paths of the images with respect to the file. Therefor, in order to resolve these path correctly once deployed we need to add an additional `../` to correct the effective path encoded in the static HTML pages.
