---
title: Adding Rubocop to an existing Rails project
description: A simple guide on adding Rubocop to an existing (and big) Rails project, with a separate CI configuration for retroactive enforcement
date: 2024-10-18
permalink: add-rubocop
---

---

If you're trying to add Rubocop to an existing project, you might've already stumbled upon the `--auto-gen-config` option for rubocop. This option will generate a `rubocop_todo.yml` configuration, which will allow you to run Rubocop without enforcing all the rules at once, and you can enable them one by one as you resolve them.

This is great, but we wanted to enable all the rules right at the start, so our editors would report all the issues, but we also wanted to add rubocop to the CI pipeline, and we didn't want the pipeline to be failing until we fixed *all* of the issues.

So our solution was to create a regular rubocop configuration file that would be used by our editors, and a separate `.rubocop-ci.yml` file to be used by the CI pipeline. That second config would extend the first one and exclude all of the files in the codebase which have linting issues. This way, instead of fixing rule by rule, we can fix issues file by file.

So let's say you're working on some file where you're adding a new feature. Now it should be your responsibility to fix all the rubocop issues in that file, and remove it from the exclude list in the `.rubocop-ci.yml` file.

One issue with this is that there will probably be a lot of files which won't be touched for a long while, meaning they won't be fixed anytime soon. You could plan some sessions in which you will take a chunk of those files, and go through them to fix the issues.

You can also run `rubocop -a` to autofix some issues.

## Configuration setup

After you created the `.rubocop.yml` configuration file with the rules you want to use, you can create the `.rubocop-ci.yml` file with the following content:

```yaml
inherit_from: .rubocop.yml

AllCops:
  Exclude:
    - "path/to/file/with/linting/issues/**/*"
    - "another/path/to/file/with/linting/issues/**/*"
```

Now here's the fun part, generating the list of files with linting issues. Since rubocop doesn't provide a way to do this out of the box (at least that I know of), I used the following command to generate the list:

```sh
bundle exec rubocop -L | uniq | sed -e "s/\(.*\)/- '\1'/"
```

This will return a list of files with linting issues, which you can copy and paste into the `.rubocop-ci.yml` file.

Now you can run rubocop with this configuration in your CI pipeline, and it will only fail if you introduce new issues in the files that are not excluded.

```sh
bundle exec rubocop -c .rubocop-ci.yml
```
