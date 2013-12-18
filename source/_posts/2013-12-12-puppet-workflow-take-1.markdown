---
layout: post
title: "puppet workflow take 1"
date: 2013-12-12 01:13
comments: true
categories: 
---

When it comes to puppet development workflow, my colleagues and I have tried various solutions, and we are getting pretty close to something that we believe works for us.
<!-- more -->
In our environment, we have three groups with slightly different requirements.  The infrastructure group is deploying to file servers, dns servers, etc., and as one might expect, changes need to be carefully vetted before being pushed out.  The research support group is the testing ground.  They are constantly deploying new hardware, trying new versions of the OS and software, and stability frequently is less important than flexibility.  The instruction support group is a hybrid of the other two.  During the semester, there needs to be very few changes in order to ensure consistency in the labs.  But during the breaks, rapid development to update the repos and pull in new fixes, as well as to support any new hardware being deployed is the rule.  The last complication is that none of these environments is necessarily a strict subset of the others.  In some cases, infrastructure and research have a very devops-y feel, with research doing development, and then infrastructure hardening and preparing the changes for production. There are also components and implementations that only one or two of the groups will use.  It is the kind of code management that git is good at, if you use it properly, and we are still working on that.

We use git and dynamic environments in puppet.  It is a relatively common setup, except for the way we deal with hiera, although I do not believe there is a standard for that yet.  Our directories look roughly like this:

```
/etc/puppet
├── environments
│   ├── [environment]
│   │   ├── hieradata
│   │   │   ├── modules
│   │   │   ├── nodes
│   │   │   └── roles
│   │   └── modules
│   │       ├── apache
│   │       ├── apachepassenger
│   │       ├── autofs
│   │       ├── ccbp
|   |       ...            
```

The tree is managed with multiple git repos.  There is one for each environment (subdirectory of `environments`).  Each module either comes from its own repo or the forge.  Initially we had the entire hieradata subdirectory in a repo, but we found that our usage made it realy hard to maintain it that way.  Currently, `hieradata/nodes/` and `hieradata/roles` is part of the environment repo, and `hieradata/modules` is in a separate repo.  I will mention more about how we do hiera here later, but the latter subdirectory contains **site-specific** in-module hieradata overrides, and we found that it made more sense to split that data out so we could easily keep updates in sync with the module.  We manage all of this using a script that clones the environment, grafts in the hieradata, and then runs librarian-puppet, which checks out all the modules either from a git repo or the forge.

I will just point out a couple of things: most of the repos are shared among our three groups, with each of us managing our own branch.  The rest of the repos (e.g., the environment repo) are completely different repos.  We could have used branches in a single repo, but we do not think that the code bases will stay similar enough to warrant that.  The second point is that our hiera data is *in the environment*; other folks in the community separate the hiera data directories completely from the environment.  So rather than `/etc/puppet/environments/myenvironment/hieradata`, they would have `/etc/puppet/hieradata/myenvironment/'.  Six one way, one half dozen the other ... or is it?  

