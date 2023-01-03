---
layout: post
title:  "Adding an approle for Terraform in Hashicorp Vault"
date:   2023-01-03 16:26:33 +0000
categories: posts
---

I recently set up a new Hashicorp Vault instance and wanted to use it with Terraform. I followed the instructions on the Hashicorp website and got it working. However, I wanted to use an `approle` instead of a token. I found the instructions on the Hashicorp website to be a bit confusing, here is what I did to get it working.

First, I created a policy called `terraform` with the following permissions.

```bash
$ vault policy write terraform - <<EOF
path "*" {
  capabilities = ["list", "read"]
}

path "secrets/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "auth/token/create" {
capabilities = ["create", "read", "update", "list"]
}
EOF
```

Then I enabled the `approle` auth method.

```bash
$ vault auth enable approle
```

Next, I created an approle called `terraform`.

```bash
$ vault write auth/approle/role/terraform \
  secret_id_ttl=0 \
  token_num_uses=0 \
  token_ttl=0 \
  token_max_ttl=0 \
  secret_id_num_uses=0
```

<!-- markdownlint-disable MD033 -->
<div class="note" style='position: relative; padding:1rem 1rem; margin-bottom: 0.5em; background-color:#cfe2ff; border: 1px solid transparent; border-radius: 0.25rem;'>
  <p><strong>Note:</strong> The <code>secret_id_num_uses=0</code> option will mean that the secret id does not expire, this is useful so we can useful in CI/CD pipelines without having to regenerate after x number of uses.</p>
</div>

Let's assign the policy we created earlier to the `approle`.

```bash
$ vault write auth/approle/role/terraform/policy terraform
```

Now we can get the role id and secret id for the `approle`, this is what we will use in Terraform to authenticate with Vault.

```bash
$ vault read auth/approle/role/terraform/role-id
$ vault write -f auth/approle/role/terraform/secret-id
```

Copy the role id and secret id and add them to your Terraform configuration I pass them in as variables and store in my CI/CD pipeline.

```hcl
provider "vault" {
  address          = var.vault_address
  skip_child_token = true # https://stackoverflow.com/questions/73034161/permission-denied-on-vault-terraform-provider-token-creation

  auth_login {
    path = "auth/approle/login"

    parameters = {
      role_id   = var.vault_role_id
      secret_id = var.vault_role_secret_id
    }
  }
}
```

That's it you should now be able to authenticate with Vault using an `approle` instead of a token.