# Hledger Flow

# What is it?

`hledger-flow` is a command-line executable program that gives you a
guided hledger workflow. It focuses on automated processing of
electronic statements as much as possible, as opposed to manually adding
your own hledger journal entries.

Manual entries are still possible, we just think it saves time in the
long run to automatically process a statement whenever one is available.

It started when I realized that the scripts I wrote while playing around
with the ideas in [adept's Full-fledged Hledger](https://github.com/adept/full-fledged-hledger/wiki) isn't
really specific to my own finances, and can be shared.

# Overview of the Basic Workflow

1.  Save an input CSV file to a [specific directory](https://github.com/apauley/hledger-flow#input-files).
2.  Add an hledger [rules file](https://github.com/apauley/hledger-flow#rules-files).
    Include some classification rules if you want.
3.  Run `hledger-flow import`

Add all your files to your favourite version control system.

The generated journal that you most likely want to use as your
`LEDGER_FILE` is called `all-years.journal`. This has include directives
to all the automatically imported journals, as well as includes for your
own manually managed journal entries.

In a typical software project we don't add generated files to version
control, but in this case I think it is a good idea to add all the
generated files to version control as well - when you inevitably change
something, e.g. how you classify transactions in your rules file, then
you can easily see if your change had the desired effect by looking at a
diff.

# Who should use this?

`hledger-flow` is intended for you if:

  - You are interested in getting started with
    [hledger](http://hledger.org/) and you wouldn't mind pointers to the
    right docs along the way.
  - You want a way to organise your finances into a structure that will
    be maintainable over the long term.
  - You want to automate as much as possible when dealing with your
    financial life.
  - You don't mind writing some scripts when needed, as long as it saves
    you time over the long term.
  - You want the ability to model your entire financial life in one
    tool, as opposed to just the parts that some online tool currently
    supports.
  - You appreciate the fact that all your financial information stays
    within your control.

# How Stable is it?

We're not close to a 1.0 release yet, which means that we can still make
changes if needed.

That being said, some parts have been used and tested extensively and
are likely to remain stable. Have a look at the "Stability of this
Feature" sections in the feature reference below.

I add future work, ideas and thoughts as [Github
issues](https://github.com/apauley/hledger-flow/issues) and in
[TODO.org](TODO.org), so have a look there for more clues as to what may
likely change.

Let me know if you can think of some improvements.

# Detailed Step-By-Step Guide

Have a look at the [detailed step-by-step instructions](docs/README.org)
and the files in the [documentation directory](docs/).

For a visual overview, check out the slide show version of the same
step-by-step instructions:

<https://pauley.org.za/hledger-flow/>

You can see the example imported financial transactions as it was
generated by the step-by-step instructions here:

<https://github.com/apauley/hledger-flow-example-finances>

# After Cloning This Repository

This repository has some submodules included, mostly related to the
examples in the documentation.

You need to initialise and update the submodules:

``` bash
git submodule init
git submodule update
```

# Build Instructions

[![CircleCI](https://circleci.com/gh/apauley/hledger-flow.svg?style=svg)](https://circleci.com/gh/apauley/hledger-flow) [![Build Status](https://travis-ci.com/apauley/hledger-flow.svg?branch=master)](https://travis-ci.com/apauley/hledger-flow)

You need a recent version of
[stack](https://docs.haskellstack.org/en/stable/README/) installed.

Then run:

``` bash
stack test
stack install
```

Which should end with this:

``` org
Copied executables to ~/.local/bin:
- hledger-flow
```

Ensure that `${HOME}/.local/bin` is in your `PATH`.

Usually this means adding this to your `~/.bashrc`:

``` bash
PATH="${HOME}/.local/bin:${PATH}"
```

## Building with older Haskell Versions

To build using an older version of GHC and related dependencies, point
stack to one of the other yaml files:

``` bash
stack test --stack-yaml stack-8.4.4.yaml
stack test --stack-yaml stack-8.2.2.yaml
```

# Feature Reference

## Input Files

Your input files will probably be CSV files with a line for each
transaction, although other file types will work fine if you use a
`preprocess` or a `construct` script that can read them. These scripts
are explained later.

We mostly use conventions based on a predefined directory structure for
your input statements.

For example, assuming you have a `savings` account at `mybank`, you'll
put your first CSV statement here:
`import/john/mybank/savings/1-in/2018/123456789_2018-06-30.csv`.

Some people may want to include accounts belonging to their spouse as
part of the household finances:
`import/spouse/otherbank/checking/1-in/2018/987654321_2018-06-30.csv`.

### More About Input Files

All files and directories under the `import` directory is related to the
automatic importing and classification of transactions.

The directory directly under `import` is meant to indicate the owner or
custodian of the accounts below it. It mostly has an impact on
reporting. You may want to have separate reports for `import/mycompany`
and `import/personal`.

Below the directory for the owner we can indicate where an account is
held. For a bank account you may choose to name it `import/john/mybank`.

If your underground bunker filled with gold has CSV statements linked to
it, then you can absolutely create `import/john/secret-treasure-room`.

Under the directory for the financial institution, you'll have a
directory for each account at that institution, e.g.
`import/mycompany/bigbankinc/customer-deposits` and
`import/mycompany/bigbankinc/expense-account`.

Next you'll create a directory named `1-in`. This is to distinguish it
from `2-preprocessed` and `3-journal` which will be auto-generated
later.

Under `1-in` you'll create a directory for the year, e.g. `2018`, and
within that you can copy the statements for that year:
`import/john/mybank/savings/1-in/2018/123456789_2018-06-30.csv`

### Stability of this Feature

The basic owner/bank/account/year structure has been used and tested
fairly extensively, I don't expect a need for it to change.

I'm open to suggestions for improvement though.

## Rules Files

If your input file is in CSV format, or converted to CSV by your
`preprocess` script, then you'll need an [hledger rules
file](http://hledger.org/csv.html).

`hledger-flow` will try to find a rules file for each statement in a
few places. The same rules file is typically used for all statements of
a specific account, or even for all accounts of the same specific bank.

  - A global rules file for any `mybank` statement can be saved here:
    `import/mybank.rules`
  - A rules file for all statements of a specific account:
    `import/spouse/bigbankinc/savings/bigbankinc-savings.rules`

### Statement-specific Rules Files

What happens if some of the statements for an account has a different
format than the others?

This can happen if you normally get your statements directly from your
bank, but some statements you had to download from somewhere else, like
Mint, because your bank is being daft with older statements.

In order to tell `hledger-flow` that you want to override the rules
file for a specific statement, you need to add a suffix, separated by an
underscore (`_`) and starting with the letters `rfo` (rules file
override) to the filename of that statement.

For example: assuming you've named your statement
`99966633_20171223_1844_rfo-mint.csv`.

`hledger-flow` will look for a rules file named `rfo-mint.rules` in
the following places:

  - in the import directory, e.g. `import/rfo-mint.rules`
  - in the bank directory, e.g. `import/john/mybank/rfo-mint.rules`
  - in the account directory, e.g.
    `import/john/mybank/savings/rfo-mint.rules`

### Example rules file usage

A common scenario is multiple accounts that share the same file format,
but have different `account1` directives.

One possible approach would be to include a shared rules file in your
account-specific rules file.

If you are lucky enough that all statements at `mybank` share a common
format across all accounts, then you can `include` a rules file that
just defines the parts that are shared across accounts.

Two accounts at `mybank` may have rules files similar to these.

A checking account at mybank:

``` hledger
# Saved as: import/john/mybank/checking/mybank-checking.rules
include ../../../mybank-shared.rules
account1 Assets:Current:John:MyBank:Checking
```

Another account at mybank:

``` hledger
# Saved as: import/alice/mybank/savings/mybank-savings.rules
include ../../../mybank-shared.rules
account1 Assets:Current:Alice:MyBank:Savings
```

Where `import/mybank-shared.rules` may define some shared attributes:

``` hledger
skip 1

fields date, description, amount, balance

date-format %Y-%m-%d
currency $
```

Another possible approach could be to use your `preprocess` script to
write out a CSV file that has extra fields for `account1` and
`account2`.

You could then create the above mentioned global `import/mybank.rules`
with the fields defined more or less like this:

``` hledger
fields date, description, amount, balance, account1, account2
```

### Stability of this Feature

Rules files are a stable feature within [hledger](http://hledger.org/),
and we're just using the normal hledger rules files. The account, bank
and statement-specific rules files have been used and tested fairly
extensively, I don't expect this to change.

Let me know if you think it should change.

## Opening and Closing Balances

### Opening Balances

`hledger-flow` looks for a file named `YEAR-opening.journal` in each
account directory, where `YEAR` corresponds to an actual year directory,
eg. **1983** (if you have electronic statements [dating back
to 1983](https://en.wikipedia.org/wiki/Online_banking#First_online_banking_services_in_the_United_States)).
Example: `import/john/mybank/savings/1983-opening.journal`

If it exists the file will automatically be included at the beginning of
the generated journal include file for that year.

You need to edit this file for each account to specify the opening
balance at the date of the first available transaction.

An opening balance may look something like this:

``` hledger
2018-06-01 Savings Account Opening Balance
assets:Current:MyBank:Savings               $102.01
equity:Opening Balances:MyBank:Savings
```

### Closing Balances

Similar to opening balances, `hledger-flow` looks for an optional
file named `YEAR-closing.journal` in each account directory. Example:
`import/john/mybank/savings/1983-closing.journal`

If it exists the file will automatically be included at the end of the
generated journal include file for that year.

A closing balance may look something like this:

``` hledger
2018-06-01 Savings Account Closing Balance
assets:Current:MyBank:Savings               $-234.56 = $0.00
equity:Closing Balances:MyBank:Savings
```

### Example Opening and Closing Journal Files

As an example, assuming that the relevant year is `2019` and
`hledger-flow` is about to generate
`import/john/mybank/savings/2019-include.journal`, then one or both of
the following files will be added to the include file if they exist:

1.  `import/john/mybank/savings/2019-opening.journal`
2.  `import/john/mybank/savings/2019-closing.journal`

The `opening.journal` will be included just before the other included
entries, while the `closing.journal` will be included just after the
other entries in that include file.

An include file may look like this:

``` bash
cat import/john/mybank/savings/2019-include.journal
```

``` hledger
### Generated by hledger-flow - DO NOT EDIT ###

!include 2019-opening.journal
!include 3-journal/2019/123456789_2019-01-30
!include 2019-closing.journal
```

### Stability of this Feature

The opening balances file works well in my opinion, I don't expect it to
change. I'm only using closing balances in one or two places, so maybe
that could do with some suggestions from people who use this more than
myself.

## The `preprocess` Script

Sometimes the statements you get from your bank is [less than
suitable](https://github.com/apauley/fnb-csv-demoronizer) for automatic
processing. Or maybe you just want to make it easier for the hledger
rules file to do its thing by adding some useful columns.

If you put a script called `preprocess` in the account directory, e.g.
`import/john/mybank/savings/preprocess`, then `hledger-flow` will
call that script for each input statement.

The `preprocess` script will be called with 4 positional parameters:

1.  The path to the input statement, e.g.
    `import/john/mybank/savings/1-in/2018/123456789_2018-06-30.csv`
2.  The path to an output file that can be sent to `hledger`, e.g.
    `import/john/mybank/savings/2-preprocessed/2018/123456789_2018-06-30.csv`
3.  The name of the bank, e.g. `mybank`
4.  The name of the account, e.g. `savings`
5.  The name of the owner, e.g. `john`

Your `preprocess` script is expected to:

  - read the input file
  - write a new output file at the supplied path that works with your
    rules file
  - be idempotent. Running `preprocess` multiple times on the same files
    will produce the same result.

### Stability of this Feature

Stable and tested.

## The `construct` Script

If you need even more power and flexibility than what you can get from
the `preprocess` script and `hledger`'s CSV import functionality, then
you can create your own custom script to `construct` transactions
exactly as you need them.

At the expense of more construction work for you, of course.

As an example, `hledger`'s CSV import currently [only supports two
postings per
transaction](https://github.com/simonmichael/hledger/issues/627), even
though `hledger` itself is perfectly happy with transactions containing
more than two postings, e.g.:

``` hledger
2019-02-01 Mortgage Payment
Liabilities:Mortgage                                $1000.00
Expenses:Interest:Real Estate                         $833.33
Assets:Cash                                         -$1833.33
```

The `construct` script can be used in addition to the `preprocess`
script, or on it's own. But since the `construct` script is more
powerful than the `preprocess` script, you could tell your `construct`
script to do anything that the `preprocess` script would have done.

Save your `construct` script in the account directory, e.g.
`import/john/mybank/savings/construct`.

`hledger-flow` will call your `construct` script with 4 positional
parameters:

1.  The path to the input statement, e.g.
    `import/john/mybank/savings/1-in/2018/123456789_2018-06-30.csv`
2.  A "-" (indicating that output should be sent to `stdout`)
3.  The name of the bank, e.g. `mybank`
4.  The name of the account, e.g. `savings`
5.  The name of the owner, e.g. `john`

Your `construct` script is expected to:

  - read the input file
  - generate your own `hledger` journal transactions
  - be idempotent. Running `construct` multiple times on the same files
    should produce the same result.
  - send all output to `stdout`. `hledger-flow` will pipe your
    output into `hledger` which will format it and save it to an output
    file.

### Stability of this Feature

Stable and tested.

## Manually Managed Journals

Not every transaction in your life comes with CSV statements.

Sometimes you just need to add a transaction for that time you loaned a
friend some money.

`hledger-flow` looks for `pre-import` and `post-import` files
related to each generated include file as part of the import.

You can enter your own transactions manually into these files.

You can run `hledger-flow import --verbose` to see exactly which
files are being looked for.

As an example, assuming that the relevant year is `2019` and
`hledger-flow` is about to generate
`import/john/2019-include.journal`, then one or both of the following
files will be added to the include file if they exist:

1.  `import/john/_manual_/2019/pre-import.journal`
2.  `import/john/_manual_/2019/post-import.journal`

The `pre-import.journal` will be included just before the other included
entries, while the `post-import.journal` will be included just after the
other entries in that include file.

An include file may look like this:

``` bash
cat import/john/2019-include.journal
```

``` hledger
### Generated by hledger-flow - DO NOT EDIT ###

!include _manual_/2019/pre-import.journal
!include mybank/2019-include.journal
!include otherbank/2019-include.journal
!include _manual_/2019/post-import.journal
```

### Stability of this Feature

It works, but the naming of `_manual_` looks a bit weird. Should it be
changed?

# Compatibility with Ledger

When writing out the journal include files, `hledger-flow` sorts the
include statements by filename.

[Ledger](https://www.ledger-cli.org/) fails any balance assertions when
the transactions aren't included in chronological order.

An easy way around this is to name your input files so that March's
statement is listed before December's statement.

Another option is to add `--permissive` to any
[ledger](https://www.ledger-cli.org/) command.

So you should easily be able to use both `ledger` and `hledger` on these
journals.

# Project Goals

My `hledger` files started to collect a bunch of supporting code that
weren't really specific to my financial situation.

I want to extract and share as much as possible of that supporting code.

[Adept's](https://github.com/adept/full-fledged-hledger/wiki) goals also
resonated with me:

  - Tracking expenses should take as little time, effort and manual work
    as possible
  - Eventual consistency should be achievable: even if I can't record
    something precisely right now, maybe I would be able to do it later,
    so I should be able to leave things half-done and pick them up later
  - Ability to refactor is a must. I want to be able to go back and
    change the way I am doing things, with as little effort as possible
    and without fear of irrevocably breaking things.

I've given [a talk](https://pauley.org.za/functional-finance-hledger/)
at [Lambda Luminaries
Johannesburg](https://www.meetup.com/lambda-luminaries/events/qklkvpyxmbnb/)
featuring hledger and hledger-flow.

# FAQ

## How does `hledger-flow` differ from `Full-fledged Hledger`?

[Full-fledged Hledger](https://github.com/adept/full-fledged-hledger/wiki#full-fledged-hledger-tutorial)
is a brilliant system, and hledger-flow continues to learn much from
it.

It has great documentation that does an excellent job of not only
showing **how** things can be done, but also **why** it is such a great
idea.

hledger-flow can be seen as a specific implementation of the
Full-fledged Hledger system, with a few implementation details that are
different.

| Full-fledged Hledger                                                                                                                                                                                                       | Hledger Flow                                                                                                                                                                                                                             |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|                                                                                                                                                                                                                            |                                                                                                                                                                                                                                                    |
| FFH describes itself as a [tutorial](https://github.com/adept/full-fledged-hledger/wiki#full-fledged-hledger-tutorial) with helper scripts that you can start using and adapt to your needs.                               | I started by following the FFH tutorial, and changed bits and pieces over time to suit my needs. The "owner/bank/account" structure for example.                                                                                                   |
|                                                                                                                                                                                                                            |                                                                                                                                                                                                                                                    |
| FFH is more open-ended: you can start with the basic scripts and over time turn it into something that solves your needs exactly. But you'll also end up with more code that you need to maintain yourself.                | Hledger Flow is less open-ended. For example, you have to adopt the "owner/bank/account" structure precisely as specified. But this allows Hledger Flow to do more work for you.                                                                                   |
|                                                                                                                                                                                                                            |                                                                                                                                                                                                                                                    |
| Maintaining include files are currently part of the user's responsibility.                                                                                                                                                 | Hledger Flow generates flexible include files for you, and automatically includes opening/closing journals if the appropriately named files are present on disk.                                                                                           |
|                                                                                                                                                                                                                            |                                                                                                                                                                                                                                                    |
| FFH actually generates some useful reports right now. Hledger Flow still [plans to get this done](https://github.com/apauley/hledger-flow/pull/4) one day.                                                                     | The "owner/bank/account" structure may look a bit much at first, but it allows us to run separate queries/reports for me/my spouse/my business etc and also allows for reports covering all of it in a an overall view.                            |
|                                                                                                                                                                                                                            |                                                                                                                                                                                                                                                    |
| FFH chose scripts and build files that you can easily modify as you go along, but this requires a Haskell runtime to be installed everywhere it needs to run. The included docker image helps to make it less of an issue. | Hledger Flow distributes a compiled binary. This means users or deployment targets don't need extra dependencies installed, they can just run a CLI program. This also provides a clearer distinction between what is provided, and what users need to do. |
|                                                                                                                                                                                                                            |                                                                                                                                                                                                                                                    |
| The FFH build scripts requires familiarity with Haskell and the Shake build system.                                                                                                                                        | Users may need to write `preprocess` or `construct` hooks, but in a language of their choice.                                                                                                                                                      |
|                                                                                                                                                                                                                            |                                                                                                                                                                                                                                                    |
| Input files are assumed to always be CSV files.                                                                                                                                                                            | Hledger Flow de-empasises the need that input files must be in CSV format. Input files can be in any format that a `preprocess` or `construct` hook can read.                                                                                              |
