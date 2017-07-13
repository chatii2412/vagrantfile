require "yaml"

conf = YAML.load_file(File.join(__dir__, "config.yaml"))

Vagrant.configure("2") do |config|
	conf["vm"].each do |key, value|
		config.vm.send("#{key}=", value)
	end

	conf["synced_folders"].each do |item|
		config.vm.synced_folder item["src"], item["dest"]
	end

	conf["providers"].each do |provider, provider_conf|
		config.vm.provider provider do |v|
			provider_conf.each do |key, value|
				v.send("#{key}=", value)
			end
		end
	end

	config.vm.provision "Copy the SSH private key", type: :file, source: "~/.ssh/id_rsa", destination: "~/id_rsa"
	config.vm.provision "Configure the SSH private key", type: :shell do |s|
		s.privileged = false
		s.inline = <<-SHELL
			rm -f ~/.ssh/id_rsa
			mv ~/id_rsa ~/.ssh/
			chmod 400 ~/.ssh/id_rsa
		SHELL
	end

	config.vm.provision "Update APT repositories", type: :shell do |s|
		s.inline = <<-SHELL
			add-apt-repository ppa:git-core/ppa -y
			apt-get update
		SHELL
	end

	config.vm.provision "Install and configure Git", type: :shell do |s|
		s.privileged = false
		s.inline = <<-SHELL
			sudo apt-get install git -y

			sudo chmod +x /usr/share/doc/git/contrib/diff-highlight/diff-highlight
			git config --global diff.compactionHeuristic true
			git config --global pager.log '/usr/share/doc/git/contrib/diff-highlight/diff-highlight | less'
			git config --global pager.show '/usr/share/doc/git/contrib/diff-highlight/diff-highlight | less'
			git config --global pager.diff '/usr/share/doc/git/contrib/diff-highlight/diff-highlight | less'
			git config --global interactive.diffFilter /usr/share/doc/git/contrib/diff-highlight/diff-highlight
		SHELL
	end

	config.vm.provision "Install and configure ZSH", type: :shell do |s|
		s.privileged = false
		s.inline = <<-SHELL
			sudo apt-get install zsh -y

			sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

			rm -rf ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
			git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

			sed -i 's/plugins=(git)/plugins=(zsh-syntax-highlighting git docker docker-compose)/g' ~/.zshrc

			echo '\ncd #{conf["default_path"]}' >> ~/.zshrc
		SHELL
	end

	config.vm.provision :docker
	config.vm.provision :docker_compose,
		yml: "/vagrant/docker-compose.yaml",
		run: "always"

	config.vm.network :private_network, type: "dhcp"
	config.vm.network :public_network
end
