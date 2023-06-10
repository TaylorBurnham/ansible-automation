# Ansible Roles & Scripts

This is a collection of Ansible scripts that I use for my homelab. All of this comes without warranty and is to be used at your own risk.

## Roles & References

 - debian-k3s: Installs k3s across all nodes, with a single master. This was adapted from [k3s-ansible](https://github.com/k3s-io/k3s-ansible). Use `debian-k3s-purge` to remove the installation.

## Scripts and Fixes
### 1Password & ansible-lint

I use the `op get item` command to pull a password from my 1Password vault, which then is used to unlock the local `vault.yaml`. This is hardcoded in my `.zshrc` for all environments to use.

```sh
export ANSIBLE_VAULT_PASSWORD_FILE=$(which op-ansible-vault)
```

This is the content of the `op-ansible-vault` script.

```bash
#!/usr/bin/env bash
VAULT_NAME="< Vault Name >"
ENTRY_NAME="< Vault Entry Name >"
op item get --format="json" \
    --vault="${VAULT_NAME}" \
    --fields label=password "${ENTRY_NAME}" | \
    jq '.value' | tr -d '"'
```

The issue I had was `ansible-lint` will run the command below, which prompts to unlock my keyring, and it occurs on every save.

```sh
ansible-playbook -i localhost, --syntax-check ${PLAYBOOK_PATH}
```

The way I worked around this was to add this to my `~/.local/bin` and set the vscode property below, which clears the variable for the invocation of the following command, removing the prompts from running.

```bash
#!/usr/bin/env bash
# Overrides the OP prompt
unset ANSIBLE_VAULT_PASSWORD_FILE
ansible-lint "$@"
```

```json
{
    "ansible.validation.lint.path": "vscode-ansible-lint"
}
```
