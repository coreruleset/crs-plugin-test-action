# CRS Plugin Test GitHub Action

This repository contains the workflows that plugins use to test they work properly.

There are two workflows:

- regular linting: checks that plugin configuration files and yaml files are sane.
- plugin ftw tests: each plugin should have their own tests in the `tests/regression` directory.
  - this workflow receives an input named `crs-config` that receives a string that is stored as a
    `crs-setup.conf` file

Take a look at the [template plugin](https://github.com/coreruleset/template-plugin) for an example on how to use these two workflows.

## Extending CRS config

If your test needs to configure additional variables or you need a particular setup, the `crs-config` variable can be used like this:
```
jobs:
  plugin-lint:
  integration-tests:
    uses: coreruleset/crs-plugin-test-action/.github/workflows/integration.yaml@main
    with:
      crs-config: |
        SecRule &TX:my-rule-exclusions-plugin_enabled "@eq 0" \
          "id:10101010,\
          phase:1,\
          pass,\
          nolog,\
          setvar:'tx.my-rule-exclusions-plugin_enabled=0'"
```

And that will include the file on the running config for the tests.

## Writing tests for plugins

Many plugins will target product exclusions, so when writing tests it is best to use the values on a rule and check that the targeted excluded rule is not being logged. An example tests could work like this:

For the rule:
```
SecRule REQUEST_FILENAME "@endsWith /tbl_find_replace.php" \
    "id:9513360,\
    phase:1,\
    pass,\
    t:none,\
    nolog,\
    ver:'phpmyadmin-rule-exclusions-plugin/1.0.0',\
    chain"
    SecRule &REQUEST_COOKIES_NAMES:'/^(?:phpMyAdmin|phpMyAdmin_https)$/' "@eq 1" \
        "t:none,\
        ctl:ruleRemoveTargetById=942140;ARGS:db"
```

We create a test like:
```
...
  input:
    dest_addr: 127.0.0.1
    headers:
      Host: localhost
      User-Agent: OWASP ModSecurity Core Rule Set
      Accept: text/xml,application/xml,application/xhtml+xml,text/html;q=0.9,text/plain;q=0.8,image/png,*/*;q=0.5
      Cookie: phpMyAdmin=1
    port: 80
    method: POST
    uri: /post/tbl_zoom_select.php
    data: db=mytestdb
  output:
    no_log_contains: id "942140"
```

In the test you can see we added the relevant parts, and that there is no logging on the rule that is being removed.
