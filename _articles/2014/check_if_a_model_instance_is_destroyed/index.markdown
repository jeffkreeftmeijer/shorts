Ever do something like this?

``` ruby
delete :destroy
Session.exist?(@session).should.be.false
```

Or this:

``` ruby
lambda {
  delete :destroy
}.should.change('Session.count' -1)
```

Or even this?

``` ruby
lambda { @session.reload }.should.raise(ActiveRecord::RecordNotFound)
```

I still see this sometimes. Please stop. Since [2009(!)](https://github.com/rails/rails/commit/7034272354ad41dd4d1c90138a79e7f14ebc2bed#diff-391caa9b9464021e932ebf657fa9b13cR2496), there's something called `ActiveRecord::Base#destroyed?`, which will allow you to ask the model instance if it was destroyed, so you don't have to check your database if the record is actually gone:

``` ruby
describe SessionsController do
  before do
    @session = sessions(:harry)
    cookies[:authorization] = @session.token
  end

  it "logs the user out" do
    delete :destroy
    assigns(:session).should.be.destroyed
  end
end
```

Be sure to test this on the same instance you called `#destroy` on, though. Calling `#destroyed?` on `@session` instead of `assigns(:session)` in the above example would return `false`. That's because `#destroy` and `#delete` actually set a `@destroyed` flag on the object, which is checked when you call `#destroyed?`.
