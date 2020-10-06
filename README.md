# ansible-awx

Some notes on building a ansible/awx/kvm/vagrant virtual environment.

vagrant winrm -s cmd -c ipconfig

# ssh

vagrant winrm -e -s cmd -c "powershell -Command Start-Service ssh-agent"

#  vagrant winrm
VAGRANT_DISABLE_STRICT_DEPENDENCY_ENFORCEMENT=1 vagrant plugin install winrm winrm-elevated winrm-fs

#
