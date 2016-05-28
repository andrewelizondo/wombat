require 'erb'

# Style tests. Rubocop and Foodcritic
namespace :packerize do
  desc 'Build Chef Server'
  task :chef_server do
    sh "cd packer && packer build -only=amazon-ebs chef-server.json | tee -a logs/ami-chef-server.log"
  end

  desc 'Build Delivery Server'
  task :delivery_server do
    sh "cd packer && packer build -only=amazon-ebs delivery-server.json | tee -a logs/ami-delivery-server.log"
  end

  desc 'Build Delivery Builder'
  task :delivery_builder do
    sh "cd packer && packer build -only=amazon-ebs delivery-builder.json | tee -a logs/ami-delivery-builder.log"
  end

  desc 'Build Workstation'
  task workstation: [:vendor] do
    sh "cd packer && packer build -only=amazon-ebs workstation.json | tee -a logs/ami-workstation.log"
  end

  desc 'Cleanup Vendor directory'
  task :cleanup_vendor do
    sh 'rm -rf packer/vendored-cookbooks/*'
  end

  task vendor: [:cleanup_vendor] do
    sh 'berks vendor -b packer/cookbooks/workstation/Berksfile packer/vendored-cookbooks'
  end
end


desc 'Build all AMIs'
task packerize: ['packerize:chef_server', 'packerize:delivery_server', 'packerize:delivery_builder', 'packerize:workstation']

namespace :terraform do
  desc 'Update AMIS'
  task :update_amis, :chef_server_ami, :delivery_server_ami, :workstation_ami do |t, args|
    chef_server = args[:chef_server_ami] || File.read('./packer/logs/ami-chef-server.log').split("\n").last.split(" ")[1]
    delivery = args[:delivery_server_ami] || File.read('./packer/logs/ami-delivery-server.log').split("\n").last.split(" ")[1]
    builder = args[:delivery_builder_ami] || File.read('./packer/logs/ami-delivery-builder.log').split("\n").last.split(" ")[1]
    workstation = args[:workstation_ami] || File.read('./packer/logs/ami-workstation.log').split("\n").last.split(" ")[1]
    fail "packer build logs not found, nor were image ids provided" unless chef_server && delivery && builder && workstation
    puts "Updating tfvars based on most recent packer logs"
    @chef_server_ami = chef_server
    @delivery_server_ami = delivery
    @delivery_builder_ami = builder
    @workstation_ami = workstation
    rendered_tfvars = ERB.new(File.read('terraform/templates/terraform.tfvars.erb')).result
    File.open('terraform/terraform.tfvars', "w") {|file| file.puts rendered_tfvars }
    puts "\n" + rendered_tfvars
  end
end
