[![Version on Galaxy](https://img.shields.io/badge/available%20on%20ansible%20galaxy-jonaspammer.motd-brightgreen)](https://galaxy.ansible.com/jonaspammer/motd) [![Testing CI](https://github.com/JonasPammer/ansible-role-motd/actions/workflows/ci.yml/badge.svg)](https://github.com/JonasPammer/ansible-role-motd/actions/workflows/ci.yml)

An Ansible role for configuring static and dynamic login banners on linux machines.

For configuring OpenSSH’s Banner, see [ansible-role-openssh](https://github.com/JonasPammer/ansible-role-openssh/).

# 🔎 Metadata

Below you can find information on…

- the role’s required Ansible version

- the role’s supported platforms

- the role’s [role dependencies](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html#role-dependencies)

**[meta/main.yml](meta/main.yml)**

    ---
    galaxy_info:
      role_name: "motd"
      description:
        "An ansible role for configuring either static (traditional) or dynamic
        (using `update-motd` framework) login banners on linux machines."
      standalone: true

      author: "jonaspammer"
      license: "MIT"

      min_ansible_version: "2.11"
      platforms:
        - name: EL # (Enterprise Linux)
          versions:
            - "7" # actively tested: centos7
            - "8" # actively tested: rockylinux8, centos8
        - name: Fedora
          versions:
            - "35"
        - name: Debian
          versions:
            - buster # debian10 (actively tested)
            - bullseye # debian11 (actively tested)
        - name: Ubuntu
          versions:
            - xenial # ubuntu1604 (actively tested)
            - bionic # ubuntu1804 (actively tested)
            - focal # ubuntu2004 (actively tested)

      galaxy_tags: []

    dependencies: []

# 📌 Requirements

The Ansible User needs to be able to `become`.

The [`community.general` collection](https://galaxy.ansible.com/community/general) must be installed on the Ansible controller.

# 📜 Role Variables

    motd_type: [OS-specific for v1, see TODO in setup-update-motd.yml]

A MOTD can be implemented in 2 manners:

“static”
Generate a single static `/etc/motd` text file and dismantle update-motd’s scripts so only this file is being shown by `pam_motd` (traditional motd).

“update-motd”
Activate the `update-motd` functionality [(nicely described in Ubuntu’s MAN Page)](https://www.systutorials.com/docs/linux/man/5-update-motd/), by which the motd is dynamically assembled from a collection of scripts at login or periodically through the use of a `cron` job.

This role will delete the traditional `/etc/motd` file, leaving only the output of the scripts to be included in the assembled motd.

This role will install its own `update-motd` Implementation on systems without this functionality built into their version of `pam_motd`.

Additional Relevant Reading Material on the `update-motd` topic:

- [ Blog Post describing update-motd’s history and implementation details](https://ownyourbits.com/2017/04/05/customize-your-motd-login-message-in-debian-and-ubuntu/)

- [UbuntuWiki’s Proposal](https://wiki.ubuntu.com/UpdateMotd)

## Static MOTD Configuration Variables

    motd_static_motd_template: "issue.net"

Path to Ansible template ending in `.jinja2`.

This role comes bundled with a pre-defined legal banner. To use your own templates, ensure your template do not wear the exact names as the one in [this roles' `templates` directory](templates).

## Dynamic MOTD Configuration Variables

    motd_dynamic_scripts_system_packages: [OS-specific, see /defaults directory]

Packages to be installed by this role using [Ansible’s package module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/package_module.html).

    motd_dynamic_scripts_templates:
      - "00-legal" # in case SSH-Banner didn't show
      - "10-sysinfo"

Path to Ansible templates ending in `.jinja2` that are to be generated into `motd_dynamic_scripts_directory`.

This role comes bundled with some already defined templates. To use your own templates, ensure your templates do not wear the exact names as the ones in [this roles' `templates` directory](templates).

    motd_dynamic_scripts_directory: [OS-specific by default, see /vars directory]

Path to store the templated `motd_dynamic_scripts_templates`. Must **not** end with `/`.

    motd_dynamic_scripts_backup: false
    motd_dynamic_scripts_backup_path: "{{ motd_dynamic_scripts_directory }}-backup"

This role ensures that `motd_dynamic_scripts_directory` **only** contains the files stated in `motd_dynamic_scripts_templates`.

These variables define whether and where to backup files found in the mentioned directory that are not included in the list of this role’s defined script template names.

    motd_static_motd_backup: false
    motd_static_motd_backup_path: "/etc/motd-backup"

This role ensure’s that only the dynamic scripts have influence on the resulting motd.

If `/etc/motd` is found to be a normal text file, these variables define whether and where to backup this file.

# 📜 Facts/Variables defined by this role

Each variable listed in this section is dynamically defined when executing this role (and can only be overwritten using `ansible.builtin.set_facts`) _and_ is meant to be used not just internally.

# 🏷️ Tags

Tasks are tagged with the following [tags](https://docs.ansible.com/ansible/latest/user_guide/playbooks_tags.html#adding-tags-to-roles):

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;">Tag</th>
<th style="text-align: left;">Purpose</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td colspan="2" style="text-align: left;"><p>This role does not have officially documented tags yet.</p></td>
</tr>
</tbody>
</table>

You can use Ansible to skip tasks, or only run certain tasks by using these tags. By default, all tasks are run when no tags are specified.

# 👫 Dependencies

# 📚 Example Playbook Usages

This role is part of [ many compatible purpose-specific roles of mine](https://github.com/JonasPammer/ansible-roles).

The machine needs to be prepared. In CI, this is done in `molecule/resources/prepare.yml` which sources its soft dependencies from `requirements.yml`:

**[molecule/resources/prepare.yml](molecule/resources/prepare.yml)**

    ---
    - name: prepare
      hosts: all
      become: true
      gather_facts: false

      roles:
        - name: jonaspammer.bootstrap
        - name: jonaspammer.core_dependencies
        - name: jonaspammer.shellcheck

The following diagram is a compilation of the "soft dependencies" of this role as well as the recursive tree of their soft dependencies.

![requirements.yml dependency graph of jonaspammer.motd](https://raw.githubusercontent.com/JonasPammer/ansible-roles/master/graphs/dependencies_motd.svg)

    roles:
      - jonaspammer.motd

    vars:
      motd_legal_location_name: MY COMPANY INTRA # OPTIONAL variable used by built-in template

Resulting dynamic MOTD (example):

     _____________________________________________________________________________________
    /\                                                                                    \
    \_| You are connecting to the computer system 'srvweb' at MY COMPANY INTRA.           |
      |                                                                                   |
      | Any or all uses of this system and all files on this system may be                |
      | intercepted, monitored, recorded, copied, audited, inspected,                     |
      | and disclosed to authorized corporation and law enforcement personnel,            |
      | as well as authorized individuals of other organizations.                         |
      | By using this system, the user consents to such interception,                     |
      | monitoring, recording, copying, auditing, inspection,                             |
      | and disclosure at the discretion of authorized personnel.                         |
      |                                                                                   |
      | Unauthorized or improper use of this system may result in                         |
      | administrative disciplinary action, civil charges/criminal penalties,             |
      | and/or other sanctions as according to the european codes and/or countries codes. |
      |                                                                                   |
      | LOG OFF IMMEDIATELY if you do not agree to the conditions stated in this warning. |
      |   ________________________________________________________________________________|_
       \_/__________________________________________________________________________________/

           _,met$$$$$gg.          user@srvweb
        ,g$$$$$$$$$$$$$$$P.       ------------
      ,g$$P"     """Y$$.".        OS: Debian GNU/Linux 9.13 (stretch) x86_64
     ,$$P'              `$$$.     Model: Standard PC (i440FX + PIIX, 1996) pc-i440f
    ',$$P       ,ggs.     `$$b:   Kernel: 4.9.0-16-amd64
    `d$$'     ,$P"'   .    $$$    Uptime: 74 days, 19 hours, 42 minutes
     $$P      d$'     ,    $$P    Packages: 920
     $$:      $$.   -    ,d$$'    Shell: bash 4.4.12
     $$;      Y$b._   _,d$P'      Terminal: run-parts
     Y$$.    `.`"Y$$$$P"'         CPU: Common KVM (2) @ 1.7GHz
     `$$b      "-.__              GPU: Vendor 1234 Device 1111
      `Y$$                        Memory: 1858MB / 3955MB
       `Y$$.
         `$$b.                    ████████████████████████
           `Y$$b.
              `"Y$b._
                  `"""

    roles:
      - jonaspammer.motd

    vars:
      motd_type: static
      motd_legal_location_name: My Company # OPTIONAL variable used by built-in template

Resulting static MOTD (example):

    You are connecting to the computer system 'srvweb' at My Company.

    Any or all uses of this system and all files on this system may be
    intercepted, monitored, recorded, copied, audited, inspected,
    and disclosed to authorized corporation and law enforcement personnel,
    as well as authorized individuals of other organizations.
    By using this system, the user consents to such interception,
    monitoring, recording, copying, auditing, inspection,
    and disclosure at the discretion of authorized personnel.

    Unauthorized or improper use of this system may result in
    administrative disciplinary action, civil charges/criminal penalties,
    and/or other sanctions as according to the european codes and/or countries codes.

    LOG OFF IMMEDIATELY if you do not agree to the conditions stated in this warning.

    roles:
      - jonaspammer.motd

    vars:
      motd_type: static
      motd_static_motd_template: my-template

**templates/my-template.jinja2**

    {{ ansible_managed | comment }}
    Welcome to {{ ansible_host }}

# 📝 Development

[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg)](https://conventionalcommits.org) [![pre-commit.ci status](https://results.pre-commit.ci/badge/github/JonasPammer/ansible-role-motd/master.svg)](https://results.pre-commit.ci/latest/github/JonasPammer/ansible-role-motd/master)

## 📌 Development Machine Dependencies

- Python 3.8 or greater

- Docker

## 📌 Development Dependencies

Development Dependencies are defined in a [pip requirements file](https://pip.pypa.io/en/stable/user_guide/#requirements-files) named `requirements-dev.txt`. Example Installation Instructions for Linux are shown below:

    # "optional": create a python virtualenv and activate it for the current shell session
    $ python3 -m venv venv
    $ source venv/bin/activate

    $ python3 -m pip install -r requirements-dev.txt

## ℹ️ Ansible Role Development Guidelines

Please take a look at my [ Ansible Role Development Guidelines](https://github.com/JonasPammer/cookiecutter-ansible-role/blob/master/ROLE_DEVELOPMENT_GUIDELINES.adoc).

If interested, I’ve also written down some [ General Ansible Role Development (Best) Practices](https://github.com/JonasPammer/cookiecutter-ansible-role/blob/master/ROLE_DEVELOPMENT_TIPS.adoc).

## 🔢 Versioning

Versions are defined using [Tags](https://git-scm.com/book/en/v2/Git-Basics-Tagging), which in turn are [recognized and used](https://galaxy.ansible.com/docs/contributing/version.html) by Ansible Galaxy.

**Versions must not start with `v`.**

When a new tag is pushed, [ a GitHub CI workflow](https://github.com/JonasPammer/ansible-role-motd/actions/workflows/release-to-galaxy.yml) (![Release CI](https://github.com/JonasPammer/ansible-role-motd/actions/workflows/release-to-galaxy.yml/badge.svg)) takes care of importing the role to my Ansible Galaxy Account.

## 🧪 Testing

Automatic Tests are run on each Contribution using GitHub Workflows.

The Tests primarily resolve around running [Molecule](https://molecule.readthedocs.io/en/latest/) on a varying set of linux distributions and using various ansible versions, as detailed in [JonasPammer/ansible-roles](https://github.com/JonasPammer/ansible-roles).

The molecule test also includes a step which lints all ansible playbooks using [`ansible-lint`](https://github.com/ansible/ansible-lint#readme) to check for best practices and behaviour that could potentially be improved.

To run the tests, simply run `tox` on the command line. You can pass an optional environment variable to define the distribution of the Docker container that will be spun up by molecule:

    $ MOLECULE_DISTRO=centos7 tox

For a list of possible values fed to `MOLECULE_DISTRO`, take a look at the matrix defined in [.github/workflows/ci.yml](.github/workflows/ci.yml).

### 🐛 Debugging a Molecule Container

1.  Run your molecule tests with the option `MOLECULE_DESTROY=never`, e.g.:

        $ MOLECULE_DESTROY=never MOLECULE_DISTRO=ubuntu1604 tox -e py3-ansible-5
        ...
          TASK [ansible-role-pip : (redacted).] ************************
          failed: [instance-py3-ansible-5] => changed=false
        ...
         ___________________________________ summary ____________________________________
          pre-commit: commands succeeded
        ERROR:   py3-ansible-5: commands failed

2.  Find out the name of the molecule-provisioned docker container:

        $ docker ps
        30e9b8d59cdf   geerlingguy/docker-debian10-ansible:latest   "/lib/systemd/systemd"   8 minutes ago   Up 8 minutes                                                                                                    instance-py3-ansible-5

3.  Get into a bash Shell of the container, and do your debugging:

        $ docker exec -it 30e9b8d59cdf /bin/bash

        root@instance-py3-ansible-2:/#
        root@instance-py3-ansible-2:/# python3 --version
        Python 3.8.10
        root@instance-py3-ansible-2:/# ...

    If the failure you try to debug is part of `verify.yml` step and not the actual `converge.yml`, you may want to know that the output of ansible’s modules (`vars`), hosts (`hostvars`) and environment variables have been stored into files on both the provisioner and inside the docker machine under: \* `/var/tmp/vars.yml` \* `/var/tmp/hostvars.yml` \* `/var/tmp/environment.yml` `grep`, `cat` or transfer these as you wish!

    You may also want to know that the files mentioned in the admonition above are attached to the **GitHub CI Artifacts** of a given Workflow run.
    This allows one to check the difference between runs and thus help in debugging what caused the bit-rot or failure in general. image::https://user-images.githubusercontent.com/32995541/178442403-e15264ca-433a-4bc7-95db-cfadb573db3c.png\[\]

4.  After you finished your debugging, exit it and destroy the container:

        root@instance-py3-ansible-2:/# exit

        $ docker stop 30e9b8d59cdf

        $ docker container rm 30e9b8d59cdf
        or
        $ docker container prune

## 🧃 TIP: Containerized Ideal Development Environment

This Project offers a definition for a "1-Click Containerized Development Environment".

This Container even allow one to run docker containers inside of them (Docker-In-Docker, dind), allowing for molecule execution.

To use it:

1.  Ensure you fullfill the [ the System requirements of Visual Studio Code Development Containers](https://code.visualstudio.com/docs/remote/containers#_system-requirements), optionally following the _Installation_-Section of the linked page section.
    This includes: Installing Docker, Installing Visual Studio Code itself, and Installing the necessary Extension.

2.  Clone the project to your machine

3.  Open the folder of the repo in Visual Studio Code (_File - Open Folder…_).

4.  If you get a prompt at the lower right corner informing you about the presence of the devcontainer definition, you can press the accompanying button to enter it. **Otherwise,** you can also execute the Visual Studio Command `Remote-Containers: Open Folder in Container` yourself (_View - Command Palette_ → _type in the mentioned command_).

I recommend using `Remote-Containers: Rebuild Without Cache and Reopen in Container` once here and there as the devcontainer feature does have some problems recognizing changes made to its definition properly some times.

You may need to configure your host system to enable the container to use your SSH Keys.

The procedure is described [ in the official devcontainer docs under "Sharing Git credentials with your container"](https://code.visualstudio.com/docs/remote/containers#_sharing-git-credentials-with-your-container).

## 🍪 CookieCutter

This Project shall be kept in sync with [the CookieCutter it was originally templated from](https://github.com/JonasPammer/cookiecutter-ansible-role) using [cruft](https://github.com/cruft/cruft) (if possible) or manual alteration (if needed) to the best extend possible.

> ![Official Example Usage of `cruft update`](https://raw.githubusercontent.com/cruft/cruft/master/art/example_update.gif)

### 🕗 Changelog

When a new tag is pushed, an appropriate GitHub Release will be created by the Repository Maintainer to provide a proper human change log with a title and description.

## ℹ️ General Linting and Styling Conventions

General Linting and Styling Conventions are [**automatically** held up to Standards](https://stackoverflow.blog/2020/07/20/linters-arent-in-your-way-theyre-on-your-side/) by various [`pre-commit`](https://pre-commit.com/) hooks, at least to some extend.

Automatic Execution of pre-commit is done on each Contribution using [`pre-commit.ci`](https://pre-commit.ci/)[\*](#note_pre-commit-ci). Pull Requests even automatically get fixed by the same tool, at least by hooks that automatically alter files.

Not to confuse: Although some pre-commit hooks may be able to warn you about script-analyzed flaws in syntax or even code to some extend (for which reason pre-commit’s hooks are **part of** the test suite), pre-commit itself does not run any real Test Suites. For Information on Testing, see [🧪 Testing](#testing).

Nevertheless, I recommend you to integrate pre-commit into your local development workflow yourself.

This can be done by cd’ing into the directory of your cloned project and running `pre-commit install`. Doing so will make git run pre-commit checks on every commit you make, aborting the commit themselves if a hook alarm’ed.

You can also, for example, execute pre-commit’s hooks at any time by running `pre-commit run --all-files`.

# 💪 Contributing

[![Open in Visual Studio Code](https://img.shields.io/static/v1?logo=visualstudiocode&label=&message=Open%20in%20Visual%20Studio%20Code&labelColor=2c2c32&color=007acc&logoColor=007acc)](https://open.vscode.dev/JonasPammer/ansible-role-motd) ![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)

The following sections are generic in nature and are used to help new contributors. The actual "Development Documentation" of this project is found under [📝 Development](#development).

## 🤝 Preamble

First off, thank you for considering contributing to this Project.

Following these guidelines helps to communicate that you respect the time of the developers managing and developing this open source project. In return, they should reciprocate that respect in addressing your issue, assessing changes, and helping you finalize your pull requests.

## 🍪 CookieCutter

This Project owns many of its files to [the CookieCutter it was originally templated from](https://github.com/JonasPammer/cookiecutter-ansible-role).

Please check if the edit you have in mind is actually applicable to the template and if so make an appropriate change there instead. Your change may also be applicable partly to the template as well as partly to something specific to this project, in which case you would be creating multiple PRs.

## 💬 Conventional Commits

A casual contributor does not have to worry about following [_the spec_](https://github.com/JonasPammer/JonasPammer/blob/master/demystifying/conventional_commits.adoc) [_by definition_](https://www.conventionalcommits.org/en/v1.0.0/), as pull requests are being squash merged into one commit in the project. Only core contributors, i.e. those with rights to push to this project’s branches, must follow it (e.g. to allow for automatic version determination and changelog generation to work).

## 🚀 Getting Started

Contributions are made to this repo via Issues and Pull Requests (PRs). A few general guidelines that cover both:

- Search for existing Issues and PRs before creating your own.

- If you’ve never contributed before, see [ the first timer’s guide on Auth0’s blog](https://auth0.com/blog/a-first-timers-guide-to-an-open-source-project/) for resources and tips on how to get started.

### Issues

Issues should be used to report problems, request a new feature, or to discuss potential changes **before** a PR is created. When you [ create a new Issue](https://github.com/JonasPammer/ansible-role-motd/issues/new), a template will be loaded that will guide you through collecting and providing the information we need to investigate.

If you find an Issue that addresses the problem you’re having, please add your own reproduction information to the existing issue **rather than creating a new one**. Adding a [reaction](https://github.blog/2016-03-10-add-reactions-to-pull-requests-issues-and-comments/) can also help be indicating to our maintainers that a particular problem is affecting more than just the reporter.

### Pull Requests

PRs to this Project are always welcome and can be a quick way to get your fix or improvement slated for the next release. [In general](https://blog.ploeh.dk/2015/01/15/10-tips-for-better-pull-requests/), PRs should:

- Only fix/add the functionality in question **OR** address wide-spread whitespace/style issues, not both.

- Add unit or integration tests for fixed or changed functionality (if a test suite already exists).

- **Address a single concern**

- **Include documentation** in the repo

- Be accompanied by a complete Pull Request template (loaded automatically when a PR is created).

For changes that address core functionality or would require breaking changes (e.g. a major release), it’s best to open an Issue to discuss your proposal first.

In general, we follow the "fork-and-pull" Git workflow

1.  Fork the repository to your own Github account

2.  Clone the project to your machine

3.  Create a branch locally with a succinct but descriptive name

4.  Commit changes to the branch

5.  Following any formatting and testing guidelines specific to this repo

6.  Push changes to your fork

7.  Open a PR in our repository and follow the PR template so that we can efficiently review the changes.

# 🗒 Changelog

Please refer to the [Release Page of this Repository](https://github.com/JonasPammer/ansible-role-motd/releases) for a human changelog of the corresponding [Tags (Versions) of this Project](https://github.com/JonasPammer/ansible-role-motd/tags).

Note that this Project adheres to Semantic Versioning. Please report any accidental breaking changes of a minor version update.

# ⚖️ License

**[LICENSE](LICENSE)**

    MIT License

    Copyright (c) 2022, Jonas Pammer

    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in all
    copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    SOFTWARE.
