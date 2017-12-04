To locally compile and view the reference guide, run these commands:

$ npm install gitbook-cli

(npm will complain there is no package.json file, that can be ignored)

$ ./node_modules/gitbook-cli/bin/gitbook.js install

This installs the necessary plugins and the latest gitbook version

$ ./node_modules/gitbook-cli/bin/gitbook.js build

This builds the book into the _book folder

Alternatively, you can use:

$ ./node_modules/gitbook-cli/bin/gitbook.js serve

This starts a webserver at port 4000; it also monitors your .md files for
changes, rebuilds the book and refreshes your browser tab.