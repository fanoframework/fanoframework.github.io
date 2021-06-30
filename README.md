## Fano Web Application Framework Documentation

You can use the [editor on GitHub](https://github.com/fanowebframework/fanowebframework.github.io/edit/master/README.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Build site locally

Following steps explain how to build and preview site locally, before you commit and make pull request.

#### Install Jekyll

Follow [Jekyll installation manual](https://jekyllrb.com/docs/installation/).

#### Install dependencies

Run
```
$ bundle install
```
#### Serve site locally

Change active directory to this repository and run

```
$ bundle exec jekyll serve
```

Open browser and go to URL `http://127.0.0.1:4000/`

When you run command above and you get following error
```
Can't find gem bundler (>= 0.a) with executable bundle (Gem::GemNotFoundException)
```

This is [RubyGemas and Bundler bug](https://bundler.io/blog/2019/05/14/solutions-for-cant-find-gem-bundler-with-executable-bundle.html). To remedy this, you need to install Bundler exact version used in `Gemfile.lock`.

```
$ gem install bundler -v "$(grep -A 1 "BUNDLED WITH" Gemfile.lock | tail -n 1)"
```

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/fanowebframework/fanowebframework.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.


### Preview Local Sites

Make sure you have Ruby 2.4 installed and clone this repository.

```
$ gem install bundler
$ bundle install
$ bundle exec jekyll serve
```

You can view local sites by opening `http://127.0.0.1:4000` on your browser.

### Contributing

Fork this repository and create pull request.
