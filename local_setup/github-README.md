# Github

## How to use fine grained access token?
GitHub has "classic" and "fine-grained" personal access tokens (PATs):

Go to Settings > Developer Settings to see these.
```bash
git pull https://${token}@github.com/${owner}/${repo}.git
```

## How to create multiple git profiles and ssh tokens on Mac
* By default, Git uses the global configuration file located at ~/.gitconfig, you can create separate configuration files for each profile and include them conditionally based on the repository's path

1. Create profile-specific config files
```bash
# Personal profile
touch ~/.gitconfig-personal
# Work profile
touch ~/.gitconfig-work
```

2. Edit profile-specific config files
```bash
# ~/.gitconfig-personal
[user]
    name = Personal Name
    email = personal@example.com

# ~/.gitconfig-work
[user]
    name = Work Name
    email = work@example.com
```

3. Modify the global .gitconfig:
```bash
[includeIf "gitdir:~/personal/"]
    path = ~/.gitconfig-personal
[includeIf "gitdir:~/work/"]
    path = ~/.gitconfig-work
```

4. SSH Keys for Authentication (if using SSH)
```bash
ssh-keygen -t ed25519 -C "personal@example.com" -f ~/.ssh/id_personal
ssh-keygen -t ed25519 -C "work@example.com" -f ~/.ssh/id_work
```

5. Add the public keys to the respective Git hosting services (e.g., GitHub, GitLab)

6. Configure SSH to use the correct key based on the host by editing `~/.ssh/config`

```text
# Personal GitHub account
Host github.com-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_personal

# Work GitHub account
Host github.com-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_work
```

7. Update repository remote URLs to use the appropriate host alias
```bash
# For personal repos
git remote set-url origin git@github.com-personal:username/repo.git
# For work repos
git remote set-url origin git@github.com-work:username/repo.git
```

8. Navigate to a repository in ~/personal/ or ~/work/ and check the active Git config
```bash
git config --get user.name
git config --get user.email
```

## How to switch the profiles

1. Automatic Switching with includeIf: If your .gitconfig is set up with conditional includes (as shown above), Git automatically applies the correct profile when you work in repositories under ~/personal/ or ~/work/. No manual action is needed as long as repositories are organized in the specified directories

2. Manually Switching Profiles, to temporarily switch profiles for a specific repository, navigate to the repository and set the local Git config
```bash
cd ~/path/to/repo
git config user.name "Work Name"
git config user.email "work@example.com"
```

