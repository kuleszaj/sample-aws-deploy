require 'aws'
require 'yaml'
require 'logger'
require 'simple-cloudfront-invalidator'

log = Logger.new(STDOUT)
log.level = Logger::INFO

config = YAML.load_file("config.yml")
pwd = Dir.getwd

desc "Do all the things."
task :deploy => ["haml:generate","aws:deploy"]

namespace :haml do
  desc "Generate the HTML from HAML"
  task :generate do
    sh 'bundle exec haml src/index.haml > build/build.html'
    sh 'cp src/index.css build/index.css'
  end
end

namespace :aws do
  desc "Deploy to S3."
  task :s3 do

    s3 = Aws::S3.new(
      config[:aws][:access_key_id],
      config[:aws][:secret_access_key]
    )

    bucket = s3.bucket(config[:deploy][:s3_bucket],true)

    Dir.chdir(config[:deploy][:from_folder])

    log.info("Beginning to deploy files")

    Dir["**/*"].each do |file|
      log.debug("Uploading #{file}")
      bucket.put(file,File.read(file)) unless File.directory?(file)
    end

    log.info("Done!")

    Dir.chdir(pwd)
  end

  desc "Invalidate Cloudfront distribution."
  task :invalidate do

    acf = SimpleCloudfrontInvalidator::CloudfrontClient.new(
      config[:aws][:access_key_id],
      config[:aws][:secret_access_key],
      config[:acf][:distribution_id]
    )

    Dir.chdir(config[:deploy][:from_folder])

    log.info("Beginning invalidation request.")

    log.debug("Invalidating the following files:\n" + Dir['**/*'].join("\n"))

    acf.invalidate(Dir["**/*"])

    log.info("Done!")

    Dir.chdir(pwd)
  end

  desc "Deploy files to S3 and invalidate Cloudfront distribution"
  task :deploy => [:s3, :invalidate]

  task :default => :deploy
end
