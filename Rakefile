require 'aws-sdk'
require 'yaml'
require 'pathname'
require 'dracula'
require 'yui/compressor'
require 'html_press'
require 'open-uri'

desc "Update analytics.js"
task :update_analytics do
  `curl http://google-analytics.com/analytics.js -o analytics.js`
end

desc "Generate the site to _output"
task :generate do
  `rm -rf _output`
  Dracula::Generator.new(File.dirname(__FILE__)).generate
end

desc "Compress HTML"
task :compress_html do
  output_directory = Pathname.new('_output')
  files = Dir["#{output_directory }/**/*"].reject { |file| File.directory?(file) }

  files.each do |file|
    if File.extname(file) == '.html'
      puts "Compressing #{file}..."
      contents = HtmlPress.press File.read(file)
      File.open(file, 'w') { |f| f.write contents }
    end
  end
end

desc "GZip html, css and javascript files"
task :gzip do
  output_directory = Pathname.new('_output')
  files = Dir["#{output_directory }/**/*"].reject { |file| File.directory?(file) }

  files.each do |file|
    if ['.html', '.css', '.js'].include? File.extname(file)
      puts "GZipping #{file}..."
      `gzip -9 #{file} && mv #{file}.gz #{file}`
    end
  end
end

desc "Upload to S3"
task :upload do
  load 'env.rb'

  service = AWS::S3.new(
    :access_key_id     => ENV['AWS_ACCESS_KEY_ID'],
    :secret_access_key => ENV['AWS_SECRET_ACCESS_KEY']
  )

  output_directory = Pathname.new('_output')
  files = Dir["#{output_directory }/**/*"].reject { |file| File.directory?(file) }
  bucket = service.buckets["shorts.jeffkreeftmeijer.com"]

  files.each do |file|
    pathname = Pathname.new(file).relative_path_from(output_directory)

    options = {:acl => :public_read}

    extname = File.extname(file)

    options[:content_type] = 'text/html' if extname == '.html'
    options[:content_type] = 'text/css' if extname == '.css'
    options[:content_type] = 'text/javascript' if extname == '.js'

    if ['.js', '.ico'].include? extname
      options[:cache_control] = 'max-age=2592000'
    end

    if ['.html', '.css', '.js'].include? File.extname(file)
      options[:content_encoding] = 'gzip'
    end

    if pathname.to_s == 'feed.xml'
      options[:website_redirect_location] = 'http://feedpress.me/shorts'
    end

    if pathname.to_s == '2015/automatically_remove_magic_encoding_comments/index.html'
      options[:website_redirect_location] = '/2014/automatically_remove_magic_encoding_comments'
    end

    if pathname.to_s == '2015/glob_a_directory_but_exclude_files_listed_in_gitignore/index.html'
      options[:website_redirect_location] = '/2014/glob_a_directory_but_exclude_files_listed_in_gitignore'
    end

    if pathname.to_s == '2015/check_if_a_model_instance_is_destroyed/index.html'
      options[:website_redirect_location] = '/2014/check_if_a_model_instance_is_destroyed'
    end

    if pathname.to_s == '2015/taking_screenshots_without_status_bars_with_uiautomation/index.html'
      options[:website_redirect_location] = '/2014/taking_screenshots_without_status_bars_with_uiautomation'
    end

    puts "Uploading #{pathname} with options: #{options}..."
    bucket.objects[pathname].write(File.read(file), options)
  end
end

desc "Generate and upload to S3"
task update: [:update_analytics, :generate, :compress_html, :gzip, :upload]
