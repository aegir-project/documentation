Aegir Documentation Project
===========================

This project aims to document the Aegir Hosting System. It uses mkdocs to build the site at http://docs.aegirproject.org. To get started contributing to this project, fork it on Github. Then install mkdocs and clone this repo:

    $ brew install python;               # For OSX users
    $ sudo aptitude install python-pip   # For Debian/Ubuntu users
    $ pip install mkdocs
    $ git clone https://github.com/aegir-project/aegir-docs.git
    $ cd aegir-docs
    $ git remote add sandbox https://github.com/<username>/aegir-docs.git
    $ mkdocs serve

Your local aegir-docs site should now be available for browsing: http://127.0.0.1:8000/. When you find a typo, an error, unclear or missing explanations or instructions, hit ctrl-c, to stop the server, and start editing. Find the page you’d like to edit; everything is in the docs/ directory. Make your changes, commit and push them, and start a pull request:

    $ git checkout -b fix_typo
    $ ls -p docs/
    index.md    install/    install.md  quick-start.md
    $ vim docs/quick-start.md               # Add/edit/remove whatever you see fit. Be bold!
    $ mkdocs build --clean; mkdocs serve    # Go check your changes. We’ll wait...
    $ git diff                              # Make sure there aren’t any unintended changes.
    diff --git a/docs/quick-start.md b/docs/quick-start.md
    index 88f9974..e69de29 100644
    ...
    $ git commit -am”Fixed typo.”           # Useful commit message are a good habit.
    $ git push sandbox fix_typo

Visit your fork on Github and start a Pull Request. After we accept a pull request or two, we’ll probably invite you to join the project team, and thus grant you commit rights to the main documentation project.

