A few things to remember:
  - The static site files (everything under ``docs/`` so html/css/xml etc.) are in a separate repo [here](https://github.com/sibbs-gambling/sibbs-gambling.github.io) and excluded here (as per the .gitignore)
  - This repo contains all the raw content markdowns as well as ``TODO.md`` (plans for future posts)
  - The reason for this repo is to have a second copy (with version control) of the "master" files


How to re-generate the site after writing a new post:
  1. Write new post under a ``content/<whatever directory>/<new post>/<post name>.md``
  2. If <whatever directory> didn't exist yet you need to add it the ``config.toml`` as an entry under [[menu.main]]
  3.
