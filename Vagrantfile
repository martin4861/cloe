# vim: ft=ruby:

Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2004"
  config.vm.provider :libvirt do |libvirt|
    libvirt.cpus = 4
    libvirt.memory = 8192
  end
  config.vm.synced_folder "./", "/vagrant", type: "nfs"
  config.vm.provision "shell", inline: <<-EOF
    # This provisioning lifted from: ./dist/docker/Dockerfile.focal
    #
    # Install system dependencies
    apt-get update
    apt-get install -y make ccache locales
    make -f /vagrant/Makefile.setup \
      DEBIAN_FRONTEND=noninteractive \
      APT_ARGS="--no-install-recommends -y" \
      install-system-deps
    locale-gen

    # Install Python dependencies
    pip3 install --upgrade pip
    make -f /vagrant/Makefile.setup \
      PIP_INSTALL_ARGS="" \
      install-python-deps

    # Setup Conan
    sudo -u vagrant conan profile new --detect default
    sudo -u vagrant conan profile update settings.build_type=RelWithDebInfo default
    sudo -u vagrant conan profile update settings.compiler.libcxx=libstdc++11 default
  EOF
end
