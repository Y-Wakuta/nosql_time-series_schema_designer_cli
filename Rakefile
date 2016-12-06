# frozen_string_literal: true

require 'rspec/core/rake_task'
require 'yard'
require 'yard-thor'

# XXX: Patch OpenStruct for yard-thor
class OpenStruct
  def delete(name)
    delete_field name
  end
end

RSpec::Core::RakeTask.new(:spec)
YARD::Rake::YardocTask.new(:doc)
task :completions do
  require 'nose'
  require_relative './lib/nose_cli'

  commands = NoSE::CLI::NoSECLI.all_commands.to_a.sort_by(&:first)

  tmpl = File.read File.join(File.dirname(__FILE__),
                             'templates/completions.erb')
  ns = OpenStruct.new commands: commands
  puts ERB.new(tmpl, nil, '>').result(ns.instance_eval { binding })
end

task :man do
  require 'erb'
  require 'fakefs/safe'
  require 'fileutils'
  require 'ronn'
  require_relative './lib/nose/cli'

  # Write the generated Markdown to a fake file then process with ronn
  def output_man(markdown, name)
    FakeFS.activate!

    path = "#{name}.md"
    File.open(path, 'w') { |f| f.write markdown }
    doc = Ronn::Document.new(path)
    roff = doc.convert('roff')

    FakeFS.deactivate!

    # Output the generated man page
    FileUtils.mkdir_p 'man'
    File.open("man/#{name}.1", 'w') { |f| f.write roff }
  end

  # Generate the man page for the main command
  ns = OpenStruct.new commands: NoSE::CLI::NoSECLI.commands,
                      options: NoSE::CLI::NoSECLI.class_options
  tmpl = File.read File.join(File.dirname(__FILE__), 'templates/man.erb')
  out = ERB.new(tmpl, nil, '>').result(ns.instance_eval { binding })
  output_man out, 'nose'

  # Generate man pages for each subcommand
  NoSE::CLI::NoSECLI.commands.each_value do |command|
    ns = OpenStruct.new command: command
    tmpl = File.read File.join(File.dirname(__FILE__), 'templates/subman.erb')
    out = ERB.new(tmpl, nil, '>').result(ns.instance_eval { binding })

    output_man out, "nose-#{command.name}"
  end
end

task default: :spec
