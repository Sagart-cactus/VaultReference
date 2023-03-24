# Vault Demo

## install vault on local machine
```bash
brew tap hashicorp/tap
brew install hashicorp/tap/vault
```

## Run vault on dev mode
```bash
vault server -dev
vault server -dev -dev-root-token-id=root
```

## Login from Web browser
http://127.0.0.1:8200

## Login from CLI
Open Terminal
```bash
    export VAULT_ADDR="http://127.0.0.1:8200"
    vault login {roottoken}
```

## Authentication

## Adding authentication using UI
--> Access --> Enable new method + --> Select Username & Password --> Enable --> Update options

## Adding a user using UI
Select Userpass --> Create user + --> Enter username --> Enter password --> Save ... You have create a user

## Adding user using CLI
```
vault write auth/userpass/users/trivedi password=sagar policies=admin
```
Check the user in the Browser

## Adding authentication using CLI
```
vault auth enable -path=tw-devops userpass
```
Check the method in browser: http://127.0.0.1:8200/ui/vault/access

```
vault auth enable -path=tw-dev userpass
```
Check the browser: http://127.0.0.1:8200/ui/vault/access

## Take a look at all the auth method that the vault provides
Now goto Enable new method and take a look at all the authentication methods for more details visit: https://developer.hashicorp.com/vault/api-docs/auth


## Adding secrets
Just like there are many auth methods there are many secret engine also. Goto Secrets in browser and Click on Enable new engine
- Create a new Engine with name `tw-dev`
- add 2 secrets with name `sample1` and `sample2`
- add key value pair of secrets in each secret.

Take a look at all the different options that are being provided.

## Getting and setting value from CLI
### Getting
```
vault kv get tw-dev/sample1
```

For complex jq logics you can use -format-json option
```
vault kv get -format=json tw-dev/sample1
```
### Setting
```
vault kv put tw-dev/sample3 key3="value3"
```
check on the UI: http://127.0.0.1:8200/ui/vault/secrets/tw-dev/list


## Versions of the Secrets
```
vault kv put tw-dev/sample3 key3="updated_value3"
```
Look closely at the ouput you now see a version 2

```
vault kv get tw-dev/sample3
```
Gives you the latest value


QQ what will be the output of this command ??
```
vault kv get -format=json -version=3 tw-dev/sample3
```
Yes it gives no value found as the version 3 does not exists yet.
Lets create a new version
```
vault kv put tw-dev/sample3 key3="new_updated_value3"
```
Now check the output of the command again
```
vault kv get -format=json -version=3 tw-dev/sample3
```

Get value of previous version

```
vault kv get -format=json -version=2 tw-dev/sample3
```

## Create Policy using UI
http://127.0.0.1:8200/ui/vault/policies/acl/create

Name: tw-dev-read-only
Policy: 
```
path "tw-dev/*" {
    capabilities = ["list"]
}
```
- Go to tw-dev auth method: http://127.0.0.1:8200/ui/vault/access/tw-dev/item/user
- Select tw-developer1
- Edit User
- Expand Tokens
- In Generated Token's Policies : Enter the name of the policy `tw-dev-read-only`

Login via the tw-developer1 user in an incognito window
goto the tw-dev
You can see the list of secrets but cannot see the value of the secret

- Go to the root browser window and visit: http://127.0.0.1:8200/ui/vault/policy/acl/tw-dev-read-only
- Edit the policy to include `read`
- Go to the tw-developer1 window and under the user menu on top right corner select renew token
- Now you will be able to see the values of secret

Repeat the same steps to add `update` and `create` in the policy


## Create Policy using CLI
```
vault policy write restricted-read-sample2 - <<EOF
path "tw-dev/data/sample2" {
    capabilities = ["read","update"]
}
EOF
```
Create a new user with the newly created policies 
```
vault write auth/tw-dev/users/tw-developer2 password=sagar policies=restricted-read-sample2
```

Login using the tw-developer2 username and password in new terminal
```
export VAULT_ADDR="http://127.0.0.1:8200"
vault login -method=userpass -path=tw-dev\
    username=tw-developer2\
    password=sagar
```

Try to get sample2 and sample3 value

```
vault kv get tw-dev/sample2
vault kv get tw-dev/sample3
```

As you can see sample2 is accessible and sample3 is not accessible


## Seal Vault
```bash
vault operator seal
```

## Unseal Vault
```bash
vault operator unseal
```
