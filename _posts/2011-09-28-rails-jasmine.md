---
title: Jasmine with Rails 3.1
layout: post
---

I have been writing a JavaScript intensive Rails 3.1 application recently and haven been using Jasmine to test my JavaScript.  When I started this application I was using the RC version of Rails 3.1 and was generating my assets based on [Jeff Dean's post](http://pivotallabs.com/users/jdean/blog/articles/1778-writing-and-running-jasmine-specs-with-rails-3-1-and-coffeescript) however with the release of 3.1 stable I found this method does not work any more.  So I had to change the jasmine_config.rb to make it work again.


Also because Rails is out of beta now so I am no longer calling the rake task, instead I am. Using the API that the rake task uses.


So to keep things brief here is what my jasmine.rb looks like (located in spec/javascript/support)

{% highlight ruby linenos %}
module Jasmine
  class Config

    def js_files(spec_filter = nil)
      generated_files_directory = File.expand_path("../../generated", __FILE__)
      rm_rf generated_files_directory, :secure => true
      precompile_app_assets
      compile_jasmine_javascripts

      # this is code from the original jasmine config js_files method - you could also just alias_method_chain it
      spec_files_to_include = spec_filter.nil? ? spec_files : match_files(spec_dir, [spec_filter])
      src_files.collect {|f| "/" + f } + helpers.collect {|f| File.join(spec_path, f) } + spec_files_to_include.collect {|f| File.join(spec_path, f) }
    end

    private

    # this method compiles all the same javascript files your app will
    def precompile_app_assets
      puts "Precompiling assets..."

      # make sure the Rails environment is loaded
      require 'config/environment'
      # ::Rake.application['environment'].invoke

    

      config = Rails.application.config
      env    = Rails.application.assets
      target = Rails.root.join("spec/javascripts/generated/assets")


      config.assets.precompile.each do |path|
        env.each_logical_path do |logical_path|
          if path.is_a?(Regexp)
            next unless path.match(logical_path)
          elsif path.is_a?(Proc)
            next unless path.call(logical_path)
          else
            next unless File.fnmatch(path.to_s, logical_path)
          end

          if asset = env.find_asset(logical_path)
            asset_path = config.assets.digest ? asset.digest_path : logical_path
            filename = target.join(asset_path)

            mkdir_p filename.dirname
            asset.write_to(filename)
            asset.write_to("#{filename}.gz") if filename.to_s =~ /\.(css|js)$/
          end
        end
      end
    end

    # this method compiles all of the spec files into js files that jasmine can run
    def compile_jasmine_javascripts
      puts "Compiling jasmine coffee scripts into javascript..."
      root = File.expand_path("../../../../spec/javascripts/coffee", __FILE__)
      destination_dir = File.expand_path("../../generated/specs", __FILE__)

      glob = File.expand_path("**/*.js.coffee", root)

      Dir.glob(glob).each do |srcfile|
        srcfile = Pathname.new(srcfile)
        destfile = srcfile.sub(root, destination_dir).sub(".coffee", "")
        FileUtils.mkdir_p(destfile.dirname)
        File.open(destfile, "w") {|f| f.write(CoffeeScript.compile(File.new(srcfile)))}
      end
    end

  end
end


# Note - this is necessary for rspec2, which has removed the backtrace
module Jasmine
  class SpecBuilder
    def declare_spec(parent, spec)
      me = self
      example_name = spec["name"]
      @spec_ids << spec["id"]
      backtrace = @example_locations[parent.description + " " + example_name]
      parent.it example_name, {} do
        me.report_spec(spec["id"])
      end
    end
  end
end

{% endhighlight %}


Here is an exert from jasmine.yml, also located in spec/javascript/support.


{% highlight yaml %}
src_files:
  - spec/javascripts/generated/assets/application*.js 

{% endhighlight %}


With these changes I have not had any problems with my jasmine suite running correctly. 
