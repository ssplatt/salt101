# Salt 101 - The Basics of Saltstack
Welcome to the world of Saltstack Salt. Simply put, it is a configuration management system written in Python and controlled using YAML with Jinja templating.

## Prerequisites
  - Virtualbox
    - https://www.virtualbox.org/wiki/Downloads
    - `brew cask install virtualbox`
  - Vagrant
    - https://www.vagrantup.com/downloads.html
    - `brew cask install vagrant`
  - Bento Boxes
    - https://app.vagrantup.com/bento
    - `mkdir tmp; cd tmp; vagrant init bento/debian-8.8; vagrant up`

## References
  - https://docs.saltstack.com/en/getstarted/
  - https://docs.saltstack.com/en/latest/topics/using_salt.html
  - http://www.yaml.org/start.html
  - http://jinja.pocoo.org/docs/2.9/templates/

## Install Salt
  * Bootstrap Script
    * `vagrant ssh`
    * `sudo su`
    * `wget https://raw.githubusercontent.com/saltstack/salt-bootstrap/develop/bootstrap-salt.sh`
    * `sh bootstrap-salt.sh`

## Basics of YAML
  * Key / Value pairs
    * Strings
      * `myvar: value`
    * Dictionaries (Mapping)
      * ```
        mydict:
          key: value
        ```
    * Lists (Sequence)
      * ```
        mylist:
          - one
          - two
          - three
        ```
    * Multiline input
      * ```
        myvar: |
          this is some big input
        ```
  * Comments
    * `#`
  * Quotes vs. No Quotes
    * dates
    * true, True, yes, false, False, no
  * Merging
    * matches on Key
    * Dictionaries will merge
    * Strings will overwrite
    * Lists will overwrite

## Basics of Jinja
  * Comments
    * `{# ... #}`
  * Expressions
    * Variables
      * `{{ foo.bar }}`
      * `{{ foo['bar'] }}`
  * Statements
    * `{% ... %}`
    * Tests
      * `If`
        * `{% if myvar %} ... {% endif %}`
        * `{% if ... %} ... {% elif ... %} ... {% else %} ... {% endif %}`
    * Loops
      * `For`
        * `{% for i in mylist %} ... {% endfor %}`
        * `{% for k,v in myvar.iteritems() %} ... {% endfor %}`
      * Special Variables
        * `loop.first`
        * `loop.last`
    * Assignments
      * `{% set myvar = "my string" %}`
  * Filters
    * `{{ listx|join(', ') }}`
  * Whitespace
    * `{%- ... -%}`
    * ```{% for item in seq -%}
        {{ item }}
      {%- endfor %}```
  * Operators
    * `in`
      * `{% if "string" in myvar %} ...`
    * `is`
      * `{% if myvar is defined %} ...`
    * `|`
      * `{{ myvar | capitalize }}`
    * `~`
      * `{{ myvar ~ " concatenate this string" }}`

## Basics of Salt
  * Masters and Minions
    * Master is the controlling node
    * Minion is the node being controlled
  * States
    * Actions to perform on a Minion
      * `pkg.installed`
      * `file.managed`
  * Pillar
    * Variable definitions for a Minion
      * `pkg_name: vim`
      * `enabled: false`
  * Grains
    * Facts that a Minion knows about itself
    * `salt-call --local grains.items`
    * `salt-call --local grains.get init`
    * `{{ grains.init }}`
  * Targeting
    * Glob
      * "db\*"
