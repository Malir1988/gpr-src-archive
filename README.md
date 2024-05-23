# gpr-src-archive

This is a sample repository which is used to transfer an encrypted archive containing multiple GitHub repostitories to another repository.
We use this for cooperations where one contractor has a contracted privilege to gain source access, governed by an external party (Notar).

What will be convered:
* One time setup of the required key
* Export the private key to an external drive, which is handed over to the external party
* Customization of the provided action workflow

## Setup access to all required source repos need a PAT

This specific repository is not the repository which contains the actual sources. So the workflow needs to clone different repositories.
This can't be achived with the default token provided to github actions, because they are only scoped to the repository containing the action.

So we need to create a PAT as documented in https://docs.github.com/en/enterprise-server@3.9/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token

It needs to be created by a user which has access to all repos, and have `repo` scope.

Once the PAT is created, store it in the `USER_PAT` action secret. Is is a short token starting with `ghp_`

If you get an error like on workflow execution:
```
 remote: The 'world-direct' organization has enabled or enforced SAML SSO.
  remote: To access this repository, visit https://github.com/orgs/world-direct/sso?...Q and try your request again.
  Error: fatal: unable to access 'https://github.com/world-direct/technology/': The requested URL returned error: 403
```

you have to click on the link to authorize the use of the PAT on time.

## Configure destination repository

The Archive action pushes the encrpyted source to a destination git repo. You have to create a (ssh-ed25519) keypair and 
register is as a "Deploy Key" to the destination repo (https://docs.github.com/en/authentication/connecting-to-github-with-ssh/managing-deploy-keys).

Once done, add the following Action secrets:

* DESTINATION_REPO: owner/repo
* DESTINATION_REPO_SSH_KEY: -----BEGIN OPENSSH PRIVATE KEY----- ...

The action will a generic user `archiver <archiver@robot>` to create the commit in the destination repository.
This is currently hardcoded and do not need / cant be configured

We will have a keypair already created, which will be handed over to the party.

## Create pgp key to encrypted the sources

We will use the `gpg` tool to create the keypair.
You have to enter an email address which serves as the ID for the key.

```sh
gpg --full-generate-key

# answer questions with default values
# (9) ECC (sign and encrypt) *default*
# (1) Curve 25519 *default*
#  0 = key does not expire
# Validate input, and no expiration
# Enter username:  test key
# Enter email: test@test.com
```

### Export the keys

Once the key is created, we need to export both (private and public) parts of the key

```sh
# export public key
gpg --export --armor test@test.com
```

If you check the file it should be like this:

```
-----BEGIN PGP PUBLIC KEY BLOCK-----
...
-----END PGP PUBLIC KEY BLOCK-----
```

For workflow configuration, you must create the following action secrets in the archive (this) repo:
* `GPG_PUBLIC_KEY` must contain the -----BEGIN PGP PUBLIC KEY BLOCK----- ... -----END PGP PUBLIC KEY BLOCK-----
* `GPG_EMAIL` must contain the mail use to create the key

The private key is not needed to encrypt, only to decrypt. So it will be exported directly on an external drive to be handed over to the external party.
Your destination directory may vary:

```sh
gpg --export-secret-key --armor --output /Volumes/"My USB Drive"
```

## Customize the workflow 

There is a workflow created at (/.github/workflows/archive.yml). Most configuration is done with action secrets, you only need to configure the source repositories 
which will be archived.

In the example workflow the `world-direct/technology` and `world-direct/suss` repositories are shown.
Change this according to the requirements. You only need to change the following options:

```yaml
      # BEGIN configuration of source repos
      - name: Checkout source repo world-direct/technology # << put the repo here for log output
        uses: actions/checkout@v4.1.5
        with:
          repository: world-direct/technology # << put the repo here for checkout
          ref: refs/heads/main                # << put the repo here for branch / ref
          path: ./repos/world-direct/technology  # << put the repo here after ./repos/
          token: ${{ secrets.USER_PAT }} # token stays the same
      # BEGIN configuration of source repos
```

Note: Instead of editing the worlflow we could use the "Matrix" strategy for a kind of loop in the workflow. Because this only is allowed on a job, not a step level,
we would need to define artifacts and dependencies between the `clone` and the `archive` stage. Because this is V1, I didn't want to over-complicate this, 
also because `repository` and `ref` is needed, no we can't just use a list of strings as an input.

You may also customize the workflow schedule. It is configured on `workflow_dispatch` (manual execution) and `cron: 0 0 * * FRI`, every Friday 00:00

## Test the workflow

After execution you should see the `src.tar.gz.enc` and `src.lst` files in the destination directory
