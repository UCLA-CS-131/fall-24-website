# CS 131 Course Website - Fall 2024

Hey there! This is the source code for the Fall 2024 CS 131 course website. It is built with:

- [Jekyll](https://jekyllrb.com/), a [Ruby](https://www.ruby-lang.org/en/)-based static site generator
- [just-the-docs](https://just-the-docs.github.io/just-the-docs/), a Jekyll theme providing the base styling and structure
- [just-the-class](https://kevinl.info/just-the-class/), a template that extends just-the-docs for class-specific features

## Development Setup

This project follows general Ruby conventions. We highly suggest you use [rbenv](https://github.com/rbenv/rbenv) to manage your Ruby environment.

First, clone the repository.

```sh
git clone https://github.com/UCLA-CS-131/fall-24-website.git
# or, with SSH
git clone git@github.com:UCLA-CS-131/fall-24-website.git
```

Then, go into the folder, and install the relevant dependencies with bundler:

```sh
$ cd fall-24-website
$ bundle
```

Finally, serve the site:

```sh
$ bundle exec jekyll serve
```

## Licensing and Attribution

This site is distributed under the [MIT License](https://github.com/UCLA-CS-131/fall-22/blob/main/LICENSE) with the notice deriving from [Kevin Lin](https://kevinl.info/)'s work on [just-the-class](https://kevinl.info/just-the-class/).

Matt Wang, a major author of this code, is also a maintainer for [just-the-docs](https://github.com/just-the-docs/just-the-docs).

Have you used this code? We'd love to hear from you! [Submit an issue](https://github.com/UCLA-CS-131/fall-22/issues).
