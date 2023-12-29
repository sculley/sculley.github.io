---
layout: post
title:  "Sorting Terraform variables using terraform-variable-sort"
date:   2023-12-29 07:00:00 +0000
categories: posts
---

When managing complex infrastructure with Terraform, keeping your variables organized can be a challenge. If you've ever struggled with maintaining consistency in your Terraform variable files, I have a created a solution. Meet terraform-variable-sort, a simple yet powerful script that allows you to sort your Terraform variables alphabetically with ease.

## What is terraform-variable-sort?

terraform-variable-sort is a handy script designed to bring order to your Terraform variable files. By sorting your variables alphabetically, you can improve readability and ensure that your variable definitions follow a consistent pattern. Whether you're working on a small project or a large-scale infrastructure, maintaining well-organized variable files is essential.

## Requirements

This script relies on GNU `awk`, which is readily available on most Linux distributions. If you're using macOS, you can install gawk using Homebrew:

```shell
brew install gawk
```

With `gawk` or `awk` installed, you're ready to proceed.

## Installation

Getting terraform-variable-sort up and running is a breeze. You can install it using Homebrew, making it even more convenient to manage your Terraform projects.

```shell
brew tap sculley/homebrew-formula
brew install terraform-variable-sort
```

Alternatively, you can use the script manually by running the following command:

```shell
curl -s https://raw.githubusercontent.com/sculley/terraform-variable-sort/main/terraform-variable-sort.sh -o terraform-variable-sort.sh
chmod +x terraform-variable-sort
mv terraform-variable-sort /usr/local/bin
```

With the `terraform-variable-sort` command installed, you're all set to start sorting your Terraform variables.

## Usage

The simplest way to sort your Terraform variables is by running terraform-variable-sort without any arguments. By default, it looks for a file named variables.tf in your current directory:

```hcl
variable "foo" {
    description = "foo"
    type = string
    default = "foo"
}

variable "bar" {
    description = "bar"
    type = string
    default = "bar"
}
```

```shell
terraform-variable-sort
```

If your variable file has a different name or is located elsewhere, you can specify it as the first argument:

```shell
terraform-variable-sort my_variables.tf
```

After running your variables file should now look like

```hcl
variable "bar" {
    description = "bar"
    type = string
    default = "bar"
}

variable "foo" {
    description = "foo"
    type = string
    default = "foo"
}
```

Your Terraform variable file will be automatically updated in place, ensuring that it remains well-organized.

## Conclusion

With `terraform-variable-sort`, you can maintain consistency and readability in your Terraform projects effortlessly. Say goodbye to manually sorting variables, and start enjoying a more organized infrastructure codebase. Give it a try today and experience the benefits of a neatly sorted Terraform variable file.

Download `terraform-variable-sort` now and simplify your Terraform workflow!