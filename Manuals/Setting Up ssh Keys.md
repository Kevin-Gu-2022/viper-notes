# Setting Up Keys
```bash
ssh-keygem -t ed25519 -C "Comment to rememeber key, e.g. Github Email"
```
Press Enter when prompted on where to store key. By default it will go in `~/.ssh/id_ed25519` 
Add password to key if you wish in next prompt. Press Enter immediately if do not want password

# Adding to ssh-agent
```bash
eval "$(ssh-agent -s)" # Starts ssh-agent in background
ssh-add ~/.ssh/id_ed25519
```
`ssh-agent` stores SSH private keys in memory so you don't have to type password every time it's used. Use `ssh-add -l` to see listed keys

# Adding to GitHub
Now add the *public* key (one with `.pub`) to GitHub website: Settings > Access > SSH and GPG keys > New SSH key