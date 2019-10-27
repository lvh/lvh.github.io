.. title: Switched to Nikola
.. slug: switched-to-nikola
.. date: 2014-07-28 23:46:16 UTC-07:00
.. tags:
.. link:
.. description:
.. type: text

I've migrated from Octopress to Nikola.

Nothing personal. Octopress is fine software, but:

- External packages, like themes, made my installation quickly
  impossible to understand and manage. What's the difference between
  ``javascripts`` and ``js``, ``css`` and ``style``? I don't know.
- Performance. Nikola builds sites nearly instantly. Even with a
  moderate amount of pages, I found Octopress too slow. Maybe I just
  need a better laptop.
- Python. While I'm sure the Ruby tools are of similar quality to
  Python's; I *know* Python's tools. Porting to Nikola was easier
  than learning about A-grade Ruby developer installations.

Anyway, I'm on Nikola now. Pretty happy with it. I used
nikola-octopress-import_. Did most of what I wanted; I contributed
some minor PRs so that it would also handle extended date formats
(with seconds and timezones) and legacy HTML posts.

.. _nikola-octopress-import: https://www.github.com/mikemccracken/nikola-octopress-import
