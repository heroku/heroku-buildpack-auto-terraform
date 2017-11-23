# Heroku Auto-Terraform Buildpack

Provides support for running Terraform from your application, including as part
of the release process.

## Basic Installation

```bash
heroku buildpacks:add --index 1 (this URL)
```

Once installed, this buildpack will check that the root of your project
includes at least one Terraform manifest. Your build will fail if no manifests
are present in the root of your application.

## Automatic Deployment

**Automatic deployment is a high-risk configuration.** This will automatically
apply your Terraform manifests to the environment without prompting. It is
strongly recommended that you use this feature in tandem with Heroku Pipelines,
or that you manually review and apply plans if you are not comfortable with the
possibility of automation-induced downtime. **You have been warned.**

<details>
  <summary>If you're aware of the risks and still think this is a good idea, click here.</summary>
<details>
  <summary>Have you considered what to do if automation destroys your data?</summary>
<details>
  <summary>Okay, but I warned you.</summary>
Add the following to your `Procfile`:

```
release: auto-terraform
```

If you have an existing release script, call `auto-terraform` from that script.
</details>
</details>
</details>

## Configuration

The following config vars can be set with `heroku config:set` to affect the
build-time behaviour of this buildpack:

* `TERRAFORM_VERSION`: Specify the version of Terraform to install. By default,
  this will install version 0.11.0.

The following config vars can be set to control the runtime behaviour of this
buildpack:

* `TERRAFORM_WORKSPACE`: Specify the workspace to switch to when automatically
  running Terraform. This requires that the workspace already exist, which
  generally requires that your manifests specify an external state store, such
  as S3.

* `TERRAFORM_EXPORT`: If set to `yes`, any top-level `output`s in your
  Terraform manifest will be exported as environment variables in your dynos.
  This requires a remote state store, and is not enabled by default.

  The `terraform-export` command generates the necessary shell script to export
  outputs. Running `eval "$(terraform-export)"` will simulate
  `TERRAFORM_EXPORT`'s effects, using the current workspace.

## Providing Variables

It's generally a bad idea to commit `.tfvars` files to source control if you
can avoid it. If your manifest requires variables, consider writing a script
that generates a `.tfvars` file from environment variables, or using `TF_VAR_*`
[environment
variables](https://www.terraform.io/intro/getting-started/variables.html)
directly.
