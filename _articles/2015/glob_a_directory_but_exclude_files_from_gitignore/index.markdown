A while back, I needed to list all files from a directory, but exclude everything in the `.gitignore` file. Here's the shortest and most easy to understand solution I could think of:

``` ruby
require 'rake/file_list'
Rake::FileList['**/*'].exclude(*File.read('.gitignore').split)
```

I don't really like depending on Rake for something like this, so if you know a simple way to do this without depending on `rake/file_list`, be sure to let me know!
