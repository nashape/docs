# Ubuntu Rails Development Machine Install HOWTO

This howto details creating a Ubuntu (virtual) machine for Rails development. I
do all my work on my dev vm as the root user, so these instructions reflect
this.

Not this is an entirely manual process! Not very DevOps-y...next is to write
puppet manifests to automate this.

1.  Minimal install of Ubuntu Server  
    10.04 lucid64 in this case  
    Install as a virtual machine using vmware fusion (or whatever takes your
    fancy)  
    On the first screen of installing Ubuntu Server, hit ```F4``` then select:  
    "Install a minimal system", then start the install process (select "Install Ubuntu Server").

2.  Install process  
    Lots of screens, including:  
    Choosing a ```hostname```  
    Partitioning - choose "Guided - use entire disk and set up LVM"  
    Then it does "Installing the base system"  
    Choose "No automatic updates"  
    Install "OpenSSH Server" from the "Software selection" page (although you
    could just do it with apt-get after installation)  
    Choose "Yes" to installing GRUB

3.  Allow root to login  
    Reboot and login as the user created during installation  
    ```sudo su``` to get access to root  
    ```passwd``` to set root's password  
    Exit (twice) and login as root to ensure that this worked

4.  Configure interfaces  
    Shouldn't actually be needed  
    ```ifconfig``` to check ```eth0``` is up and what it's IP address is (for
    setting in your main machine's ```hosts``` file for example)

5.  ssh access  
    ssh in as root from your main machine

6.  Setup ssh keys  
    Read https://help.ubuntu.com/10.04/serverguide/openssh-server.html  
    Create the ```.ssh``` directory under your home directory  
    Either copy in existing keys, or create a new key.  
    Write config  
    See https://help.github.com/articles/generating-ssh-keys

7.  Update aptitude  
    ```apt-get update```

8.  Install vim  
    ```apt-get install vim```

9.  Install git  
    See http://git-scm.com/book/en/Getting-Started-Installing-Git  
    and http://evgeny-goldin.com/blog/3-ways-install-git-linux-ubuntu/  
    ```cd ~```  
    ```mkdir src```  
    ```cd src```  
    ```apt-get install libcurl4-gnutls-dev libexpat1-dev gettext libz-dev libssl-dev```  
    ```apt-get install git-core``` # you will now have an out-of-date git  
    ```git clone https://github.com/git/git.git```  
    ```cd git```  
    ```git tag``` # and look for the latest release  
    ```git checkout v1.7.12```  
    ```git describe``` # just to make sure  
    ```make prefix=/usr/local all```  
    ```make prefix=/usr/local install```  
    ```which git``` # this was correct without logging out and in  
    ```git --version``` # I had to log out and back in again for this to be correct

10. Configure git  
    ```git config --global color.branch auto```  
    ```git config --global color.diff auto```  
    ```git config --global color.interactive auto```  
    ```git config --global color.status auto```  
    ```git config --global user.name <username>```  
    ```git config --global user.email <email address>```  
    This is worth reading: https://help.github.com/articles/set-up-git#platform-linux

11. Install zsh  
    ```apt-get install zsh```

12. Change to zsh  
    ```which zsh```  
    Should be installed to <code>/usr/bin/zsh</code>  
    ```chsh /usr/bin/zsh``` # log out and back in again

13. Configure zsh  
    Add ```.zshrc``` to your home directory  
    One option is oh-my-zsh: https://github.com/robbyrussell/oh-my-zsh  
    Read the README for instructions on how to install this

14. Configure vim  
    add ```.vimrc``` to your home directory  
    First install pathogen: https://github.com/tpope/vim-pathogen  
    Then install plugins (just ```git clone``` the repos in the
    ```~/.vim/bundle``` directory):  
    Solarized (http://ethanschoonover.com/solarized): https://github.com/altercation/vim-colors-solarized  
    NERDTree: git://github.com/scrooloose/nerdtree.git  
    vim-rails: https://github.com/tpope/vim-rails  
    vim-cucumber: https://github.com/tpope/vim-cucumber  
    vim-fugative: https://github.com/tpope/vim-fugitive  
    etc.

15. Install nginx  
    As described here: http://wiki.nginx.org/Install  
    Add entries to ```/etc/apt/sources.list```  
    Then run ```apt-get update```  
    Before installing, add nginx's public key (as noted here: http://mailman.nginx.org/pipermail/nginx/2011-December/030918.html):  
    ```wget -O - http://nginx.org/packages/keys/nginx_signing.key | apt-key add -```  
    ```apt-key adv --keyserver pgp.mit.edu --recv-keys 7BD9BF62```  
    And to install:  
    ```apt-get install nginx```  
    Which uses a Debian style Apache layout for nginx.  
    Note: nginx installs an ```nginx``` user and group
    See http://articles.slicehost.com/2009/2/27/ubuntu-intrepid-installing-nginx-via-aptitude (but this is quite old now) and  
    http://articles.slicehost.com/2009/3/4/ubuntu-intrepid-installing-nginx-from-source if you want to install from source  

16. Configure nginx  
    Edit the ```/etc/nginx/nginx.conf``` file  
    See http://articles.slicehost.com/2009/3/5/ubuntu-intrepid-nginx-configuration (again, an oldie but a goodie)  
    Here is a minimal conf that for a unicorn served rails app (see below):  
    http://unicorn.bogomips.org/examples/nginx.conf  
    This may need quite a bit of modifcation depending on your needs.

17. Install MongoDB  
    Follow the very good instructions here http://docs.mongodb.org/manual/tutorial/install-mongodb-on-debian-or-ubuntu-linux/

18. Install RVM  
    curl isn't installed: ```apt-get install curl```  
    I install RVM for the root user only (not the multiuser install): https://rvm.io/support/faq/#root  
    And add ```.rvmrc``` to home directory:  
    ```
    export rvm_prefix="$HOME"
    export rvm_path="$HOME/.rvm"
    ```  
    Then to install (see https://rvm.io/rvm/install/):  
    ```curl -L https://get.rvm.io | bash -s stable --ruby```  
    which installs ruby as well  
    Don't forget to run: ```source /root/.rvm/scripts/rvm```  
    ```which rvm``` # ugh dumps the contents of the ```rvm``` script in my zsh shell  
    ```which ruby```  
    ```ruby -v``` # this should be 1.9.3

19. Install Ruby  
    Now already installed!

20. Install Rails  
    See https://github.com/nashape/docs/blob/master/rails_install_howto.md  
    The only issue is that you need to delay creating the ruby/unicorn wrapper
    until after the unicorn binary has been installed. This will get installed
    when, from a Rails app, you run ```bundle install``` for the first time.  
    However before running ```bundle install``` run:  
    ```rvm requirements```  
    It will tell you that you also need to run:  
    ```
    apt-get install build-essential openssl libreadline6 libreadline6-dev curl git-core \  
    zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt-dev \  
    autoconf libc6-dev ncurses-dev automake libtool bison subversion pkg-config
    ```  
    Now you can run ```bundle install```  
    Now don't forget to create the ruby/unicorn wrapper.

21. Install unicorn  
    Now already installed!  
    (Github use it: https://github.com/blog/517-unicorn)  
    See http://unicorn.bogomips.org/  
    Start unicorn from a Rails app directory:  
    ```r193_unicorn -c config/unicorn.rb```

22. Sinatra  
    Sinatra is installed using gem, see http://www.sinatrarb.com/  
    ```rvm use 1.9.3@sinatra``` # to use the correct gemset  
    ```gem install sinatra```

23. Install ack  
    See http://betterthangrep.com/install/ but put into ```/usr/local/bin```  
    ```curl http://betterthangrep.com/ack-standalone > /usr/local/bin/ack && chmod 0755 !#:3```

24. Install tmux and tmuxinator  
    ```
    apt-get install tmux
    gem install tmuxinator
    echo '[[ -s $HOME/.tmuxinator/scripts/tmuxinator ]] && source $HOME/.tmuxinator/scripts/tmuxinator' >> ~/.zshrc
    ```

25. Fix time  
    On my install the time was incorrect, despite it displaying the correct timezone.  
    Run ```ntpdate ntp.ubuntu.com```  
    This corrects it once, but run a cron job to keep it correct, or use ntpd
    (which, actually, isn't installed yet), see:  
    https://help.ubuntu.com/community/UbuntuTime#Changing_the_Time_Zone (also
    has instructions for changing the timezone).  
    This is actually a vm issue:  
    See http://www.vmware.com/files/pdf/techpaper/Timekeeping-In-VirtualMachines.pdf


