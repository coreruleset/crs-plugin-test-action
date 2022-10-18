# CRS Plugin Test GitHub Action

This repository contains the workflows that plugins use to test they work properly.

There are two workflows:

- regular linting: checks that plugin configuration files and yaml files are sane.
- plugin ftw tests: each plugin should have their own tests in the `tests/regression` directory.
  - this workflow receives an input named `config_file` that receives a string that is stored as a
    `crs-setup.conf` file
