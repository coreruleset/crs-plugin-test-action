# CRS Plugin Test GitHub Action

This repository contains the workflows that plugins use to test they work properly.

There are two workflows:

- regular linting: checks that plugin configuration files and yaml files are sane.
- plugin ftw tests: each plugin should have their own tests in the `tests/regression` directory.
  - this workflow receives an input named `config_file` that receives a string that is stored as a
    `crs-setup.conf` file

An example input could be:
```
    with:
      plugin-config: |
        SecRule &TX:my-rule-exclusions-plugin_enabled "@eq 0" \
          "id:1234567890,\
          phase:1,\
          pass,\
          nolog,\
          setvar:'tx.my-rule-exclusions-plugin_enabled=1'"
```

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
