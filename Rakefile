# frozen_string_literal: true

require "bundler/gem_tasks"
require "rake/testtask"

Rake::TestTask.new(:test) do |t|
  t.libs << "test"
  t.test_files = FileList["test/**/*_test.rb"]
end

require "rubocop/rake_task"

RuboCop::RakeTask.new

task default: %i[test conformance rubocop]

desc "Run the conformance tests"
task conformance: %w[conformance:setup] do
  rm_rf ["/tmp/user_data_dir", "/tmp/user_cache_dir"]
  sh({ "GHA_SIGSTORE_CONFORMANCE_XFAIL" =>
       "test_verify_trust_root_with_invalid_ct_keys test_verify_dsse_bundle_with_trust_root" },
     "env/bin/pytest", "test",
     "--entrypoint=#{File.join(__dir__, "bin", "conformance-entrypoint")}", "--skip-signing",
     chdir: "test/sigstore-conformance")
end

namespace :conformance do
  file "test/sigstore-conformance/.git/config" do
    sh "git", "clone", "https://github.com/sigstore/sigstore-conformance", chdir: "test"
  end
  file "test/sigstore-conformance/.git/HEAD" => "test/sigstore-conformance/.git/config" do
    sh "git", "checkout", "36c89ee", chdir: "test/sigstore-conformance"
  end
  file "test/sigstore-conformance/version" => %w[test/sigstore-conformance/.git/HEAD] do
    sh "git", "describe", "--tags", "--always", chdir: "test/sigstore-conformance",
                                                out: "test/sigstore-conformance/version"
  end
  file "test/sigstore-conformance/env/pyvenv.cfg" => "test/sigstore-conformance/version" do
    sh "make", "dev", chdir: "test/sigstore-conformance"
  end
  task setup: "test/sigstore-conformance/env/pyvenv.cfg" # rubocop:disable Rake/Desc
end

# namespace :test do
#   task :generate do
#     FileList["lib/**/*.rb"].each do |file|
#       test = file.sub(/\.rb$/, "_test.rb").sub("lib", "test")
#       next if File.exist?(test)

#       mkdir_p File.dirname(test)

#       File.write(test, <<~RUBY)
#         # frozen_string_literal: true

#         require "test_helper"

#         class #{file.sub(/\.rb$/, "").split("/").drop(1).join("::")}Test < Test::Unit::TestCase
#           def test_something
#
#           end
#         end
#       RUBY
#     end
#   end
#
#
# end
