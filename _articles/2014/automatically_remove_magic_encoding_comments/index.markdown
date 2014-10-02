Writing a Ruby script to remove Ruby 1.9's magic encoding comments is still faster for me than figuring out how to do it with Vim.

``` ruby
#!/usr/bin/env ruby
 
Dir['**/*.rb'].each do |path|
  lines = File.readlines(path)
  new_lines = lines.dup
 
  lines.each do |line|
    if line =~ /^# encoding: utf-8/i || line !~ /\S/
      new_lines.delete_at(new_lines.index(line))
    else
      break
    end
  end
 
  print '.'
 
  File.open(path, 'w') do |file|
    file.write(new_lines.join)
  end
end
```
