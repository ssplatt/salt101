# Salt 101 - The Basics of Saltstack Salt
Welcome to the world of Saltstack Salt. Simply put, it is a configuration management system written in Python and controlled using YAML with Jinja templating.

## Prerequisites
Before arriving at class, please make sure these prerequisites are installed on your system.

  - Virtualbox
    - https://www.virtualbox.org/wiki/Downloads
    - Install with brew on Mac OS: `brew cask install virtualbox`
  - Vagrant
    - https://www.vagrantup.com/downloads.html
    - Install with brew on Mac OS: `brew cask install vagrant`
  - Bento Boxes
    - https://app.vagrantup.com/bento
    - Pull down the Debian 8 Image: `vagrant box add bento/debian-8.8`
      - If you choose not to use Debian 8, you may have to change some of the provided directions to fit your environment
    - I've provided a minimal vagrant config in vagrant-tmp; you can use that or make your own
      - to use the one provided
        - `cd vagrant-tmp; vagrant up`
      - to make your own
        - `mkdir tmp; cd tmp; vagrant init bento/debian-8.8; vagrant up`

## References
You may find it useful to browse these links prior to the class, or leave them open for reference during the class.

  - https://docs.saltstack.com/en/getstarted/
  - https://docs.saltstack.com/en/latest/topics/using_salt.html
  - http://www.yaml.org/start.html
  - http://jinja.pocoo.org/docs/2.9/templates/
  - https://docs.saltstack.com/en/latest/salt-modindex.html

## Install Salt
Set up your experimental environment. We will be running 'masterless', so we will only install the 'minion' package and run all commands locally.

  - Bootstrap Script
    1. `vagrant ssh`
    2. `sudo su`
    3. `wget https://raw.githubusercontent.com/saltstack/salt-bootstrap/develop/bootstrap-salt.sh`
    4. `sh bootstrap-salt.sh`
  - Manually through the package manager
    1. `vagrant ssh`
    2. `sudo su`
    3. `apt install apt-transport-https ca-certificates`
    4. `echo "deb https://repo.saltstack.com/apt/debian/8/amd64/latest/ jessie main" > "/etc/apt/sources.list.d/saltstack.list"`
    5. `apt-key add https://repo.saltstack.com/apt/debian/8/amd64/latest/SALTSTACK-GPG-KEY.pub`
    6. `apt update`
    7. `apt install salt-minion`

## Basics of Salt
Salt can be used as a simple configuration management system, but automation and orchestration are possible with more

  - Masters and Minions
    - Master is the controlling node
    - Minion is the node being controlled
    - A Master is not always needed
    - Commands can be run locally on a minion to query and configure itself
      - `salt` run from a master and targets one or more minions
      - `salt-call` runs from a minion and targets the same minion
        - `salt-call --local` does not reach out to a master, uses local cache
  - Execution Modules
    - "one off" execution runs
    - usually beneficial to remotely execute a command on a minion
    - `salt-call --local sys.doc`
    - `salt-call --local sys.doc pkg`
    - `salt-call --local sys.doc pkg.install`
    - `salt-call --local cmd.run 'dpkg -l | grep systemd'`
    - Example:
      ```
      # salt-call --local pkg.install vim
      local:
          ----------
          vim:
              ----------
              new:
                  2:7.4.488-7+deb8u3
              old:
          vim-runtime:
              ----------
              new:
                  2:7.4.488-7+deb8u3
              old:
      ```
  - SLS, Salt State files
    - YAML and Jinja rendering by default
  - States
    - Actions to perform on a Minion
    - Similar to execution modules
      - `pkg.installed`
        - if the defined package is not installed, install it
      - `file.managed`
        - if the file does not have the defined content, change the contents
        - if the file is not owned by the defined user, change the ownership
        - if the file does not have the defined mode, change it
    - [Idempotent](http://www.dictionary.com/browse/idempotent)
      - applying a salt state over and over with the same pillar definition should not result in changes on the minion
  - Pillar
    - User defined values for a Minion
    - Pillar data is referenced in state files and Jinja templates
    - `pkg_name: vim`
    - `enabled: false`
  - Grains
    - Facts about the environment on the minion
      - OS type, version, kernel version
      - CPU info
      - Hostname, IPs, MAC addresses
      - Disk drives
    - Minions cannot see the grains of other minions
    - `salt-call --local grains.items`
    - `salt-call --local grains.get virtual`
    - Example:
      ```
      # salt-call --local grains.get ip4_interfaces
      local:
          ----------
          eth0:
              - 10.0.2.15
          lo:
              - 127.0.0.1
      ```
    - Can be used in Jinja `{{ grains.oscodename }}`
  - Targeting
    - When commanding minions from a master, you can run the module against all minions or a subset of them.
    - Used in the Top files
    - The most basic: The Glob
      - `salt \* pkg.install vim`
      - `salt db\* pkg.install vim`
  - Highstate
    - This will apply all of the configured state definitions on the targeted minions
    - merges `state/top.sls` and `pillar/top.sls` to configure the minion
    - `salt-call --local state.highstate`
  - Top.sls
    - The main definition files for the Configuration of Minions in your environment
    - Used during `highstate` runs
    - Can define multiple environments, `base` is the default
    - `states/top.sls`
      - define which state files to apply to which minions
    - `pillar/top.sls`
      - define which pillar files to apply to which minions
  - init.sls
    - a special sls file that will be called if only the Directory name is referenced
    - similar to `index.html` on a website
    - in a top.sls file, `mysql` could mean one of these two files:
      - `mysql.sls`
      - `mysql/init.sls`

## Basics of YAML
The majority of the code you touch in Salt is YAML. YAML is very human readable and easy to use.

  - Key / Value pairs
    - Strings
      - `myvar: value`
    - Dictionaries (Mapping)
      - ```
        mydict:
          key: value
        ```
    - Lists (Sequence)
      - ```
        mylist:
          - one
          - two
          - three
        ```
    - Multiline input
      - ```
        myvar: |
          this is some big input
          on multiple lines
          that will act as one
        ```
  - Comments
    - `#` begins a comment, even if not at the very start of a line
    - CAUTION: using this style of comment will not stop Jinja from parsing on that line
  - Quotes vs. No Quotes
    - In general, single and double quotes work the same
    - Most of the time, quotes are unneccesary
    - Quotes are neccessary to escape certain characters and to prevent auto-formatting of special terms
      - dates or strings that may look like dates
      - yes, true, True
      - no, false, False
  - Merging
    - matches on Key
    - Dictionaries will merge
    - Strings will overwrite
    - Lists will overwrite

### YAML Excercises
In this exercise, we will produce three YAML files. The states top file, the pillar top file, and a basic pillar file. This will get us ready to start applying state to a machine.

#### Create the pillar top file

  1. Get into your Vagrant VM, `vagrant ssh` if you haven't already.
  2. Become root, `sudo su` if you haven't already.
  3. Make the salt pillar directory, `mkdir -p /srv/pillar`
  4. Enter the pillar directory, `cd /srv/pillar`
  5. Create a file named "top.sls", `touch top.sls`
  6. Edit top.sls, `vi top.sls` enter this data into it: (if using vi, hit `i` to insert text)
  
```
base:
  "*":
    - testing
```

  7. Save the file and exit the editor. If using vi, hit the `escape` key to leave input mode, then `ZZ` or `:wq` to write and quit.

#### Create the testing pillar file

  1. While in `/srv/pillar`, make a new file "testing.sls", `touch testing.sls`
  2. Edit testing.sls and enter this text into it:
  
```
anewtest:
  enabled: true
  config:
    testvar: hello world
    testloop:
      varone: foo
      vartwo: bar
```

  3. Save the file and exit the editor
  4. Check to see if Salt recognizes the pillar data, run `salt-call --local pillar.items`
  
```
~# salt-call --local pillar.items
local:
    ----------
    anewtest:
        ----------
        enabled:
            True
        config:
            ----------
            testvar:
                hello world
            testloop:
                ----------
                varone:
                    foo
                vartwo:
                    bar
```

#### Create the states top file

  1. Create the states directory, `mkdir -p /srv/salt`
  2. Change to the states directory, `cd /srv/salt`
  3. Create the top file, `touch top.sls`
  4. Edit the top file and put this data in it:
  
```
base:
  "*":
    - teststates
```

  5. Save the file and exit the editor

## Basics of Jinja
Jinja is the default templating language for Salt. Jinja is not a programming language. It can do some things that you'd find in Python, for example, but it is not intended to replace a true scripting or programming language. If you are trying to do advanced things with Jinja, you will know when it is time to move onto one of the other templating options.

  - Comments
    - Can be multiline or single line
    - `{# ... #}`
  - Expressions
    - Variables
      - `{{ foo.bar }}`
      - `{{ foo['bar'] }}`
  - Statements
    - `{% ... %}`
    - Tests
      - `If`
        - `{% if myvar %} ... {% endif %}`
        - `{% if ... %} ... {% elif ... %} ... {% else %} ... {% endif %}`
    - Comparisons
      - `==` is equal
      - `!=` is not equal
      - `>` greater than
      - `>=` greater than or equal to
      - `<` less than
      - `<=` less then or equal to
    - Operators
      - `in` test if thing is in other thing
        - `{% if "string" in myvar %} ...`
      - `is` test if 
        - `{% if myvar is defined %} ...`
      - `|` 
        - `{{ myvar | capitalize }}`
      - `~`
        - `{{ myvar ~ " concatenate this string" }}`
    - Logic
      - and
      - or
      - not
    - Loops
      - `For`
        - `{% for i in mylist %} ... {% endfor %}`
        - `{% for k,v in myvar.iteritems() %} ... {% endfor %}`
      - Special Variables
        - `loop.first`
        - `loop.last`
    - Assignments
      - `{% set myvar = "my string" %}`
  - Filters
    - `{{ listx|join(', ') }}`
  - Whitespace
    - `{%- ... -%}`
    - ```
      {% for item in seq -%}
        {{ item }}
      {%- endfor %}
      ```

### Jinja Excercise
In this exercise, we will create a configuration file with Jinja templating.

  1. Change into the states directory, `cd /srv/salt`
  2. Make a new file, "testfile.conf.j2", `touch testfile.conf.j2`
  3. Edit the file and put this data into it:
  
```
# This is a test configuration file
# Managed by Salt

A Value = {{ varpass.testvar }}

{% if varpass.testloop is defined %}
{% for key, val in varpass.testloop.iteritems() %}
{{ key }} = {{ val }}
{% endfor %}
{% endif %}
```

  4. Save the file and exit the editor

## Beginning Configuration Management
Now that we know the basics of Salt, YAML, and Jinja, we can create a state file that will allow Salt to manage the configuration of the minion.

### State file exercise
This exercise will result in a Salt state file which will define what commands and which values should be applied to the minion.

  1. Change into the salt states directory, `cd /srv/salt`
  2. Create the "teststates.sls" file, `touch teststates.sls`
  3. Edit the file and enter this data into it:
  
```
{% set anewtest = pillar['anewtest'] %}

{% if anewtest.enabled %}

test_configure_file:
  file.managed:
    - name: /root/testfile.conf
    - source: salt://testfile.conf.j2
    - user: root
    - group: root
    - mode: 600
    - template: jinja
    - varpass: {{ anewtest.config }}

{% else %}

anewtest_not_enabled:
  test.succeed_without_changes

{% endif %}
```

  4. Save the file and exit the editor

### Running your first highstate
With all of the files in place from the previous exercises, you should now be able to run your first "highstate".

  1. From any directory, execute the highstate, `salt-call --local state.highstate`
  
```
~# salt-call --local state.highstate
local:
----------
          ID: test_configure_file
    Function: file.managed
        Name: /root/testfile.conf
      Result: True
     Comment: File /root/testfile.conf updated
     Started: 20:53:24.839959
    Duration: 17.802 ms
     Changes:
              ----------
              diff:
                  New file

Summary for local
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:  17.802 ms
```

  2. Change directories to `/root`
  3. Check to see if testfile.conf exists and has the proper permissions, `ls -la testfile.conf`
  
```
~# ls -la testfile.conf
-rw------- 1 root root 109 Jul 21 20:53 testfile.conf
```

  4. Check that the contents of the file is what we expect, `cat testfile.conf`
  
```
~# cat testfile.conf

# This is a test configuration file
# Managed by Salt

A Value = hello world



varone = foo

vartwo = bar
```

  5. Run another highstate to see if our state definition is idempotent, `salt-call --local state.highstate`
  
```
~# salt-call --local state.highstate
local:
----------
          ID: test_configure_file
    Function: file.managed
        Name: /root/testfile.conf
      Result: True
     Comment: File /root/testfile.conf is in the correct state
     Started: 20:57:08.127061
    Duration: 16.498 ms
     Changes:

Summary for local
------------
Succeeded: 1
Failed:    0
------------
Total states run:     1
Total run time:  16.498 ms
```

  6. If you'd like to see the file change, you can edit one of the values in `/srv/pillar/testing.sls`

# Congratulations! You are now able to manage the configuration of a server using Salt.
