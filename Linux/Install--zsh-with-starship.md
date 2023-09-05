# Install ZSH with Starship 🚀

Install ZSH at first

```bash
sudo apt update && sudo apt install zsh
```

Install Oh my ZSH

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

*more information you can find https://github.com/ohmyzsh/ohmyzsh*

## Install Starship theme 

Run following script

```bash
curl -fsSL https://starship.rs/install.sh | bash
```

update your `.zshrc` file

```bash
# do not use sudo
nano ~/.zshrc
```
add following at end 

```bash
eval "$(starship init zsh)"
```

Logout and login. 🎉 open terminal