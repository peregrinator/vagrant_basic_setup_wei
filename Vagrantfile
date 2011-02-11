require 'yaml'

def chef_cloud_attributes(instance_type)
  
  @app = {}
  @app[:name] = 'app'
  @app[:sever_port] = 80
  
  @rails_env = 'development'
  
  case instance_type
  when 'vagrant'
    @app[:url] = 'app.local'
  end
  
  return {
    :ec2 => false,
    :lsb => { :code_name  => 'lucid' },
    :bootstrap => {:chef => {:client_version => '0.9.12'}},
    :fqdn => @app[:url],
    :hostname => @app[:name],
    :chef => { :roles => ['vagrant', 'app', 'database', 'sphinx'] },
    :ubuntu => { :servers => [
                               {:ip => '127.0.0.1', :fqdn => @app[:url], :alias => @app[:name]},
                               {:ip => '127.0.1.1', :fqdn => 'vagrant-lucid64.vagrantup.com', :alias => 'vagrant-lucid64'},
                               {:ip => '127.0.0.1', :fqdn => "database.#{@app[:name]}.#{@app[:server_type]}.internal", :alias => 'database'},
                               {:ip => '127.0.0.1', :fqdn => "sphinx.#{@app[:name]}.#{@app[:server_type]}.internal",   :alias => 'sphinx'},
                               {:ip => '127.0.0.1', :fqdn => "app.#{@app[:name]}.#{@app[:server_type]}.internal",      :alias => 'app'}
                             ],
                  :database => { :fqdn => "database.#{@app[:name]}.#{@app[:server_type]}.internal" },
                  :mysql => true,
                  :users => { :accounts => {} }
               },
    :app => { :root_dir => "/var/www/apps/#{@app[:name]}", :app_name => @app[:name], :url => @app[:url] },
    :server => { :name => "#{@app[:name]}_#{@rails_env}" },
    :authorization => { :sudo => { :groups => ['admin'], :users => ['vagrant'] } },
    :ruby => { :version => 'ree' },
    :apache => {
                  :vhost_port     => @app[:server_port],
                  :vhosts         => [
                                        { :server_name    => @app[:url],
                                          :server_aliases => '',
                                          :docroot        => "/var/www/apps/#{@app[:name]}/public",
                                          :name           => @app[:url]
                                        }
                                     ], 
                  :server_name    => @app[:url],
                  :web_dir        => '/var/www',
                  :docroot        => "/var/www/apps/#{@app[:name]}/public",
                  :name           => @app[:name],
                  :enable_mods    => ["rewrite", "deflate", "expires"],
                  :prefork => {
                               :startservers        => 128,
                               :minspareservers     => 32,
                               :maxspareservers     => 128
                              }
               },
    :rails => {
                :environment  => @rails_env,
                :db_adapter   => 'mysql',
                :version      => '2.3.8',
                :using_shared => false,
                :app_root_dir => "/var/www/apps/#{@app[:name]}",
                :db_directory => "/var/www/apps/#{@app[:name]}"
              },
    :mysql  => {
                :bind_address           => "database.#{@app[:name]}.#{@app[:server_type]}.internal",
                :database_name          => "#{@app[:name]}_#{@rails_env}",
                :server_root_password   => '',
                :server_repl_password   => '',
                :server_debian_password => '',
                :tunable => {
                              :query_cache_size        => '40M',
                              :tmp_table_size          => '100M',
                              :max_heap_table_size     => '100M',
                              :innodb_buffer_pool_size => '1GB'
                            }
               },
     :passenger_enterprise => {
                                :pool_idle_time => 100000,
                                :max_requests   => 10000,
                                :max_pool_size  => 10
                              },
     :sphinx => {
                  :server_address => "sphinx.#{@app[:name]}.#{@app[:server_type]}.internal"
                }
  }        
end

Vagrant::Config.run do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "lucid64"


  # Assign this VM to a host only network IP, allowing you to access it
  # via the IP.
  config.vm.network "33.33.33.40"

  # Forward a port from the guest to the host, which allows for outside
  # computers to access the VM, whereas host only networking does not.
  # config.vm.forward_port "http", 80, 8080

  # Share an additional folder to the guest VM. The first argument is
  # an identifier, the second is the path on the guest to mount the
  # folder, and the third is the path on the host to the actual folder.
  config.vm.share_folder("app-data", "/var/www/apps/app", ".")

  # Enable provisioning with chef solo, specifying a cookbooks path (relative
  # to this Vagrantfile), and adding some recipes and/or roles.
  #
  config.vm.provisioner = :chef_solo
  config.chef.cookbooks_path = "vendor/cookbooks"

  recipes = ["apt",
             "ubuntu",
             "openssl",
             "mysql::server",
             "sphinx",
             "apache2",
             "passenger_enterprise::apache2",
             "rubygems",
             "git",
             "rails"]
  recipes.each do |recipe|
    config.chef.add_recipe(recipe)
  end
  
  # config.chef.add_role "web"

  # We need to pass our attributes for chef to set up properly
  config.chef.json.merge!( chef_cloud_attributes('vagrant') )

end
