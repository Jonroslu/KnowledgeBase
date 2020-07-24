## Everything is done via the aws CLI

To follow this guide, install 

1) the [awscli](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html).

2) jq to help with json parsing `brew install jq`.

3) `kubectl`

Note that the `awscli` requires access keys. You may get them by going to `<my username> -> My Security Credentials -> Access Keys`.

After downloading the key ID and password, you will have to give your credentials to the `awscli`. Simply type `awscli configure` and it will prompt you for them.

[Next](https://https://github.com/Jonroslu/KnowledgeBase/blob/master/awk-eks-setup/2-create-role.md)


