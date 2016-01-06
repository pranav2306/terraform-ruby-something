# Terraform + Ruby

## Requirements

- Linux or OS X
- Ruby
- [direnv](https://github.com/direnv/direnv)

## Preparation

```
$ bundle install --path vendor/bundle
```

Put your environmental variables to `.envrc` (see `.envrc.example`) then

```
$ direnv allow
```

And install Terraform!

```
$ bundle exec rake terraform:install
```

## Usage

All of your `terraform plan` and `terraform apply` should be executed as

```
$ bundle exec rake terraform:plan
$ bundle exec rake terraform:apply
```

`*.tf.erb` will be parsed as ERB template and converted to `*.tf`.

Default rake task `$ bundle exec rake` is equivalent to `$ bundle exec rake terraform:plan`.

## Security

For security reason, *DO NOT git add terraform.tfstate*. It will be encrypted with your secret key to `terraform.tfstate.encrypted` automatically (unless you avoid using rake tasks).

See `.gitignore`.

## Tips

If you already have `.tf` files, just rename it to `.tf.erb`.
