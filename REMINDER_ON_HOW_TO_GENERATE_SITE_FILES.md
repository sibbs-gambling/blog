A few things to remember:
  - The static site files (everything under ``docs/`` so html/css/xml etc.) are in a separate repo [here](https://github.com/sibbs-gambling/sibbs-gambling.github.io) and excluded in this repo (as per the .gitignore)
  - The repo you are in now contains all the raw content markdowns as well as ``TODO.md`` (plans for future posts)
  - The reason for this repo is to have a second copy (with version control) of the "master" files


How to re-generate the site after writing a new post:
  1. Write new post under a ``content/<whatever directory>/<new post>/<post name>.md``
  2. If ``<whatever directory>`` didn't exist yet you need to add it the ``config.toml`` as an entry under [[menu.main]]
  3. Double check that your post is ready to publish (by looking at local version ``hugo server -D``)
  4. ``cd`` to the ``blog_dot_star/`` directory
  5. Run ``hugo``
  6. ``cd`` to ``docs/``
  7. Add, commit, and push the changes to the static-site repo
  8. ``cd`` back to to the ``blog_dot_star/`` directory and add, commit, and push the changes to the content repo
  9. Confirm the changes are made in the gh-pages site: https://sibbs-gambling.github.io/
