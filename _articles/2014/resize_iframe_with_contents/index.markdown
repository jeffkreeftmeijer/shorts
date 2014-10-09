<style>
  iframe{
    width: 100%;
    border: 1px solid #eee;
    height: 92px;
    margin: 1em 0;
    overflow: hidden;
  }
</style>

<script src="/zepto.min.js"></script>
<script>
  window.addEventListener('message', function(event) {
    if(height = event.data['height']) {
    console.log(height)
      $('iframe').css('height', height + 'px')
    }
  })
</script>

The simplest way I could find to [resize an iframe to match the size of its contents](https://gist.github.com/jeffkreeftmeijer/fbd572e06e6dbcbe7fbe) is using `window.postMessage()` to have the iframe send its height to `window.parent` when adding or removing content.

<iframe src="frame/index.html"></iframe>

*Please note*: this will work cross-origin, but only if you can run javascript in both the frame and the parent window.

In the iframe, there's a method named `sendHeight()`, that checks the height of the contents and posts it to the parent:

``` js
sendHeight = function(){
  height = $('div').height()
  window.parent.postMessage({"height": height}, "*")
}
```

That function gets called whenever something in the content changes, like adding or removing a new line item:

``` js
$(document).
  on('click', 'button.add', function(){
    $('ol').append("<li>I'm a line item in an automatically resizing iframe.</li>")
    sendHeight()
  }).
  on('click', 'button.remove', function(){
    $('ol li:last-child').remove()
    sendHeight()
  })
```

The iframe's width changes when resizing the parent, which causes lines to wrap and the content inside the iframe to expand vertically.
By listening for window resizes inside the iframe, we can send the new height as soon as it changes:

``` js
$(window).on('resize', sendHeight)
```

Finally, to actually resize the iframe element, there's an event listener on the parent page:

``` js
window.addEventListener('message', function(event) {
  if(height = event.data['height']) {
    $('iframe').css('height', height + 'px')
  }
})
```

Be sure to try out the demo above (don't forget to try resizing your browser window). Here's [the source](https://gist.github.com/jeffkreeftmeijer/fbd572e06e6dbcbe7fbe).
