---
layout: default
title: "Facter 2.0: Frequently Asked Questions"
---

Frequently Asked Questions
==========================

### How can I make a new fact based on a shell command or script?

Facter's API makes it very easy to write new facts in Ruby that use the shell to do all the heavy lifting. Try modeling your fact on [this example](custom_facts.html#example-minimal-fact-that-relies-on-a-single-shell-command). If you'd rather not use Ruby at all, you can write an [external fact](custom_facts.html#external-facts) in the language of your choice --- just be aware that this approach has [drawbacks](custom_facts#external-facts).

### Where should I put custom facts?

The best practice is to put all of your custom facts into the appropriate module(s). More specifically, custom facts should go in `{moduledir}/{modulename}/lib/facter`. That way, the puppet master can use [pluginsync](/guides/plugins_in_modules) to distribute the facts to the agent nodes. Other options are discussed in the [custom facts page](custom_facts.html#loading-custom-facts).

### How can I make my fact run differently depending on the OS (or some other fact)?

You'll want to use `confine` statements to specify which resolutions go with which operating system. [This example](fact_overview.html#example-different-resolutions-for-different-operating-systems) should be enough to get you started. To see a great example of this from one of Facter's core facts, check out the source for the [`operatingsystem` fact](https://github.com/puppetlabs/facter/blob/master/lib/facter/operatingsystem.rb).
