require 'rake'
require 'rake/testtask'
require 'rake/rdoctask'

WYSIHAT_ROOT          = File.expand_path(File.dirname(__FILE__))
WYSIHAT_SRC_DIR       = File.join(WYSIHAT_ROOT, 'src')
WYSIHAT_DIST_DIR      = File.join(WYSIHAT_ROOT, 'dist')
WYSIHAT_DOC_DIR       = File.join(WYSIHAT_ROOT, 'doc')
WYSIHAT_TEST_DIR      = File.join(WYSIHAT_ROOT, 'test')
WYSIHAT_TEST_UNIT_DIR = File.join(WYSIHAT_TEST_DIR, 'unit')
WYSIHAT_TMP_DIR       = File.join(WYSIHAT_TEST_UNIT_DIR, 'tmp')

desc "Update git submodules"
task :update_submodules do
  system("git submodule init")
  system("git submodule update")
end

task :default => :dist

desc "Builds the distribution."
task :dist => ["sprocketize:prototype", "sprocketize:wysihat"]

task :watch do
  ENV['WATCH'] = "1"
  Rake::Task["sprocketize:wysihat"].invoke
end

namespace :sprocketize do
  task :dist_dir do
    FileUtils.mkdir_p(WYSIHAT_DIST_DIR)
  end

  task :wysihat => [:update_submodules, :dist_dir] do
    require File.join(WYSIHAT_ROOT, "vendor", "sprockets", "lib", "sprockets")
    require File.join(WYSIHAT_ROOT, "vendor", "sprockwatch", "lib", "sprockwatch")

    secretary = Sprockets::Secretary.new(
      :root         => File.join(WYSIHAT_ROOT, "src"),
      :load_path    => [WYSIHAT_SRC_DIR],
      :source_files => ["wysihat.js"]
    )

    save = lambda {
      secretary.concatenation.save_to(File.join(WYSIHAT_DIST_DIR, "wysihat.js"))
    }

    ENV['WATCH'] ? secretary.watch!(&save) : save.call
  end

  task :prototype => [:update_submodules, :dist_dir] do
    require File.join(WYSIHAT_ROOT, "vendor", "sprockets", "lib", "sprockets")

    prototype_root    = File.join(WYSIHAT_ROOT, "vendor", "prototype")
    prototype_src_dir = File.join(prototype_root, 'src')

    secretary = Sprockets::Secretary.new(
      :root         => File.join(prototype_root, "src"),
      :load_path    => [prototype_src_dir],
      :source_files => ["prototype.js"]
    )

    secretary.concatenation.save_to(File.join(WYSIHAT_DIST_DIR, "prototype.js"))
  end
end

desc "Empties the output directory and builds the documentation."
task :doc => 'doc:build'

namespace :doc do
  desc "Builds the documentation"
  task :build => [:update_submodules, :clean] do
    require File.join(WYSIHAT_ROOT, "vendor", "sprockets", "lib", "sprockets")
    require File.join(WYSIHAT_ROOT, "vendor", "pdoc", "lib", "pdoc")
    require 'tempfile'

    Tempfile.open("pdoc") do |temp|
      secretary = Sprockets::Secretary.new(
        :root         => File.join(WYSIHAT_ROOT, "src"),
        :load_path    => [WYSIHAT_SRC_DIR],
        :source_files => ["wysihat.js"],
        :strip_comments => false
      )

      secretary.concatenation.save_to(temp.path)
      PDoc::Runner.new(temp.path,
        :output => WYSIHAT_DOC_DIR
      ).run
    end
  end

  desc "Empties documentation directory"
  task :clean do
    rm_rf WYSIHAT_DOC_DIR
  end
end

desc "Builds the distribution, runs the JavaScript unit tests and collects their results."
task :test => ['test:build', 'test:run']

namespace :test do
  task :run do
    testcases        = ENV['TESTCASES']
    browsers_to_test = ENV['BROWSERS'] && ENV['BROWSERS'].split(',')
    tests_to_run     = ENV['TESTS'] && ENV['TESTS'].split(',')
    runner           = UnittestJS::WEBrickRunner::Runner.new(:test_dir => WYSIHAT_TMP_DIR)

    Dir[File.join(WYSIHAT_TMP_DIR, '*_test.html')].each do |file|
      file = File.basename(file)
      test = file.sub('_test.html', '')
      unless tests_to_run && !tests_to_run.include?(test)
        runner.add_test(file, testcases)
      end
    end

    UnittestJS::Browser::SUPPORTED.each do |browser|
      unless browsers_to_test && !browsers_to_test.include?(browser)
        runner.add_browser(browser.to_sym)
      end
    end

    trap('INT') { runner.teardown; exit }
    runner.run
  end

  task :build => [:clean, :dist] do
    require File.join(WYSIHAT_ROOT, "vendor", "unittest_js", "lib", "unittest_js")
    builder = UnittestJS::Builder::SuiteBuilder.new({
      :input_dir  => WYSIHAT_TEST_UNIT_DIR,
      :assets_dir => WYSIHAT_DIST_DIR
    })
    selected_tests = (ENV['TESTS'] || '').split(',')
    builder.collect(*selected_tests)
    builder.render
  end

  task :clean do
    require File.join(WYSIHAT_ROOT, "vendor", "unittest_js", "lib", "unittest_js")
    UnittestJS::Builder.empty_dir!(WYSIHAT_TMP_DIR)
  end
end
