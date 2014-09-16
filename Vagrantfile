# -*- mode: ruby -*-
# vi: set ft=ruby :

# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.


STRATOS_IP="192.168.56.5"

Vagrant.configure("2") do |config|

    # use the opscode vagrant box definitions because they have a 40Gb disk which should be enough for
    # stratos. ubuntu cloud images only have 10Gb which is not enough.

    # 64 bit machine
    config.vm.box = "chef/ubuntu-14.04"

    config.vm.hostname = "vagrant.stratos.com"

    # Use vagrant cachier if it is available - it will speed up repeated 
    # 'vagrant destroy' and 'vagrant up' calls
    if Vagrant.has_plugin?("vagrant-cachier")
      # monkey-patch / disable the :apt_lists bucket until this is fixed:
      # https://github.com/fgrehm/vagrant-cachier/issues/113
      module VagrantPlugins
        module Cachier
          class Bucket
            class AptLists < Bucket
              def self.capability
                :none
              end
            end
          end
        end
      end
      config.cache.scope = :box
    end

    config.vm.network :private_network, :ip => STRATOS_IP

    # apply provisioning script
    config.vm.provision "shell", inline: $setup_stratos_script, privileged: false
    config.vm.provision "shell", inline: $run_stratos_script, privileged: false

    # virtualbox customisations
    config.vm.provider "virtualbox" do |v|
      v.customize ["modifyvm", :id, "--memory", 4096 ]

      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]

      # uncomment these to use the virtualbox gui:
      # v.gui = true
      # v.customize ["modifyvm", :id, "--clipboard", "bidirectional"]
    end

    # If you want to customise the Vagrantfile just for your environment, try
    # putting your customisations in Vagrantfile.stratos.extensions
    # 
    # For example, I use it to join my LAN network when running vagrant on a server
    #   stratos.vm.network "public_network"
    begin
      eval(File.open("Vagrantfile.extensions").read)
      puts "Loaded Vagrantfile.extensions"
    rescue
      # do nothing
    end

end


$setup_stratos_script = <<SCRIPT

  set -x # debug output
  set -e # fail on error

  sudo apt-get install -y curl sed

  # install docker
  curl -s https://get.docker.io/ubuntu/ | sudo sh
  sudo ln -sf /usr/bin/docker.io /usr/local/bin/docker
  sudo sed -i '$acomplete -F _docker docker' /etc/bash_completion.d/docker

  sudo gpasswd -a ${USER} docker
  sudo service docker restart

  # fetch a script to run the docker instances
  wget "https://git-wip-us.apache.org/repos/asf?p=stratos.git;a=blob_plain;f=tools/stratos-docker-images/run-example.sh;hb=HEAD" -O /home/vagrant/run-example.sh

  # fetch a script to stop the stratos docker instances
  wget "https://git-wip-us.apache.org/repos/asf?p=stratos.git;a=blob_plain;f=tools/stratos-docker-images/stop_stratos_containers.sh;hb=HEAD" -O /home/vagrant/stop_stratos_containers.sh

  # fetch a script to stop the stratos docker instances
  wget "https://git-wip-us.apache.org/repos/asf?p=stratos.git;a=blob_plain;f=tools/stratos-docker-images/remove_stratos_images.sh;hb=HEAD" -O /home/vagrant/remove_stratos_images.sh

  # fetch a script to stop the stratos docker instances
  wget "https://git-wip-us.apache.org/repos/asf?p=stratos.git;a=blob_plain;f=tools/stratos-docker-images/run-nsenter.sh;hb=HEAD" -O /home/vagrant/run-nsenter.sh

SCRIPT

$run_stratos_script = <<SCRIPT

  # the scripts below are run with sudo because the vagrant user 
  # account does not have permission to run the scripts at this point

  # first make sure no stratos containers are already running
  sudo bash -x /home/vagrant/stop_stratos_containers.sh

  # now start the stratos containers
  sudo bash -x /home/vagrant/run-example.sh
SCRIPT
