## Building the Site

1. If you haven't already, install Jekyll (this might take a while):

    ```
    gem install jekyll
    ```

    The site uses Autoprefixer to handle the CSS, so you'll need that too:

    ```
    gem install octopress-autoprefixer
    ```

    If you're using Rbenv, don't forget to `rbenv rehash` after installation completes.

1. To build the site once, `jekyll build`. If you're working with a *draft*, `jekyll build --drafts`.
2. To make lots of changes to the site, `jekyll watch`.
