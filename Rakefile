require 'bundler'
Bundler.require

require 'yaml'

Sequel.extension :migration

DB = Sequel.connect(ENV['DATABASE_URL'])
Sequel::Migrator.run(DB, ::File.join(::File.dirname(__FILE__), 'lib/migrations'))

require './lib/models/session'

namespace :db do
  task :seed do
    Dir["data/*"].each do |directory|
      next unless File.directory? directory
      year = Integer(directory.split(/\//).last)

      YAML.load(File.open(File.join(directory, "_sessions.yml"))).each do |number, attributes|
        previous = nil

        filename = "data/#{year}/#{number}.vtt"
        next unless File.exist?(filename)

        session = Session.find(year: year, number: number) || Session.new(attributes)
        session.number = number
        session.year = year
        session.transcript = File.read(filename).lines.delete_if{|line|
          line == "\n" ||
          line[0] == "[" ||
          /^\d{2}\:\d{2}\:\d{2}\.\d{3}/ === line ||
          /^WEBVTT/ === line ||
          /^X-TIMESTAMP-MAP/ === line
        }.delete_if{|line|
          line == previous and previous = line
        }.collect{|line|
          line.gsub(/[\r\n]+/, " ").gsub(/(&gt\;|\-\-)/, "")
        }.join

        puts session
        session.save
      end
    end
  end
end
