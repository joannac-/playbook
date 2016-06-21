# WiredTiger Support Playbook

The goal of the support playbook is to spread knowledge about how to support
MongoDB running WiredTiger. 

We're going to try to work towards a common format where each playbook entry is
a separate wiki page describing a class of problem with:

-   symptoms

-   background

-   diagnosis and remediation

Types of issues to cover:

-   Performance issues (tools e.g: `timeseries`)

-   Cache usage problems

-   Machine configuration problems, identifying them and outlining solutions

-   Crashes that involve a stack trace (search JIRA, can we have a protocol to
    make tickets tracking crashes reliably searchable)

-   Issues that mean recovery can’t startup a `mongod` (e.g., checksum
    mismatches)

Process:

-   Use JIRA to find common / important cases and use them to drive the doc

-   Add a “playbook” label to JIRA tickets that have relevant information

-   Add a line to the [tracking page](https://docs.google.com/spreadsheets/d/1IgTm7De5FKFp_xhu_0BUztkRFGsTejc36ezLRjCfb-U/edit#gid=2061226700)

-   Add a developer-only comment with notes about why / what should go in the
    playbook

-   Ultimate output will be on the wiki, arranged so that relevant playbook
    pages turn up in searches
