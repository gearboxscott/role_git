# role_git Role
---
This role can be used to do a `git pull` and `git push` within a playbook. This role can be added to allow for more git commands.  This role doesn't contain the normal `tasks/main.yml` but it can be added if there need to serialize several tasks together to accomplish a goal, yet allow for them to be call separately. 
<br><br>

## Requirements
---
Access to the following with permission to create and modify respositories

* GitHub Account or GitLab Account
* Private Repository Server

Have credentials setup:

* ssh keys
* it will be modified to allow tokens.
<br><br>

## Role Variables
---
| Variable | Type | Defaults | Comments |
|-|-|-|-|
| `git_commit_message` | String/<span style="color:red">_required_</span> | none | Commit message. |
| `git_repo_branch` | String/<span style="color:red">_required_</span> | `master` | Branch in the repository |
| `git_repo_path` | String/<span style="color:red">_required_</span> | `origin` | The remote path of the repository |
| `git_repo_url` | String/<span style="color:red">_required_</span> | none | The URL of the git repository in the from `git@server.com:namespace/repo.git` |
| `git_user_email` | String/<span style="color:red">_required_</span> | none | `your_email_address@some_domain` | The user`s email address for git to use |
| `git_username` | String/<span style="color:red">_required_</span> | none | The git username account to access the repositories |
| `git_working_dir_path` | String/<span style="color:red">_required_</span> | none | Where to run the shell command for git from, typically the root of the git repository |

A list of tasks available tasks:

| Task Name | Purpose |
|-|-|
| `add.yml` | Add files to the git metadata for staging to push to the version control server |
| `clone.yml` | Download a copy of the repository from the version control server |
| `commit.yml` | Comment and commit the staged files for push to the version control server |
| `config.yml` | Configure the `.git/config` in the repository these properties for `user.email`, `user.name` and `push.default` to simple.
| `gconfig.yml` | Configure the global git config` for any repository these properties for `user.email`, `user.name` and `push.default` to simple.|
| `pull.yml` | Pull the latest updates from the repository |
| `push.yml` | Push to the version control server the latest updates from the local repository |
<br><br>

## Dependencies
---
- git needs to be installed on the server where git commands are going to be used.
<br><br>

## Example Playbook
---
```yaml
---
- hosts: localhost
  vars:
   git_url: 'git@gitserver:username/repository.git'
   git_key: "{{ lookup('file','./id_rsa') }}"

  tasks:
  - name: Create temporary directory for GIT Operations
    ansible.builtin.tempfile:
      state: directory
      suffix: git
    register: git_temp
    delegate_to: localhost
    run_once: true

  - name: git clone
    include_role:
      name: git_role
      tasks_from: clone.yml
    apply:
      delegate_to: localhost
      run_once: true
    vars:
      git_repo_url: "{{ git_url }}"
      git_working_dir_path: "{{ git_temp.path }}"
  
  - name: Add a test file
    ansible.builtin.copy:
      dest: "{{ git_temp.path }}/testfile01.txt"
      content: |
        this is line 01
        this is line 02
    delegate_to: localhost

  - name: git add
    include_role:
      name: git_role
      tasks_from: '{{ item }}'
    apply:
      delegate_to: localhost
      run_once: true
    vars:
      git_file_path: name_of_repo
      git_working_dir_path: "{{ git_temp.path }}"
      git_commit_message: "add a new file"
      git_repo_branch: master
      git_repo_path: origin
    tags:
      - always
    with_items:
      - config.yml
      - add.yml
      - commit.yml
      - push.yml

  - name: Remove temporary directory for GIT Operations
    ansible.builtin.file:
      path: "{{ git_temp.path }}"
      state: absent
    when: git_temp.path is defined
    delegate_to: localhost
    run_once: true

```
<br><br>

## License
---
GPL
<br><br>

## Author Information
---
Scott Parker (sparker@redhat.com)