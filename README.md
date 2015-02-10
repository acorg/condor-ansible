## Using ansible

Here are basic instructions on using [Ansible](http://www.ansible.com) to
update our dark-matter code, BLAST databases, and BLAST executables on the
cluster machines.

## Directories

Everything you need is under `/usr/local/dark-matter` on all cluster machines.

    $ ls -l
    lrwxrwxrwx 1 terry terry    18 Oct 26 16:57 blast -> ncbi-blast-2.2.28+/
    drwxrwxr-x 2 terry ncbi  28672 Feb 10 17:06 blast-dbs/
    drwxrwxr-x 9 terry ncbi   4096 Feb 10 15:58 dark-matter/
    drwxr-xr-x 4 terry terry  4096 Mar 13  2013 ncbi-blast-2.2.28+/
    drwxrwxr-x 8 terry ncbi   4096 Oct 24 18:06 virtualenv/

## To update the dark matter source on all htcondor machines

    $ ssh albertine
    $ cd /usr/local/dark-matter/dark-matter
    $ git pull origin master
    $ ansible-playbook src.yml -i hosts

## To update BLAST databases

    $ ssh albertine
    $ cd /usr/local/dark-matter/blast-dbs
    $ ../blast/bin/makeblastdb -in xxx.fasta -dbtype nucl -title 'Some title' -out dbname -parse_seqids
    $ ansible-playbook blast-dbs.yml -i hosts

## To update BLAST executables

    # Change the BLAST version in blast-executables.yml in this repo (and push it to master).
    $ ssh albertine
    $ cd /usr/local/dark-matter
    $ rm blast
    # Unpack/unzip etc. your new BLAST source tree. Now make a symbolic link to the new dir.
    $ ln -s ncbi-blast-2.2.28+ blast
    $ ansible-playbook blast-executables.yml -i hosts

## To update everything

Make any combination of the above changes, as needed, but don't run `ansible-playbook`. Then when
you want to push it all to the other htcondor machines:

    $ ansible-playbook blast-executables.yml -i hosts

## To run a one-off command (`date` in this case) on all htcondor machines:

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
