---
title: "StudySwipe Git Workflow"
tags: git studyswipe
---

I wrote this up in an answer on a Slack thread, so I figured I'd take a minute to polish it up a little and post it here so I can reference it in the future. This is the Git workflow that [Ben](https://twitter.com/benhakes) and I follow for our project [StudySwipe](https://apps.apple.com/us/app/studyswipe/id1470980976):

We work out of the same repo, which has a `master` branch, a `development` branch and then we each have our own branch for what we're working on:
```plain
'master' (what we publish on Github and to the AppStore)
    |
    |
    'development' (our main working branch prior to
      publishing to master. Each PR into master is
      a new version release following
      semantic versioning)
        |
        ––– 'dillon' (my branch to make PRs
             into development)
        |
        |
        ––– 'ben' (ben's branch same as mine)
        |
        |
        ––– Other feature branches (as needed)
```

We start by branching off of `development`. To do that, we first make sure that our local version of `development` is up to date:

```bash
# switch to the development branch
git checkout development
# pull anything the remote repo has that isn't here locally
git pull
```

Then we make the new branch (or checkout the existing one):
```bash
# if we need to create a branch
git checkout -b some-branch-name
# otherwise we don't need the -b flag
git checkout dillon
```

Then we do some work. This is the part that takes up the most time, but it is a small part of the Git workflow. We commit as regularly as we can with [meaningful commit messages](https://chris.beams.io/posts/git-commit/) of the changes and we try to group the code for a PR into features that are a part of a user story.
```bash
git commit -m "Made a change to the api for CoreDataController" -m "Explanation and reasoning behind the change..."
```

When a feature is complete, or some bugs are fixed, or whatever, we get ready to make a pull request. First, we make sure `development` is up to date:
```bash
git checkout development
git pull
```

Then we merge it into our branch. This gives a great opportunity to fix any merge conflicts between anything we've written and what was added since we last pulled. It is nice because it leaves things clean and conflict-free for anyone who is going to review the PR.
```bash
# switch back to our branch
git checkout some-branch-name
# merge development into it
git merge development
# cross our fingers that it just works
# but fix any merge conflicts that occur
# then push it up
git push

# if this branch hasn't been pushed before
# you might have to do something like
git push --set-upstream origin branch-name
# but git will tell you and even give you
# a command that you can just copy and paste
# if you need to.
```

Then we actually make a pull request into development. Depending on how things are going, we will sometimes review them before merging, but this project started as a hack-a-thon submission, so we didn't start with great review habits. Once the PR is merged, we start the loop over again.

The part that I don't have an exact handle on the flow of, because [Ben](https://twitter.com/benhakes) does it, is exactly how we get from `development` to `master` to the App Store. My guess is something like this (I'll confirm with [Ben](https://twitter.com/benhakes) when I have a chance):
```bash
# Make sure development is up to date
git checkout development
git pull
```
Then update the version numbers and build numbers and what not, in Xcode. Then pull request and merge into `master`:
```bash
# Make sure master is up to date
git checkout master
git pull
```
Once the latest stable version is merged into `master` and on the local machine, build and run the project in Xcode. Verify that it’s working and what not. Then, finally, archive it and push to the store.

I don't know that it is the best flow to follow but it seems to be working for us right now. At some point we may look into some sort of CICD that would build the archive and push it to the store for us when we merge into development, but that is quite a bit of overhead for a side project, especially because right now we're not even doing any testing. When/if we get there, [CircleCI](https://circleci.com/) seems like a really solid option. They have great documentation and they will even give you some free time on a machine if your project is open source.

Let me know if you have a better Git workflow, or suggestions on how to improve this one, or suggestions on CICD products or whatever.
