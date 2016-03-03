# Web and Mobile App Development Club @ Virginia Tech

Website: http://wmad.cs.vt.edu/

## Want to make changes?

* Fork this repository
* Make changes in a branch
* Make a PR!

### Setup

1. Make sure ruby is on your system

A) Install a version of Ruby (if you have a 64 bit machine make sure you download the x64 one!) https://www.ruby-lang.org/en/documentation/installation/ 
B) Install a devkit (read the side notes they put- download the right one!) : http://rubyinstaller.org/downloads/
-Configuring your DevKit/binding it to your path:
" Quick start (https://github.com/oneclick/rubyinstaller/wiki/Development-Kit)
Download it, run it to extract it somewhere (permanent). 
Then (with the "Command Prompt with Ruby", you installed this in A):
cd to it, run ruby dk.rb init and ruby dk.rb install to bind it to ruby installations in your path."


Then with the Command Prompt with Ruby again do...

2. If you don't already have it, install bundler `gem install bundler`
3. In the project root, run `bundle install` to get dependencies for Jekyll
4. Start the development server with `bundle exec jekyll serve`


You should now be able to see the site running locally in a browser at http://localhost:4000

### Adding a new post

Take a look at the other posts on the site to see how they are formatted.

1. Create a new file in the `_posts` directory
2. Add the frontmatter (stuff at the top) in the correct format
3. Below that, add your Markdown post!

Commit changes to your local fork, push, then submit a PR!
