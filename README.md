## Security warning

**Make sure this repo does not contain any sensitive information (e.g.,
passwords).**  The repo is public,
[on github](https://github.com/acorg/condor-ansible).

Ansible sends its commands to remote machines via `ssh`, so you shouldn't
have any need to store sensitive information here.

## Using Ansible

Here are basic instructions on using [Ansible](http://www.ansible.com) to
update important local files on the cluster machines.

You will need to do this if you want to update any of

* dark-matter code
* BLAST databases
* BLAST executables

for an `htcondor` run. The above resources are stored locally on each
cluster machine. The idea is that you can make changes on `albertine` and
then push those changes to the other cluster machines using Ansible
commands (as below).

## Clone the `condor-ansible` repo

First, grab a copy of our Ansible config files

    $ git clone git@github.com:acorg/condor-ansible.git

The `ansible-playbook` and `ansible` commands shown below need to be run in
the `condor-ansible` directory that the above `git clone` command created
for you.

## ssh to albertine

You must make your changes on albertine, so log in there

    $ ssh albertine.antigenic-cartography.org

## Directories

Everything available to be changed and pushed to the other cluster machines
is under `/usr/local/dark-matter` on albertine (and on all the other
cluster machines)

    $ ls -l
    lrwxrwxrwx 1 terry terry    18 Oct 26 16:57 blast -> ncbi-blast-2.2.28+/
    drwxrwxr-x 2 terry ncbi  28672 Feb 10 17:06 blast-dbs/
    drwxrwxr-x 9 terry ncbi   4096 Feb 10 15:58 dark-matter/
    drwxr-xr-x 4 terry terry  4096 Mar 13  2013 ncbi-blast-2.2.28+/
    drwxrwxr-x 8 terry ncbi   4096 Oct 24 18:06 virtualenv/

## Updating dark matter sources on all other cluster machines

    $ ssh albertine.antigenic-cartography.org
    $ cd /usr/local/dark-matter/dark-matter
    $ git pull origin master
    $ ansible-playbook src.yml -i hosts

## Updating BLAST databases on all other cluster machines

    $ ssh albertine.antigenic-cartography.org
    $ cd /usr/local/dark-matter/blast-dbs
    $ ../blast/bin/makeblastdb -in xxx.fasta -dbtype nucl -title 'Some title' -out dbname -parse_seqids
    $ ansible-playbook blast-dbs.yml -i hosts

## Updating BLAST executables on all other cluster machines

    # Change the BLAST version in blast-executables.yml in this repo (and push it to master).
    $ ssh albertine.antigenic-cartography.org
    $ cd /usr/local/dark-matter
    $ rm blast
    # Unpack/unzip etc. your new BLAST source tree. Now make a symbolic link to the new dir.
    $ ln -s ncbi-blast-2.2.31+ blast
    $ ansible-playbook blast-executables.yml -i hosts

Note that this will not delete the old blast executables from the other hosts. It
will copy across the new executables and update the `blast` symbolic link, which is
all you really need. If you run the `all.yml` playbook (below), the old executables
will be removed as well.

## To update everything

Make any combination of the above changes, as needed, but don't bother
running `ansible-playbook` yet. When you want to push it all to the other
htcondor machines:

    $ ansible-playbook all.yml -i hosts

## To run a one-off command on all htcondor machines:

The quotes around `date` in the following aren't needed, but you'll need
quotes if your command has more than a single word in it.

    $ ansible htcondor -a 'date' -i hosts
    i19 | success | rc=0 >>
    Tue Feb 10 23:39:52 GMT 2015

    i20 | success | rc=0 >>
    Tue Feb 10 23:37:07 GMT 2015

    i21 | success | rc=0 >>
    Tue Feb 10 23:36:49 GMT 2015

    i18 | success | rc=0 >>
    Tue Feb 10 23:39:54 GMT 2015

    o17 | success | rc=0 >>
    Tue Feb 10 23:44:30 GMT 2015

    o16 | success | rc=0 >>
    Tue Feb 10 23:44:35 GMT 2015

## Troubleshooting

The other cluster machines are listed in the `[htcondor_others]` section of the `hosts` file
in this repo. Currently, that's `o16`, `o17`, `i18`, `i20`, and `i21`.

If you're having a problem, make sure you can `ssh` to all those
machines. Make sure you can run a remote command on them (e.g., `ssh o16
date`).

Try running a single command on all the remote machines (see above section).

You can add `-vvvv` to an `ansible` or `ansible-playbook` command to have
it show you what it's trying to do (usually sending commands to remote
machines using `ssh`).

Sometimes `ansible` hangs at the `GATHERING FACTS` stage. I (Terry) am not
sure how to fix that yet (perhaps via `ssh_options` in
`/etc/ansible/ansible.cfg`, though the `-t` and `-n` options both cause
Ansible to emit an error saying those options are unknown).  If you retry a
little later the hanging may have gone away - that's what I do, for now.

As a last resort, you could always
[read the Ansible docs](http://docs.ansible.com/). I find them a little
confusing - there seem to be so many ways to do the same thing, plus
there's quite a lot of terminology.
