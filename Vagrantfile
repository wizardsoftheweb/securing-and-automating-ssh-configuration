# -*- mode: ruby -*-
# vi: set ft=ruby :

unless Vagrant.has_plugin?('vagrant-triggers')
  `vagrant plugin install vagrant-triggers`
end

# TODO: pull these from ENV with defaults
# HYPER_V_SWITCH = 'NATWotwSsh'
# HYPER_V_SWITCH_IP_FIRST_THREE = '10.47.1'

HYPER_V_SWITCH = 'Default Switch'

Vagrant.configure('2') do |config|
  # Be careful; triggers are run for each machine
  # if Vagrant::Util::Platform.windows?
  #   [:up, :reload].each do |command|
  #     config.trigger.before command do
  #       run "powershell.exe powershell/network.ps1 -Switch #{HYPER_V_SWITCH} -LeadingThree #{HYPER_V_SWITCH_IP_FIRST_THREE} -Create"
  #     end
  #   end
  #   config.trigger.after :destroy do
  #     run "powershell.exe powershell/network.ps1 -Switch #{HYPER_V_SWITCH} -Destroy"
  #   end
  # end

  config.vm.box = 'centos/7'
  config.vm.network 'private_network', type: 'dhcp'

  config.vm.provider 'hyperv' do |hyperv|
    config.vm.network 'public_network', bridge: HYPER_V_SWITCH
  end

  # machines = ['trantor', 'terminus', 'kalgan']

  # machines.each do |machine|
  #   config.vm.define machine do |virtual_machine|
  #     virtual_machine.vm.hostname = machine

  #     virtual_machine.vm.provider 'virtualbox' do |virtualbox|
  #       virtualbox.name = machine
  #     end

  #     virtual_machine.vm.provider 'hyperv' do |hyperv|
  #       hyperv.vmname = machine
  #     end
  #   end
  # end

  config.vm.define 'controller' do |controller|
    controller.vm.hostname = 'generic'

    controller.vm.provider 'virtualbox' do |virtualbox|
      virtualbox.name = controller.vm.hostname
    end

    controller.vm.provider 'hyperv' do |hyperv|
      hyperv.vmname = controller.vm.hostname
    end

    controller.vm.provision "shell" do |sh|
      sh.inline = "yum list installed cifs-utils || yum install -y cifs-utils"
      sh.privileged = true
    end

    controller.vm.synced_folder "./provisioning/", "/tmp/provisioning", type: "cifs"

    # WARNING: DUPLICATE THIS IN YOUR ENVIRONMENT AT YOUR OWN RISK
    # If you're running on Windows, you have to virtualize ansible anyway, so no
    # harm there. Either way, I highly recommend you test it virtually to
    # understand how it works before applying it to your environment.
    # See https://www.vagrantup.com/docs/provisioning/ansible_local.html#install
    controller.vm.provision "ansible_local" do |ansible|
      ansible.playbook = "initial-setup.yml"
      ansible.install_mode = "pip"
      ansible.config_file = "/tmp/provisioning/ansible.cfg"
      ansible.provisioning_path = "/tmp/provisioning"
    end
    # END WARNING

    # This one's okay to duplicate
    controller.vm.provision "ansible" do |ansible|
      ansible.playbook = "main.yml"
      ansible.config_file = "/tmp/provisioning/ansible.cfg"
      ansible.provisioning_path = "/tmp/provisioning"
    end
  end
end
