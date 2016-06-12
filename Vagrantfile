# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'etc'

VAGRANTFILE_API_VERSION = "2"
ENV['VAGRANT_DEFAULT_PROVIDER'] ||= 'docker'
ENV['VAGRANT_NO_PARALLEL'] = 'yes'

ENV['NAMESPACE'] = 'example'
ENV['USERNAME'] = 'exampleuser'
ENV['UID'] = Etc.getpwnam("#{ENV['USER']}").uid.to_s
ENV['DOMAINNAME'] = 'example.com'
ENV['PHPFPMSERVER'] = 'php'
ENV['PORT'] = '8080'

ENV['MARIADB_ROOT_PASS'] = "#{ENV['USERNAME']}-dev-root"
ENV['MARIADB_USER'] = "#{ENV['USERNAME']}"
ENV['MARIADB_PASS'] = 'examplepass'
ENV['MARIADB_DATABASE'] = "exampledb"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.define "app" do |app|
    app.vm.provider "docker" do |d|
      d.build_dir = "app"
      d.build_args = ["-t", "local/#{ENV['NAMESPACE']}-app",
                      "--build-arg", "USERNAME=#{ENV['USERNAME']}",
                      "--build-arg", "UID=#{ENV['UID']}",]
      d.name = "#{ENV['NAMESPACE']}_app"
      d.create_args = ["--log-driver=journald"]
      d.remains_running = false
      d.vagrant_vagrantfile = "HostVagrantfile"
      d.volumes = ["/sys/fs/cgroup:/sys/fs/cgroup:ro"]
      if ENV.has_key?('VAGRANT_FORCE_VM')
        d.force_host_vm = true
      end
    end
    app.vm.synced_folder ".", "/vagrant", disabled: true
  end

  config.vm.define "db" do |db|
    db.vm.provider "docker" do |d|
      d.build_dir = "db"
      d.build_args = ["-t", "local/#{ENV['NAMESPACE']}-db",
                      "--build-arg", "MARIADB_ROOT_PASS=#{ENV['MARIADB_ROOT_PASS']}",
                      "--build-arg", "MARIADB_USER=#{ENV['MARIADB_USER']}",
                      "--build-arg", "MARIADB_PASS=#{ENV['MARIADB_PASS']}",
                      "--build-arg", "MARIADB_DATABASE=#{ENV['MARIADB_DATABASE']}",]
      d.name = "#{ENV['NAMESPACE']}_db"
      d.create_args = ["--log-driver=journald"]
      d.vagrant_vagrantfile = "HostVagrantfile"
      d.volumes = ["/sys/fs/cgroup:/sys/fs/cgroup:ro",]
      if ENV.has_key?('VAGRANT_FORCE_VM')
        d.force_host_vm = true
      end
    end
    db.vm.synced_folder ".", "/vagrant", disabled: true
  end

  config.vm.define "php" do |php|
    php.vm.provider "docker" do |d|
      d.build_dir = "php"
      d.build_args = ["-t", "local/#{ENV['NAMESPACE']}-php",
                      "--build-arg", "USERNAME=#{ENV['USERNAME']}",
                      "--build-arg", "UID=#{ENV['UID']}",]
      d.name = "#{ENV['NAMESPACE']}_php"
      d.link("#{ENV['NAMESPACE']}_db:db")
      d.create_args = ["--volumes-from", "#{ENV['NAMESPACE']}_app",
                       "--log-driver=journald",]
      d.vagrant_vagrantfile = "HostVagrantfile"
      d.volumes = ["/sys/fs/cgroup:/sys/fs/cgroup:ro"]
      if ENV.has_key?('VAGRANT_FORCE_VM')
        d.force_host_vm = true
      end
    end
    php.vm.synced_folder ".", "/vagrant", disabled: true
  end

  config.vm.define "httpd" do |httpd|
    httpd.vm.provider "docker" do |d|
      d.build_dir = "httpd"
      d.build_args = ["-t", "local/#{ENV['NAMESPACE']}-httpd",
                      "--build-arg", "USERNAME=#{ENV['USERNAME']}",
                      "--build-arg", "UID=#{ENV['UID']}",
                      "--build-arg", "DOMAINNAME=#{ENV['DOMAINNAME']}",
                      "--build-arg", "PHPFPMSERVER=#{ENV['PHPFPMSERVER']}",]
      d.name = "#{ENV['NAMESPACE']}_httpd"
      d.link("#{ENV['NAMESPACE']}_php:php")
      d.link("#{ENV['NAMESPACE']}_db:db")
      d.ports = ["8080:80", "4430:443"]
      d.create_args = ["--volumes-from", "#{ENV['NAMESPACE']}_app",
                       "--log-driver=journald",]
      d.vagrant_vagrantfile = "HostVagrantfile"
      d.volumes = ["/sys/fs/cgroup:/sys/fs/cgroup:ro"]
      d.remains_running = false
      if ENV.has_key?('VAGRANT_FORCE_VM')
        d.force_host_vm = true
      end
    end
    httpd.vm.synced_folder ".", "/vagrant", disabled: true
  end

  config.vm.define "drush" do |drush|
    drush.vm.provider "docker" do |d|
      d.build_dir = "drush"
      d.build_args = ["-t", "local/#{ENV['NAMESPACE']}-drush",
                      "--build-arg", "USERNAME=#{ENV['USERNAME']}",
                      "--build-arg", "UID=#{ENV['UID']}",
                      "--build-arg", "DOMAINNAME=#{ENV['DOMAINNAME']}",
                      "--build-arg", "PORT=8080",]
      d.name = "#{ENV['NAMESPACE']}_drush"
      d.link("#{ENV['NAMESPACE']}_db:db")
      d.create_args = ["--volumes-from", "#{ENV['NAMESPACE']}_app",
                      "--user", "#{ENV['UID']}"]
      d.remains_running = false
      d.vagrant_vagrantfile = "HostVagrantfile"
      if ENV.has_key?('VAGRANT_FORCE_VM')
        d.force_host_vm = true
      end
    end
    drush.vm.synced_folder ".", "/vagrant", disabled: true
    # This next line is used to aid in making the drush alias auto-complete in bash.
    #   @see https://github.com/drush-ops/drush/issues/2213
    drush.vm.hostname = "#{ENV['USERNAME']}"
  end

end
