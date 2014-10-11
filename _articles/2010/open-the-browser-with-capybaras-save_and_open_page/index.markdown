When using [Capybara](https://github.com/jnicklas/capybara), sometimes you get an error you don't quite understand. 
Maybe there's a missing button or link you're trying to use, or the field you're trying to type in doesn't exist.

Sometimes, it would be easy to just get a quick glance at the page at the moment your test fails. 
If you're using [Selenium](), you could add a `sleep`, but that probably wouldn't give you enough time to figure out what's going wrong.

Going through Capybara’s source last week, I found the `save_and_open_page` methoid, which saves the current page —complete with styling and images—  and opens it in your favorite browser:

``` ruby
it 'should register successfully' do
  visit registration_page
  save_and_open_page
  fill_in 'username', :with => 'jkreeftmeijer'
end
```
