# AutoMirror
## Purpose

AutoMirror is a Git post-receive hook designed to automatically mirror a self-hosted Git repository
to a remote location when a push to the original repository occurs. The utility is especially useful
for mirroring repositories to standard cloud platforms like [GitHub](https://github.com). AutoMirror
was designed to facilitate the publication of specific releases or branches to publicly accessible
repositories to supports standard fork and pull-request workflows for newcomers. However, pull requests
should be manually applied and pushed to the original remote. AutoMirror is activated after a `git receive`
operation in the source repository, enabling the mirroring of specific Git references (`refs`) such
as entire repositories, specific branches, or tags. Additionally, when provided with a GitHub Access
Token, it can create GitHub repositories and configure remotes automatically.

## Scope and Limitations

- AutoMirror requires that the user executing `git-receive-pack` has push access to the specified remote repository. The utility is compatible with all standard Git protocols (`ssh`, `git`, `https`, `file`, and `ftps`). `ssh` keys or other automatic authentication methods should be configured in advance.

- Installation of a Git `post-receive` hook is necessary for using AutoMirror.

- AutoMirror is limited to creating repositories on GitHub. For other Git providers, repository setup must be done manually. Contributions through pull requests are welcome.

## Installation

### Dependencies

AutoMirror is developed for systems with Python 3.x and the `python3-requests` package. The required dependencies can be installed using the following distro-specific commands:

For Ubuntu/Debian:

```bash
sudo apt install python3 python3-requests git
```

For Arch Linux:

```bash
sudo pacman -S python git python-requests
```

### Installing a Key File

Automatic authentication is required for ssh. The standard method to accomplish this is to setup SSH key for pushing to the remote repository and to specify the key should be used in the `~/.ssh/config` file:

```bash
Match host github.com
	IdentityFile ~/.ssh/github-mirror
```

### Installing in a Single Repository

To activate AutoMirror for a single repository, copy the `post-receive` script into the repository's hooks directory:

```bash
cp post-receive /some/repo.git/hooks/post-receive
chmod +x /some/repo.git/hooks/post-receive
```

### Global Installation

For a global installation, the following steps are required (which are compatible with newer Git versions):

```bash
mkdir -p /etc/git/hooks
git config --global core.hooksPath /etc/git/hooks
cp post-receive /etc/git/hooks/post-receive
chmod +x /etc/git/hooks/post-receive
```

### Gitolite Installation

For Gitolite installations:

```bash
cp post-receive ~/.gitolite/hooks/common/post-receive
gitolite setup --hooks
chmod +x ~/.gitolite/hooks/common/post-receive
```

Append the following keys to the `GIT_CONFIG_KEYS` variable in `/etc/gitolite3/gitolite.rc`:

```bash
automirror.refs automirror.remote automirror.gh-create automirror.gh-desc automirror.gh-homepage
```

Per-repository AutoMirror settings can be configured through `git config` entries in the `gitolite.conf` file:

```bash
repo example
  config automirror.remote ssh://git.example.com:/example
```

## Configuration

### Remote and Refs

A remote must be specified to enable mirroring. When this setting is configured through `automirror.remote`,
AutoMirror mirrors the repository to the specified remote following a `git receive` operation. Optionally,
you can specify which `refs` to mirror using `automirror.refs`:

```bash
git config --local --set automirror.remote <REMOTE_URL>
git config --local --set automirror.refs <REF_SPEC>
```

The `<REMOTE_URL>` is the URL of the Git remote repository to which updates will be pushed. The
`<REF_SPEC>` is a set of refs that will be synchronized. If `automirror.refs` is not set, the remote
symbolic ref "HEAD" will be mirrored. By default, HEAD is set to `git config --get init.defaultbranch`
which is typically either `master` or `main`.

For example, `<REF_SPEC>` could be:

```bash
refs/heads/*                       # all heads
refs/tags/*                        # all tags
refs/{heads,tags}/*                # all heads and tags
refs/heads/main,refs/heads/release # main and release branches
refs/tags/*,refs/heads/main        # all tags and main branch
```

### GitHub Integration

GitHub integration is optional but offers automated repository creation via GitHub's API. If you set
`automirror.gh-create` to true and leave `automirror.remote` unspecified, AutoMirror will automatically
create a GitHub repository. This feature requires AutoMirror to have access to a GitHub personal access
token stored in `automirror.gh-token-file`. It's important to secure this file with restricted permissions
(`chmod 600 <file>`).

```bash
git config --local  --add automirror.gh-create true
```

After the repository is successfully created, the value of `automirror.remote` is automatically set
to the repository's `ssh_url`.

#### Configuring the Repository Description and Homepage

You can optionally specify the repository description and homepage. Use the `automirror.gh-desc` and
`automirror.gh-homepage` settings, respectively. These strings utilize Python's `string.format` method,
allowing you to use `{repo_name}` as a placeholder in your templates.

```bash
git config --global --add automirror.gh-desc '{repo_name} Mirror - This is an automatically maintained mirror of https://git.flu0r1ne.net/{repo_name}.'
git config --global --add automirror.gh-homepage 'https://git.flu0r1ne.net/{repo_name}/'
```

#### Obtaining and Configuring a Personal Access Token

For this feature to work, a personal access token from GitHub is required. This token must have read
and write access to the Administration scope, as well as read access to Metadata. You can obtain such
a token as follows:

1. Navigate to GitHub and go to `Settings -> Developer settings -> Personal access tokens -> Fine-grained tokens`.
2. Click `Generate new token`.
3. In the account permissions section, enable read and write access to Administration and read access to Metadata.
4. Generate and copy the token.

Once obtained, store this token in a file and set its permissions. You can use `cat` to echo  the token
without it being added to your `bash` history. Press `Ctrl-D` to exit:

```bash
cat </dev/stdin >/path/to/token_file
chmod 600 /path/to/token_file
```

Finally, configure AutoMirror to use this token file:

```bash
git config --global --add automirror.gh-token-file '/path/to/token_file'
```

This completes the setup for GitHub integration.

GitHub integration is optional. If `automirror.gh-create` is set to true and `automirror.remote`
is not defined, AutoMirror will create the GitHub repository using the GitHub API. The API requires
a personal access token stored in `automirror.gh-token-file`, which should have restricted permissions
(`chmod 600 file`).

For optional GitHub integration:

```bash
git config --local  --add automirror.gh-create true
```

The description and homepage templates:

```bash
git config --global --add automirror.gh-desc '{repo_name} Mirror - This is an automatically maintained mirror of https://git.flu0r1ne.net/{repo_name}.'
git config --global --add automirror.gh-homepage 'https://git.flu0r1ne.net/{repo_name}/'
```

## Example Usage

To mirror all heads and tags from a local repository to a remote repository:

1. Configure the remote URL:
    ```bash
    git config --local --set automirror.remote ssh://git.example.com:/example
    ```

2. Specify the refs to mirror:
    ```bash
    git config --local --set automirror.refs "refs/{heads,tags}/*"
    ```

For additional queries or issues, consult the project documentation or contact the maintainers.
