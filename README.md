# Adding a new blog post

- Add new file to `_posts` directory with date prefix. 
- Create a new folder in `/assets/img/` with the same path as the post filename
- Add a new cover photo. Preferably from unsplash
- Add the cover photo to front matter

# Run local server
```
bundle exec jekyll serve
```

# Notes
This project was originally created with the following version of ruby:

```
# ruby --version
ruby 2.6.8p205 (2021-07-07 revision 67951) [universal.x86_64-darwin21]

# jekyll --version
jekyll 4.2.2
```

# TODO
- deploy blog through github actions on github pages
- have highligher perform code syntax highlighting during compilation (see: ruby rouge??)
- Fix paging on blog.html page
- search?
- tags?
- Move over some of the TOS/Privacy pages from vijaysharma.ca