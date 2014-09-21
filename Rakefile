require 'aws-sdk'
require 'yaml'
require 'pathname'
require 'dracula'
require 'yui/compressor'
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
    options[:content_type] = 'text/html' if File.extname(file) == '.html'

    puts "Uploading #{pathname} with options: #{options}..."
    bucket.objects[pathname].write(File.read(file), options)
  end
end

desc "Generate and upload to S3"
task update: [:update_analytics, :generate, :upload]
