# tmux

A shell script to get started.

{% code overflow="wrap" fullWidth="true" %}
```bash
configure_tmux() {

	echo "Installing latest Tmux from source.";
	apt-get -yqq remove tmux &> /dev/null && rm -rf /tmp/latest_tmux;
	apt-get -yqq install libevent-dev ncurses-dev build-essential bison pkg-config automake git zsh ruby-full &> /dev/null;
	git clone -q https://github.com/tmux/tmux.git /tmp/latest_tmux && cd /tmp/latest_tmux;
	sh autogen.sh &> /dev/null && ./configure &> /dev/null && make &> /dev/null && make install &> /dev/null;
	if [[ $? -ne 0 ]]; then echo "Error installing Tmux."; fi
	
	echo "Initializing Tmux configuration.";
	cd /root/ && mkdir -p /root/logs/tmux;
	tee /root/.tmux.conf <<-'EOF' > /dev/null
		set-option -g default-shell /bin/zsh
		set -g @plugin 'tmux-plugins/tmux-logging'
		set -g @plugin 'tmux-plugins/tpm'
		set -g @plugin 'tmux-plugins/tmux-sensible'
		set -g history-limit 250000
		set -g allow-rename off
		set -g escape-time 50
		set-window-option -g mode-keys vi
		run '/root/.tmux/plugins/tpm/tpm'
		run '/root/.tmux/plugins/tmux-logging/logging.tmux'
		run '/root/.tmux/plugins/tmux-logging/scripts/toggle_logging.sh'
		bind-key "c" new-window \; run-shell "/root/.tmux/plugins/tmux-logging/scripts/toggle_logging.sh"
		bind-key '"' split-window \; run-shell "/root/.tmux/plugins/tmux-logging/scripts/toggle_logging.sh"
		bind-key "%" split-window -h \; run-shell "/root/.tmux/plugins/tmux-logging/scripts/toggle_logging.sh"
	EOF

	echo "Installing the Tmux Plugin Manager (TPM).";
	rm -rf /root/.tmux/plugins/tpm && git clone -q https://github.com/tmux-plugins/tpm.git /root/.tmux/plugins/tpm;
	/bin/bash /root/.tmux/plugins/tpm/scripts/install_plugins.sh &> /dev/null;
	if [[ $? -ne 0 ]]; then echo "Error installing Tmux plugins."; fi

	sed -i 's/default_logging_path="$HOME"/default_logging_path="\/root\/logs\/tmux"/' /root/.tmux/plugins/tmux-logging/scripts/variables.sh;
	tmux new-session -d; # initialize tmux
	tmux source-file /root/.tmux.conf;
	
	echo "Installing tmuxinator";
	gem install tmuxinator &> /dev/null;
	mkdir -p /root/.config/tmuxinator;
	tee /root/.config/tmuxinator/default.yml <<-'EOF' > /dev/null
		name: default
		root: /root/
		windows:
		    - main: tmux source /root/.tmux.conf
		    - msf: msfconsole
	EOF
}

configure_tmux;
```
{% endcode %}



